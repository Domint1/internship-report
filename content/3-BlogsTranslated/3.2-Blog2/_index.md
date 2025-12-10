---
title: "Blog 2"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Tracing ETL Workloads using AWS X-Ray and AWS Distro for OpenTelemetry

by Praneeth Reddy Tekula, Deepak Kovvuri, and Viveyk Karri on 30 APR 2025 in [AWS Distro for OpenTelemetry](https://aws.amazon.com/blogs/mt/category/management-tools/aws-distro-for-opentelemetry/), [AWS Glue](https://aws.amazon.com/blogs/mt/category/analytics/aws-glue/), [AWS X-Ray](https://aws.amazon.com/blogs/mt/category/developer-tools/aws-x-ray/), [Management Tools](https://aws.amazon.com/blogs/mt/category/management-tools/), [Technical How-to](https://aws.amazon.com/blogs/mt/category/post-types/technical-how-to/) [Permalink](https://aws.amazon.com/blogs/mt/tracing-etl-workloads-using-aws-x-ray-and-aws-distro-for-opentelemetry/)  [Share](https://aws.amazon.com/vi/blogs/mt/tracing-etl-workloads-using-aws-x-ray-and-aws-distro-for-opentelemetry/#)

## **Introduction**

Data pipelines are essential for modern data-driven companies to gain critical business insights. However, data pipelines commonly fail when new files or datasets from data sources do not conform to the expected schema, leading to downstream job failures, workflow breakdowns, and delayed insights. Additionally, fluctuating data volumes, from a few gigabytes to multiple terabytes, can significantly impact the efficiency and reliability of these pipelines. This can make it challenging for customers to identify the specific ETL job within the data pipeline that failed.

To address these challenges, monitoring data pipelines in real-time and maintaining an end-to-end view is critical to identify issues early and minimize disruptions. In this blog post, we will explore how to orchestrate an end-to-end ETL pipeline using [AWS Glue](https://aws.amazon.com/glue), [AWS Step Functions](https://aws.amazon.com/pm/step-functions/), [Amazon Simple Storage Service](http://aws.amazon.com/s3) (Amazon S3), [AWS Distro for OpenTelemetry](https://aws.amazon.com/otel/) (ADOT) and [AWS X-ray](https://aws.amazon.com/xray/).

## **Solution overview**

This solution addresses a data engineering use case with two-phase pipeline approach: –

* Data Cleaning: The data cleaning phase is handled by an ETL job to convert arbitrary data formats (CSV, JSON) to Parquet and performs various transformations, including standardizing column names, dropping unnecessary columns, removing duplicates, filling null values, converting data types, fixing skewness in certain columns.  
* Data Processing: Next, the data processing phase is handled by another ETL job that calculates, ranks and groups data.

The data pipeline is orchestrated end to end using AWS Step Functions, leveraging AWS Glue for ETL jobs that are instrumented with ADOT and custom X-ray helper model for comprehensive tracing. Figure (1) illustrates architectural components involved in the orchestration of an end-to-end ETL pipeline. Here’s how it works: –

1. Upload a dataset, to the S3 bucket by creating a folder input.  
2. This triggers lambda function, that starts a new X-Ray trace and invokes Step Function workflow by passing traceID and S3 object details as an input.  
3. Step Function workflow passes this traceID as a parameter. Glue Jobs use X-ray helper module to retrieve the parent segment for the current job, and correlate with input traceID from step functions.  
4. Glue Job – 1 fetches input data set from S3 bucket to perform data cleaning and uploads the output to the same bucket by creating a folder clean.  
5. After completion of Glue Job \-1, step function workflow triggers Glue Job – 2\.  
6. Glue Job – 2 fetches input data set from S3 bucket to perform data processing job and uploads the output to the same bucket by creating a folder processed.  
7. Each Glue Job is instrumented with ADOT, to send collected trace data to OpenTelemetry Collector that is running as a sidecar on ECS.  
8. OpenTelemetry Collector then pushes collected trace data to AWS X-Ray.

{{< figure 
    src="/images/Blog2/Image-1.jpg" 
    alt="Solution Architecture"
    class="img-box"
>}}
<p style="text-align:center;">Figure 1: Solution Architecture</p>

By leveraging this solution, we maintain a single trace across the entire ETL pipeline that provides end to end visibility into the data processing workflow. Data engineers and MLOps professionals can gain deep insights, quickly identify bottlenecks or failures, and optimize their data processing workflows for improved efficiency and reliability.

## **Prerequisites**

To deploy this solution, you need:

* An AWS account. If you don’t already have an AWS account, you can [create one](https://aws.amazon.com/).  
* An [AWS Identity and Access Management (IAM)](http://aws.amazon.com/iam) role that launches [AWS CloudFormation](http://aws.amazon.com/cloudformation) templates which creates IAM roles to access services such as [AWS Glue](https://aws.amazon.com/glue), [AWS X-Ray](https://aws.amazon.com/xray/), [AWS Step Functions](https://aws.amazon.com/step-functions/), [AWS Lambda](https://aws.amazon.com/pm/lambda/), [Amazon ECS](https://aws.amazon.com/ecs/), [AWS CloudMap](https://aws.amazon.com/cloud-map/) and [Amazon Simple Storage Service](http://aws.amazon.com/s3).  
* Install [AWS CLI.](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)  
* Python 3.12 or later  
* Install [AWS CDK](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_install).

## **Deploy the Solution with AWS CDK**

We will use [AWS CDK](https://aws.amazon.com/cdk/) to deploy the solution. CDK allows defining the infrastructure through a familiar programming language such as Python.

* Clone the repository.
    ```bash
    git clone https://github.com/aws-samples/tracing-etl-workloads-otel.git cd tracing-etl-workloads-otel
    ```
* Create and activate a virtual environment:  
    * For Linux and macOS.
    ```bash
    python3 \-m venv .venv
    source .venv/bin/activate
    ```
    * For Windows.
    ```bash
    python \-m venv .venv
    .venv\\Scripts\\activate
    ```
* Install dependencies.
    ```bash
    pip install \-r requirements.txt
    ```
* Bootstrap the CDK environment (if not already done).
    ```bash
    cdk bootstrap
    ```
* Deploy the stack.
    ```bash
    cdk deploy
    ```
This will create all the necessary resources, including S3 buckets, Glue jobs, Step Functions state machine, and Lambda function. Once, the solution is deployed, on your AWS CloudFormation console, you can find the deployed resources on the “OtelSolutionStack” CloudFormation stack resource section.

## **Walk-through of the solution**

* Upload [Airbnb sample dataset](https://www.kaggle.com/datasets/arianazmoudeh/airbnbopendata) to the S3 ingestion bucket under input prefix folder.

    {{< figure 
        src="/images/Blog2/Image-2.png" 
        alt="Sample dataset uploaded to S3 bucket"
        class="img-box"
    >}}
    <p style="text-align:center;">Figure 2: Sample dataset uploaded to S3 bucket</p>

* Wait for the AWS StepFunction state machine workflow to succeed.

    {{< figure 
        src="/images/Blog2/Image-3.png" 
        alt="State machine workflow running Glue jobs"
        class="img-box"
    >}}
    <p style="text-align:center;">Figure 3: State machine workflow running Glue jobs</p>


* Navigate to [AWS CloudWatch console.](https://us-east-2.console.aws.amazon.com/cloudwatch/home?region=us-east-2#home:) Click on the “Xray traces” in left navigation pane and select the recent traceID.

    {{< figure 
        src="/images/Blog2/Image-4.png" 
        alt="Trace retrieval on AWS X-ray"
        class="img-box"
    >}}
    <p style="text-align:center;">Figure 4: Trace retrieval on AWS X-ray</p>


* A trace map is displayed by default. It is a visual, end-to-end representation of data flow through each component within the data pipeline
    {{< figure 
        src="/images/Blog2/Image-5.png" 
        alt="Trace Map, an end to end representation of data flow"
        class="img-box"
    >}}
    <p style="text-align:center;">Figure 5: Trace Map, an end to end representation of data flow.</p>


* Each component i.e., AWS Lambda, AWS Step Function, and AWS Glue will create its own segment within the trace.
    {{< figure 
        src="/images/Blog2/Image-6.png" 
        alt="Segments in a trace"
        class="img-box"
    >}}
    <p style="text-align:center;">Figure 6: Segments in a trace</p>



* You will see sub-segments under AWS Glue Segment because of the instrumentation done using AWS Distro for Open Telemetry.
    {{< figure 
        src="/images/Blog2/Image-7.png" 
        alt="Sub-segments in AWS Glue Segment"
        class="img-box"
    >}}
    <p style="text-align:center;">Figure 7: Sub-segments in AWS Glue Segment</p>


Figure 7: Sub-segments in AWS Glue Segment

This level of detail allows data engineers to identify which specific operation in a glue job is taking the most time.

## **Detailed Explanation of Trace Correlation**

The solution uses a custom X-Ray helper module to correlate traces across the ETL pipeline. Here’s how it works

* Glue Jobs:  
    * At the start of each Glue job, X-Ray helper module is used to retrieve the parent segment for the current job:
        ```python
        xray_trace = xray_helper.XRayTrace(TRACE_ID) 
        parent_id = xray_trace.retrieve_segment_for_step(JOB_NAME)  
        ```
    * If a parent segment is found, a new ADOT context is created using the X-Ray propagator:
        ```python
        carrier = {'X-Amzn-Trace-Id': f"Root={xray_trace.trace_id};Parent={parent_id};Sampled=1"} 
        propagator = AwsXRayPropagator() context = propagator.extract(carrier=carrier)
        ```
    * The Glue job execution is wrapped in a span using this context:
        ```python
        with tracer.start_as_current_span("Glue Job Execution", context=context, kind=trace.SpanKind.SERVER):     
        # Job execution code
        ```

    * Throughout the job, individual operations are wrapped in spans for detailed tracing:
        ```python
        with tracer.start_as_current_span("Read Data", attributes={'S3Path': s3_path}):     
        # Read data operation
        ```
 
* Trace Emission:  
    * The ADOT instrumentation is configured to send trace data to the OpenTelemetry Collector running as a sidecar. The OpenTelemetry Collector then forwards the trace data to AWS X-Ray.
        ```python
        processor = BatchSpanProcessor(OTLPSpanExporter(endpoint=f"http://{OTLP_ENDPOINT}:4318/v1/traces")) 
        tracer_provider = TracerProvider(resource=resource, active_span_processor=processor)
        ```
By using this approach, we maintain a single trace across the entire ETL pipeline, from the initial S3 upload trigger through the Step Functions execution and individual Glue job runs.

## **Understanding Distributed Tracing Concepts**

It’s important to understand some key concepts in distributed tracing:

* **Traces:** A trace represents the entire journey of a request or operation as it moves through a distributed system. In our ETL pipeline, a trace begins when data is uploaded to S3 and ends when the final processed data is written to Amazon S3.  
* **Spans:** Spans are the building blocks of a trace. Each span represents a unit of work or operation within the trace. For example, in our Glue jobs, we create spans for operations like “Read Data”, “Drop Columns”, or “Calculate Minimum Total Spend”. Spans have a start time, end time, and can include additional metadata like logs or tags. Example from our Glue job:
```python
with tracer.start_as_current_span("Calculate Minimum Total Spend"): 
spark_df = spark_df.withColumn("minimum_total_spend", (col("price") + col("service_fee")) * col('minimum_nights'))
```

* **Parent-Child Relationships:** Spans can have parent-child relationships, creating a hierarchical structure within a trace. In our solution, the overall Glue job execution is a parent span, while individual operations within the job are child spans.  
* **Segments:** In AWS X-Ray, a segment is similar to a span, but represents a unit of work done by a single component in your application. In our case, each Lambda invocation, Step Functions execution, and Glue job run creates a segment.  

* **Subsegments:** These are equivalent to child spans. They represent smaller units of work within a segment. In our Glue jobs, the individual operations we trace become subsegments in X-Ray.  

* **Trace Context Propagation:** To maintain a single trace across different components of a distributed system, we need to propagate the trace context. This is what allows us to link the Lambda function, Step Functions execution, and Glue jobs into a single trace. We do this by passing the trace ID and parent segment ID between components. Example of propagating context in our Glue job:
```python
carrier = {'X-Amzn-Trace-Id': f"Root={xray_trace.trace_id};Parent={parent_id};Sampled=1"}       
propagator = AwsXRayPropagator()     
context = propagator.extract(carrier=carrier)  
```

* **Attributes and Annotations:** These are key-value pairs that provide additional context to spans or segments. We use these to add useful information like S3 paths, job names, or data statistics. Example from our code:  
```python
with tracer.start_as_current_span("Glue Job Execution", context=context, kind=trace.SpanKind.SERVER, attributes={'job_name': JOB_NAME, 'job_run_id': JOB_RUN_ID})
```
* **Sampling:** In high-volume systems, it’s often impractical to trace every single request. Sampling is the practice of only tracing a subset of requests. AWS X-Ray provides configurable sampling rules to control how much data you collect and store.

## **OpenTelemetry Collector Configuration**

A key component in our tracing setup is the OpenTelemetry Collector, which runs as a sidecar alongside Glue jobs. The collector is responsible for receiving trace data from our instrumented Glue jobs and forwarding it to AWS X-Ray.

Let’s break down the configuration file (otel-agent-config.yaml) that defines how the collector operates.

* Receivers:  
  * The collector is configured to receive OpenTelemetry Protocol (OTLP) data via both gRPC (port 4317) and HTTP (port 4318). This allows our Glue jobs to send trace data using either protocol.  
* Exporters:  
  * The awsxray exporter is configured to send the collected trace data to AWS X-Ray. This is how our trace data ultimately ends up in X-Ray for visualization and analysis.  
* Processors:  
  * A memory limiter processor is configured to prevent the collector from consuming too much memory. It’s set to limit memory usage to 100 MiB and check every 5 seconds.

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
exporters:
  awsxray:
    region: us-east-2
processors:
  memory_limiter:
    limit_mib: 100
    check_interval: 5s
service:
  pipelines:
    traces:
      processors:
        - memory_limiter
      receivers:
        - otlp
      exporters:
        - awsxray
```
This OpenTelemetry Collector configuration ensures the effective capture, processing, and export of trace data to AWS X-Ray for visibility and analysis of our data processing pipelines

## **Clean up**

You can either delete the stack through the [CloudFormation console](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks?filteringText=&filteringStatus=active&viewNested=true) or use AWS CDK destroy from the root folder.
```bash
cdk destroy
```
When deleting a stack, most resources will be deleted upon stack deletion, however that’s not the case for all resources. In this solution, lambda functions will generate [Amazon CloudWatch](https://aws.amazon.com/cloudwatch) logs that are retained even after stack deletion. These won’t be tracked by CloudFormation because they’re not part of the stack, so the logs will persist. Follow steps [here](https://docs.aws.amazon.com/solutions/latest/video-on-demand-on-aws-foundation/deleting-the-cloudwatch-logs.html), to manually delete any logs that you don’t want to retain from CloudWatch console.

## **Conclusion**

This solution provides an end-to-end observability view of a data pipeline workflow by propagating context and tracing individual operations through AWS Distro for OpenTelemetry and AWS X-Ray. This enables data engineers to quickly troubleshoot failures, optimize performance, and gain insights into how data characteristics impact downstream processing. Such observability is crucial for modern data platforms involving complex, distributed workflows across multiple components and services.
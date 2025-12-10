---
title: "Blog 2"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# Theo dõi quy trình ETL với AWS X-Ray và AWS Distro for OpenTelemetry

**Tác giả**: *Praneeth Reddy Tekula, Deepak Kovvuri, và Viveyk Karri*

**Ngày đăng**:*30 tháng 8, 2025*

*Chuyên mục:* [*AWS Distro for OpenTelemetry*](https://aws.amazon.com/blogs/mt/category/management-tools/aws-distro-for-opentelemetry/)*,* [*AWS Glue*](https://aws.amazon.com/blogs/mt/category/analytics/aws-glue/)*,* [*AWS X-Ray*](https://aws.amazon.com/blogs/mt/category/developer-tools/aws-x-ray/)*,* [*Management Tools*](https://aws.amazon.com/blogs/mt/category/management-tools/)*,* [*Technical How-to*](https://aws.amazon.com/blogs/mt/category/post-types/technical-how-to/)

## Giới thiệu

Các data pipeline là yếu tố thiết yếu đối với các công ty định hướng dữ liệu hiện đại nhằm thu được những insight kinh doanh quan trọng. Tuy nhiên, các pipeline này thường gặp lỗi khi những file hoặc dataset mới từ nguồn dữ liệu không tuân theo schema mong đợi, dẫn đến việc các job ở tầng downstream bị lỗi, workflow bị gián đoạn, và việc tạo ra insight bị trì hoãn. Bên cạnh đó, sự dao động của khối lượng dữ liệu — từ vài gigabyte đến hàng terabyte — có thể ảnh hưởng đáng kể đến hiệu suất và độ tin cậy của các pipeline này. Điều này khiến việc xác định ETL job cụ thể bị lỗi trong toàn bộ pipeline trở nên khó khăn hơn đối với người dùng.

Để giải quyết những thách thức này, việc giám sát data pipeline theo thời gian thực và duy trì góc nhìn end-to-end là rất quan trọng nhằm phát hiện sớm sự cố và giảm thiểu gián đoạn. Trong bài viết này, chúng ta sẽ cùng tìm hiểu cách orchestrate một ETL pipeline end-to-end bằng cách sử dụng [AWS Glue](https://aws.amazon.com/glue), [AWS Step Functions](https://aws.amazon.com/pm/step-functions/), [Amazon Simple Storage Service](http://aws.amazon.com/s3) (Amazon S3), [AWS Distro for OpenTelemetry](https://aws.amazon.com/otel/) (ADOT) và [AWS X-Ray](https://aws.amazon.com/xray/).

## Tổng quan về giải pháp

Giải pháp này giải quyết một trường hợp sử dụng trong kỹ thuật dữ liệu (data engineering) với cách tiếp cận pipeline hai giai đoạn:

*   **Giai đoạn làm sạch dữ liệu (Data Cleaning)**: Giai đoạn làm sạch dữ liệu được xử lý bởi một ETL job để chuyển đổi các định dạng dữ liệu tùy ý (CSV, JSON) sang Parquet và thực hiện nhiều phép biến đổi (transformations) khác nhau, bao gồm chuẩn hóa tên cột (standardizing column names), loại bỏ các cột không cần thiết (dropping unnecessary columns), xóa các bản ghi trùng lặp (removing duplicates), điền các giá trị null (filling null values), chuyển đổi các kiểu dữ liệu (converting data types), và khắc phục độ lệch (fixing skewness) trong một số cột nhất định.
*   **Giai đoạn xử lý dữ liệu (Data Processing)**: Tiếp theo, giai đoạn xử lý dữ liệu được xử lý bởi một ETL job khác, có nhiệm vụ tính toán, xếp hạng và nhóm dữ liệu.

Data pipeline được điều phối từ đầu đến cuối (end to end) bằng AWS Step Functions, tận dụng AWS Glue cho các ETL job được tích hợp ADOT và X-ray helper model tùy chỉnh để có khả năng truy vết toàn diện (comprehensive tracing). Hình (1) minh họa các thành phần kiến trúc liên quan đến việc điều phối một ETL pipeline end-to-end. Cách thức hoạt động như sau:

1.  Tải một dataset lên S3 bucket bằng cách tạo một thư mục input.
2.  Thao tác này kích hoạt lambda function, khởi tạo một X-Ray trace mới và gọi Step Function workflow bằng cách truyền `traceID` và chi tiết đối tượng S3 làm đầu vào (input).
3.  Step Function workflow truyền `traceID` này dưới dạng một tham số. Glue Jobs sử dụng X-ray helper module để truy xuất parent segment cho job hiện tại và tương quan với input `traceID` từ step functions.
4.  Glue Job – 1 tìm nạp input data set từ S3 bucket để thực hiện data cleaning và tải đầu ra (output) lên cùng một bucket bằng cách tạo một thư mục clean.
5.  Sau khi Glue Job – 1 hoàn thành, step function workflow kích hoạt Glue Job – 2.
6.  Glue Job – 2 tìm nạp input data set từ S3 bucket để thực hiện data processing job và tải đầu ra (output) lên cùng một bucket bằng cách tạo một thư mục processed.
7.  Mỗi Glue Job được tích hợp ADOT để gửi dữ liệu trace đã thu thập đến OpenTelemetry Collector, chạy dưới dạng một sidecar trên ECS.
8.  OpenTelemetry Collector sau đó đẩy dữ liệu trace đã thu thập đến AWS X-Ray.

{{< figure 
    src="/images/Blog2/Image-1.jpg" 
    alt="Solution Architecture"
    class="img-box"
>}}
<p style="text-align:center;">Hình 1: Solution Architecture</p>


Nhờ cách này, chúng ta duy trì một trace duy nhất xuyên suốt pipeline ETL, giúp mọi người có thể nhìn thấy luồng dữ liệu end-to-end. Các kỹ sư dữ liệu và người làm MLOps có thể có insight sâu, nhanh chóng xác định điểm tắc nghẽn hoặc lỗi, và tối ưu hóa workflow để tăng hiệu quả và độ tin cậy.

## Yêu cầu trước khi thực hiện

Để triển khai giải pháp này, bạn cần:

*   Một tài khoản AWS. Nếu bạn chưa có, bạn có thể [tạo một tài khoản](https://aws.amazon.com/).
*   Một [AWS Identity and Access Management (IAM)](http://aws.amazon.com/iam) role để khởi tạo template [CloudFormation](http://aws.amazon.com/cloudformation), tạo các role IAM cần thiết để truy cập các dịch vụ như [AWS Glue](https://aws.amazon.com/glue), [AWS X-Ray](https://aws.amazon.com/xray/), [AWS Step Functions](https://aws.amazon.com/step-functions/), [AWS Lambda](https://aws.amazon.com/pm/lambda/), [Amazon ECS](https://aws.amazon.com/ecs/), [AWS CloudMap](https://aws.amazon.com/cloud-map/) và [Amazon Simple Storage Service](http://aws.amazon.com/s3).
*   Cài đặt [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
*   Python 3.12 hoặc mới hơn.
*   Cài đặt [AWS CDK](https://docs.aws.amazon.com/cdk/v2/guide/getting_started.html#getting_started_install).

## Triển khai giải pháp với AWS CDK

Chúng ta sẽ dùng [AWS CDK](https://aws.amazon.com/cdk/) để triển khai giải pháp này. CDK cho phép định nghĩa cơ sở hạ tầng qua một ngôn ngữ lập trình quen thuộc như Python.

1. Clone repository:

```
git clone <https://github.com/aws-samples/tracing-etl-workloads-otel.git> cd tracing-etl-workloads-otel
``` 

2. Tạo và kích hoạt môi trường ảo (virtual environment):
    * Đối với Linux và macOS:
       ```bash
       python3 -m venv .venv
       source .venv/bin/activate
       ``` 
    * Đối với Windows:
       ```bash
       python -m venv .venv
       .venv\\Scripts\\activate
       ``` 

3. Cài đặt các dependencies.
    ```bash
    pip install -r requirements.txt
    ``` 

4.  Bootstrap môi trường CDK (nếu chưa thực hiện).

    ```bash
    cdk bootstrap
    ``` 

5.  Triển khai (Deploy) stack.

    ```bash
    cdk deploy
    ```` 

Điều này sẽ tạo các tài nguyên cần thiết, bao gồm: S3 buckets, Glue jobs, Step Functions state machine, and Lambda function. Sau khi triển khai xong, bạn có thể xem các tài nguyên trong console CloudFormation, trong stack “OtelSolutionStack”.

## Hướng dẫn chi tiết về giải pháp

1.  Tải [dataset mẫu Airbnb](https://www.kaggle.com/datasets/arianazmoudeh/airbnbopendata) lên S3 ingestion bucket, trong thư mục con (prefix folder).
  {{< figure 
      src="/images/Blog2/Image-2.png" 
      alt="Solution Architecture"
      class="img-box"
  >}}

  <p style="text-align:center;">Hình 2: Dataset mẫu đã được tải lên S3 bucket.</p>


2.  Chờ AWS Step Functions hoàn tất workflow của state machine và trạng thái hiển thị “Succeeded”.

{{< figure 
    src="/images/Blog2/Image-3.png" 
    alt="Solution Architecture"
    class="img-box"
>}}
<p style="text-align:center;">Hình 3: Workflow của State Machine đang chạy các Glue Jobs</p>

3.  Điều hướng đến [AWS CloudWatch Console](https://us-east-2.console.aws.amazon.com/cloudwatch/home?region=us-east-2#home:). Trong thanh điều hướng bên trái, chọn “X-Ray traces”, sau đó chọn `traceID` gần nhất.

{{< figure 
    src="/images/Blog2/Image-4.png" 
    alt="Solution Architecture"
    class="img-box"
>}}
<p style="text-align:center;">Hình 4: Truy xuất Trace trên AWS X-Ray</p>


4.  Một bản đồ trace (trace map) sẽ được hiển thị mặc định. Đây là biểu diễn trực quan, end-to-end của luồng dữ liệu đi qua từng thành phần trong data pipeline.

{{< figure 
    src="/images/Blog2/Image-5.png" 
    alt="Solution Architecture"
    class="img-box"
>}}
<p style="text-align:center;">Hình 5: Trace Map – biểu diễn end-to-end của luồng dữ liệu (data flow)</p>


5.  Mỗi thành phần, bao gồm AWS Lambda, AWS Step Functions, và AWS Glue, sẽ tạo một segment riêng biệt trong trace.

{{< figure 
    src="/images/Blog2/Image-6.png" 
    alt="Solution Architecture"
    class="img-box"
>}}
<p style="text-align:center;">Hình 6: Các Segment trong một Trace</p>


6.  Bạn sẽ thấy các sub-segment nằm trong AWS Glue Segment, do quá trình instrumentation được thực hiện bằng AWS Distro for OpenTelemetry (ADOT).

{{< figure 
    src="/images/Blog2/Image-7.png" 
    alt="Solution Architecture"
    class="img-box"
>}}
<p style="text-align:center;">Hình 7: Các Sub-segment trong AWS Glue Segment</p>


7.  Mức độ chi tiết này giúp data engineers xác định được thao tác cụ thể nào trong Glue job đang tiêu tốn nhiều thời gian nhất.

## Giải thích chi tiết về việc tương quan trace (Trace Correlation)

Giải pháp sử dụng mô-đun hỗ trợ X-Ray tùy chỉnh để tương quan các trace giữa các phần trong pipeline ETL. Cách hoạt động như sau:

*   **Glue Jobs:**
    *   Khi bắt đầu mỗi job Glue, mô-đun X-Ray helper dùng để lấy segment cha cho job hiện tại:
    ```python
    xray_trace = xray_helper.XRayTrace(TRACE_ID)
    parent_id = xray_trace.retrieve_segment_for_step(JOB_NAME)

    ````

    * Nếu tìm được segment cha, ta tạo một context mới cho ADOT sử dụng propagator của X-Ray:

    ``` python
    carrier = {'X-Amzn-Trace-Id': f"Root={xray_trace.trace_id};Parent={parent_id};Sampled=1"}
    propagator = AwsXRayPropagator() context = propagator.extract(carrier=carrier)
    ```


    *   Việc thực thi job Glue được wrap trong một span với context này:
    ``` python
    with tracer.start_as_current_span("Glue Job Execution", context=context, kind=trace.SpanKind.SERVER):
    # Job execution code
    ```


    *   Trong job, các thao tác riêng biệt được wrap bằng các span con để trace chi tiết hơn:
    ``` python
    with tracer.start_as_current_span("Read Data", attributes={'S3Path': s3_path}):
    # Read data operation
    ```

  * **Trace Emission:**
      * Instrumentation ADOT được cấu hình để gửi dữ liệu trace đến OpenTelemetry Collector chạy như sidecar. Collector sau đó gửi dữ liệu trace đến AWS X-Ray:
    ``` python
    processor = BatchSpanProcessor(OTLPSpanExporter(endpoint=f"http://{OTLP_ENDPOINT}:4318/v1/traces"))
    tracer_provider = TracerProvider(resource=resource, active_span_processor=processor)

    ```

Nhờ cách này, chúng ta giữ một trace duy nhất xuyên suốt pipeline ETL, từ khi dữ liệu được upload lên S3, qua Step Functions đến các job Glue.

## Khái niệm về Distributed Tracing

Một số khái niệm quan trọng cần hiểu:

  * **Trace:** Một trace đại diện cho toàn bộ hành trình của một yêu cầu (request) hoặc một hoạt động (operation) khi nó di chuyển qua một hệ thống phân tán. Trong pipeline ETL của chúng ta, một trace bắt đầu khi dữ liệu được tải lên Amazon S3 và kết thúc khi dữ liệu đã được xử lý cuối cùng được ghi trở lại vào Amazon S3.
  * **Span:** Thành phần cấu tạo của trace. Mỗi span đại diện cho một đơn vị công việc hay hoạt động trong trace. Ví dụ: trong job Glue, ta tạo span cho các thao tác như “Read Data”, “Drop Columns”, “Calculate Minimum Total Spend”. Span có thời điểm bắt đầu, thời điểm kết thúc và có thể chứa metadata (như logs hoặc tags). Ví dụ từ Glue Job của chúng tôi:
    ``` python
    with tracer.start_as_current_span("Calculate Minimum Total Spend"):
    spark_df = spark_df.withColumn("minimum_total_spend", (col("price") + col("service_fee")) * col('minimum_nights'))

    ```

  * **Quan hệ Cha – Con (Parent-Child):** Spans có thể có quan hệ cha – con, tạo nên cấu trúc phân cấp trong một trace. Ở giải pháp này, span thực thi Glue job là span cha, và các thao tác bên trong là span con.
  * **Segment:** Trong AWS X-Ray, một segment tương đương một span, nhưng nó đại diện cho một đơn vị công việc được thực hiện bởi một thành phần duy nhất trong ứng dụng. (ví dụ: Lambda invocation, Step Functions execution, Glue job run).
  * **Subsegment:** Tương đương với span con, đại diện tầng con trong một segment. Trong các Glue jobs của chúng ta, những hoạt động riêng lẻ được theo dõi (trace) sẽ trở thành subsegments trong X-Ray.
  * **Trace Context Propagation:** Để duy trì một trace xuyên suốt giữa các thành phần của hệ thống phân tán, cần truyền ngữ cảnh trace (trace context). Điều này cho phép chúng ta liên kết Lambda, Step Functions và các Glue jobs vào cùng một trace. Chúng ta thực hiện điều này bằng cách truyền trace ID và parent segment ID giữa các thành phần. Ví dụ:
    ``` python
    carrier = {'X-Amzn-Trace-Id': f"Root={xray_trace.trace_id};Parent={parent_id};Sampled=1"}
    propagator = AwsXRayPropagator()
    context = propagator.extract(carrier=carrier)

    ```

  * **Attributes và Annotations:** Là các cặp key‐value cung cấp thêm ngữ cảnh cho span hoặc segment. Ta sử dụng chúng để thêm thông tin như S3 path, tên job, thống kê dữ liệu. Ví dụ:

<!-- end list -->

``` python
with tracer.start_as_current_span("Glue Job Execution", context=context, kind=trace.SpanKind.SERVER, attributes={'job_name': JOB_NAME, 'job_run_id': JOB_RUN_ID})

```

  * **Sampling:** Trong hệ thống có lưu lượng cao, việc trace mọi yêu cầu là không khả thi. Sampling là phương pháp chỉ trace một phần yêu cầu. AWS X-Ray cung cấp các quy tắc sampling cấu hình được để kiểm soát lượng dữ liệu trace thu thập và lưu trữ.

## Cấu hình OpenTelemetry Collector

Một thành phần quan trọng trong setup tracing là OpenTelemetry Collector, chạy như sidecar kèm theo các job Glue. Collector chịu trách nhiệm nhận dữ liệu trace từ các job đã instrumentation và chuyển tiếp đến AWS X-Ray.

Hãy phân tích file cấu hình (otel-agent-config.yaml) để xác định mô hình cấu trúc dịch vụ trong collector:

  * **Receivers:** Collector được cấu hình để nhận dữ liệu OpenTelemetry Protocol (OTLP) qua gRPC (cổng 4317) và HTTP (cổng 4318). Điều này cho phép job Glue gửi dữ liệu trace theo cả hai giao thức.
  * **Exporters:** `awsxray` exporter được cấu hình để gửi dữ liệu trace đã thu thập đến AWS X-Ray. Đây là cách dữ liệu trace của chúng tôi kết thúc bằng việc đưa vào X-Ray để trực quan hóa và phân tích.
  * **Processors:** Sử dụng bộ giới hạn bộ nhớ (`memory_limiter processor`) để ngăn collector tiêu tốn quá nhiều bộ nhớ. Giới hạn đặt 100 MiB và kiểm tra mỗi 5 giây.

Dưới đây là mô hình cấu trúc dịch vụ trong collector:

``` yaml
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

Cấu hình này đảm bảo việc thu thập, xử lý và xuất dữ liệu trace đến AWS X-Ray để quan sát và phân tích pipeline xử lý dữ liệu.

## Dọn dẹp tài nguyên

Bạn có thể xóa stack bằng console [CloudFormation](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=us-east-2#/stacks?filteringText=&filteringStatus=active&viewNested=true) hoặc dùng lệnh sau từ thư mục gốc:

``` bash
cdk destroy

```

Khi xóa stack, hầu hết các tài nguyên sẽ bị xóa cùng với stack, nhưng có một số ngoại lệ. Trong giải pháp này, các hàm Lambda sẽ tạo ra các log của [Amazon CloudWatch](https://aws.amazon.com/cloudwatch) — những log này vẫn được giữ lại ngay cả sau khi stack bị xóa. Các log này sẽ không được CloudFormation theo dõi vì chúng không phải là một phần của stack, do đó chúng sẽ tiếp tục tồn tại. Hãy làm theo các bước được hướng dẫn [tại đây](https://docs.aws.amazon.com/solutions/latest/video-on-demand-on-aws-foundation/deleting-the-cloudwatch-logs.html) để xóa thủ công bất kỳ log nào bạn không muốn giữ lại từ giao diện CloudWatch console.

## Kết luận

Giải pháp này cung cấp cái nhìn quan sát (observability) end-to-end cho workflow pipeline dữ liệu bằng cách truyền ngữ cảnh và trace các hoạt động cá nhân qua AWS Distro for OpenTelemetry và AWS X-Ray. Điều này giúp kỹ sư dữ liệu nhanh chóng xử lý lỗi, tối ưu hiệu suất và có insight về cách tính chất dữ liệu ảnh hưởng đến quá trình xử lý sau đó. Quan sát này rất quan trọng đối với các nền tảng dữ liệu hiện đại bao gồm workflow phân tán phức tạp qua nhiều thành phần và dịch vụ.


---
title: "Translated Blogs"
date: "`r Sys.Date()`"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

List blogs translated:

###  [Blog 1 - Training AI models for skill-based matchmaking using Amazon SageMaker AI](3.1-Blog1/)
This blog introduces how to build a machine learning workflow to generate more accurate, data-driven skill ratings for multiplayer matchmaking using Amazon SageMaker AI. You will learn why traditional skill formulas struggle with large, diverse gameplay metrics, and how supervised learning can automatically identify which factors best represent true player skill. The article walks you through setting up a SageMaker AI domain, preparing a regression-based pipeline with Autopilot, deploying the ML workflow, and integrating it with game backend components such as API Gateway, Lambda, DynamoDB, and GameLift FlexMatch. By following the steps, developers can automate data preprocessing, model training, evaluation, and deployment, ultimately enabling more balanced, fair, and engaging matchmaking experiences.

###  [Blog 2 - Tracing ETL Workloads using AWS X-Ray and AWS Distro for OpenTelemetry](3.2-Blog2/)
This blog explains how to add end-to-end observability to an ETL pipeline using AWS Glue, Step Functions, Lambda, AWS Distro for OpenTelemetry, and AWS X-Ray. When a dataset is uploaded to S3, a Lambda function starts a new trace, and this trace ID is passed through the entire workflow so both Glue jobs (data cleaning and data processing) appear as connected spans on X-Ray. ADOT instrumentation and an OpenTelemetry Collector sidecar send detailed sub-segment data from each transformation step, allowing engineers to see the entire pipeline flow, pinpoint failures, identify slow operations, and optimize performance. The article also shows how to deploy the architecture with AWS CDK, configure the collector, and clean up resources.

###  [Blog 3 - The AWS Imagine Grant launches the 2025 cycle in six countries, expanding its global reach](3.3-Blog3/)
This blog announces the launch of the 2025 AWS Imagine Grant cycle, now open to registered nonprofit organizations in six countries (US, UK, Ireland, Canada, Australia, and New Zealand) that are using cloud technology to accelerate their missions. It explains how the program supports nonprofits with unrestricted financial funding, AWS Promotional Credit, and technical guidance, especially for projects using advanced services like AI/ML, HPC, IoT, and generative AI. The article outlines three award categories—Pathfinder – Generative AI for data-driven, mission-critical generative AI projects; Go Further, Faster for highly innovative, scalable cloud solutions; and Momentum to Modernize for core infrastructure modernization—along with typical award amounts and regional availability. It also provides key dates (applications opening April 14, 2025, with first deadlines on June 2, 2025), links to application instructions by country, and points readers to past winners and resources that show how nonprofits are using AWS to drive social and environmental impact.

---
title: "Blog 1"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Training AI models for skill-based matchmaking using Amazon SageMaker AI

*Authors: Christina Defoor and Alexander Qin - April 16, 2025*

***Categories:*** [Amazon DynamoDB](https://aws.amazon.com/blogs/gametech/category/database/amazon-dynamodb/), [Amazon GameLift](https://aws.amazon.com/blogs/gametech/category/game-development/amazon-gamelift/), [Amazon SageMaker](https://aws.amazon.com/blogs/gametech/category/artificial-intelligence/sagemaker/), [Artificial Intelligence](https://aws.amazon.com/blogs/gametech/category/artificial-intelligence/), [AWS Lambda](https://aws.amazon.com/blogs/gametech/category/compute/aws-lambda/), [Compute](https://aws.amazon.com/blogs/gametech/category/compute/), [Database](https://aws.amazon.com/blogs/gametech/category/database/), [Game Development](https://aws.amazon.com/blogs/gametech/category/game-development/), [Industries](https://aws.amazon.com/blogs/gametech/category/industries/), [Media & Entertainment](https://aws.amazon.com/blogs/gametech/category/industries/entertainment/)

In competitive multiplayer games, **skill-based matchmaking** is crucial for creating fun and competitive games. Determining player skill today is difficult due to the vast array of metrics games record (such as hits, misses, assists, time played, level, and more), making it challenging to determine which factors are most indicative of skill. Instead of manually creating algorithms to determine player skill, machine learning (ML) techniques (particularly supervised learning) can automatically identify patterns across game metrics to produce more accurate skill ratings. These ML-derived ratings enable more balanced matchmaking, ultimately enhancing player satisfaction and engagement.

In this first part of our two-part blog series, we'll show you how to use [Amazon SageMaker AI](https://aws.amazon.com/sagemaker-ai/) to quickly build and deploy an automated ML pipeline. Amazon SageMaker AI provides the capabilities to build, train, and deploy ML and foundation models, with fully managed infrastructure, tools, and workflows. The model and pipeline we build will produce a value that is a more reflective and precise rating of each player's skill.

To accomplish this task, we will be building upon the [Guidance for AI-Driven Player Insights on Amazon Web Services (AWS)](https://aws.amazon.com/solutions/guidance/ai-driven-player-insights-on-aws/). The architecture diagram for the guidance shows how game studios can leverage this low-code solution to quickly build, train, and deploy high-quality models that predict player skill using historic player data. Operators just upload their historic player data to [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/). This invokes a complete workflow to extract insights, select algorithms, tune hyperparameters, evaluate models, and deploy the best performing model for your dataset to a prediction API orchestrated by [Amazon SageMaker Pipelines](https://aws.amazon.com/sagemaker-ai/pipelines/).

<img src="/images/Blog1/Image-1.png" alt=" Architecture diagram of the Guidance for AI-Driven Player Insights on AWS" >
*Figure 1: Architecture diagram of the Guidance for AI-Driven Player Insights on AWS.*

Figure 2 shows the architecture you will be implementing in this blog. The diagram shows the flow of how a player's matchmaking request is handled. The matchmaking request triggers [Amazon API Gateway](https://aws.amazon.com/api-gateway/?nc2=type_a), invoking an [AWS Lambda](https://aws.amazon.com/lambda/?nc2=type_a) function which receives the relevant player data from [Amazon DynamoDB](https://aws.amazon.com/dynamodb/?nc2=type_a). The data is then passed to the Amazon SageMaker AI endpoint, which runs an inference to produce the more holistic skill value used by [Amazon GameLift FlexMatch](https://docs.aws.amazon.com/gamelift/latest/flexmatchguide/match-intro.html) in the matchmaking process.

<img src="/images/Blog1/Image-2.png" alt=" Matchmaking workflow using matchmaking simulator" >
*Figure 2: Matchmaking workflow using matchmaking simulator.*

The following architecture diagram shows how you will implement this solution in an actual game by connecting FlexMatch to an [Amazon GameLift Servers](https://aws.amazon.com/gamelift/servers/) queue. This triggers GameLift Servers to place or spin up game servers for the newly created matches.

<img src="/images/Blog1/Image-3.png" alt=" Matchmaking workflow as part of a game backend" >
*Figure 3: Matchmaking workflow as part of a game backend.*

## Walkthrough:

### Prerequisites

Before starting, you should have the following prerequisites:

*   An AWS account with [IAM permissions (administrative access) to create and use a SageMaker AI Domain](https://docs.aws.amazon.com/sagemaker/latest/dg/gs-set-up.html).
*   A code editor.
*   To download the data file you will be using for this tutorial, choose [PlayerStats.csv](https://github.com/aws-samples/sample-ai-powered-multiplayer-matchmaking-sagemaker-and-gamelift-flexmatch/blob/main/PlayerStats.csv) on the GitHub repository and select the "Download raw file" option, as shown in Figure 4.
<img src="/images/Blog1/Image-4.png" alt=" PlayerStats.csv download page" >
*Figure 4: PlayerStats.csv download page.*

*   Optional: If you would like to replace the code files we will be modifying throughout this tutorial rather than modifying the code line by line, choose the following links to download each of the files using the "Download raw file" option:
    *   [workflow.py](https://github.com/aws-samples/sample-ai-powered-multiplayer-matchmaking-sagemaker-and-gamelift-flexmatch/blob/main/workflow.py)
    *   [evaluation.py](https://github.com/aws-samples/sample-ai-powered-multiplayer-matchmaking-sagemaker-and-gamelift-flexmatch/blob/main/evaluation.py)
    *   [skill_inference.py](https://github.com/aws-samples/sample-ai-powered-multiplayer-matchmaking-sagemaker-and-gamelift-flexmatch/blob/main/skill_inference.py)
*   The prerequisites found in the [Guidance for AI-driven player insights on AWS](https://github.com/aws-solutions-library-samples/guidance-for-ai-driven-player-insights-on-aws) GitHub repository.

### Setting up SageMaker AI domain

1.  Open the [SageMaker AI console](https://console.aws.amazon.com/sagemaker/).
2.  Under the left side menu, choose **Domains**.
3.  Choose **Create domain**.
    <img src="/images/Blog1/Image-5.png" alt=" Amazon SageMaker domain dashboard" >
  *Figure 5: Amazon SageMaker domain dashboard.*
4.  Select the **Quick Setup** option and choose **Set up**.
    <img src="/images/Blog1/Image-6.png" alt=" Amazon SageMaker domain creation" >
  *Figure 6: Amazon SageMaker domain creation.*

### Access the Amazon SageMaker Studio dashboard

1.  Select the domain you created in the previous section.
    <img src="/images/Blog1/Image-7.png" alt=" Amazon SageMaker domain selection" >
  *Figure 7: Amazon SageMaker domain selection.*   

2.  Under the User profiles section, select the **Launch** dropdown button located next to the profile you want to use. Select **SageMaker Studio** which will open in another tab. Leave this tab open, we will be coming back to this in a later section.
    <img src="/images/Blog1/Image-8.png" alt=" Domain user profiles" >
  *Figure 8: Domain user profiles.*

### Deploying the AI-driven player insights solution

Follow the deployment steps provided by the AWS AI-Driven Player Insights repository linked in the prerequisites section of this blog. You will need to deploy the solution using your own device if you do not have access to an AWS Cloud9 development environment. Follow the Deployment Steps up to Step 5 within the guide.

### Understanding AI-driven player insights on AWS

Traditionally, developing effective machine learning models requires data science experience since builders must determine appropriate data pre-processing methods based on metric relationships, select optimal machine learning algorithms, and establish model performance evaluation strategies. With this solution, you will no longer need extensive machine learning experience to build and deploy machine learning models. Instead, you will leverage the feature [Amazon SageMaker Autopilot](https://docs.aws.amazon.com/sagemaker/latest/dg/use-auto-ml.html).

Amazon SageMaker Autopilot automates the complete process of building, training, tuning, and deploying machine learning models. Amazon SageMaker Autopilot analyzes your data, selects algorithms suitable for your problem type, and preprocesses the data for training. It also handles automatic model training and performs hyper-parameter optimization to find the best performing model for your dataset. To accelerate the process of training and deploying machine learning models as data changes over time, our solution provides a pre-defined machine learning pipeline. The pipeline triggers the entire process and model deployment from the moment your data is uploaded to your S3 bucket.

This solution requires a player statistics dataset. The dataset should include relevant performance metrics for your game and the current skill rating used for matchmaking. For this tutorial, we will use the PlayerStats.csv file you downloaded in the prerequisites section.

### Preparing the machine learning pipeline for linear regression

The AI-driven player insights machine learning pipeline, by default, is configured for predicting player churn with an output of "True" or "False". Since the output value for this example refers to categorical data with discrete outcomes, this is set up for a classification problem. In our tutorial, we are looking to output numeric data for a player's "Skill". Since we are looking at supervised learning for numeric data with continuous outcomes, we need to modify this pipeline for linear regression, the most common supervised learning model for predicting numeric data.

1.  In your preferred code editor, open the `/player-insights/constants.py` file and replace its contents with the following (Be sure to replace the values for `SM_DOMAIN_ID` and `REGION` to your own specific values.):

    ```python
    WORKLOAD_NAME = "PlayerSkills"
    REGION = "[YOUR REGION]"
    SM_DOMAIN_ID = "[YOUR SAGEMAKER AI DOMAIN ID]"
    DATA_FILE = "PlayerStats.csv"
    TARGET_ATTRIBUTE = "playerSkill"
    PERFORMANCE_THRESHOLD = 0.00
    ENDPOINT_TYPE = "SERVERLESS"
    ```

2.  Since we will be training a linear regression model, we will need to change the evaluation to Mean Squared Error (MSE). Replace the `/player-insights/evaluation.py` main function with the following code:

    ```python
    if __name__ == "__main__":
        logger.debug("Starting Evaluation ...")
        logger.info("Reading Test Predictions")
        y_pred_path = "/opt/ml/processing/input/predictions/x_test.csv.out"
        y_pred = pd.read_csv(y_pred_path, header=None).squeeze()
        logger.info("Reading Test Labels")
        y_true_path = "/opt/ml/processing/input/true_labels/y_test.csv"
        y_true = pd.read_csv(y_true_path, header=None).squeeze()
        mse = mean_squared_error(y_true, y_pred)
        logger.info(f"Mean Squared Error: {mse}")
        report_dict = {
            "regression_metrics": {
                "mean_squared_error": {
                    "value": mse,
                    "standard_deviation": "NaN",
                },
            },
        }
        output_dir = "/opt/ml/processing/evaluation"
        pathlib.Path(output_dir).mkdir(parents=True, exist_ok=True)
        evaluation_path = os.path.join(output_dir, "evaluation_metrics.json")
        logger.info("Saving Evaluation Report")
        with open(evaluation_path, "w") as f:
            f.write(json.dumps(report_dict))
    ```

3.  Open the `/player-insights/workflow.py` file. Here we are adjusting the machine learning pipeline to use the MSE metric for evaluation rather than the F1 score (since F1 score is used for classification problems). To do this, we will modify the failure step defined around line 227 to send a message indicating the MSE is less than the specified threshold.

    ```python
    failure_step = FailStep(
        name="ModelEvaluationFailure",
        error_message=Join(
            on=" ",
            values=["Pipeline execution failure: MSE is less than the specified Evaluation Threshold"]
        )
    )
    ```

4.  You will also need to modify the conditional step defined around line 259 in this same file to use the MSE metric for evaluation instead of the weighted F1 score.

    ```python
    conditional_step = ConditionStep(
        name="ModelQualityCondition",
        conditions=[
            ConditionGreaterThanOrEqualTo(
                left=JsonGet(
                    step_name=evaluation_step.name,
                    property_file=evaluation_report,
                    json_path="regression_metrics.mean_squared_error.value"
                ),
                right=metric_threshold
            )
        ],
        if_steps=[step_register_model, deployment_step],
        else_steps=[failure_step]
    )
    ```

5.  Verify that the cloud development kit (CDK) deployment correctly synthesizes the proper [AWS CloudFormation](https://aws.amazon.com/cloudformation/?nc2=type_a) templates to ensure your changes are updated in the stack by executing the following command in your terminal: `cdk synth`

6.  Deploy your modified solution by executing the following command in your terminal: `cdk deploy`

7.  Locate the S3 bucket created by the CloudFormation template by opening the [AWS CloudFormation console](https://console.aws.amazon.com/cloudformation/home). Choose the **PlayerSkills-Stack**.
    <img src="/images/Blog1/Image-9.png" alt=" CloudFormation stack" >
  *Figure 9: CloudFormation stack.*

8.  On the right side of your console screen, you will see a tab labeled **Outputs**. Select the **Outputs** tab and take note of the value for the key **DataBucketName**.
    <img src="/images/Blog1/Image-10.png" alt=" CloudFormation stack outputs" >
  *Figure 10: CloudFormation stack outputs.*


9.  Open the [Amazon S3 console](https://console.aws.amazon.com/s3) and choose the bucket that has the name you noted in the previous step. Choose **Upload**.
    <img src="/images/Blog1/Image-11.png" alt=" Amazon S3 object upload" >
  *Figure 11: Amazon S3 object upload.*


10. Choose **Add files**, choose the CSV file you downloaded in the prerequisites, and choose **Upload**. This will trigger your machine learning pipeline to begin. Training and model deployment will take roughly 30 minutes to be completed.
    <img src="/images/Blog1/Image-12.png" alt=" Amazon S3 object upload – adding files" >
  *Figure 12: Amazon S3 object upload – adding files.*


### Checking pipeline completion progress

1.  Navigate back to the tab you have with the SageMaker Studio page open. Within the left side menu, choose **Pipelines**.
    <img src="/images/Blog1/Image-13.png" alt=" Amazon SageMaker Studio home page" >
  *Figure 13: Amazon SageMaker Studio home page.*

2.  Choose the **PlayerSkills-AutoMLPipeline** and choose your most recent execution.
    <img src="/images/Blog1/Image-14.png" alt=" Amazon SageMaker pipeline" >
  *Figure 14: Amazon SageMaker pipeline.*
    <img src="/images/Blog1/Image-15.png" alt=" Amazon SageMaker pipeline executions" >
  *Figure 15: Amazon SageMaker pipeline executions.*

3.  After the execution is completed, you will see a graph showing the steps of the pipeline and their results.
    <img src="/images/Blog1/Image-16.png" alt=" Amazon SageMaker pipeline execution details graph" >
  *Figure 16: Amazon SageMaker pipeline execution details graph.*

### Testing your machine learning model

1.  Once training and model deployment is completed successfully, there is an Amazon SageMaker AI endpoint that will be created. Locate the Amazon SageMaker AI endpoint by navigating to Amazon SageMaker in the AWS Management Console. On the left side menu, under the **Inference** section, choose the **Endpoints** option. The name of the endpoint in this tutorial is **PlayerSkills-Endpoint**. Note the name of your created endpoint, you will be referring to this later.
    <img src="/images/Blog1/Image-17.png" alt=" Amazon SageMaker AI endpoint" >
  *Figure 17: Amazon SageMaker AI endpoint.*

2.  To test the newly created model and endpoint, you can replace lines 23-25 in the `/player-insights/assets/examples/churn_inference.py` file with the following code:

    ```python
    response = predictor.predict(
        "10597,20312,602,205,1916,56266,76578,9725,8.0"
    ).decode("utf-8")
    ```

3.  Rename the file `churn_inference.py` to `skill_inference.py`.

4.  To run the script, in your command line, navigate to the `/player-insights/assets/examples` directory and run the script with the following commands:

    ```bash
    cd assets/examples
    python3 skill_inference.py --endpoint-name PlayerSkills-Endpoint
    ```

5.  If the script runs successfully, it will return a response that is a value between 0-1 of the endpoint's output. Changing the values entered in the previous step will change the output values.

### Cleaning up resources

You'll be using what you deployed in this blog in the second part of this series, so we won't clean this up yet. We'll show you how to clean up any resources you've deployed in the next blog.

In the second part of this series, we'll show you how to use the newly determined "Skill" value in conjunction with Amazon GameLift FlexMatch. Amazon GameLift FlexMatch will handle the logic of matchmaking players while giving you, the developer, a way to adjust which matches are created through a rules-based syntax called **FlexMatch rule sets**.

## Conclusion

We showed how to deploy a solution for AI-driven player insights on AWS, and how to build an ML model to more holistically infer a player's Skill value. Based on what makes a player skillful within your game, you can choose what in-game factors that the ML model uses to determine player skill. This results in a more precise player skill that you can use to create balanced and competitive matches.

Be certain to join us again for [Implementing AI-Powered Matchmaking with Amazon GameLift FlexMatch](https://aws.amazon.com/blogs/gametech/implementing-ai-powered-matchmaking-with-amazon-gamelift-flexmatch/), the second blog in this series. We'll show you how to use the results from this blog's model to match players using Amazon GameLift FlexMatch. You will also learn how to simulate matchmaking using the Amazon GameLift Testing Toolkit in order to test both the ML model and your matchmaking parameters.

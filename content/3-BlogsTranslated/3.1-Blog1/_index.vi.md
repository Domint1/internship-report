---
title: "Blog 1"
date: "`r Sys.Date()`"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Đào tạo các mô hình AI để ghép trận theo kỹ năng (skill-based matchmaking) bằng cách sử dụng Amazon SageMaker AI

*Tác giả: Christina Defoor và Alexander Qin - 16 tháng 8,2025*

***Chuyên mục:*** [Amazon DynamoDB](https://aws.amazon.com/blogs/gametech/category/database/amazon-dynamodb/), [Amazon GameLift](https://aws.amazon.com/blogs/gametech/category/game-development/amazon-gamelift/), [Amazon SageMaker](https://aws.amazon.com/blogs/gametech/category/artificial-intelligence/sagemaker/), [Artificial Intelligence](https://aws.amazon.com/blogs/gametech/category/artificial-intelligence/), [AWS Lambda](https://aws.amazon.com/blogs/gametech/category/compute/aws-lambda/), [Compute](https://aws.amazon.com/blogs/gametech/category/compute/), [Database](https://aws.amazon.com/blogs/gametech/category/database/), [Game Development](https://aws.amazon.com/blogs/gametech/category/game-development/), [Industries](https://aws.amazon.com/blogs/gametech/category/industries/), [Media & Entertainment](https://aws.amazon.com/blogs/gametech/category/industries/entertainment/)

Trong các trò chơi trực tuyến cạnh tranh nhiều người chơi, **skill-based matchmaking** rất quan trọng để tạo ra các trận đấu thú vị và cân bằng. Việc xác định năng lực người chơi hiện nay không dễ vì có rất nhiều chỉ số mà trò chơi ghi lại (như số lần trúng đích, trượt, hỗ trợ, thời gian chơi, cấp độ, v.v.), khiến việc chọn những yếu tố nào là biểu hiện tốt nhất của năng lực trở nên khó khăn. Thay vì tự viết thuật toán thủ công để đánh giá năng lực người chơi, các kỹ thuật máy học (machine learning, đặc biệt là supervised learning) có thể tự động nhận diện các mẫu từ các chỉ số trò chơi để đưa ra đánh giá kỹ năng chính xác hơn. Những đánh giá “ML-derived” này cho phép việc ghép trận (matchmaking) cân bằng hơn, từ đó nâng cao sự hài lòng và tương tác của người chơi.

Trong phần đầu của chuỗi blog hai phần này, chúng tôi sẽ cho bạn thấy cách dùng [Amazon SageMaker AI](https://aws.amazon.com/vi/sagemaker-ai) để nhanh chóng tạo và triển khai pipeline ML tự động. Amazon SageMaker AI cung cấp các chức năng để xây dựng, huấn luyện và triển khai các mô hình ML và mô hình nền tảng (foundation models), với hạ tầng, công cụ và quy trình được quản lý hoàn toàn. Mô hình và pipeline mà chúng ta xây sẽ tạo ra một giá trị phản ánh sát hơn và chính xác hơn năng lực của từng người chơi.

Để thực hiện nhiệm vụ này, chúng ta sẽ dựa trên [Guidance for AI-Driven Player Insights on Amazon Web Service (AWS)](https://aws.amazon.com/solutions/guidance/ai-driven-player-insights-on-aws/). Sơ đồ kiến trúc của hướng dẫn cho thấy cách các studio game có thể dùng giải pháp low-code này để nhanh chóng xây dựng, huấn luyện và triển khai các mô hình chất lượng cao dự đoán kỹ năng người chơi dựa trên dữ liệu lịch sử người chơi. Người vận hành chỉ việc upload dữ liệu lịch sử người chơi lên [Amazon Simple Storage Service (Amazon S3)](https://aws.amazon.com/s3/). Điều này sẽ kích hoạt một workflow hoàn chỉnh để trích xuất thông tin chi tiết, chọn thuật toán, điều chỉnh siêu tham số (hyperparameter), đánh giá và triển khai mô hình có hiệu suất cao nhất làm API dự đoán, tất cả được điều phối bởi [Amazon SageMaker Pipelines](https://aws.amazon.com/sagemaker-ai/pipelines/).

<img src="/images/Blog1/Image-1.png" alt=" Sơ đồ kiến trúc của Guidance for AI-Driven Player Insights trên AWS" >
*Hình 1: Sơ đồ kiến trúc của Guidance for AI-Driven Player Insights trên AWS*.


Hình 2 dưới đây cho thấy kiến trúc bạn sẽ triển khai trong bài này. Sơ đồ mô tả luồng xử lý khi một yêu cầu ghép trận được gửi lên. Yêu cầu ghép trận kích hoạt [Amazon API Gateway](https://aws.amazon.com/api-gateway/?nc2=type_a), gọi một hàm [AWS Lambda](https://aws.amazon.com/lambda/?nc2=type_a) nhận dữ liệu người chơi từ [Amazon DynamoDB](https://aws.amazon.com/dynamodb/?nc2=type_a). Dữ liệu sau đó được truyền đến endpoint Amazon SageMaker AI, nơi chạy quá trình suy luận để đưa ra giá trị kỹ năng toàn diện hơn, được Amazon [GameLift FlexMatch ](https://docs.aws.amazon.com/gamelift/latest/flexmatchguide/match-intro.html)sử dụng trong quá trình ghép trận.

<img src="/images/Blog1/Image-2.png" alt="Workflow Matchmaking sử dụng Matchmaking simulator." >
*Hình 2: Workflow Matchmaking sử dụng Matchmaking simulator.*

Sơ đồ kiến trúc tiếp theo cho thấy cách bạn sẽ kết nối FlexMatch với hàng đợi máy chủ [Amazon GameLift Servers](https://aws.amazon.com/gamelift/servers/) trong game thực tế. Điều này kích hoạt GameLift Servers để đặt hoặc khởi tạo máy chủ trò chơi cho các trận mới được tạo. (Hình 3)

<img src="/images/Blog1/Image-3.png" alt=" Workflow ghép trận như một phần backend của game" >
*Hình 3: Workflow ghép trận như một phần backend của game.*

## Hướng dẫn các bước:

### Yêu cầu (prerequisites)

Trước khi bắt đầu bạn cần có:

*   Một tài khoản AWS với quyền IAM (quyền quản trị) để tạo và sử dụng domain SageMaker AI ([IAM permissions (administrative access) to create and use a SageMaker AI Domain](https://docs.aws.amazon.com/sagemaker/latest/dg/gs-set-up.html)).
*   Một trình soạn thảo code (Code editor).
*   Để tải file dữ liệu dùng trong hướng dẫn này, chọn [PlayerStats.csv](https://github.com/aws-samples/sample-ai-powered-multiplayer-matchmaking-sagemaker-and-gamelift-flexmatch/blob/main/PlayerStats.csv) trên repo GitHub (click “Download raw file”) như hình 4 phía dưới

  <img src="/images/Blog1/Image-4.png" alt="Trang tải PlayerStats.csv" >
  *Hình 4: Trang tải PlayerStats.csv*

*   (Tùy chọn) Nếu bạn muốn thay thế các file code, chúng tôi sẽ sửa đổi trong suốt hướng dẫn này thay vì sửa đổi từng dòng mã, hãy chọn các liên kết sau để tải xuống từng tệp:
    *   [workflow.py](https://github.com/aws-samples/sample-ai-powered-multiplayer-matchmaking-sagemaker-and-gamelift-flexmatch/blob/main/workflow.py)
    *   [evaluation.py](https://github.com/aws-samples/sample-ai-powered-multiplayer-matchmaking-sagemaker-and-gamelift-flexmatch/blob/main/evaluation.py)
    *   [skill\_inference.py](https://github.com/aws-samples/sample-ai-powered-multiplayer-matchmaking-sagemaker-and-gamelift-flexmatch/blob/main/skill_inference.py)
*   Có thể xem các yêu cầu trong [Guidance for AI-driven player insights on AWS](https://github.com/aws-solutions-library-samples/guidance-for-ai-driven-player-insights-on-aws) GitHub repository.

### Thiết lập SageMaker AI domain

1.  Mở [SageMaker AI console](https://console.aws.amazon.com/sagemaker/).
2.  Trong menu bên trái, chọn **Domains**.
3.  Chọn **Create domain**.
    <img src="/images/Blog1/Image-5.png" alt=" Amazon SageMaker domain dashboard" >
  *Hình 5:  Amazon SageMaker domain dashboard.*
4.  Chọn tùy chọn **Quick Setup** và nhấn **Set up**.
    <img src="/images/Blog1/Image-6.png" alt="Tạo Amazon SageMaker domain" >
  *Hình 6: Tạo Amazon SageMaker domain*


### Truy cập Amazon SageMaker Studio dashboard

1.  Chọn domain đã tạo ở phần trước
    <img src="/images/Blog1/Image-7.png" alt="Chọn Amazon SageMaker domain" >
  *Hình 7: Chọn Amazon SageMaker domain.*

2.  Ngay dưới phần **User profiles** chọn **Launch button** nằm cạnh profile bạn muốn sử. chọn **SageMaker Studio** sẽ được mở ra ở tab mới. Giữ tab này mở, chúng ta sẽ quay lại sau.
    <img src="/images/Blog1/Image-8.png" alt="Thông tin các domain user." >
  *Hình 8: Thông tin các domain user.*


### Triển khai giải pháp AI-driven Player Insights

Làm theo **Deployment Steps** được cung cấp trong **AWS AI-Driven Player Insights repository** (được liên kết trong phần *Prerequisites* của blog này). Nếu bạn không có quyền truy cập môi trường phát triển AWS Cloud9, bạn sẽ cần triển khai giải pháp trên thiết bị cá nhân của mình. Thực hiện **Deployment Steps** đến **Step 5** trong hướng dẫn.

### Hiểu về AI-driven Player Insights trên AWS

Truyền thống, việc phát triển các mô hình machine learning hiệu quả đòi hỏi phải có kinh nghiệm trong lĩnh vực khoa học dữ liệu. Với giải pháp này, bạn sẽ tận dụng tính năng [Amazon SageMaker Autopilot](https://docs.aws.amazon.com/sagemaker/latest/dg/use-auto-ml.html). Amazon SageMaker Autopilot sẽ tự động hóa toàn bộ quy trình xây dựng, huấn luyện, tinh chỉnh và triển khai các mô hình machine learning. Giải pháp này cung cấp một pipeline machine learning được định nghĩa sẵn, sẽ tự động kích hoạt toàn bộ quy trình và triển khai mô hình ngay từ thời điểm dữ liệu của bạn được tải lên **S3 bucket**.

Giải pháp này yêu cầu một bộ dữ liệu thống kê người chơi (player statistics dataset). Bộ dữ liệu cần bao gồm các chỉ số hiệu suất liên quan đến trò chơi của bạn và điểm kỹ năng hiện tại (**skill rating**) được sử dụng cho hệ thống matchmaking. Trong hướng dẫn này, chúng ta sẽ sử dụng file **PlayerStats.csv**.

### Chuẩn bị pipeline machine learning cho linear regression

Pipeline machine learning trong giải pháp AI-driven player insights, theo mặc định, được cấu hình cho bài toán phân loại (classification problem). Vì chúng ta muốn mô hình xuất ra dữ liệu dạng số đại diện cho “Skill” của người chơi (bài toán hồi quy tuyến tính), nên cần chỉnh sửa pipeline.

1.  Trong code editor bạn ưa thích, mở file ***/player-insights/constants.py*** và thay thế toàn bộ nội dung bằng đoạn mã sau (Đảm bảo thay các giá trị của `SM_DOMAIN_ID` và `REGION` bằng giá trị cụ thể của bạn) :

    ```python
    WORKLOAD_NAME = “PlayerSkills”
    REGION = "[YOUR REGION]"
    SM_DOMAIN_ID = "[YOUR SAGEMAKER AI DOMAIN ID]"
    DATA_FILE = "PlayerStats.csv"
    TARGET_ATTRIBUTE = "playerSkill"
    PERFORMANCE_THRESHOLD = 0.00
    ENDPOINT_TYPE = "SERVERLESS"
    ```

2.  Vì chúng ta sẽ huấn luyện một mô hình hồi quy tuyến tính, nên cần thay đổi phương pháp đánh giá sang **Mean Squared Error (MSE)**. Hãy thay thế hàm main trong file ***/player-insights/evaluation.py*** bằng đoạn code sau:

    ```python
    if __name__ == "__main__":
        logger.debug("Starting Evaluation ...")
        logger.info("Reading Test Predictions")
        y_pred_path = "/opt/ml/processing/input/predictions/x_test.csv.out"
        y_pred = pd.read_csv(y_pred_path, header=None).squeeze()  # Assuming one column
        logger.info("Reading Test Labels")
        y_true_path = "/opt/ml/processing/input/true_labels/y_test.csv"
        y_true = pd.read_csv(y_true_path, header=None).squeeze()  # Assuming one column
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

3.  Mở file ***/player-insights/workflow.py***. Chỉnh sửa bước "failure step" được định nghĩa ở dòng 227 để sử dụng MSE thay vì F1 score:

    ```python
       failure_step = FailStep(
            name="ModelEvaluationFailure",
            error_message=Join(
                on=" ",
                values=["Pipeline execution failure: MSE is less than the specified Evaluation Threshold"] #CHANGED: Updated to reflect evaluating MSE rather than F1 score
            )
        )
    ```

4.  Chỉnh sửa “conditional step” ở dòng 259 trong cùng file này để sử dụng MSE làm chỉ số đánh giá thay vì weighted F1 score:

    ```python
    conditional_step = ConditionStep(
            name="ModelQualityCondition",
            conditions=[
                ConditionGreaterThanOrEqualTo(
                    left=JsonGet(
                        step_name=evaluation_step.name,
                        property_file=evaluation_report,
                        json_path="regression_metrics.mean_squared_error.value"  #CHANGED: changed line to evaluate MSE instead of F1 score
                    ),
                    right=metric_threshold
                )
            ],
            if_steps=[step_register_model, deployment_step],
            else_steps=[failure_step]
        )
    ```

5.  Xác minh rằng Cloud Development Kit (CDK) triển khai đúng các template [AWS CloudFormation](https://aws.amazon.com/cloudformation/?nc2=type_a), đảm bảo rằng những thay đổi của bạn đã được cập nhật vào stack, bằng cách chạy lệnh sau trong terminal: `cdk synth`

6.  Triển khai (deploy) giải pháp đã được chỉnh sửa của bạn bằng cách chạy lệnh: `cdk deploy`

7.  Tìm S3 bucket được tạo bởi CloudFormation template bằng cách mở [**AWS CloudFormation Console**](https://console.aws.amazon.com/cloudformation/home). Chọn **PlayerSkills-Stack**.
    <img src="/images/Blog1/Image-9.png" alt="CloudFormation stack" >
  *Hình 9: CloudFormation stack.*

8.  Ở phía bên phải màn hình console, bạn sẽ thấy một tab có nhãn **Outputs**. Chọn tab **Outputs** và ghi lại giá trị của key **DataBucketName**.
    <img src="/images/Blog1/Image-10.png" alt="CloudFormation stack" >
  *Hình 10: CloudFormation stack.*

9.  Mở [Amazon S3 Console](https://console.aws.amazon.com/s3), chọn bucket có tên trùng với giá trị bạn vừa ghi ở bước trước, sau đó chọn **Upload**.
    <img src="/images/Blog1/Image-11.png" alt="Amazon S3 object upload" >
  *Hình 11: Amazon S3 object upload.*


10. Chọn **Add files**, sau đó chọn file **csv** bạn đã tải xuống trong phần prerequisites, và nhấn **Upload.** Việc này sẽ kích hoạt pipeline machine learning của bạn để bắt đầu. Quá trình huấn luyện và triển khai mô hình sẽ mất khoảng 30 phút để hoàn thành.
    <img src="/images/Blog1/Image-12.png" alt="Amazon S3 object upload – thêm files" >
  *Hình 12: Amazon S3 object upload – thêm files.*


### Kiểm tra tiến trình hoàn thành của pipeline

1.  Quay lại tab **SageMaker Studio**, chọn menu **Pipelines**.
    <img src="/images/Blog1/Image-13.png" alt="Trang chủ Amazon SageMaker Studio" >
  *Hình 13: Trang chủ Amazon SageMaker Studio.*

2.  Chọn pipeline tên **PlayerSkills-AutoMLPipeline** và chọn execution mới nhất.
    <img src="/images/Blog1/Image-14.png" alt="Amazon SageMaker pipeline" >
  *Hình 14: Amazon SageMaker pipeline.*
    <img src="/images/Blog1/Image-15.png" alt="Amazon SageMaker pipeline executions" >
  *Hình 15: Amazon SageMaker pipeline executions.*

3.  Khi execution hoàn tất, bạn sẽ thấy đồ thị các bước trong pipeline và kết quả từng bước.
    <img src="/images/Blog1/Image-16.png" alt="Biểu đồ Amazon SageMaker pipeline execution chi tiết" >
  *Hình 16: Biểu đồ Amazon SageMaker pipeline execution chi tiết.*

### Kiểm thử mô hình machine learning

1.  Khi quá trình huấn luyện và triển khai mô hình hoàn tất thành công, một Amazon SageMaker AI endpoint sẽ được tạo ra. Hãy tìm endpoint này bằng cách truy cập Amazon SageMaker trong AWS Management Console. Trong menu bên trái, dưới phần **Inference**, chọn **Endpoints**. Tên của endpoint trong hướng dẫn này là **PlayerSkills-Endpoint**. Ghi lại tên endpoint mà bạn đã tạo.
    <img src="/images/Blog1/Image-17.png" alt="Amazon SageMaker AI endpoint" >
  *Hình 17: Amazon SageMaker AI endpoint.*

2.  Để kiểm thử mô hình và endpoint vừa tạo, hãy thay thế dòng 23–25 trong file ***/player-insights/assets/examples/churn\_inference.py*** bằng đoạn code sau:

    ```python
    response = predictor.predict(
            "10597,20312,602,205,1916,56266,76578,9725,8.0"
        ).decode("utf-8")
    ```

3.  Đổi tên file **churn\_inference.py** thành **skill\_inference.py**.

4.  Để chạy script, trong command line, điều hướng đến thư mục ***/player-insights/assets/examples*** và chạy script bằng các lệnh sau:

    ```bash
    cd assets/examples
    python3 skill_inference.py --endpoint-name PlayerSkills-Endpoint
    ```

5.  Nếu script chạy thành công, nó sẽ trả về một giá trị trong khoảng 0–1 là đầu ra của endpoint. Thay đổi các giá trị được nhập ở Bước 2 sẽ làm thay đổi giá trị đầu ra tương ứng.

### Dọn dẹp tài nguyên

Bạn sẽ tiếp tục sử dụng những tài nguyên đã triển khai trong blog này ở phần hai của loạt bài, vì vậy chúng ta chưa cần dọn dẹp ngay bây giờ. Chúng tôi sẽ hướng dẫn bạn cách xóa và dọn dẹp toàn bộ tài nguyên đã triển khai trong bài viết kế tiếp.

Trong phần hai, chúng tôi sẽ hướng dẫn bạn sử dụng giá trị “Skill” vừa được xác định, kết hợp với Amazon **GameLift FlexMatch**. Amazon GameLift FlexMatch sẽ xử lý logic ghép trận người chơi (matchmaking), đồng thời cho phép bạn, với tư cách là nhà phát triển, tùy chỉnh cách ghép trận thông qua ngữ pháp quy tắc (rules-based syntax) được gọi là **FlexMatch rule sets**.

## Kết luận

Chúng tôi đã trình bày cách triển khai một giải pháp AI-driven player insights trên AWS, và xây dựng một mô hình machine learning nhằm suy luận toàn diện hơn về “Skill” của người chơi. Điều này giúp tạo ra chỉ số kỹ năng chính xác hơn, giúp bạn xây dựng các trận đấu cân bằng và cạnh tranh hơn.

Hãy tiếp tục theo dõi bài tiếp theo [Implementing AI-Powered Matchmaking with Amazon GameLift FlexMatch](https://aws.amazon.com/blogs/gametech/implementing-ai-powered-matchmaking-with-amazon-gamelift-flexmatch/), bài thứ hai trong loạt blog này. Chúng tôi sẽ hướng dẫn bạn sử dụng kết quả từ mô hình trong blog này để ghép trận người chơi thông qua Amazon GameLift FlexMatch, đồng thời mô phỏng quá trình ghép trận bằng Amazon GameLift Testing Toolkit để kiểm thử mô hình machine learning và các tham số ghép trận của bạn.

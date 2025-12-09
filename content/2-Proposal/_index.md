---
title: "Proposal"
date: "2025-12-08"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
[Document version of the proposal](https://docs.google.com/document/d/1-WtW93MMZPBzKaP8AzaQ2Zy3GQCm-i-ZW2sAS80ZsUg/edit?usp=sharing)

# Rafilm: AI-Powered Movie Logging & Recommendation Platform

## A Serverless AWS Solution for Intelligent Movie Discovery

### 1. Executive Summary

Rafilm is a Letterboxd-inspired movie logging and recommendation platform designed to help general users track their watched films, share reviews, and discover new favorites through AI-powered recommendations. Built as part of the AWS First Cloud Journey (FCJ) internship, Rafilm integrates Amazon Personalize and Bedrock to deliver tailored movie suggestions and conversational recommendations via a chatbot interface.

The platform runs fully on AWS Serverless architecture, featuring Amplify-hosted Next.js frontend, Lambda-based backend services connected through API Gateway, and DynamoDB for scalable user and movie data storage. TMDb provides external movie data integration, while Amazon Cognito manages user authentication. Rafilm aims to demonstrate a scalable, intelligent, and cost-efficient architecture capable of supporting multi-user access and interactive experiences.

### 2. Problem Statement

#### What’s the Problem?

While existing movie platforms like Letterboxd and IMDb offer robust logging and social features, they lack **personalized recommendation systems** and **interactive discovery experiences**. Users often rely on external sources or generic trending lists to find what to watch next, leading to irrelevant or repetitive suggestions.

#### The Solution

Rafilm integrates a **custom recommendation pipeline** powered by **Amazon Personalize**, combined with a **Bedrock LLM chatbot** that interprets user preferences and generates conversational movie recommendations. Users can log movies, write reviews, and receive curated suggestions—all within one seamless experience. Unlike Letterboxd, Rafilm focuses on data-driven personalization and AI-assisted interaction rather than pure social networking.

#### Benefits and Return on Investment

By leveraging AWS Serverless services, Rafilm achieves near-zero maintenance cost, pay-per-use scalability, and real-time AI-driven personalization. For the FCJ internship, the project serves as both a **technical showcase** and a **learning artifact** for integrating AI services in serverless architectures. Projected cost remains under $1/month during testing, with AWS Free Tier coverage for most usage.

### 3. Solution Architecture

Rafilm employs a modular serverless architecture using AWS services for scalability, integration, and cost optimization.

![architecture-diagram](/images/2-Proposal/solution-architect-rafilm.jpg)

#### AWS Services Used

- **AWS Amplify**: Hosts the Next.js frontend for movie browsing, logging, and chatbot interaction.
- **Amazon Cognito**: Handles user registration, login, and session management.
- **Amazon API Gateway**: Routes client requests to backend Lambda functions.
- **AWS Lambda**: Executes serverless business logic (e.g., CRUD for reviews, fetching TMDb data, triggering recommendations).
- **Amazon DynamoDB**: Stores user logs, movie interactions, and preferences.
- **Amazon Personalize**: Trains and serves personalized recommendation models.
- **Amazon Bedrock**: Provides conversational chatbot functionality for recommendation dialogue.
- **Amazon S3**: Stores static assets and backups for logs and model outputs.

#### Component Design

- **Frontend (Next.js)**: User-friendly interface for movie discovery, logging, and chat-based recommendations.
- **Backend (Lambda + API Gateway)**: Stateless logic layer handling user operations, movie fetching, and recommendation retrieval.
- **Data Layer (DynamoDB + S3)**: Stores structured user interactions and movie metadata for model training.
- **AI Layer (Personalize + Bedrock)**: Personalize analyzes historical user interactions; Bedrock chatbot provides natural language access to personalized results.
- **Authentication (Cognito)**: Securely manages multi-user access.

#### Architecture Overview

1. Users log in via Cognito and interact with the Next.js interface.
2. Actions such as logging or rating trigger API Gateway → Lambda → DynamoDB workflows.
3. The Bedrock chatbot accesses Personalize results to generate conversational movie suggestions.
4. Amplify hosts the frontend for seamless deployment and scalability.

### 4. Technical Implementation

#### Implementation Phases

1. **Architecture Design (Month 1):** Research AWS serverless and AI integration patterns; finalize architecture diagrams.
2. **Prototype Integration (Month 2):** Implement Amplify hosting, Cognito authentication, and Lambda-based backend APIs.
3. **Recommendation System (Month 3):** Connect Personalize and Bedrock for end-to-end AI recommendation and chatbot response.
4. **Testing & Deployment:** Conduct functional testing, optimize costs, and deploy production-ready version on Amplify.

#### Technical Requirements

- **Frontend:** Next.js + React hosted via AWS Amplify, using TMDb API for movie data.
- **Backend:** AWS Lambda (Node.js runtime) connected through API Gateway.
- **Database:** Amazon DynamoDB for scalable user and review data.
- **AI Components:** Amazon Personalize (user-item recommendations) and Bedrock (chatbot dialogue).
- **Authentication:** Amazon Cognito for secure, multi-user access.
- **Automation:** AWS SDK & CloudFormation for provisioning; AWS SAM for deployment workflows.

### 5. Timeline & Milestones

| Phase       | Duration                 | Key Deliverables                                            |
| ----------- | ------------------------ | ----------------------------------------------------------- |
| Month 1     | Research & Architecture  | AWS design finalizing                                       |
| Month 2     | Core Development         | Amplify hosting, Cognito setup, Lambda API, DynamoDB schema |
| Month 3     | AI Integration & Testing | Personalize training, Bedrock chatbot, system deployment    |
| Post-Launch | Continuous Improvement   | Cost optimization, new features, UX refinement              |

### 6. Budget Estimation

**Estimated Monthly Cost:** $40.09 USD ($481.08 for 12 months)

The cost estimate of this project is projected in this link: [https://calculator.aws/#/estimate?id=dab7fb57dabfb76041cdba98ac2bac7ba9630046](https://calculator.aws/#/estimate?id=dab7fb57dabfb76041cdba98ac2bac7ba9630046)

The monthly cost estimate of **$40.09 USD** is calculated based on the following specific usage assumptions derived from the AWS Pricing Calculator:

- **Standard Usage:** Assumes **10,000 requests per month** for both **AWS Lambda** and **Amazon API Gateway**.
- **User Base:** Assumes a maximum of **10,000 Monthly Active Users (MAU)** for **Amazon Cognito**.
- **AI/ML Usage (Primary Cost Driver):**
    - **Amazon Bedrock** is estimated for continuous operation (24 hours per day) with an average of **1 request per minute** and **100 input/output tokens per request**.
    - **Amazon Personalize** includes **1 GB of data ingested** and **15 training hours per month**.
- **Security Overhead:** Assumes use of **1 AWS WAF Web ACL** with 4 Rules and 3 Managed Rule Groups.
- **Data Storage:** Assumes **0.5 GB** of data storage in **Amazon DynamoDB**.


### 7. Risk Assessment

| Risk                           | Probability | Impact | Mitigation                                    |
| ------------------------------ | ----------- | ------ | --------------------------------------------- |
| API rate limits from TMDb      | Medium      | Medium | Cache requests via Lambda                     |
| Model training cost escalation | Low         | Medium | Use limited training dataset for testing      |
| Chatbot latency                | Medium      | Low    | Optimize Bedrock model type and response size |
| Authentication or token expiry | Medium      | Low    | Use short-lived JWTs and refresh tokens       |

### 8. Expected Outcomes

#### Technical Improvements

- Demonstrates **serverless integration** of AI/ML and LLM services in real-world use.
- Establishes a reusable AWS architecture for recommendation-based apps.

#### Long-Term Value

- Provides a foundation for future expansion into a **social movie discovery network**.
- Serves as an **AWS FCJ internship showcase project** highlighting scalability, personalization, and conversational AI.
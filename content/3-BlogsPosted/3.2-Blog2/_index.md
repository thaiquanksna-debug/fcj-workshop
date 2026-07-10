---
title: "Designing a Vietnam AI Travel Assistant MVP with Amazon Bedrock, AWS Amplify, and Amazon Location Service"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 3.2 Blog 2. </b> "
---


## Introduction

When speaking about AI applications for travel, many beginners immediately think of a simple chatbot setup: a user inputs a question, the backend calls a Large Language Model (LLM), and the model generates a response. This approach is straightforward to demo since it only requires a chat interface, a backend API, and a model to output responses.

However, when moving closer to a real-world product, a travel chatbot should not rely solely on the general knowledge of an LLM. Users often ask highly specific, context-dependent questions such as:
- "Where should I go in Hoi An if I only have 1 day?"
- "Which restaurants are suitable for someone who cannot eat seafood?"
- "Can you suggest a 3-day Da Nang itinerary for a family traveling with toddlers?"
- "Which places should I avoid visiting during the rainy season?"

If the system answers purely based on the model's pre-trained knowledge, the response might sound confident but could lack validation, miss local context, or conflict with the verified data an enterprise actually wants to provide.

Therefore, for this MVP, we are not designing a completely unconstrained chatbot. Instead, the architecture strictly separates the interface layer, orchestration layer, AI layer, knowledge base layer, location layer, and observability layer. The goal is to establish a robust baseline for building a structured AI application rather than just displaying raw model responses.

---

## Defining the MVP Scope

The core user interaction can be defined as follows: a user enters their travel requirements in natural language, for example: 

> "I have 3 days in Da Nang, I love beaches and local food, my budget is moderate, and I don't want to travel too much between locations."

The system will analyze the input, retrieve relevant information from the knowledge base, call Amazon Bedrock to generate a tailored itinerary, invoke Amazon Location Service to fetch coordinates or place data, and finally return a structured itinerary to the user.

A clean output structure should consist of:
- A detailed day-by-day itinerary.
- Specific recommendations for dining and sightseeing.
- Transportation notes and alternative options.
- Contextual follow-up questions (e.g., *"Would you prefer your itinerary to focus more on relaxation, food exploration, or cultural experiences?"*).

> **Note:** This architecture should not be considered fully "production-ready." A true production system requires comprehensive inputs checking, intensive response evaluation, cost control, strict rate limiting, CI/CD pipelines, Infrastructure as Code (IaC), and more. However, this MVP is perfect for junior cloud engineers or AWS learners practicing production-oriented GenAI design patterns.

---

# Architecture Overview and Core Components

The system's data flow routes traffic across several decoupled layers:
- **Frontend Layer:** Built and hosted using **AWS Amplify** for web or mobile access.
- **Backend Orchestration:** Managed seamlessly by **AWS Lambda** via an API gateway.
- **Generative AI & RAG Layer:** Powered by **Amazon Bedrock** and **Amazon Bedrock Knowledge Bases** for accurate information retrieval.
- **Location Layer:** Handled by **Amazon Location Service** for map visualization and geolocation.
- **Storage & Observability:** **Amazon DynamoDB** tracks user sessions, while **Amazon CloudWatch** handles monitoring.

---

## Frontend and Backend Orchestration

### AWS Amplify
Amplify acts as the ideal frontend foundation because it accelerates web and mobile application deployment. It natively handles common full-stack development needs such as hosting, authentication, data management, and backend integrations with a highly developer-friendly experience.

### AWS Lambda
Lambda functions as the lightweight backend orchestration mechanism. It receives API requests, validates inputs, queries the Knowledge Base to retrieve context, calls Amazon Bedrock to generate responses, communicates with Amazon Location Service for coordinates, and formats the final payload. This serverless model removes server management overhead, making it cheap and flexible for iterative testing.

---

## Generative AI and RAG Strategy

### Amazon Bedrock
Bedrock represents the core Generative AI layer. The backend forwards prompts, contextual data, and user preferences to foundation models to synthesize itineraries. Here, Bedrock is utilized to transform unconstrained natural language into a highly structured schema (Overview, Itinerary by day, Places, Food suggestions, Transport notes, Safety notes).

### Retrieval-Augmented Generation (RAG)
To mitigate model hallucination, the system relies on RAG. When a request comes in, the backend locates relevant information chunks within **Amazon Bedrock Knowledge Bases**, appends them into the LLM prompt context, and instructs the model to generate a response constrained by that specific context.

The raw verified data is stored in **Amazon S3** and structured into:
- Standardized destination details across Vietnamese cities.
- Sample itinerary templates and FAQs for international tourists.
- Regional cultural guidelines and seasonal weather advisories.

---

## Mapping and Data Storage

### Amazon Location Service
This service powers map visuals and location data integration. In the context of Vietnamese tourism, geography is a major constraint. Itineraries need optimization based on physical distance and localized travel times (e.g., clustering My Khe Beach and Son Trara Peninsula together into a single afternoon instead of suggesting distant locations in a single session).

### Amazon DynamoDB
DynamoDB stores structured operational data such as user profiles, session IDs, conversation logs, generated itineraries, and user feedback. At the MVP stage, a single table design using a partition key based on `userId` or `sessionId` along with item types like `PROFILE`, `CONVERSATION`, `ITINERARY`, and `FEEDBACK` is sufficient.

---

# Step-by-Step Request Flow

The sequence of processing a single user inquiry follows these clear operational steps:

1. **Submit Request:** The user inputs their preferences on the frontend app (e.g., *"I want a 2-day trip to Hoi An focusing on photography and local food..."*). The frontend submits this to the backend API.
2. **Validate & Sanitize:** AWS Lambda sanitizes the inputs (validating location, duration, budget). If crucial parameters are missing, the backend can instruct the model to ask clarifying questions instead of generating an inaccurate plan.
3. **Retrieve Context (RAG):** The backend invokes Amazon Bedrock Knowledge Bases to fetch verified Hoi An destination logs from S3, embedding these data fragments into the prompt context.
4. **Generate Response:** The prompt is sent to Amazon Bedrock to generate a structured itinerary separated cleanly into daily routines (Morning, Afternoon, Evening) along with rationale.
5. **Geolocate Places:** If the generated text contains specific venues, the backend queries Amazon Location Service to append exact geographical coordinates for interactive rendering on the frontend map.
6. **Save & Stream:** The system saves the conversation logs to DynamoDB, streams operational logs to CloudWatch, and serves the finalized response back to the frontend.

---

# Production-Oriented Operations

## Guardrails (Amazon Bedrock Guardrails)
AI applications are inherently vulnerable to malicious inputs, prompt injections, and off-topic queries. **Amazon Bedrock Guardrails** provides a vital security layer checking user inputs and model outputs against established safety policies:
- Content filters to detect harmful themes.
- Denied topics to block sensitive political, medical, or legal claims.
- Word filters and sensitive information filters to mask PII data.
- Contextual grounding checks to prevent hallucinations.

## Cost Control
Foundation models charge per input and output token. To keep operational costs controlled during the MVP phase:
- Impose strict character limits on frontend user inputs.
- Control maximum output sizes using the `max_tokens` parameter.
- Cache responses for highly repeated generic queries.
- Never allow the frontend to interact with the LLM directly; routing must go through the serverless backend to enforce robust rate-limiting.

## Observability & Feedback Loop
- **CloudWatch:** Monitors system health, measuring request success/failure ratios, function latency, Bedrock API failures, and resource exhaustion metrics.
- **Feedback Loop:** Empowers users to rate itineraries via simple thumbs-up/down choices. This telemetry logs directly into DynamoDB to drive ongoing prompt modifications and knowledge base updates.

---

# Architecture Trade-offs and Best Practices

When building Generative AI platforms, engineers frequently fall into two distinct traps:

1. **Treating the prompt as the entire system:** Prompts are just one piece of the puzzle. Reliable architectures require rigorous data ingestion, strict boundaries, and elegant fallbacks. If your knowledge base lacks relevant data, the system should gracefully respond *"I do not have enough verified data for this region"* rather than fabricating details.
2. **Scope Creep:** Overloading an MVP with hotel booking systems, flight scanners, billing integrations, or automated multi-currency calculators will quickly destabilize the core product.

### Effective Scoping Strategy for the MVP:
- **Geographic Limits:** Limit your data scope to three popular travel hubs first: Da Nang, Hoi An, and Ha Noi. Ensure this baseline data is perfectly clean before expanding.
- **Language Localization:** Launch initially with English and Vietnamese. True localization goes beyond simple translation; it requires tailored knowledge bases reflecting the cultural habits and travel behaviors of different demographics.
- **Security:** Utilize **Amazon Cognito** or **Amplify Auth** for simple user identity management. Avoid collecting non-essential sensitive information like passport numbers or credit card credentials until your data compliance structures are fully implemented.

---

# Future Roadmap and Scale

Once your basic architecture runs stably via manual AWS Console deployments, consider migrating onto the following automated workflows:

- **Infrastructure as Code (IaC):** Standardize your entire AWS stack (S3, Lambda, Bedrock, DynamoDB) using **Terraform**, **AWS CDK**, or **CloudFormation** to make your environments reproducible and peer-reviewable.
- **CI/CD Integration:** Set up automated delivery pipelines (GitHub Actions, GitLab CI/CD, or AWS CodePipeline) split across three functional paths:
  - Frontend delivery to automatically deploy changes to AWS Amplify.
  - Data ingestion pipelines to sync raw S3 updates directly into Bedrock Knowledge Bases.
  - Backend application pipelines to run automated unit tests and update Lambda code.
- **Advanced Enhancements:** Introduce an Admin Portal for content managers, export plans to downloadable PDFs, and pull real-time weather and event feeds to dynamically alter suggestions.

---

# Conclusion

Building a "Vietnam AI Travel Assistant" serves as an exceptional practical project for AWS engineers because it unifies foundational disciplines: frontend hosting, serverless execution, stateful data stores, geolocation, monitoring, and cutting-edge GenAI implementations.

Moving past a basic "black box" chat demo teaches engineers the realities of production development—where data must be controlled, costs managed, applications monitored, and system boundaries strictly maintained.

<h2 style="text-align:center;">Proposed Architecture Diagram</h2>

![Photo](/images/3-Blog/diagram2.jpg)

**Reference Article Link** : https://www.facebook.com/groups/awsstudygroupfcj/permalink/2201739207257706
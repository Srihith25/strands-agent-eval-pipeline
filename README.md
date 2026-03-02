# Production-Grade AI Agent with End-to-End Observability and Automated Evaluation

A multi-tool restaurant assistant agent built with Strands SDK on Amazon Bedrock Agentcore, fully instrumented with OpenTelemetry tracing, LLM-as-a-Judge evaluations, and production monitoring via Arize AX.

---

## Overview

This project goes beyond building a demo agent. It implements the full production lifecycle for an AI agent: development, instrumentation, evaluation, prompt optimization, and monitoring.

The agent handles restaurant bookings, information retrieval, and customer queries — but the focus is on the **infrastructure around it**: how to trace nondeterministic workflows, catch silent failures with automated evaluations, and iterate on prompts using regression testing.

### Key Insight

> Nondeterministic systems fail in ways you cannot predict by reading code. Without tracing and automated evaluations, failures ship silently.

For example, when asked "find me a restaurant on the moon," the agent initially scored **0** on graceful handling — it confidently offered to search for lunar restaurants. Automated evaluation caught this, and prompt iteration fixed it to **100% edge-case correctness**.

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                Local Environment (Strands SDK)       │
│                                                      │
│   ┌──────────────────────┐    ┌──────────────────┐  │
│   │  Restaurant Assistant │───▶│ Tools            │  │
│   │  Agent               │    │ • create_booking  │  │
│   │                      │    │ • get_booking     │  │
│   │                      │    │ • delete_booking  │  │
│   │                      │    │ • current_time    │  │
│   │                      │    │ • retrieve (KB)   │  │
│   └──────┬───────────────┘    └────────┬─────────┘  │
│          │                             │             │
│          │  Traces & Logs              │ Updates     │
│          ▼                             ▼             │
│   ┌──────────┐              ┌──────────────────┐    │
│   │ Arize AX │              │ Amazon DynamoDB  │    │
│   └──────────┘              └──────────────────┘    │
└─────────────────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────┐
│                    AWS Cloud                         │
│                                                      │
│   ┌────────────────┐    ┌─────────────────────────┐ │
│   │ Amazon Bedrock  │    │ Bedrock Knowledge Base  │ │
│   │ LLMs           │    │ → OpenSearch Serverless  │ │
│   │                │    │ → S3 Documents           │ │
│   └────────────────┘    └─────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Layer              | Technology                                      |
|--------------------|--------------------------------------------------|
| Agent Framework    | Strands SDK                                      |
| LLM Backend        | Amazon Bedrock (Claude)                          |
| Agent Runtime      | Amazon Bedrock Agentcore                         |
| Data Store         | Amazon DynamoDB                                  |
| Knowledge Base     | Bedrock Knowledge Base + OpenSearch Serverless + S3 |
| Tracing            | OpenTelemetry + OpenInference                    |
| Observability      | Arize AX                                         |
| Evaluation         | LLM-as-a-Judge (3 evaluators)                    |
| Language           | Python 3.11                                      |

---



---

## Notebooks

### Notebook 1: Build & Deploy

Covers the complete agent construction pipeline:

- Environment setup and AWS resource provisioning (DynamoDB, S3, OpenSearch)
- Five tool definitions: `create_booking`, `get_booking_details`, `delete_booking`, `current_time`, `retrieve`
- Bedrock Knowledge Base integration for restaurant information retrieval
- Strands SDK agent assembly with system prompt
- Test scenarios including adversarial edge cases
- Packaging and deployment to Amazon Bedrock Agentcore

### Notebook 2: Observability, Evaluation & Monitoring

Covers the full instrumentation and quality pipeline:

- OpenTelemetry tracing via `arize-phoenix-otel` and `openinference-instrumentation-strands`
- Trace visualization: Trace Trees, Agent Graphs, Timeline views
- Three LLM-as-a-Judge evaluators:
  - **graceful_handling** — Does the agent handle impossible requests correctly?
  - **response_relevance** — Is the response on-topic?
  - **tool_usage_correctness** — Did the agent use the right tools?
- 7-case regression dataset across 4 categories
- Prompt iteration workflow: identified and fixed graceful-handling failure (0% → 100%)
- Production monitoring: dashboards (latency, cost, eval scores, errors) and 5 alert rules

---

## Metrics from the Project

| Metric                        | Value                          |
|-------------------------------|--------------------------------|
| Total trace latency           | 5.69s                          |
| LLM call latency (per span)  | 2.75s, 2.44s                   |
| Tool retrieval latency        | 0.43s                          |
| Token count (span 1)         | 2,618                          |
| Token count (span 2)         | 3,150                          |
| Total tokens (trace)         | 5,768                          |
| Graceful handling (before)    | Score: 0 (not_graceful)        |
| Graceful handling (after)     | Score: 1 (graceful)            |
| Edge-case correctness         | 0% → 100%                     |
| Evaluators                    | 3                              |
| Regression test cases         | 7                              |

---

## Setup

### Prerequisites

- Python 3.11+
- AWS account with Bedrock, DynamoDB, OpenSearch Serverless, and S3 access
- Arize AX account (free tier available)

### Installation

```bash
# Clone the repository
git clone https://github.com/<your-username>/agent-observability-pipeline.git
cd agent-observability-pipeline

# Install dependencies
pip install strands-agents strands-agents-tools boto3 python-dotenv \
    arize-phoenix-otel opentelemetry-api opentelemetry-sdk \
    opentelemetry-exporter-otlp openinference-instrumentation-strands \
    arize-ax pandas
```

### Environment Variables

Create a `.env` file in the project root:

```env
AWS_REGION=us-east-1
DYNAMODB_TABLE_NAME=restaurant-bookings
KNOWLEDGE_BASE_ID=your-kb-id
S3_BUCKET_NAME=your-restaurant-docs-bucket
ARIZE_API_KEY=your-arize-api-key
ARIZE_SPACE_ID=your-space-id
ARIZE_PROJECT_NAME=restaurant-assistant-agent
AGENTCORE_IAM_ROLE_ARN=arn:aws:iam::role/AgentcoreExecutionRole
```

### Running

1. Open `01_build_and_deploy_strands_agent.ipynb` and run cells sequentially to build and test the agent.
2. Open `02_observability_evaluation_monitoring.ipynb` to add tracing, run evaluations, optimize prompts, and configure monitoring.


---

## Acknowledgments

Built during the **"Observing and Evaluating Agentic Workflows"** workshop hosted by [Arize AI](https://arize.com) and [AWS](https://aws.amazon.com). Special thanks to Richard Young for his detailed walkthrough of agent evaluation in production and testing environments.

---

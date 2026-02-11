# AI Meeting Intelligence Platform (Video-Audio → Decisions → Actions)

**Concept:**

Upload meeting audio/video → system extracts decisions, action items, risks, and auto-creates tasks.

## What it does

- Upload meeting recording
- Pipeline:
    - transcription
    - topic segmentation
    - decision detection per QA/R&D/ PR
    - action item extraction per QA/R&D/ PR
    - responsibility detection per QA/R&D/ PR
- Outputs:
    - structured JSON decisions
    - action tasks
    - risk flags
    - summary timeline
- Sends:
    - SMS/email/Slack notification
    - creates Jira/GitHub issues (API)

## Architecture

- microservices
- event-driven
- queue pipeline
- AI extraction service
- rules + LLM hybrid scoring

## Tech reuse

- Your transcription app ✅
- Your event microservices ✅
- Your summarization pipeline ✅
- RabbitMQ orchestration ✅

## 

> “Turns meetings into structured, actionable data automatically.”
>
# AI Meeting Intelligence Platform - Critical Questions

## The 4 Questions You Must Answer Before Building

These questions are ordered by dependency — answer them in this sequence.

---

## 1. Who are your target users and what's the real pain point?

**Why this matters:**

- A solo freelancer ≠ enterprise team with 100s of meetings/week
- Internal engineering meetings vs. client meetings have different structures
- Slack notifications are noise for some, essential for others
- This determines your architecture priorities and whether anyone will actually pay for/use it

**Key questions to resolve:**

- What specific meeting type(s) first (QA, R&D, PR — pick one to start)?
- What's the current workflow pain (e.g., "we forget action items after standups")?
- Who will buy/use this — engineering managers, product owners, Scrum masters?

---

## 2. What is your complete tech stack, and is it cohesive?

You mention reusing existing components, but you need a cohesive stack for *this* specific product.

**Why this matters:**

- Microservices + event-driven + queues = multiple languages/frameworks can create maintenance nightmare
- AI extraction needs model serving infrastructure
- Video/audio processing has specific library dependencies
- Each integration (Jira, GitHub, Slack, SMS, Email) has different SDK considerations

**Key questions to resolve:**

| Component | Options |
|-----------|---------|
| **Backend API** | FastAPI, Django, Node.js/Express, Go? |
| **AI/LLM** | OpenAI, Anthropic, open-source (Llama 3), hosted vs self-hosted? |
| **Queue System** | RabbitMQ (mentioned), Redis, AWS SQS, Kafka? |
| **Database** | PostgreSQL, MongoDB, or hybrid? |
| **Video/Audio** | FFmpeg, pydub, cloud processing (AWS Transcribe, Google Speech-to-Text)? |
| **Task Scheduling** | Celery, BullMQ, AWS Lambda + EventBridge? |
| **Frontend** | Next.js, React, Vue, or pure API-first? |
| **Hosting** | AWS, GCP, Azure, self-hosted? |

---

## 3. How accurate must the AI extraction be, and what's your validation strategy?

Your core value proposition is automatically extracting *decisions*, *action items*, *risks*, and *responsibilities* from free-form meeting transcripts. This is the hardest technical challenge.

**Why this matters:**

- False positives = spam tasks created in Jira/GitHub
- False negatives = missed commitments that impact projects
- Different meeting types (QA, R&D, PR) have different patterns

**Key questions to resolve:**

- What minimum accuracy threshold is acceptable (e.g., 90% precision)?
- How will users correct/validate extracted items?
- LLM-only, rules-based, or hybrid approach?
- Will you fine-tune models or use prompt engineering?

---

## 4. How do your existing reusable components actually fit this architecture?

You mention reusing transcription, event microservices, summarization, and RabbitMQ. But integration complexity can kill a project.

**Why this matters:**

- Microservices + event-driven + queues = distributed systems hell if not architected carefully
- Existing components may have assumptions that don't fit this use case
- API integrations (Jira/GitHub/Slack) each have rate limits, auth, and edge cases

**Key questions to resolve:**

- Does your transcription service handle the meeting formats your users have (MP4, WebM, Zoom recordings, etc.)?
- What's the queue strategy for long-running AI extraction vs. quick notification delivery?
- How will you handle failures (e.g., Jira API is down mid-task creation)?

---

## Recommended Next Steps

1. Answer these questions in order
2. Document your decisions in this file
3. Once answered, create an implementation plan based on concrete choices

---

*Generated for AI Meeting Intelligence Platform project*

Here are the most important questions to answer so the app becomes *actually useful* (not just “it transcribes and summarizes”).

    ## 1) What exactly is the input and what quality do you need?

    * What file types will you accept (mp3/mp4/wav/m4a/zoom recording)?
    * Typical length (5 min, 1 hour, 3 hours)?
    * Languages/accents? One speaker or many?
    * Do you need speaker diarization (“who said what”)?
    * How accurate must transcription be (good-enough vs near-perfect)?
    * Do you need timestamps and “jump to this moment in audio”?

    ## 2) What should the output look like and how will you measure “good”?

    Define your output schema clearly:

    * **Summary**: short + detailed? bullets? max length?
    * **Decisions**: what counts as a decision (explicit vote? “we agreed…”?)
    * **Action items**: format like “Owner + Task + Due date + Priority”?
    * **Risks**: what counts as a risk vs just “issue”?
    * Should the app extract **open questions** too?
    * How will you validate quality: user edits, thumbs up/down, “was this correct?”, diff vs ground truth?

    ## 3) How will tasks be created and where do they go?

    * Do you create tasks **automatically** or only after user confirms?
    * Which task systems: Notion / Jira / Trello / Asana / Todoist / Google Tasks?
    * What fields must be filled (owner, due date, tags, project)?
    * How do you detect **ownership** (names, “I’ll do it”, “John will…”)?
    * What happens when owner/due date is missing (leave blank? suggest? ask user)?

    ## 4) What’s the UX flow that saves time in 2 minutes?

    * Is it “upload → wait → review → export”?
    * Do you want a **review screen** where you can quickly correct owners/dates?
    * Do you want “approve all tasks” button?
    * Do you need multi-meeting workspace, search, filters, history?

    ## 5) Privacy, security, and compliance (this matters a lot for meeting recordings)

    * Can users upload sensitive info? (work meetings usually = yes)
    * Where do files live (cloud storage vs local)?
    * Do you delete audio after processing? After how long?
    * Do you need encryption at rest / in transit?
    * Any compliance requirements (SOC2-ish, GDPR-like, company policies)?

    ## 6) What’s your tech approach for extraction?

    * Pipeline steps: upload → transcription → segmentation → extraction → task creation
    * Do you use one LLM call or multiple (better reliability with multi-step)?
    * Do you need “structured output” JSON validation + retries?
    * How do you handle hallucinations (tasks that were never said)?
    * Do you store embeddings to enable “ask questions about this meeting”?

    ## 7) What are the failure modes and how will the app recover?

    * Bad audio, overlapping speakers, noisy rooms
    * Very long meetings (timeouts/cost)
    * Mixed languages
    * “No decisions/action items found” (what should UI show?)
    * Incorrect owners/dates—how will users fix quickly?

    ## 8) Cost & speed constraints

    * Target processing time (real-time? 2–5 minutes after upload?)
    * Per-hour processing cost you can accept
    * Will you offer “fast/cheap” vs “slow/better” modes?

    ---

    ### If you only pick **3** questions to start with:

    1. **What is your exact output schema** (summary/decisions/actions/risks fields and rules)?
    2. **How will tasks be created and confirmed** (auto vs review + which integrations)?
    3. **What level of transcription quality** do you need (diarization, timestamps, languages, typical meeting length)?

    If you tell me:

    * typical meeting length + language,
    * where you want tasks created (Notion/Jira/etc),
    * and whether you need speaker names,
    I’ll propose a clean MVP scope + pipeline architecture.

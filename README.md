1. Keyword Matching Engine

How exactly is "partial matching" implemented — substring match, regex with wildcards, or something else (e.g., tokenization + fuzzy match like Levenshtein)?
Is matching case-insensitive by default?
How are overlapping matches handled (e.g., keyword "OSFI" and keyword "OSFI Report" both matching the same text)?
How is performance handled when a file has thousands of words and a keyword list has hundreds of entries — what's the algorithmic approach (naive O(n*m) scan, Aho-Corasick, indexing)?
How are false positives/negatives handled or reduced?
What file types are supported (PDF, DOCX, TXT, scanned/image PDFs)? How is text extracted from each (e.g., PyMuPDF, pdfplumber, OCR)?

2. Session Management (no DB — this is the big one)

Since there's no database, where is session state actually stored — in-memory (per server process), Redis, filesystem/temp storage, or browser-side?
How does "session sharing for secondary review" work technically without a DB — is there a session ID/token, and where does the underlying file + redaction state live so a second user can open the same session?
What happens if the server restarts — is session state lost? Is there any persistence at all (e.g., temp files on disk)?
How long does a session live — is there a TTL/expiration and cleanup process?
With FastAPI being async, how is session state kept consistent if multiple async requests touch the same session concurrently (race conditions on shared session data)?
If deployed across multiple server instances/pods, how is session state shared across instances (since no DB/no shared cache would mean sessions are pinned to one instance)?

3. Concurrency & Scalability (why FastAPI/async mattered)

What specifically does FastAPI's async model buy us here — is it async I/O (file reads, TeamMate+ API calls) or also concurrent CPU-bound keyword scanning?
Are CPU-heavy tasks (keyword scanning across large files) run in a thread pool / background workers to avoid blocking the event loop?
How is the system expected to behave under the "double load at month/quarter-end" scenario — is there load testing data, or is this an assumption?
Is there any queueing (e.g., background task queue) for processing large files, or is it all synchronous request/response?
How is the app deployed — single instance, multiple instances behind a load balancer, containerized (Docker/Kubernetes)?
Are there any caching layers (e.g., caching keyword lists in memory per session)?

4. Security Controls

Encryption at rest: what exactly is encrypted at rest — uploaded files, temp storage, keyword lists? What's the mechanism (disk-level encryption, encrypting individual files with a key, cloud provider-managed encryption)?
Encryption in transit: is this just HTTPS/TLS for all traffic, or is there additional encryption for specific calls (e.g., to TeamMate+)?
Authentication: how exactly does LDAP + AD group integration work — is it validating credentials against AD directly, or via an SSO/token layer? What determines a user's permissions (are AD groups mapped to app roles)?
Authorization: are there different permission levels (e.g., who can redact vs who can only review) tied to AD groups?
Locks — what are the "locks" you mentioned? (e.g., file locks to prevent two users editing the same file/session simultaneously? Or session locks during the redaction review process?) Get specifics: what triggers a lock, how is it released, what happens if a user closes the browser without releasing it.
Is uploaded file data ever logged (e.g., in application logs, error logs) — any risk of sensitive content leaking into logs?
How are temp files cleaned up after a session ends — securely deleted, or just standard file deletion?
Any audit logging of who redacted what, when? (Important for a compliance-focused tool.)

5. File Import/Export & TeamMate+ Integration

How does the TeamMate+ integration authenticate (API key, OAuth, service account)?
Is there error handling/retry logic if TeamMate+ is unavailable or slow?
How are multiple files handled — processed in parallel or sequentially?
What does the "export" step actually produce — is redaction "true" redaction (content permanently removed/burned in) or just visual overlay (important distinction, since visual-only redaction is a common real-world compliance bug)?

6. Frontend

What's the frontend built in (React, Vue, plain JS)?
How does the UI render highlights on top of a PDF/document — is there a PDF rendering library (e.g., pdf.js) with an overlay layer for highlights?
How does the frontend communicate redaction state to the backend — REST calls per action, or batched?

7. AI-Assisted Development Specifics (for your "how AI built this" narrative)

Which AI tool(s) exactly were used to write code — GitHub Copilot, Claude, ChatGPT, Cursor?
For the POC→FastAPI migration: what did the generated "feature specs" actually look like (format — markdown, structured JSON, plain prose)? Do those spec files still exist and could you show one as an example in the interview?
Roughly what % of the FastAPI codebase was AI-generated vs. hand-written/hand-modified?
Can you find one concrete example of AI-generated code that a team member had to correct or rewrite — this is very valuable for your interview story about critically reviewing AI output.

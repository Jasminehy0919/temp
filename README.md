Context: The app currently has no durable, on-disk application log. What we saw earlier in err.log was uvicorn/console output only (printed via Python’s logging module to stdout), without timestamps, without rotation, and lost on restart. We want to build a proper logging system from scratch.

Please design the following, and answer the questions below, before writing any code:

1. File structure — exactly 2 log files, not more

	•	audit.log: every user-facing event, in one chronological stream. Must include at minimum: precise timestamp (date+time, explicit timezone or UTC), username, event type, and relevant IDs (session_id, doc_id, share_id) — structured consistently (e.g., key=value pairs) so it’s easy to grep/filter by user or time range.
	•	app.log: everything else — internal errors, warnings, startup/shutdown, cache operations, unhandled exception tracebacks with full stack trace and request context.

Please don’t split further into per-event-type files — one audit.log with consistent structure is easier to review than scattered files.

2. Audit events to capture

	•	Login: username, timestamp, source IP if available.
	•	Logout: both explicit logout and session expiry/timeout (TTL-based), since users may just close the tab.
	•	Session lifecycle: session created, session expired — include session duration.
	•	Sharing: when a share is created (who, when, which doc), and when/if it’s accessed by someone other than the original session owner — referencing the _share_store.py mechanism seen earlier.
	•	Also flag any other actions worth audit-logging for this compliance-sensitive (audit-document) context — e.g., file upload, redaction export, download — and propose which ones matter most.

3. Timestamps

All log lines (both files) must include precise timestamps. This applies to existing log statements too (auth, cache, sessions), not just new ones.

4. Rotation

Time-based (daily) rotation for both files, retaining a configurable number of days (suggest 90 as default, adjustable via config). Files roll over at midnight (e.g., audit.log → audit.log.2026-07-17).

5. Global exception handling

Add exception-handling middleware so any unhandled error in any endpoint logs a full traceback with request context (path, method, username if available) to app.log, rather than silently returning a bare connection reset or an unlogged 500.

Please answer these questions in your analysis:

	•	Where in the code does login currently happen, and where is the most natural hook point to log it?
	•	Where does logout / session expiry currently get detected (TTL check on read? background cleanup thread?) — is there an existing hook, or does one need to be added?
	•	Where in _share_store.py is the natural hook point for share creation/access logging?
	•	Proposed log file paths and the specific logging library/mechanism you’ll use for rotation (e.g., Python’s TimedRotatingFileHandler).
	•	Any risk of the new audit logging adding meaningful latency to hot paths (e.g., logging on every /positions call, if that ends up being audit-worthy) — and how to avoid that.

Give me the full analysis and design first. I will confirm before you write any code.
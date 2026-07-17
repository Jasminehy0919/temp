好，那我把之前整理的实现要求 + 这次的"不改动现有逻辑"约束，合并成一条完整消息，你一次性发给它：


	•	Use TimedRotatingFileHandler with when='W6' so weeks align to calendar Sunday boundaries (e.g., current week through 7/19/2026, next week 7/20–7/26, etc.)
	•	Retention: approximately 13 weekly files (~90 days). Confirm this number.

app.log: daily rotation

	•	Use TimedRotatingFileHandler with when='midnight'
	•	Retention: 90 daily files (backupCount=90)Show me example rotated filenames for both (e.g., audit.log.2026-07-19 and app.log.2026-07-17) before finalizing.

Both should use utc=True for rotation timing, consistent with UTC timestamps in the log content.
---

**Approved — please proceed with implementation based on your proposed design, with these adjustments and constraints:**

## Important constraint: purely additive, no behavior changes
This work must not change any existing function's behavior, return values, response codes, or control flow. You're adding log statements and a new middleware layer, not modifying business logic.

**Before writing any code, please confirm:**
1. Adding audit log calls at the hook points you identified (e.g., `server.py:1376` for login, `_session_store.py:1043` for session cleanup) — will these be simple additions (a log line inserted into existing code paths) with zero changes to what the function returns or how it behaves? Or does any hook point require restructuring existing logic to get the data needed for logging?
2. The global exception-handling middleware — confirm it only adds a new layer that *catches and logs* unhandled exceptions before returning a controlled 500, and won't change behavior for requests that currently succeed, or alter responses for errors already explicitly handled by existing try/except blocks — only genuinely unhandled ones.
3. Rerouting existing module logs (auth/cache/session) into `app.log` via root logger config — confirm this only changes *where log output goes*, not any conditional logic that happens to also call `logging.info(...)`.

**If any hook point requires touching business logic (not just adding a log call), flag it specifically so we can decide together whether it's worth it — don't make that change silently.**

## Design adjustments from earlier discussion

**1. Share caveat — go with Option 3:** Since share routes are intentionally disabled, don't re-enable them. Use `session_link_accessed_by_non_owner` as the practical substitute for share-access auditing for now.

**2. Rotation: weekly (calendar weeks ending Sunday), not daily:**
- Use `TimedRotatingFileHandler` with `when='W6'` so weeks align to calendar Sunday boundaries (e.g., current week through 7/19/2026, next week 7/20–7/26, etc.)
- Recalculate `backupCount` for ~90 days retention using weekly files — approximately **13 weekly files** (90 ÷ 7 ≈ 13). Confirm this number.
- Show me an example rotated filename before f
## Everything else stays as designed:
- Two-file architecture (`audit.log` + `app.log`), no additional event-type files
- Structured key=value format, one line per event, consistent field order
- UTC timestamps with milliseconds
- Full event list as proposed (login/logout, session lifecycle, sharing substitute, document handling, redaction/export/download, security/admin events)
- QueueHandler + QueueListener for async writes (no blocking on hot paths)
- Config additions under a `logging` section (enabled, dir, retention, rotate_when, level, audit_level, app_level)

**Please answer the 3 confirmation questions above first. Once confirmed with no concerns, proceed with implementation. After it's done, show me:**
1. A sample of what `audit.log` looks like after a typical session (login → file upload → redact → export → logout)
2. The exact log file paths being used
3. Confirmation that existing module logs (auth/cache/session) are now flowing into `app.log` with timestamps

---

这条发过去后，它应该会先回答那 3 个确认问题（有没有需要碰业务逻辑的地方），你看完确认没问题，再让它接着实现。
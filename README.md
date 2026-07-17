1. Persistent application/error logging (following up on our earlier discussion):

	•	Ensure all logs (not just auth) are written to a rotating log file on disk (not just stdout/stderr), so logs survive service restarts and don’t grow unbounded.
	•	Add the global exception-handling middleware we discussed earlier, so any unhandled exception logs a full traceback with request context (path, method, user if available).

2. Audit logging for user sessions — who logged in, when, and when they logged out:

	•	Log a structured audit event on successful login: username, timestamp, source IP if available.
	•	Log a structured audit event on logout — including both explicit logout and session expiry/timeout (since users may just close the tab without clicking logout).
	•	Consider what other actions should be audit-logged for this kind of tool handling audit/compliance documents — e.g., file uploads, redaction exports, downloads — since this may matter for compliance purposes given the KPMG/audit context we’ve seen in test data.
	•	Keep audit logs in a separate, clearly named log file/stream from general application logs, so they’re easy to review independently (e.g., audit.log vs app.log).

Please do a quick risk/design analysis first — where in the code login/logout currently happens (_auth.py? server.py?), whether session expiry already fires an event we can hook into, and what the log file paths/rotation strategy should be — before writing any code.

这样能让它先理清楚”登录/登出的代码在哪
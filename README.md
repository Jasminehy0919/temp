

audit.log: weekly rotation, calendar weeks ending Sunday

	•	Use TimedRotatingFileHandler with when='W6' so weeks align to calendar Sunday boundaries (e.g., current week through 7/19/2026, next week 7/20–7/26, etc.)
	•	Retention: approximately 13 weekly files (~90 days). Confirm this number.

app.log: daily rotation

	•	Use TimedRotatingFileHandler with when='midnight'
	•	Retention: 90 daily files (backupCount=90)

Both should use utc=True for rotation timing, consistent with UTC timestamps in the log content.

Show me example rotated filenames for both (e.g., audit.log.2026-07-19 and app.log.2026-07-17) before finalizing.
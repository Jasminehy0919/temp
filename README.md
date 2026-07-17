Before implementing, please do a quick risk analysis on both fixes at the implementation level:

For Fix 1 (atomic meta.json writes):

	1.	Does switching to temp-file + os.replace interact safely with the existing threading.Lock in _doc_cache.py:41? Any risk of the lock being held across the temp-write + rename in a way that changes timing elsewhere?
	2.	Any risk of orphaned temp files accumulating if the process crashes mid-write (e.g., temp file naming/cleanup strategy)?
	3.	Does this change affect the TTL-slide read path’s performance noticeably given how often it’s called (once per successful get)?

For Fix 2 (frontend concurrency throttling):

	1.	Does capping concurrency to N change the user’s perceived behavior in a way that matters — e.g., does onProgress/onPageResult currently assume all pages resolve near-simultaneously, and could batching introduce a “stalling” visual effect or reorder result arrival unexpectedly?
	2.	Is there any existing timeout/abort logic (e.g., AbortController, opts.signal) that assumes all requests are in-flight together, which might break with sequential batching?
	3.	What’s the right batch size — is N=4-6 arbitrary, or should it match something concrete like the actual worker count or a measured backend capacity limit?
	4.	Could this fix alone (without Fix 1) already reduce the 404 rate enough that Fix 1’s urgency changes? Or are they independent enough that order doesn’t matter?
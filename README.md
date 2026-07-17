Fix 2 confirmed working — blackout is gone, and /positions 404s dropped from ~5-6 down to 2 out of ~79 requests in this test run. This confirms Fix 1 is still needed to fully close the race.

One new observation: this run also showed a single 404 on /sessions/current — not seen before. Can you check if this is the same class of read/write race (session store), or something else? Just flag it, no need to fix unless it’s related.

Please proceed with Fix 1 now (atomic meta.json writes) as planned.
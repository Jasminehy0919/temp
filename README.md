This all looks solid, no major concerns. Let’s proceed in the order you recommended:

1. Implement Fix 2 first (frontend concurrency throttling) — start with N=4 (matching the 4-worker count). Keep onPageResult/progress logic and AbortController signal sharing exactly as-is, just cap concurrent in-flight requests.

2. Once Fix 2 is done, implement Fix 1 (backend atomic meta.json writes) — temp file + os.replace, with unique temp naming (pid/tid/random suffix) and add stale temp file cleanup to the existing cache cleanup loop at _doc_cache.py:298.

Please implement Fix 2 now. Stop after Fix 2 is complete and let me test in IST before starting Fix 1.
**Bug report for the coding tool:**

---

After implementing Solution 1 (omit `pdf_b64` on cache hit), I'm seeing a race condition: when switching tabs quickly, the content displayed sometimes belongs to a *previous* tab instead of the one currently selected (e.g., I'm on tab 3, but tab 2's content is shown).

**Root cause:** Before Solution 1, every tab switch sent the full `pdf_b64`, so all requests were uniformly slow and rarely arrived out of order. Now that cache hits return quickly (doc_id only) while cache misses still return slowly (full pdf_b64), responses can arrive **out of order**. If the user switches tabs fast, a later request (fast, cache hit) can resolve and render before an earlier request (slow, cache miss) — and when the slow one finally arrives, it overwrites the screen with stale content, even though the user has already moved to a different tab.

**Fix needed:** Add a stale-response guard on the frontend. When switching tabs, track which tab/doc_id is currently active (e.g., a request token or the active tab id). When a response comes back, check if it still matches the currently active tab before rendering it. If the user has since switched away, discard the response instead of rendering it.

Please locate where tab-switch responses are handled and add this guard so out-of-order responses can no longer overwrite the current view.

---

你可以把这段直接复制贴给那个工具。
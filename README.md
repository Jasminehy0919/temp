**给那个工具的 prompt：**

---

Proceed with the token-gating fix as planned. After you implement it, give me a complete checklist mapping every mutation point you listed earlier in the trace (the early-reset block items 2–15, the post-load writes items 17–23, the `_loadSlotPdf` internal writes items 25–26, and the final commit items 28–37) to the specific token check that now guards it. For each one, tell me: file, line number, and which token check (1st, 2nd, or a new one you added) protects it. If any mutation point from that original list is still unguarded, call it out explicitly instead of omitting it.

Also confirm: after this fix, if switch S1 (slow) and switch S2 (fast) overlap, does S1 get cut off *before* it executes any of the early-reset mutations, or only after?

---

这段要求它在改完后逐条对照"它自己之前列出的清单"，而不是泛泛地说"修好了"。如果它真的漏了哪个点，这种逐条核对的方式更容易暴露出来。
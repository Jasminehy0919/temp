明白，那就是把日志时间戳从 UTC 改成美东时间（Eastern Time，会自动处理夏令时/标准时切换）。

不过这里有一个点需要跟你确认清楚，因为之前特意选 UTC 是有考虑的：**审计日志用 UTC 的好处是不会有夏令时切换造成的时间跳跃或重复**（比如夏令时结束那天，本地时间会出现"1:30 AM 出现两次"这种情况，对审计记录来说容易造成混淆）。如果你确定要用美东时间，这个夏令时的边界情况需要让它特别注意处理。

另外，**文件轮转的时间点（周日/午夜）现在是按 UTC 算的**，如果时间戳改成美东时间，轮转边界要不要也一起改成美东时间的周日/午夜？这样轮转边界和日志内容的时间才能对得上，不然会出现"文件名说是周日凌晨,但内容时间戳其实是美东时间周六晚上"这种不一致。

给它的英文提示：

---

**Please change all log timestamps from UTC to US Eastern Time (America/New_York), including automatic DST handling (EDT/EST transitions).**

**Please also confirm/address:**
1. Since Eastern Time observes DST, please confirm the logging setup correctly handles the "fall back" transition (when 1:00–2:00 AM occurs twice) and "spring forward" transition (2:00–3:00 AM is skipped) without creating ambiguous or duplicate timestamps in a way that breaks readability.
2. Should file rotation boundaries (currently UTC-based: `app.log` at UTC midnight, `audit.log` weekly at UTC Sunday) also switch to Eastern Time midnight/Sunday, so the rotation timing matches what the timestamps inside the file say? I'd say yes — confirm this makes sense and update rotation to use Eastern Time as well.
3. Confirm this is a config/formatter-level change only (timezone conversion for display), not a change to any underlying business logic, TTL calculations, or session expiry timing (those should likely still be computed correctly regardless of display timezone — please confirm nothing else silently changes).

**Please answer these before implementing, then proceed once confirmed.**

---

这样能确保时区改动是"只改显示"，不会不小心影响到 TTL 过期这类跟"真实时间流逝"相关的逻辑判断。
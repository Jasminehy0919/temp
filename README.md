The guards keep failing — first blank page, then tabs not switching, now content is still bleeding across tabs. Stop patching individual guards. Before making any more changes:

	1.	Show me the full _switchSlot() function as it currently stands, including every place that reads or writes currentSlot, #slotSwitchToken, #loading, and any other shared state touched during a switch.
	2.	Explain, step by step, what happens when two switches overlap (e.g., user goes Tab A → Tab B → Tab C quickly): which async calls are in flight, when each token is assigned, when each is compared, and when shared state gets mutated.
	3.	Identify every point where shared/rendered state is mutated, and confirm each mutation is guarded by the same token check.

Don’t apply a fix yet — first show me the trace so we can confirm the actual root cause before changing code again.
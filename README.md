I'm preparing for a technical interview and need to understand exactly how our 
E2E testing was implemented. Please answer using SPECIFIC details from the 
actual code/test files — file names, function names, config values — not 
general assumptions.

## 1. Test Framework & Setup
- Confirm: is Playwright (Python) used for E2E, and Vitest for unit tests? 
Where are the config files for each (e.g. playwright config, vitest.config.js)?
- How are tests organized — one file per feature/suite, or another structure?
- How is the mock or real AD (Active Directory) set up for auth tests — is 
there a mock LDAP server, stubbed responses, or a real test AD instance?

## 2. Test Scenario Coverage
- I have a document called E2E-TEST-SCENARIOS.md listing scenarios like 
AUTH-01 through AUTH-13 and SESS-01 through SESS-07+ with priorities (P0-P3) 
and coverage percentages (e.g. Auth 85%, Session lifecycle 78%, overall ~72%). 
Are these scenario IDs actually implemented as real test cases in the test 
files? Can you find the actual test file(s) and confirm how many of these 
scenario IDs have corresponding test code?
- How was the coverage percentage (e.g. "85%", "72% overall") calculated — is 
there a coverage tool, or was this an estimate written by whoever authored the 
spec?

## 3. Known Gaps (from the "Hard-to-discover gaps" section)
- For AUTH-03/04 (rate limit state is in-memory per worker): is there evidence 
in the test code of how this was handled — e.g., does the test explicitly 
restart the server, or account for the in-memory state some other way?
- For AUTH-07 (Safari private browsing blocking localStorage): is there an 
actual Playwright browser context configured for this, or is this scenario 
still just documented but not implemented?
- For AUTH-12 (server blacklist is in-memory, so a second worker wouldn't see 
revocation): is there any code addressing this limitation, or is it purely 
documented as a known risk?

## 4. How AI Was Used to Build the Tests
- Can you tell from file comments, commit messages, or structure whether these 
test scenarios/files were AI-generated, AI-assisted, or hand-written? 
- Is there any evidence of a workflow where scenarios were planned first (like 
in E2E-TEST-SCENARIOS.md) and then an AI tool was used to generate the actual 
Playwright test code from that spec?

## 5. Running the tests
- Confirm the actual commands to run the full E2E suite vs a smoke subset 
(I believe it's something like `cd tests/e2e && uv run pytest -v` for full, 
and `uv run pytest test_smoke_first_cut.py -v` for smoke — please confirm 
exact commands from AGENTS.md or the test folder itself).
- Roughly how long does the full E2E suite take to run?

Please organize your answer with the same numbered headers so I can match it 
up with my questions.

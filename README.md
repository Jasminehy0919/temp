I need a system architecture diagram for a technical interview. Based on 
the actual codebase, create a diagram (as an SVG, or describe it in a 
structured way I can turn into one) showing:

1. All major components: frontend (vanilla JS + custom elements), FastAPI 
backend, PyMuPDF processing layer, file-based session store, LDAP/JWT auth, 
and the TeamMate+ integration.

2. For each component, show the KEY data flowing between them — e.g., what 
exactly goes from frontend to backend on a redaction request, what comes 
back, what gets written to session storage.

3. Show where the two async queues live (session queue vs. find queue) in 
frontend/src/api-client.js, and what each is responsible for.

4. Show the session lifecycle stages (create/load/save/delete/claim/release/
export) as a small state diagram or flow, referencing the actual function/
endpoint names from backend/server.py.

5. Mark clearly where there is NO database — file-based JSON with file 
locking (portalocker) — since that's an important architectural decision 
to explain.

6. Mark the TeamMate+ integration boundary — what leaves our system, what 
comes back, and the auth method (Bearer API key).

Please give me the actual box/arrow layout with labels — even a simple 
text-based layout description (boxes, connections, labels) is fine, I'll 
turn it into a clean diagram myself. Reference actual file names for each 
box so I know it's grounded in the real code, not a generic guess.

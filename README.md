<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>RI Process Flow: NiFi / AutoSys</title>
<style>
:root{
  box-sizing:border-box;
  padding-top:env(safe-area-inset-top,0px);
  padding-bottom:env(safe-area-inset-bottom,0px);
  --bg:#EEF1F6; --panel:#FFFFFF; --ink:#16233B; --muted:#5B6980; --line:#C9D1DD;
  --hdr1:#1B2F6E; --hdr2:#2B47A0; --hdr-ink:#FFFFFF;
  --flow:#3B4A66; --green:#2E7D4F; --red:#C2412D; --navy:#2B3F8F;
  --trig:#6B57A8; --sys:#2F5DA8; --inc:#C2412D; --wait:#2B7A78; --data:#8A6A1F; --dec:#B5651D;
  --warn:#FFF6DD; --warn-line:#E6C766;
  color-scheme:light;
}
@media (prefers-color-scheme: dark){
  :root:not([data-theme="light"]){
    --bg:#0E1522; --panel:#16223A; --ink:#E6ECF5; --muted:#9AA9BF; --line:#2F4163;
    --hdr1:#162552; --hdr2:#22397F; --hdr-ink:#FFFFFF;
    --flow:#9DB0CF; --green:#58B27D; --red:#F08A78; --navy:#8FA6F0;
    --trig:#A895E6; --sys:#7FA6E8; --inc:#F08A78; --wait:#6CC3BF; --data:#D8B25C; --dec:#E0A060;
    --warn:#2D2A18; --warn-line:#6B5A1F;
    color-scheme:dark;
  }
}
:root[data-theme="dark"]{
  --bg:#0E1522; --panel:#16223A; --ink:#E6ECF5; --muted:#9AA9BF; --line:#2F4163;
  --hdr1:#162552; --hdr2:#22397F; --hdr-ink:#FFFFFF;
  --flow:#9DB0CF; --green:#58B27D; --red:#F08A78; --navy:#8FA6F0;
  --trig:#A895E6; --sys:#7FA6E8; --inc:#F08A78; --wait:#6CC3BF; --data:#D8B25C; --dec:#E0A060;
  --warn:#2D2A18; --warn-line:#6B5A1F;
  color-scheme:dark;
}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);
  font:15px/1.55 "Avenir Next","Segoe UI",system-ui,-apple-system,sans-serif}
.hero{background:linear-gradient(135deg,var(--hdr1),var(--hdr2));color:var(--hdr-ink);
  padding:26px 20px 22px;text-align:center}
.hero h1{margin:0;font-size:clamp(22px,3.4vw,32px);letter-spacing:.2px}
.hero p{margin:6px 0 0;opacity:.85;font-size:clamp(13px,1.8vw,16px)}
.wrap{max-width:1180px;margin:0 auto;padding:20px 16px 40px;
  display:grid;gap:18px;grid-template-columns:minmax(0,560px) minmax(0,1fr);align-items:start}
@media (max-width:940px){.wrap{grid-template-columns:minmax(0,1fr)}}
.card{background:var(--panel);border:1px solid var(--line);border-radius:12px;padding:16px 18px}
.card h2{margin:0 0 10px;font-size:17px}
.diagram{padding:10px 6px;overflow-x:auto}
svg.flow{display:block;width:100%;min-width:480px;height:auto}
svg text{font-family:inherit;fill:var(--ink)}
.ed{fill:none;stroke-width:2}
.ed.flow{stroke:var(--flow)} .ed.yes{stroke:var(--green)} .ed.no{stroke:var(--red)}
.ed.back{stroke:var(--navy);stroke-dasharray:6 4}
.m-flow{fill:var(--flow)} .m-yes{fill:var(--green)} .m-no{fill:var(--red)} .m-back{fill:var(--navy)}
.node{--c:var(--sys)}
.t-trig{--c:var(--trig)} .t-sys{--c:var(--sys)} .t-inc{--c:var(--inc)}
.t-wait{--c:var(--wait)} .t-data{--c:var(--data)} .t-dec{--c:var(--dec)}
.node .body{fill:color-mix(in srgb,var(--c) 9%,var(--panel));stroke:var(--c);stroke-width:1.5}
.node .bar{fill:var(--c)}
.node .t{font-size:13px;font-weight:700}
.node .t.d{font-size:12px}
.node .s{font-size:10.5px;fill:var(--muted)}
.yes,.no{font-size:12px;font-weight:700}
text.yes{fill:var(--green)} text.no{fill:var(--red)}
text.lbl{font-size:11px;font-weight:700}
text.note{font-size:10.5px;fill:var(--muted)}
.side{display:grid;gap:18px}
.tablewrap{overflow-x:auto;border:1px solid var(--line);border-radius:8px;
  background:color-mix(in srgb,var(--warn) 40%,var(--panel))}
table{border-collapse:collapse;width:100%;font-size:13.5px}
th,td{padding:8px 10px;text-align:left;border-bottom:1px solid var(--line);vertical-align:top}
tr:last-child td{border-bottom:0}
th{color:var(--muted);font-weight:600;font-size:12.5px}
td:first-child{font-weight:700}
.note-box{margin:10px 0 0;padding:10px 12px;background:var(--warn);border:1px solid var(--warn-line);
  border-radius:8px;font-size:13px}
.kp{margin:0;padding-left:20px}
.kp>li{margin:0 0 10px}
.kp ul{margin:4px 0 0;padding-left:18px}
.kp ul ul{list-style:circle}
.kp li li{margin:2px 0}
code{font:12.5px ui-monospace,SFMono-Regular,Menlo,Consolas,monospace;
  background:color-mix(in srgb,var(--ink) 8%,transparent);padding:1px 5px;border-radius:4px;word-break:break-all}
.legend{display:grid;grid-template-columns:repeat(2,minmax(0,1fr));gap:8px 16px;font-size:13.5px}
.legend div{display:flex;align-items:center;gap:8px}
.sw{width:22px;height:14px;border-radius:3px;border:1.5px solid var(--c);
  background:color-mix(in srgb,var(--c) 18%,var(--panel));flex:none}
.sw.dia{width:16px;height:16px;transform:rotate(45deg) scale(.8);border-radius:2px}
.ln{width:24px;height:0;border-top:2px solid var(--flow);flex:none}
.ln.d{border-top:2px dashed var(--navy)}
</style>
</head>
<body>
<header class="hero">
  <h1>RI Process Flow: NiFi / AutoSys</h1>
  <p>From RI trigger to posting (Subledger / CGL), including rerun and issue handling</p>
</header>

<div class="wrap">
  <section class="card diagram" aria-label="Process flow diagram">
<svg class="flow" viewBox="0 0 520 970" role="img" aria-labelledby="ft fd">
<title id="ft">RI process flow: NiFi / AutoSys</title>
<desc id="fd">Flow from RIWMA Run Trigger through NiFi upload and RI run, with failure handling, success trigger, waiting for the next trigger, rerun, and posting to Subledger / CGL.</desc>
<defs>
<marker id="a-flow" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" class="m-flow"/></marker>
<marker id="a-yes" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" class="m-yes"/></marker>
<marker id="a-no" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" class="m-no"/></marker>
<marker id="a-back" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto"><path d="M0 0L10 5L0 10z" class="m-back"/></marker>
</defs>
<path d="M380,84 V100" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M380,150 V180" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M380,230 V260" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M380,310 V330" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M380,400 V440" class="ed yes" marker-end="url(#a-yes)"/>
<path d="M300,365 H120 V400" class="ed no" marker-end="url(#a-no)"/>
<path d="M40,428 H24 V584 H40" class="ed no" marker-end="url(#a-no)"/>
<path d="M120,456 V480" class="ed no" marker-end="url(#a-no)"/>
<path d="M200,508 H230 V472 H260" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M120,612 V636 H8 V45 H260" class="ed back" marker-end="url(#a-back)"/>
<path d="M380,518 V548" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M380,598 V625" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M300,660 H170 V720" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M460,660 H480 V720" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M170,800 V830 H8 V45 H260" class="ed back" marker-end="url(#a-back)"/>
<path d="M410,784 V820" class="ed flow" marker-end="url(#a-flow)"/>
<path d="M410,864 V904" class="ed flow" marker-end="url(#a-flow)"/>
<g class="node t-trig">
<rect class="body" x="260" y="20" width="240" height="64" rx="7"/>
<rect class="bar" x="260" y="27" width="4" height="50" rx="2"/>
<text class="t" x="276" y="40">1. RIWMA places Run Trigger</text>
<text class="s" x="276" y="57.0">{entity}_actuarial_ri_trig_</text>
<text class="s" x="276" y="71.5">{FYYYY}_{PMM}.csv</text>
</g>
<g class="node t-sys">
<rect class="body" x="260" y="100" width="240" height="50" rx="7"/>
<rect class="bar" x="260" y="107" width="4" height="36" rx="2"/>
<text class="t" x="276" y="120">2. NiFi reads the trigger file</text>
<text class="s" x="276" y="137.0">Detects and reads the Run Trigger</text>
</g>
<g class="node t-sys">
<rect class="body" x="260" y="180" width="240" height="50" rx="7"/>
<rect class="bar" x="260" y="187" width="4" height="36" rx="2"/>
<text class="t" x="276" y="200">3. NiFi uploads files to RI</text>
<text class="s" x="276" y="217.0">Uploads the required files</text>
</g>
<g class="node t-sys">
<rect class="body" x="260" y="260" width="240" height="50" rx="7"/>
<rect class="bar" x="260" y="267" width="4" height="36" rx="2"/>
<text class="t" x="276" y="280">4. NiFi triggers the RI run</text>
<text class="s" x="276" y="297.0">For the specified entity, FYYYY, PMM</text>
</g>
<g class="node t-trig">
<rect class="body" x="260" y="440" width="240" height="78" rx="7"/>
<rect class="bar" x="260" y="447" width="4" height="64" rx="2"/>
<text class="t" x="276" y="460">5. NiFi places Success Trigger</text>
<text class="s" x="276" y="477.0">RI_NiFi_{entity}_{FYYYY}_{PMM}_</text>
<text class="s" x="276" y="491.5">{projectId}_{completedTime}_</text>
<text class="s" x="276" y="506.0">RIRunSuccess.txt (to riwma/incoming)</text>
</g>
<g class="node t-wait">
<rect class="body" x="260" y="548" width="240" height="50" rx="7"/>
<rect class="bar" x="260" y="555" width="4" height="36" rx="2"/>
<text class="t" x="276" y="568">6. Wait for next trigger</text>
<text class="s" x="276" y="585.0">Within the defined waiting window</text>
</g>
<g class="node t-trig">
<rect class="body" x="60" y="720" width="220" height="80" rx="7"/>
<rect class="bar" x="60" y="727" width="4" height="66" rx="2"/>
<text class="t" x="76" y="740">7a. Process Run Trigger</text>
<text class="s" x="76" y="757.0">Reads file, uploads to RI</text>
<text class="s" x="76" y="771.5">Triggers the RI run again</text>
<text class="s" x="76" y="786.0">Normal rerun, no incident</text>
</g>
<g class="node t-trig">
<rect class="body" x="320" y="720" width="180" height="64" rx="7"/>
<rect class="bar" x="320" y="727" width="4" height="50" rx="2"/>
<text class="t" x="336" y="740">7b. Process Post Trigger</text>
<text class="s" x="336" y="757.0">Triggers posting</text>
<text class="s" x="336" y="771.5">to Subledger / CGL</text>
</g>
<g class="node t-data">
<rect class="body" x="320" y="820" width="180" height="44" rx="7"/>
<rect class="bar" x="320" y="827" width="4" height="30" rx="2"/>
<text class="t" x="336" y="846.5">Post to Subledger / CGL</text>
</g>
<g class="node t-sys">
<rect class="body" x="320" y="904" width="180" height="44" rx="7"/>
<rect class="bar" x="320" y="911" width="4" height="30" rx="2"/>
<text class="t" x="336" y="930.5">Process Completed</text>
</g>
<g class="node t-inc">
<rect class="body" x="40" y="400" width="160" height="56" rx="7"/>
<rect class="bar" x="40" y="407" width="4" height="42" rx="2"/>
<text class="t" x="56" y="420">RI Run Failed</text>
<text class="s" x="56" y="437.0">Raise incident</text>
<text class="s" x="56" y="451.5">Investigate root cause</text>
</g>
<g class="node t-sys">
<rect class="body" x="40" y="480" width="160" height="56" rx="7"/>
<rect class="bar" x="40" y="487" width="4" height="42" rx="2"/>
<text class="t" x="56" y="500">Non-file issue</text>
<text class="s" x="56" y="517.0">Investigate and resolve</text>
<text class="s" x="56" y="531.5">Manual RI run if required</text>
</g>
<g class="node t-trig">
<rect class="body" x="40" y="556" width="160" height="56" rx="7"/>
<rect class="bar" x="40" y="563" width="4" height="42" rx="2"/>
<text class="t" x="56" y="576">File issue</text>
<text class="s" x="56" y="593.0">Reopen RIWMA UI, update files</text>
<text class="s" x="56" y="607.5">New Run Trigger to Step 1</text>
</g>
<g class="node t-dec">
<polygon class="body" points="380,330 460,365 380,400 300,365"/>
<text class="t d" x="380" y="364" text-anchor="middle">Did RI run</text>
<text class="t d" x="380" y="379" text-anchor="middle">succeed?</text>
</g>
<g class="node t-dec">
<polygon class="body" points="380,625 460,660 380,695 300,660"/>
<text class="t d" x="380" y="659" text-anchor="middle">Which trigger</text>
<text class="t d" x="380" y="674" text-anchor="middle">is received?</text>
</g>
<text class="yes" x="388" y="422" text-anchor="start">Yes</text>
<text class="no" x="292" y="357" text-anchor="end">No</text>
<text class="lbl" x="164" y="700" text-anchor="end">Run Trigger (re-run)</text>
<text class="lbl" x="474" y="700" text-anchor="end">Post Trigger (approved)</text>
<text class="note" x="16" y="653" text-anchor="start">New Run Trigger: Step 1</text>
<text class="note" x="16" y="848" text-anchor="start">Rerun: back to Step 1</text>
<text class="note" x="206" y="527" text-anchor="start">Manual RI run</text>
<text class="note" x="206" y="539" text-anchor="start">succeeded</text>
</svg>
  </section>

  <div class="side">
    <section class="card">
      <h2>Waiting window after RI run completion (by entity)</h2>
      <div class="tablewrap">
        <table>
          <thead><tr><th>Entity</th><th>Expected approval / post timing (AutoSys)</th><th>NiFi waiting window</th></tr></thead>
          <tbody>
            <tr><td>BD2</td><td>AutoSys job at 4:00 PM</td><td>Wait until 4:00 PM</td></tr>
            <tr><td>SJLI</td><td>Auto-approved at 12:00 AM on BD3</td><td>Wait until 4:00 AM</td></tr>
            <tr><td>SLTT</td><td>Approval timing to be confirmed</td><td>Wait until 4:00 AM</td></tr>
            <tr><td>SIBL</td><td>Auto-approved at 12:00 PM (noon) on BD3</td><td>Wait until 4:00 PM</td></tr>
            <tr><td>SCRE</td><td>Auto-approved at 12:00 PM (noon) on BD3</td><td>Wait until 4:00 PM</td></tr>
            <tr><td>BNS</td><td>Approval timing to be confirmed</td><td>Wait until 8:00 PM on BD3</td></tr>
          </tbody>
        </table>
      </div>
      <p class="note-box"><strong>Note:</strong> The waiting window is the expected time for NiFi to receive the next trigger (Post or Run) after the RI run completes successfully.</p>
    </section>

    <section class="card">
      <h2>Key points</h2>
      <ol class="kp">
        <li>RIWMA places a Run Trigger file: <code>{entity}_actuarial_ri_trig_{FYYYY}_{PMM}.csv</code></li>
        <li>NiFi reads the trigger file, uploads the required files to RI, and triggers the RI run.</li>
        <li>If the RI run fails, raise an incident and investigate the issue.
          <ul>
            <li><strong>File issue:</strong>
              <ul>
                <li>Reopen RIWMA UI for the user to update the file(s).</li>
                <li>A new Run Trigger will be sent by RIWMA, which returns to Step 1.</li>
              </ul>
            </li>
            <li><strong>Non-file related issue:</strong>
              <ul>
                <li>Investigate and resolve.</li>
                <li>Manually run RI if required.</li>
                <li>If the manual RI run completes successfully, NiFi manually places the same success trigger file in riwma/incoming, and the process continues from Step 5.</li>
              </ul>
            </li>
          </ul>
        </li>
        <li>When the RI run completes successfully (normal or manual), NiFi places the RI Run Success Trigger file in riwma/incoming: <code>RI_NiFi_{entity}_{FYYYY}_{PMM}_{projectId}_{completedTime}_RIRunSuccess.txt</code></li>
        <li>NiFi waits for the next trigger within the defined waiting window. The next trigger can be either:
          <ul>
            <li>Post Trigger: trigger posting to Subledger / CGL (approved result).</li>
            <li>Run Trigger: rerun RI. Normal process, no incident / ticket required.</li>
          </ul>
        </li>
        <li>Once the Post Trigger is processed, posting to Subledger / CGL is triggered and the process completes (or continues with downstream AutoSys jobs).</li>
      </ol>
    </section>

    <section class="card">
      <h2>Legend</h2>
      <div class="legend">
        <div style="--c:var(--trig)"><span class="sw"></span>Trigger file</div>
        <div style="--c:var(--sys)"><span class="sw"></span>System action</div>
        <div style="--c:var(--wait)"><span class="sw"></span>Waiting</div>
        <div style="--c:var(--data)"><span class="sw"></span>Posting / data store</div>
        <div style="--c:var(--inc)"><span class="sw"></span>Incident</div>
        <div style="--c:var(--dec)"><span class="sw dia"></span>Decision</div>
        <div><span class="ln"></span>Process flow</div>
        <div><span class="ln d"></span>Return to Step 1 (normal rerun)</div>
      </div>
    </section>
  </div>
</div>
</body>
</html>

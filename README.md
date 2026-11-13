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
  --bg:#F2F4F7; --panel:#FFFFFF; --ink:#16233B; --muted:#5B6980; --line:#A9B4C4;
  --accent:#E0861A; --chip:#E4E9F0;
  --trig:#6B57A8; --sys:#2F5DA8; --inc:#C2412D; --wait:#2B7A78; --data:#8A6A1F;
  color-scheme:light;
}
@media (prefers-color-scheme: dark){
  :root:not([data-theme="light"]){
    --bg:#0F1724; --panel:#17233A; --ink:#E6ECF5; --muted:#94A3B8; --line:#41536F;
    --accent:#F2A33A; --chip:#223250;
    --trig:#A895E6; --sys:#7FA6E8; --inc:#F08A78; --wait:#6CC3BF; --data:#D8B25C;
    color-scheme:dark;
  }
}
:root[data-theme="dark"]{
  --bg:#0F1724; --panel:#17233A; --ink:#E6ECF5; --muted:#94A3B8; --line:#41536F;
  --accent:#F2A33A; --chip:#223250;
  --trig:#A895E6; --sys:#7FA6E8; --inc:#F08A78; --wait:#6CC3BF; --data:#D8B25C;
  color-scheme:dark;
}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--ink);
  font:15px/1.5 "Avenir Next","PingFang SC","Hiragino Sans GB","Microsoft YaHei",system-ui,sans-serif}
main{max-width:720px;margin:0 auto;padding:0 12px}
header{padding:18px 4px 6px}
h1{margin:0;font-size:22px;letter-spacing:.2px}
header p{margin:4px 0 0;color:var(--muted);font-size:14px}
.chips{display:flex;gap:8px;overflow-x:auto;padding:10px 4px 12px;scrollbar-width:none}
.chips::-webkit-scrollbar{display:none}
.chip{flex:none;border:1px solid var(--line);background:var(--chip);color:var(--ink);
  border-radius:999px;padding:7px 14px;font:inherit;font-size:14px;cursor:pointer}
.chip[aria-pressed="true"]{background:var(--ink);color:var(--bg);border-color:var(--ink)}
button:focus-visible,select:focus-visible{outline:3px solid var(--accent);outline-offset:2px}
.stage{background:var(--panel);border:1px solid var(--line);border-radius:10px;padding:8px 4px}
svg{display:block;width:100%;height:auto}
svg text{font-family:inherit;fill:var(--ink)}
.node rect.body{fill:var(--panel);stroke:var(--line);stroke-width:1.5}
.node .bar{fill:var(--c)}
.node .t{font-size:13px;font-weight:700}
.node .s{font-size:10.5px;fill:var(--muted)}
.node polygon{fill:var(--panel);stroke:var(--line);stroke-width:1.5}
.node.visited rect.body,.node.visited polygon{stroke:var(--c)}
.node.active rect.body,.node.active polygon{stroke:var(--accent);stroke-width:3;
  fill:color-mix(in srgb,var(--accent) 14%,var(--panel))}
.edge{fill:none;stroke:var(--line);stroke-width:2;marker-end:url(#ah)}
.edge.back{stroke-dasharray:5 4}
.edge.done{stroke:var(--accent);marker-end:url(#ahl)}
.flow{fill:none;stroke:var(--accent);stroke-width:3;stroke-linecap:round}
.lbl{font-size:11px;font-weight:700}
.lbl.no{fill:var(--inc)} .lbl.yes{fill:var(--wait)}
.note{font-size:10.5px;fill:var(--muted)}
.halo{fill:var(--accent);opacity:.28}
.dot{fill:var(--accent);stroke:var(--panel);stroke-width:2}
h2{font-size:16px;margin:22px 4px 8px}
.tablewrap{overflow-x:auto;background:var(--panel);border:1px solid var(--line);border-radius:10px}
table{border-collapse:collapse;width:100%;font-size:13.5px}
th,td{padding:8px 10px;text-align:left;border-bottom:1px solid var(--line)}
tr:last-child td{border-bottom:0}
th{color:var(--muted);font-weight:600}
td:first-child{font-weight:700}
.foot{color:var(--muted);font-size:13px;margin:8px 4px 90px}
.bar-ui{position:sticky;bottom:0;z-index:5;background:var(--panel);border-top:1px solid var(--line);
  padding:10px 14px calc(10px + env(safe-area-inset-bottom,0px));margin:0 -12px}
.bar-ui .inner{max-width:720px;margin:0 auto}
#cap{margin:0 0 8px;font-size:14.5px;min-height:44px}
.ctl{display:flex;gap:8px;align-items:center}
.ctl button,.ctl select{font:inherit;font-size:14px;border:1px solid var(--line);background:var(--chip);
  color:var(--ink);border-radius:8px;padding:7px 12px;cursor:pointer}
.ctl .primary{background:var(--accent);border-color:var(--accent);color:#1b1200;font-weight:700}
#prog{margin-left:auto;color:var(--muted);font-size:13px}
</style>
</head>
<body>
<main>
  <header>
    <h1>RI Process Flow</h1>
    <p>NiFi / AutoSys: from RI trigger to posting (Subledger / CGL)</p>
  </header>

  <div class="chips" id="chips" role="group" aria-label="Choose a scenario"></div>

  <div class="stage">
    <svg id="flow" viewBox="0 0 490 970" role="img" aria-label="RI process flow diagram">
      <defs>
        <marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M0 0L10 5L0 10z" style="fill:var(--line)"/>
        </marker>
        <marker id="ahl" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="7" markerHeight="7" orient="auto">
          <path d="M0 0L10 5L0 10z" style="fill:var(--accent)"/>
        </marker>
      </defs>
    </svg>
  </div>

  <h2>Waiting window after RI run completion, by entity</h2>
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
  <p class="foot">The waiting window is the expected time for NiFi to receive the next trigger (Post or Run) after the RI run completes successfully.</p>
</main>

<div class="bar-ui">
  <div class="inner">
    <p id="cap" aria-live="polite">Pick a scenario, then press Play.</p>
    <div class="ctl">
      <button class="primary" id="play">Play</button>
      <button id="next">Next step</button>
      <button id="restart">Restart</button>
      <select id="speed" aria-label="Speed">
        <option value="0.5">0.5x</option>
        <option value="1" selected>1x</option>
        <option value="2">2x</option>
      </select>
      <span id="prog"></span>
    </div>
  </div>
</div>

<script>
(function(){
const NS='http://www.w3.org/2000/svg';
const reduced=matchMedia('(prefers-reduced-motion: reduce)').matches;
const svg=document.getElementById('flow');
function el(n,a,p){const e=document.createElementNS(NS,n);for(const k in (a||{}))e.setAttribute(k,a[k]);if(p)p.appendChild(e);return e;}

/* ---------- 1. 架构配置：节点 ---------- */
const NODES=[
 {id:'n1',x:230,y:20,w:240,h:50,c:'trig',t:'1. RIWMA places Run Trigger',s:['{entity}_actuarial_ri_trig_…csv']},
 {id:'n2',x:230,y:100,w:240,h:50,c:'sys',t:'2. NiFi reads the trigger file',s:['detects and reads the Run Trigger']},
 {id:'n3',x:230,y:180,w:240,h:50,c:'sys',t:'3. NiFi uploads files to RI',s:['uploads the required files']},
 {id:'n4',x:230,y:260,w:240,h:50,c:'sys',t:'4. NiFi triggers the RI run',s:['for the specified entity, FYYYY, PMM']},
 {id:'d1',kind:'dia',cx:350,cy:365,hw:80,hh:35,l:['Did RI run','succeed?']},
 {id:'n5',x:230,y:440,w:240,h:50,c:'trig',t:'5. NiFi places Success Trigger',s:['RI_NiFi_…_RIRunSuccess.txt to riwma/incoming']},
 {id:'n6',x:230,y:520,w:240,h:50,c:'wait',t:'6. Wait for next trigger',s:['within the defined waiting window']},
 {id:'d2',kind:'dia',cx:350,cy:640,hw:80,hh:35,l:['Which trigger','is received?']},
 {id:'n7a',x:60,y:720,w:220,h:80,c:'trig',t:'7a. Process Run Trigger',s:['Reads file, uploads to RI','Triggers the RI run again','Normal rerun, no incident']},
 {id:'n7b',x:290,y:720,w:180,h:64,c:'trig',t:'7b. Process Post Trigger',s:['Triggers posting','to Subledger / CGL']},
 {id:'post',x:290,y:820,w:180,h:44,c:'data',t:'Post to Subledger / CGL',s:[]},
 {id:'done',x:290,y:904,w:180,h:44,c:'sys',t:'Process Completed',s:[]},
 {id:'fail',x:40,y:400,w:160,h:56,c:'inc',t:'RI Run Failed',s:['Raise an incident','Investigate root cause']},
 {id:'nonfile',x:40,y:480,w:160,h:56,c:'sys',t:'Non-file issue',s:['Investigate and resolve','Manual RI run if required']},
 {id:'file',x:40,y:556,w:160,h:56,c:'trig',t:'File issue',s:['Reopen RIWMA UI, update files','New Run Trigger to Step 1']}
];

/* ---------- 2. 连线 ---------- */
const EDGES=[
 ['e_12','M350,70 V100'],['e_23','M350,150 V180'],['e_34','M350,230 V260'],['e_4d','M350,310 V330'],
 ['e_yes','M350,400 V440'],
 ['e_no','M270,365 H120 V400'],
 ['e_ff','M40,428 H24 V584 H40'],
 ['e_fn','M120,456 V480'],
 ['e_ns','M200,508 H215 V465 H230'],
 ['e_fb','M120,612 V636 H8 V45 H230','back'],
 ['e_56','M350,490 V520'],['e_6d','M350,570 V605'],
 ['e_run','M270,640 H170 V720'],
 ['e_post','M430,640 H450 V720'],
 ['e_rerun','M170,800 V830 H8 V45 H230','back'],
 ['e_7b','M380,784 V820'],['e_pd','M380,864 V904']
];

/* ---------- 3. 说明文字 ---------- */
const CAP={
 n1:'1. RIWMA places the Run Trigger file: {entity}_actuarial_ri_trig_{FYYYY}_{PMM}.csv',
 n1b:'Back to Step 1: a new Run Trigger from RIWMA starts a new cycle.',
 n2:'2. NiFi detects and reads the trigger file.',
 n3:'3. NiFi uploads the required files to RI.',
 n4:'4. NiFi triggers the RI run for the specified entity, FYYYY, PMM.',
 d1:'Decision: did the RI run complete successfully?',
 n5:'5. After a successful RI run (including a manual run), NiFi places the RI Run Success Trigger file in riwma/incoming.',
 n6:'6. NiFi waits within the defined window for the next trigger: either a Post Trigger or a Run Trigger.',
 d2:'Decision: which trigger is received?',
 n7a:'7a. Run Trigger (rerun): NiFi reads the file, uploads it, and triggers the RI run again. Normal rerun, no incident / ticket required.',
 n7b:'7b. Post Trigger (approved): NiFi reads it and triggers posting.',
 post:'Posting to Subledger / CGL.',
 done:'Process completed, or continues with downstream AutoSys jobs.',
 fail:'RI run failed: raise an incident and investigate the root cause (file issue or non-file related issue).',
 file:'File issue: reopen the RIWMA UI so the user can update the file(s).',
 nonfile:'Non-file issue: investigate and resolve, and manually run RI if required. When the manual run succeeds, NiFi places the same success trigger file.'
};
const ECAP={
 e_yes:'Yes: the RI run completed successfully.',
 e_no:'No: the RI run failed.',
 e_run:'Run Trigger received: rerun requested.',
 e_post:'Post Trigger received: approved.',
 e_rerun:'Normal rerun, no incident. Back to Step 1.',
 e_fb:'A new Run Trigger will be sent by RIWMA. Back to Step 1.',
 e_ns:'Manual RI run completed successfully. The process continues from Step 5.'
};

/* ---------- 4. 场景 = 经过的节点和连线序列 ---------- */
const HEAD=['n1','e_12','n2','e_23','n3','e_34','n4','e_4d','d1'];
const TAIL_OK=['e_yes','n5','e_56','n6','e_6d','d2'];
const POST=['e_post','n7b','e_7b','post','e_pd','done'];
const SCEN=[
 {k:'ok',name:'Normal success',ev:[...HEAD,...TAIL_OK,...POST]},
 {k:'rerun',name:'Rerun',ev:[...HEAD,...TAIL_OK,'e_run','n7a','e_rerun','n1b']},
 {k:'file',name:'Failed: file issue',ev:[...HEAD,'e_no','fail','e_ff','file','e_fb','n1b']},
 {k:'nonfile',name:'Failed: non-file issue',ev:[...HEAD,'e_no','fail','e_fn','nonfile','e_ns','n5','e_56','n6','e_6d','d2',...POST]}
];
/* e_fb 的路径和 e_fb 说明共用：把 e_ff 之后的返回线标为 e_fb */

/* ---------- 绘制 ---------- */
const gE=el('g',{},svg), gN=el('g',{},svg), gL=el('g',{},svg);
const edgeMap={}, nodeMap={}, nodeCenter={};
EDGES.forEach(([id,d,cls])=>{
  const p=el('path',{d,id,class:'edge '+(cls||'')},gE);
  const ov=el('path',{d,class:'flow'},gE);
  const len=p.getTotalLength();
  ov.style.strokeDasharray=len; ov.style.strokeDashoffset=len;
  edgeMap[id]={p,ov,len};
});
NODES.forEach(n=>{
  const g=el('g',{class:'node',id:n.id},gN);
  g.style.setProperty('--c',`var(--${n.c||'sys'})`);
  if(n.kind==='dia'){
    el('polygon',{points:`${n.cx},${n.cy-n.hh} ${n.cx+n.hw},${n.cy} ${n.cx},${n.cy+n.hh} ${n.cx-n.hw},${n.cy}`},g);
    g.style.setProperty('--c','var(--inc)');
    n.l.forEach((t,i)=>{const tx=el('text',{x:n.cx,y:n.cy-2+i*15,'text-anchor':'middle',class:'s'},g);
      tx.style.fill='var(--ink)'; tx.style.fontSize='11.5px'; tx.style.fontWeight='700'; tx.textContent=t;});
    nodeCenter[n.id]=[n.cx,n.cy];
  }else{
    el('rect',{x:n.x,y:n.y,width:n.w,height:n.h,rx:7,class:'body'},g);
    el('rect',{x:n.x,y:n.y+7,width:4,height:n.h-14,rx:2,class:'bar'},g);
    el('text',{x:n.x+16,y:n.y+19,class:'t'},g).textContent=n.t;
    n.s.forEach((t,i)=>{el('text',{x:n.x+16,y:n.y+35+i*14.5,class:'s'},g).textContent=t;});
    nodeCenter[n.id]=[n.x+n.w/2,n.y+n.h/2];
  }
  nodeMap[n.id]=g;
});
function lab(x,y,t,cls,anchor){const e=el('text',{x,y,class:'lbl '+cls,'text-anchor':anchor||'start'},gL);e.textContent=t;}
lab(358,422,'Yes','yes'); lab(262,357,'No','no','end');
lab(164,700,'Run Trigger','',"end"); lab(444,700,'Post Trigger','','end');
el('text',{x:16,y:656,class:'note'},gL).textContent='New Run Trigger: back to Step 1';
el('text',{x:16,y:822,class:'note'},gL).textContent='Rerun: back to Step 1';
el('text',{x:222,y:500,class:'note','text-anchor':'start'},gL).textContent='';
const token=el('g',{opacity:0},svg);
el('circle',{r:12,class:'halo'},token); el('circle',{r:6,class:'dot'},token);

/* ---------- 引擎 ---------- */
const capEl=document.getElementById('cap'), progEl=document.getElementById('prog');
const playBtn=document.getElementById('play'), nextBtn=document.getElementById('next');
const restartBtn=document.getElementById('restart'), speedSel=document.getElementById('speed');
const chipsEl=document.getElementById('chips'), barEl=document.querySelector('.bar-ui');
let gen=0, idx=0, playing=false, busy=false, events=SCEN[0].ev, speed=1, tokenShown=false;

const sleep=ms=>new Promise(r=>setTimeout(r,ms));
function setCap(t){capEl.textContent=t||'';}
function setProg(){progEl.textContent=events.length?`${Math.min(idx+1,events.length)} / ${events.length}`:'';}
function setPlayLabel(){playBtn.textContent=playing?'Pause':(idx>=events.length&&idx>0?'Play again':'Play');}

function inView(r,pad){const top=pad, bottom=innerHeight-barEl.offsetHeight-pad;return r.top>=top&&r.bottom<=bottom;}
function reveal(node){
  const r=node.getBoundingClientRect();
  if(inView(r,70)) return;
  const mid=(r.top+r.bottom)/2, want=(70+innerHeight-barEl.offsetHeight-70)/2;
  scrollBy({top:mid-want,behavior:reduced?'auto':'smooth'});
}

function resetVisuals(){
  gen++; idx=0; busy=false; playing=false; tokenShown=false;
  token.setAttribute('opacity',0);
  document.querySelectorAll('.node').forEach(n=>n.classList.remove('active','visited'));
  Object.values(edgeMap).forEach(({p,ov,len})=>{p.classList.remove('done');ov.style.strokeDashoffset=len;});
  setCap('Pick a scenario and press Play, or use Next step to go one step at a time.'); setProg(); setPlayLabel();
}

function travel(id,my){
  return new Promise(res=>{
    const {p,len}=edgeMap[id], ov=edgeMap[id].ov;
    const dur=reduced?250:Math.max(380,len/(230*speed)*1000);
    const t0=performance.now(); let lastRev=0;
    token.setAttribute('opacity',1); tokenShown=true;
    (function f(now){
      if(my!==gen) return res();
      const k=Math.min(1,(now-t0)/dur);
      const pt=p.getPointAtLength(len*k);
      token.setAttribute('transform',`translate(${pt.x},${pt.y})`);
      ov.style.strokeDashoffset=len*(1-k);
      if(now-lastRev>300){lastRev=now;reveal(token);}
      if(k<1) requestAnimationFrame(f); else {p.classList.add('done');res();}
    })(t0);
  });
}
async function arrive(id,my){
  const key=id==='n1b'?'n1':id, g=nodeMap[key];
  document.querySelectorAll('.node.active').forEach(n=>{n.classList.remove('active');n.classList.add('visited');});
  g.classList.add('active');
  setCap(CAP[id]);
  if(!tokenShown){const c=nodeCenter[key];token.setAttribute('transform',`translate(${c[0]},${c[1]})`);token.setAttribute('opacity',1);tokenShown=true;}
  reveal(g);
  const isDia=id==='d1'||id==='d2';
  await sleep((reduced?350:(isDia?1000:750))/speed);
}
async function exec(ev,my){
  setProg();
  if(ev.startsWith('e_')){
    const eid=(ev==='e_ff')?'e_ff':ev;
    const msg=ECAP[ev==='e_ff'?'e_ff':ev];
    if(msg) setCap(msg);
    await travel(eid,my);
  }else{
    await arrive(ev,my);
  }
}
async function loop(){
  if(busy) return; busy=true; const my=gen;
  while(playing && idx<events.length){
    await exec(events[idx],my);
    if(my!==gen){return;}
    idx++;
  }
  busy=false; playing=false; setProg(); setPlayLabel();
}
async function step(){
  if(busy) return;
  if(idx>=events.length){resetVisuals();}
  playing=false; busy=true; const my=gen;
  await exec(events[idx],my);
  if(my!==gen) return;
  idx++; busy=false; setPlayLabel();
}
function selectScenario(i){
  events=SCEN[i].ev;
  [...chipsEl.children].forEach((c,j)=>c.setAttribute('aria-pressed',String(i===j)));
  resetVisuals();
}

/* e_ff 的说明：用一个专门的提示 */
ECAP.e_ff='File issue branch.';

SCEN.forEach((s,i)=>{
  const b=document.createElement('button'); b.className='chip'; b.textContent=s.name;
  b.setAttribute('aria-pressed',i===0?'true':'false');
  b.addEventListener('click',()=>selectScenario(i));
  chipsEl.appendChild(b);
});
playBtn.addEventListener('click',()=>{
  if(playing){playing=false;setPlayLabel();return;}
  if(!busy && idx>=events.length){resetVisuals();}
  playing=true; setPlayLabel(); loop();
});
nextBtn.addEventListener('click',step);
restartBtn.addEventListener('click',resetVisuals);
speedSel.addEventListener('change',()=>{speed=parseFloat(speedSel.value);});
resetVisuals();
})();
</script>
</body>
</html>

#pragma once
// ============================================================================
//  Web pages (stored in flash, fully self-contained: no CDN, works offline)
//    PORTAL_HTML : WiFi setup page shown by the captive portal
//    APP_HTML    : the phone app (tasks, time/date, settings)
// ============================================================================

<img width="798" height="906" alt="oled-preview (1)" src="https://github.com/user-attachments/assets/53ef4d84-e96f-4925-b0c6-6859a205e985" />

#include <pgmspace.h>

static const char PORTAL_HTML[] PROGMEM = R"HTML(<!doctype html>
<html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<meta name="theme-color" content="#070b1e">
<title>BeeClock Setup</title>
<style>
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html,body{margin:0}
body{min-height:100vh;color:#eef1ff;font:16px/1.45 system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;background:radial-gradient(800px 500px at 90% -10%,#34208a77,transparent 70%),linear-gradient(180deg,#060a1c,#0a0f2e 55%,#140b33);background-attachment:fixed;overflow-x:hidden}
#sky{position:fixed;inset:0;pointer-events:none;overflow:hidden}
.st{position:absolute;top:-10px;border-radius:50%;background:#fff;animation:fall linear infinite}
@keyframes fall{to{transform:translateY(112vh)}}
.moon{position:fixed;top:14px;right:16px;width:54px;height:54px;border-radius:50%;box-shadow:inset 14px -6px 0 0 #ffe8a3,0 0 36px 4px rgba(255,220,120,.3)}
.wrap{position:relative;max-width:460px;margin:0 auto;padding:22px 16px 40px}
h1{margin:6px 0 2px;font-size:28px}h1 b{color:#ffc83d}
p.m{color:#9aa6d6;margin:0 0 14px}
.card{background:rgba(255,255,255,.07);border:1px solid rgba(255,255,255,.14);border-radius:20px;padding:14px;margin:12px 0;backdrop-filter:blur(12px)}
.net{display:flex;align-items:center;gap:10px;width:100%;text-align:left;background:rgba(255,255,255,.05);border:1px solid rgba(255,255,255,.1);color:inherit;border-radius:14px;padding:12px;margin:6px 0;font:inherit}
.net.on{border-color:#ffc83d;background:rgba(255,200,61,.12)}
.net span{margin-left:auto;color:#9aa6d6;font-size:13px}
input{width:100%;font:inherit;color:#fff;background:rgba(0,0,0,.3);border:1px solid rgba(255,255,255,.18);border-radius:12px;padding:12px;margin:6px 0}
.btn{width:100%;border:0;border-radius:14px;padding:14px;font:700 16px system-ui;color:#2a1a00;background:linear-gradient(90deg,#ffe28a,#ffb43d);margin-top:6px}
.btn.g{background:rgba(255,255,255,.1);color:#eef1ff;font-weight:600}
.msg{margin:10px 0;padding:12px;border-radius:12px;background:rgba(255,255,255,.08);display:none}
.msg.ok{background:rgba(53,232,162,.15);display:block}.msg.bad{background:rgba(255,95,126,.18);display:block}.msg.go{display:block}
</style></head><body>
<div id="sky"></div><div class="moon"></div>
<div class="wrap">
<h1>&#128029; Bee<b>Clock</b></h1>
<p class="m">Pick your home WiFi. Takes 10 seconds.</p>
<div class="card">
<div id="nets"><div class="m">Scanning WiFi...</div></div>
<button class="btn g" id="rescan">Rescan</button>
<input id="ssid" placeholder="WiFi name" autocapitalize="none" autocomplete="off">
<input id="pass" type="password" placeholder="WiFi password">
<button class="btn" id="go">Connect</button>
<div class="msg" id="msg"></div>
</div>
<div class="card"><p class="m" style="margin:0 0 8px">No router here? Use the clock directly through this hotspot.</p>
<button class="btn g" id="off">Skip - use offline</button></div>
</div>
<script>
const $=s=>document.querySelector(s);
const sky=$('#sky');for(let i=0;i<40;i++){const s=document.createElement('i');s.className='st';const z=Math.random(),sz=1+z*2;s.style.cssText='left:'+Math.random()*100+'%;width:'+sz+'px;height:'+sz+'px;opacity:'+(.3+z*.6)+';animation-duration:'+(14-z*8+Math.random()*6)+'s;animation-delay:-'+Math.random()*20+'s';sky.appendChild(s)}
async function j(p,b){try{const r=await fetch(p,b===undefined?{cache:'no-store'}:{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(b)});return await r.json()}catch(e){return null}}
function bars(r){return r>-55?'&#9608;&#9608;&#9608;&#9608;':r>-65?'&#9608;&#9608;&#9608;':r>-75?'&#9608;&#9608;':'&#9608;'}
async function scan(re){$('#nets').innerHTML='<div class="m">Scanning WiFi...</div>';const l=await j('/scan'+(re?'?refresh=1':''))||[];
 $('#nets').innerHTML=l.length?'':'<div class="m">No networks found. Type the name below.</div>';
 l.forEach(n=>{const b=document.createElement('button');b.className='net';b.innerHTML='<b></b><span>'+(n.l?'&#128274; ':'')+bars(n.r)+'</span>';b.firstChild.textContent=n.s;b.onclick=()=>{$('#ssid').value=n.s;document.querySelectorAll('.net').forEach(x=>x.classList.remove('on'));b.classList.add('on');$('#pass').focus()};$('#nets').appendChild(b)})}
$('#rescan').onclick=()=>scan(true);
function msg(t,c){const m=$('#msg');m.className='msg '+c;m.innerHTML=t}
$('#go').onclick=async()=>{const ssid=$('#ssid').value.trim();if(!ssid){msg('Choose or type a WiFi name','bad');return}
 msg('Connecting to <b></b> ...','go');$('#msg b').textContent=ssid;$('#go').disabled=true;
 const r=await j('/save',{ssid,pass:$('#pass').value});if(!r||!r.ok){msg('Could not send. Try again.','bad');$('#go').disabled=false;return}
 const t0=Date.now();const iv=setInterval(async()=>{const s=await j('/status');
  if(s&&s.state=='ok'){clearInterval(iv);msg('&#9989; Connected!<br>Now join your home WiFi on this phone, then open:<br><b>http://'+s.host+'</b><br>or <b>http://'+s.ip+'</b><br>(the clock screen shows the address too)','ok')}
  else if(s&&s.state=='fail'){clearInterval(iv);msg('Could not connect. Check the password and try again.','bad');$('#go').disabled=false}
  else if(Date.now()-t0>40000){clearInterval(iv);msg('If the clock screen says ONLINE, the setup worked. Rejoin your home WiFi and open the address shown on the screen.','ok');$('#go').disabled=false}
 },1500)};
$('#off').onclick=async()=>{await j('/offline',{});location.href='/'};
scan(false);
</script></body></html>)HTML";

static const char APP_HTML[] PROGMEM = R"HTML(<!doctype html>
<html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<meta name="theme-color" content="#070b1e">
<title>BeeClock</title>
<style>
:root{--card:rgba(255,255,255,.07);--line:rgba(255,255,255,.13);--txt:#eef1ff;--mut:#9aa6d6;--gold:#ffc83d;--ok:#35e8a2;--bad:#ff5f7e}
*{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
html,body{margin:0}
body{min-height:100vh;color:var(--txt);font:15px/1.45 system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;background:radial-gradient(900px 600px at 88% -10%,#34208a77,transparent 70%),radial-gradient(800px 600px at -10% 35%,#0b3f8a55,transparent 70%),linear-gradient(180deg,#060a1c,#0a0f2e 55%,#140b33);background-attachment:fixed;overflow-x:hidden}
#sky{position:fixed;inset:0;pointer-events:none;overflow:hidden;z-index:0}
.st{position:absolute;top:-10px;border-radius:50%;background:#fff;animation:fall linear infinite}
@keyframes fall{to{transform:translateY(112vh)}}
.met{position:absolute;top:-90px;width:2px;height:80px;background:linear-gradient(to top,#fff,transparent);opacity:0;transform:rotate(16deg);animation:met 10s linear infinite}
@keyframes met{0%{opacity:0;transform:translate(0,0) rotate(16deg)}3%{opacity:.9}20%{opacity:0;transform:translate(-300px,115vh) rotate(16deg)}100%{opacity:0}}
.moon{position:fixed;top:14px;right:16px;width:58px;height:58px;border-radius:50%;z-index:0;box-shadow:inset 15px -7px 0 0 #ffe8a3,0 0 38px 4px rgba(255,220,120,.28);animation:glow 6s ease-in-out infinite alternate}
@keyframes glow{to{box-shadow:inset 15px -7px 0 0 #fff3c4,0 0 58px 10px rgba(255,220,120,.42)}}
.bee{position:fixed;left:0;top:34vh;width:52px;height:36px;z-index:3;pointer-events:none;animation:fly 24s linear infinite}
@keyframes fly{0%{transform:translate(-80px,0) scaleX(1)}48%{transform:translate(calc(100vw + 30px),-40px) scaleX(1)}50%{transform:translate(calc(100vw + 30px),-40px) scaleX(-1)}98%{transform:translate(-80px,30px) scaleX(-1)}100%{transform:translate(-80px,30px) scaleX(1)}}
.bob{animation:bob 1.3s ease-in-out infinite alternate}
@keyframes bob{to{transform:translateY(-9px)}}
.wing{transform-box:fill-box;transform-origin:50% 100%;animation:flap .11s ease-in-out infinite alternate}
@keyframes flap{from{transform:scaleY(1)}to{transform:scaleY(.3)}}
.wrap{position:relative;z-index:1;max-width:720px;margin:0 auto;padding:max(14px,env(safe-area-inset-top)) 14px 90px}
header{display:flex;align-items:center;gap:10px;padding:6px 70px 6px 2px}
header h1{font-size:22px;margin:0}header h1 b{color:var(--gold)}
.pills{display:flex;gap:6px;flex-wrap:wrap}
.pill{font-size:12px;padding:3px 9px;border-radius:99px;background:rgba(255,255,255,.1);color:var(--mut);white-space:nowrap}
.pill.ok{background:rgba(53,232,162,.18);color:var(--ok)}.pill.bad{background:rgba(255,95,126,.2);color:var(--bad)}
.card{background:var(--card);border:1px solid var(--line);border-radius:22px;padding:16px;margin:14px 0;backdrop-filter:blur(14px);-webkit-backdrop-filter:blur(14px)}
.card h2,summary{font-size:16px;margin:0 0 10px;font-weight:700}
summary{cursor:pointer;list-style:none;margin:0}summary::-webkit-details-marker{display:none}
details[open] summary{margin-bottom:12px}
.hero{text-align:center;padding:20px 12px 16px}
.clk{display:inline-block;font-size:clamp(54px,18vw,92px);font-weight:800;letter-spacing:-2px;line-height:1;background:linear-gradient(90deg,#ffe28a,#ffb43d);-webkit-background-clip:text;background-clip:text;color:transparent;font-variant-numeric:tabular-nums}
.sec{display:inline-block;font-size:22px;color:var(--mut);margin-left:6px;font-variant-numeric:tabular-nums}
.date{color:var(--mut);margin-top:4px}
.prog{height:8px;border-radius:9px;background:rgba(255,255,255,.1);overflow:hidden;margin:14px 6px 6px}
.prog i{display:block;height:100%;width:0;background:linear-gradient(90deg,#6ee7b7,#35e8a2);transition:width .6s}
.mut{color:var(--mut);font-size:13px}
input,select,textarea{width:100%;font:inherit;font-size:16px;color:#fff;background:rgba(0,0,0,.28);border:1px solid rgba(255,255,255,.17);border-radius:13px;padding:12px;margin:5px 0;color-scheme:dark}
input[type=range]{padding:0;margin:10px 0}
input[type=checkbox]{width:22px;height:22px;margin:0;accent-color:#ffc83d}
label{display:block;font-size:12px;color:var(--mut);margin-top:6px}
.row{display:flex;gap:10px}.row>*{flex:1;min-width:0}
.btn{border:0;border-radius:14px;padding:13px 16px;font:700 15px system-ui;color:#2a1a00;background:linear-gradient(90deg,#ffe28a,#ffb43d);cursor:pointer}
.btn.w{width:100%;margin-top:8px}
.btn.g{background:rgba(255,255,255,.11);color:var(--txt);font-weight:600}
.btn.r{background:rgba(255,95,126,.2);color:#ffb3c1}
.btn:active{transform:scale(.97)}
.chips{display:flex;gap:6px;flex-wrap:wrap;margin:8px 0}
.chips button{border:1px solid var(--line);background:rgba(255,255,255,.07);color:var(--txt);padding:6px 11px;border-radius:99px;font:13px system-ui}
.seg{display:flex;background:rgba(0,0,0,.28);border-radius:13px;padding:3px;margin:5px 0}
.seg button{flex:1;border:0;background:none;color:var(--mut);padding:9px 2px;border-radius:10px;font:600 13px system-ui;white-space:nowrap}
.seg button.on{background:rgba(255,200,61,.2);color:var(--gold)}
.tabs{display:flex;gap:6px;margin-bottom:8px}
.tab{flex:1;border:0;padding:9px;border-radius:12px;background:rgba(255,255,255,.07);color:var(--mut);font:600 13px system-ui}
.tab.on{background:rgba(255,200,61,.2);color:var(--gold)}
ul{list-style:none;margin:0;padding:0}
.task{display:flex;align-items:center;gap:12px;padding:12px 8px;border-radius:14px;border-bottom:1px solid rgba(255,255,255,.06)}
.task:last-child{border-bottom:0}
.chk{flex:none;width:28px;height:28px;border-radius:50%;border:2px solid rgba(255,255,255,.35);background:none;color:#0b1a12;font:700 16px system-ui;line-height:1}
.task.done .chk{background:var(--ok);border-color:var(--ok)}
.task.p2 .chk{border-color:var(--gold)}
.tb{flex:1;min-width:0}
.tt{font-weight:600;word-break:break-word}
.task.done .tt{text-decoration:line-through;color:var(--mut)}
.tm{display:flex;gap:6px;flex-wrap:wrap;margin-top:3px}
.tm span{font-size:12px;color:var(--mut);background:rgba(255,255,255,.07);padding:1px 8px;border-radius:99px}
.tm .od{background:rgba(255,95,126,.22);color:var(--bad)}.tm .hi{color:var(--gold)}
.x{flex:none;border:0;background:none;color:var(--mut);font-size:18px;padding:6px 8px}
.empty{text-align:center;color:var(--mut);padding:24px 0}
.alarm{border-color:var(--bad);background:rgba(255,95,126,.15);animation:ring .9s ease-in-out infinite alternate}
@keyframes ring{to{box-shadow:0 0 28px rgba(255,95,126,.5)}}
.alarm h2{font-size:20px}
.tog{display:flex;align-items:center;justify-content:space-between;padding:8px 0}
.help{font-size:12.5px;color:var(--mut);line-height:1.5}
.help b{color:var(--txt)}
footer{text-align:center;color:var(--mut);font-size:12px;margin-top:20px}
#toast{position:fixed;left:50%;bottom:24px;transform:translate(-50%,40px);background:#1b2150;border:1px solid var(--line);padding:11px 18px;border-radius:99px;opacity:0;transition:.25s;z-index:9;pointer-events:none}
#toast.show{opacity:1;transform:translate(-50%,0)}#toast.bad{background:#5a1830}
[hidden]{display:none!important}
</style></head><body>
<div id="sky"></div><div class="moon"></div>
<svg class="bee" viewBox="0 0 64 44" aria-hidden="true"><defs><clipPath id="bc"><ellipse cx="32" cy="27" rx="19" ry="12"/></clipPath></defs><g class="bob">
<ellipse class="wing" cx="27" cy="13" rx="8" ry="11" fill="#cfeaff" opacity=".8"/><ellipse class="wing" cx="38" cy="13" rx="8" ry="11" fill="#cfeaff" opacity=".6" style="animation-delay:-.05s"/>
<ellipse cx="32" cy="27" rx="19" ry="12" fill="#ffc83d"/><g clip-path="url(#bc)" fill="#1a1230"><rect x="23" y="12" width="5" height="30"/><rect x="33" y="12" width="5" height="30"/></g>
<circle cx="51" cy="26" r="8" fill="#ffc83d"/><circle cx="54" cy="24" r="1.9" fill="#1a1230"/><path d="M55 19q4-6 8-6M13 28l-9 3" stroke="#1a1230" stroke-width="1.6" fill="none" stroke-linecap="round"/></g></svg>

<div class="wrap">
<header><h1>&#128029; Bee<b>Clock</b></h1><div class="pills"><span class="pill" id="bWifi">WiFi</span><span class="pill" id="bFb">Cloud</span></div></header>

<section class="card hero">
 <div><span class="clk" id="clk">--:--</span><span class="sec" id="sec"></span></div>
 <div class="date" id="dateTxt">Connecting...</div>
 <div class="prog"><i id="progBar"></i></div><div class="mut" id="progTxt">&nbsp;</div>
</section>

<section class="card alarm" id="alarm" hidden>
 <h2>&#9200; <span id="alTitle"></span></h2>
 <div class="row"><button class="btn" id="alSnz">Snooze 5 min</button><button class="btn g" id="alDone">Done &#10003;</button><button class="btn r" id="alStop">Stop</button></div>
</section>

<section class="card">
 <h2>&#10024; New task</h2>
 <input id="title" maxlength="60" placeholder="What do you need to do?" autocomplete="off">
 <div class="row"><div><label>Date (optional)</label><input type="date" id="fDate"></div><div><label>Time (alarm)</label><input type="time" id="fTime"></div></div>
 <div class="chips" id="chips"><button data-q="15">+15 min</button><button data-q="30">+30 min</button><button data-q="60">+1 hour</button><button data-q="eve">Tonight 8pm</button><button data-q="tom">Tomorrow 9am</button><button data-q="clr">Clear</button></div>
 <div class="row"><div><label>Priority</label><div class="seg" id="prio"><button data-p="0">Low</button><button data-p="1" class="on">Normal</button><button data-p="2">High</button></div></div>
 <div><label>Repeat</label><select id="rep"><option value="0">Once</option><option value="1">Every day</option><option value="2">Every week</option></select></div></div>
 <button class="btn w" id="add">Add task</button>
</section>

<section class="card">
 <div class="tabs"><button class="tab on" data-t="today">Today</button><button class="tab" data-t="all">All</button><button class="tab" data-t="done">Done</button></div>
 <ul id="list"></ul>
 <button class="btn g w" id="clearDone" style="margin-top:10px">Remove finished tasks</button>
</section>

<details class="card"><summary>&#128339; Time &amp; date</summary>
 <div class="tog"><span>Auto time from internet (NTP)</span><input type="checkbox" id="ntp"></div>
 <button class="btn w" id="phoneT">Use my phone's time now</button>
 <label>Or set manually</label>
 <input type="datetime-local" id="dt">
 <label>Time zone</label><select id="tz"></select>
 <button class="btn g w" id="applyT">Set this time (turns auto time off)</button>
 <p class="help">The ESP32 has no battery clock. If power is cut, it gets the time again from the internet when WiFi is back, or use the buttons above.</p>
</details>

<details class="card"><summary>&#9729;&#65039; Cloud &amp; display settings</summary>
 <label>Firebase Realtime Database URL</label>
 <input id="fbUrl" placeholder="https://your-project-default-rtdb.firebasedatabase.app" autocapitalize="none" autocomplete="off">
 <div class="row"><div><label>Data path</label><input id="fbPath" autocapitalize="none" autocomplete="off"></div><div><label>Auth secret (optional)</label><input id="fbAuth" type="password" autocomplete="off"></div></div>
 <div class="help" id="fbMsg"></div>
 <label>Clock scene seconds: <b id="cksV"></b></label><input type="range" id="cks" min="3" max="30">
 <label>Seconds per todo page: <b id="lssV"></b></label><input type="range" id="lss" min="3" max="30">
 <label>OLED brightness: <b id="conV"></b></label><input type="range" id="con" min="5" max="255">
 <div class="tog"><span>24 hour clock on OLED</span><input type="checkbox" id="h24"></div>
 <div class="tog"><span>Alarm buzzer</span><input type="checkbox" id="buz"></div>
 <button class="btn w" id="saveS">Save settings</button>
 <div class="row" style="margin-top:8px"><button class="btn g" id="syncNow">Sync now</button><button class="btn g" id="testAl">Test alarm</button></div>
 <button class="btn r w" id="wifiRst">Forget WiFi &amp; restart setup</button>
 <p class="help"><b>Firebase setup:</b> console.firebase.google.com &rarr; create project &rarr; Build &rarr; Realtime Database &rarr; Create. Paste the URL above. Rules: allow read/write only for your data path (see README). Without Firebase everything still works and is saved on the clock.</p>
</details>

<footer id="foot"></footer>
</div>
<div id="toast"></div>

<script>
(()=>{
const $=s=>document.querySelector(s),$$=s=>[...document.querySelectorAll(s)];
const p2=n=>String(n).padStart(2,'0');
const esc=s=>String(s).replace(/[&<>"']/g,c=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
const DAYS=['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'],MON=['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
let S={epoch:0,tz:360,at:Date.now(),tasks:[],r:{}},tab='today',prio=1,first=true;
const sNow=()=>new Date((S.epoch+S.tz*60)*1000+(Date.now()-S.at));
const dS=d=>d.getUTCFullYear()+'-'+p2(d.getUTCMonth()+1)+'-'+p2(d.getUTCDate());
const hmS=d=>p2(d.getUTCHours())+':'+p2(d.getUTCMinutes());
function toast(m,bad){const t=$('#toast');t.textContent=m;t.className='show'+(bad?' bad':'');clearTimeout(toast.h);toast.h=setTimeout(()=>t.className='',2400)}
async function api(p,b){try{const r=await fetch(p,b===undefined?{cache:'no-store'}:{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(b)});return await r.json()}catch(e){return null}}

const occurs=(t,td,wd)=>t.rep==1?true:t.rep==2?(t.date?new Date(t.date+'T12:00:00Z').getUTCDay()==wd:true):(!t.date||t.date==td);
const isDone=(t,td)=>t.done&&(t.rep==0||t.do==td);
const overdue=(t,td,hm,wd)=>{if(isDone(t,td))return false;if(!t.rep&&t.date&&t.date<td)return true;return !!(occurs(t,td,wd)&&t.time&&t.time<hm)};

function item(t){return '<li class="task'+(t._d?' done':'')+' p'+t.prio+'" data-id="'+t.id+'"><button class="chk" data-a="t" aria-label="toggle">'+(t._d?'&#10003;':'')+'</button><div class="tb"><div class="tt">'+esc(t.title)+'</div><div class="tm">'+
 (t.time?'<span>&#9200; '+t.time+'</span>':'')+(t.date&&!t.rep?'<span>&#128197; '+t.date+'</span>':'')+(t.rep?'<span>&#128257; '+(t.rep==1?'daily':'weekly')+'</span>':'')+
 (t.prio==2?'<span class="hi">&#128293; high</span>':t.prio==0?'<span>low</span>':'')+(t._o?'<span class="od">overdue</span>':'')+(t.snz?'<span>&#128164; snoozed</span>':'')+
 '</div></div><button class="x" data-a="d" aria-label="delete">&#10005;</button></li>'}

function render(){
 const n=sNow(),td=dS(n),hm=hmS(n),wd=n.getUTCDay();
 const L=S.tasks.map(t=>Object.assign({},t,{_d:isDone(t,td),_o:overdue(t,td,hm,wd),_t:occurs(t,td,wd)}));
 const tot=L.filter(t=>t._t).length,dn=L.filter(t=>t._t&&t._d).length;
 $('#progBar').style.width=(tot?dn*100/tot:0)+'%';
 $('#progTxt').textContent=tot?(dn+' of '+tot+' done today'+(dn==tot?' 🎉':'')):'No tasks for today';
 let V=tab=='today'?L.filter(t=>t._t||(!t.rep&&t.date&&t.date<td&&!t._d)):tab=='done'?L.filter(t=>t._d):L;
 const k=t=>(t.rep?td:(t.date||'0000-00-00'))+(t.time||'99:99');
 V.sort((a,b)=>((a._d?1:0)-(b._d?1:0))||k(a).localeCompare(k(b)));
 $('#list').innerHTML=V.length?V.map(item).join(''):'<div class="empty">&#127769; Nothing here yet.<br>Add a task above.</div>';
}

function tick(){
 if(!S.r.valid){$('#clk').textContent='--:--';$('#sec').textContent='';$('#dateTxt').textContent='Time not set. Open "Time & date" below.';return}
 const n=sNow();$('#clk').textContent=p2(n.getUTCHours())+':'+p2(n.getUTCMinutes());$('#sec').textContent=p2(n.getUTCSeconds());
 $('#dateTxt').textContent=DAYS[n.getUTCDay()]+', '+n.getUTCDate()+' '+MON[n.getUTCMonth()]+' '+n.getUTCFullYear();
}

function ui(){
 const r=S.r,f=r.fb||{};
 const w=$('#bWifi');w.textContent=r.online?('WiFi '+r.rssi+' dBm'):'Hotspot mode';w.className='pill '+(r.online?'ok':'');
 const b=$('#bFb');
 if(!f.on){b.textContent='Cloud off';b.className='pill';b.title=''}
 else if(f.ok){b.textContent=f.pend?'Saving...':'Cloud synced';b.className='pill ok'}
 else{b.textContent='Cloud error';b.className='pill bad';b.title=f.err||''}
 $('#fbMsg').textContent=f.on?(f.ok?'Connected to Firebase.':(f.err||'Connecting...')):'';
 $('#alarm').hidden=!r.alert;$('#alTitle').textContent=r.alertTitle||'';
 $('#ntp').checked=!!r.ntp;
 $('#foot').textContent='BeeClock v'+r.ver+' · '+r.ip+' · '+Math.round(r.heap/1024)+' KB free · up '+Math.floor(r.up/60)+' min';
 render();tick();
}

async function poll(){
 const r=await api('/api/state');
 if(!r){const w=$('#bWifi');w.textContent='Offline';w.className='pill bad';return}
 S.epoch=r.epoch;S.tz=r.tz;S.at=Date.now();S.tasks=r.tasks;S.r=r;
 if(first){first=false;initForms()}
 ui();
}

async function initForms(){
 $('#tz').value=S.tz;
 const n=sNow();$('#dt').value=dS(n)+'T'+hmS(n);
 const s=await api('/api/settings');
 if(s&&s.ok!==false){
  $('#fbUrl').value=s.fbUrl||'';$('#fbPath').value=s.fbPath||'';$('#fbAuth').value=s.fbAuth||'';
  $('#cks').value=s.clockSec;$('#lss').value=s.listSec;$('#con').value=s.contrast;$('#h24').checked=s.h24;$('#buz').checked=s.buzzer;
  ['cks','lss','con'].forEach(i=>$('#'+i+'V').textContent=$('#'+i).value);
 }
}

// ---- events ----
$('#list').addEventListener('click',async e=>{
 const b=e.target.closest('button');if(!b)return;const id=+b.closest('li').dataset.id;
 if(b.dataset.a=='t')await api('/api/task/toggle',{id});
 else if(b.dataset.a=='d'){if(!confirm('Delete this task?'))return;await api('/api/task/delete',{id})}
 poll()});
$$('.tab').forEach(b=>b.onclick=()=>{tab=b.dataset.t;$$('.tab').forEach(x=>x.classList.toggle('on',x==b));render()});
$$('#prio button').forEach(b=>b.onclick=()=>{prio=+b.dataset.p;$$('#prio button').forEach(x=>x.classList.toggle('on',x==b))});
$$('#chips button').forEach(b=>b.onclick=()=>{
 const q=b.dataset.q,n=sNow();let d;
 if(q=='clr'){$('#fDate').value='';$('#fTime').value='';return}
 if(q=='eve'){d=new Date(n);d.setUTCHours(20,0,0,0)}
 else if(q=='tom'){d=new Date(n.getTime()+864e5);d.setUTCHours(9,0,0,0)}
 else d=new Date(n.getTime()+(+q)*6e4);
 $('#fDate').value=dS(d);$('#fTime').value=hmS(d)});
$('#add').onclick=async()=>{
 const title=$('#title').value.trim();if(!title){toast('Write a task first',1);return}
 const r=await api('/api/task/add',{title,date:$('#fDate').value,time:$('#fTime').value,prio,rep:+$('#rep').value});
 if(r&&r.ok){$('#title').value='';toast('Added 🐝');poll()}else toast((r&&r.err)||'Error',1)};
$('#title').addEventListener('keydown',e=>{if(e.key=='Enter')$('#add').click()});
$('#clearDone').onclick=async()=>{await api('/api/task/clear',{});toast('Cleaned up');poll()};

const tzs=$('#tz');
for(let m=-720;m<=840;m+=30){const a=Math.abs(m),o=document.createElement('option');o.value=m;
 o.textContent='UTC'+(m<0?'-':'+')+p2(Math.floor(a/60))+':'+p2(a%60)+({360:'  Dhaka',330:'  India',0:'  London',60:'  Paris',180:'  Moscow',240:'  Dubai',480:'  Singapore',540:'  Tokyo','-300':'  New York','-480':'  Los Angeles'}[m]||'');tzs.appendChild(o)}
$('#ntp').onchange=async()=>{await api('/api/time',{ntp:$('#ntp').checked});toast($('#ntp').checked?'Auto time on':'Auto time off');poll()};
$('#phoneT').onclick=async()=>{const r=await api('/api/time',{epoch:Math.floor(Date.now()/1000),tz:-new Date().getTimezoneOffset()});toast(r&&r.ok?'Time set from phone ✔':'Failed',!(r&&r.ok));$('#tz').value=-new Date().getTimezoneOffset();poll()};
$('#applyT').onclick=async()=>{
 const v=$('#dt').value;if(!v){toast('Pick date and time',1);return}
 const tz=+$('#tz').value,[d,t]=v.split('T'),[y,m,dd]=d.split('-').map(Number),[h,mi]=t.split(':').map(Number);
 const ep=Date.UTC(y,m-1,dd,h,mi,0)/1000-tz*60;
 const r=await api('/api/time',{epoch:ep,tz,ntp:false});toast(r&&r.ok?'Time set ✔':'Failed',!(r&&r.ok));poll()};

['cks','lss','con'].forEach(i=>$('#'+i).oninput=()=>$('#'+i+'V').textContent=$('#'+i).value);
$('#saveS').onclick=async()=>{
 const r=await api('/api/settings',{fbUrl:$('#fbUrl').value.trim(),fbPath:$('#fbPath').value.trim(),fbAuth:$('#fbAuth').value,clockSec:+$('#cks').value,listSec:+$('#lss').value,contrast:+$('#con').value,h24:$('#h24').checked,buzzer:$('#buz').checked});
 toast(r&&r.ok?'Saved ✔':'Failed',!(r&&r.ok));setTimeout(poll,1500)};
$('#syncNow').onclick=async()=>{await api('/api/sync',{});toast('Syncing...');setTimeout(poll,1500)};
$('#testAl').onclick=async()=>{await api('/api/alarm/test',{});poll()};
$('#wifiRst').onclick=async()=>{if(!confirm('Forget the saved WiFi and restart in setup mode?'))return;await api('/api/wifi/reset',{});toast('Restarting... join the BeeClock hotspot')};
$('#alSnz').onclick=async()=>{await api('/api/alarm/snooze',{min:5});toast('Snoozed 5 min 💤');poll()};
$('#alDone').onclick=async()=>{await api('/api/alarm/done',{});toast('Done ✔');poll()};
$('#alStop').onclick=async()=>{await api('/api/alarm/stop',{});poll()};

// ---- falling stars + meteors ----
const sky=$('#sky');
for(let i=0;i<48;i++){const s=document.createElement('i');s.className='st';const z=Math.random(),sz=1+z*2.2;
 s.style.cssText='left:'+Math.random()*100+'%;width:'+sz+'px;height:'+sz+'px;opacity:'+(.25+z*.6)+';animation-duration:'+(15-z*9+Math.random()*6)+'s;animation-delay:-'+Math.random()*20+'s';sky.appendChild(s)}
for(let i=0;i<3;i++){const m=document.createElement('i');m.className='met';m.style.cssText='left:'+(25+Math.random()*70)+'%;animation-delay:'+(i*3.4+Math.random()*2)+'s';sky.appendChild(m)}

poll();setInterval(poll,2500);setInterval(tick,250);
})();
</script></body></html>)HTML";

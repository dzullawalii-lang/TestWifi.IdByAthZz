# TestWifi.IdByAthZz
TestWifi.id

<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>NETX SPEED TEST</title>
<style>
*{box-sizing:border-box}
body{margin:0;background:#080d18;color:white;font-family:Arial;padding:18px}
main{max-width:850px;margin:auto}
header{text-align:center;padding:20px}
header h1{color:#00e5ff;letter-spacing:3px}
.card{background:#111c2b;border:1px solid #263b50;border-radius:15px;padding:18px;margin:12px 0}
.grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.metric{text-align:center}
.metric h2{font-size:30px;color:#00e5ff}
button{background:#00d9ed;color:#001018;border:0;padding:14px;border-radius:10px;font-weight:bold;width:100%;cursor:pointer}
.secondary{background:#263b50;color:white}
#status{text-align:center;color:#00ff9d;padding:12px}
.info{display:flex;justify-content:space-between;gap:10px;padding:10px 0;border-bottom:1px solid #263b50;font-size:13px}
canvas{width:100%;height:150px}
#history{font-size:13px;line-height:2}
small{color:#9aadc0}
</style>
</head>
<body>
<main>
<header>
<h1>NETX ⚡</h1>
<p>ADVANCED NETWORK DIAGNOSTICS</p>
</header>

<div class="card">
<h3>NETWORK STATUS</h3>
<div id="status">CHECKING...</div>
<div class="info"><span>Connection</span><span id="type">-</span></div>
<div class="info"><span>Browser downlink</span><span id="downlink">-</span></div>
</div>

<div class="card">
<div class="grid">
<div class="metric"><small>DOWNLOAD</small><h2 id="download">--</h2><small>Mbps</small></div>
<div class="metric"><small>UPLOAD</small><h2 id="upload">--</h2><small>Mbps</small></div>
<div class="metric"><small>PING</small><h2 id="ping">--</h2><small>ms</small></div>
<div class="metric"><small>JITTER</small><h2 id="jitter">--</h2><small>ms</small></div>
</div>
<button id="start" onclick="startTest()">START SPEED TEST ⚡</button>
</div>

<div class="card">
<h3>LIVE SPEED GRAPH</h3>
<canvas id="graph"></canvas>
</div>

<div class="card">
<h3>NETWORK INFORMATION</h3>
<div class="info"><span>Public IP</span><span id="ip">-</span></div>
<div class="info"><span>ISP</span><span id="isp">-</span></div>
<div class="info"><span>Location</span><span id="location">-</span></div>
<div class="info"><span>Platform</span><span id="platform">-</span></div>
<div class="info"><span>Online</span><span id="online">-</span></div>
</div>

<div class="card">
<h3>NETWORK TOOLS</h3>
<button onclick="dnsTest()">DNS TEST</button><br><br>
<button onclick="httpsTest()">HTTPS TEST</button><br><br>
<button onclick="stabilityTest()">STABILITY TEST</button>
<p id="toolResult">Ready.</p>
</div>

<div class="card">
<h3>TEST HISTORY</h3>
<div id="history">No history.</div><br>
<button class="secondary" onclick="copyHistory()">COPY HISTORY</button><br><br>
<button class="secondary" onclick="downloadReport()">DOWNLOAD REPORT</button><br><br>
<button class="secondary" onclick="clearHistory()">CLEAR HISTORY</button>
</div>
</main>

<script>
const $=id=>document.getElementById(id);
let historyData=[],graphData=[];

function network(){
$('status').textContent=navigator.onLine?'● ONLINE':'● OFFLINE';
$('online').textContent=navigator.onLine?'ONLINE':'OFFLINE';
let c=navigator.connection;
$('type').textContent=c?.effectiveType||'Unknown';
$('downlink').textContent=c?.downlink?c.downlink+' Mbps':'Unknown';
$('platform').textContent=navigator.platform||'Unknown';
}
window.addEventListener('online',network);
window.addEventListener('offline',network);

async function getIP(){
try{
let r=await fetch('https://ipapi.co/json/');
let d=await r.json();
$('ip').textContent=d.ip||'-';
$('isp').textContent=d.org||'-';
$('location').textContent=(d.city||'-')+', '+(d.country_name||'-');
}catch(e){
$('ip').textContent='Unavailable';
$('isp').textContent='Unavailable';
$('location').textContent='Unavailable';
}
}

function graph(){
let c=$('graph'),ctx=c.getContext('2d');
c.width=c.clientWidth*devicePixelRatio;
c.height=150*devicePixelRatio;
ctx.scale(devicePixelRatio,devicePixelRatio);
ctx.clearRect(0,0,c.clientWidth,150);
ctx.beginPath();
graphData.forEach((v,i)=>{
let x=i*c.clientWidth/Math.max(1,graphData.length-1);
let y=145-Math.min(v,100)*1.3;
i?ctx.lineTo(x,y):ctx.moveTo(x,y);
});
ctx.strokeStyle='#00e5ff';ctx.lineWidth=3;ctx.stroke();
}

async function pingTest(){
let times=[];
for(let i=0;i<6;i++){
let t=performance.now();
let r=await fetch('https://www.cloudflare.com/cdn-cgi/trace?t='+Math.random(),{cache:'no-store'});
if(!r.ok)throw Error('Ping failed');
times.push(performance.now()-t);
}
let avg=times.reduce((a,b)=>a+b,0)/times.length;
let jitter=times.slice(1).reduce((a,b,i)=>a+Math.abs(b-times[i]),0)/(times.length-1);
$('ping').textContent=avg.toFixed(1);
$('jitter').textContent=jitter.toFixed(1);
}

async function downloadTest(){
let total=0,start=performance.now();
for(let size of [2000000,5000000]){
let r=await fetch('https://speed.cloudflare.com/__down?bytes='+size+'&t='+Math.random());
if(!r.ok)throw Error('Download failed');
let data=await r.arrayBuffer();
total+=data.byteLength;
let elapsed=(performance.now()-start)/1000;
let speed=total*8/elapsed/1000000;
graphData.push(speed);graph();
}
return total*8/((performance.now()-start)/1000)/1000000;
}

async function uploadTest(){
let data=new Uint8Array(1000000);
let start=performance.now();
let r=await fetch('https://speed.cloudflare.com/__up',{method:'POST',body:data});
if(!r.ok)throw Error('Upload failed');
return data.length*8/((performance.now()-start)/1000)/1000000;
}

async function startTest(){
let btn=$('start');
btn.disabled=true;
btn.textContent='TESTING...';
graphData=[];
try{
await pingTest();
let d=await downloadTest();
$('download').textContent=d.toFixed(2);
let u=await uploadTest();
$('upload').textContent=u.toFixed(2);
let result=`Download ${d.toFixed(2)} Mbps | Upload ${u.toFixed(2)} Mbps | Ping ${$('ping').textContent} ms | Jitter ${$('jitter').textContent} ms`;
historyData.unshift(new Date().toLocaleString()+' | '+result);
$('history').innerHTML=historyData.map(x=>'<div>'+x+'</div>').join('');
$('status').textContent='● TEST COMPLETED';
}catch(e){
$('status').textContent='● TEST FAILED: '+e.message;
}
btn.disabled=false;
btn.textContent='START SPEED TEST ⚡';
}

async function dnsTest(){
$('toolResult').textContent='Testing DNS...';
try{
let t=performance.now();
await fetch('https://cloudflare-dns.com/dns-query?name=example.com&type=A',{headers:{accept:'application/dns-json'}});
$('toolResult').textContent='DNS response: '+(performance.now()-t).toFixed(1)+' ms';
}catch(e){$('toolResult').textContent='DNS test failed';}
}

async function httpsTest(){
try{
let t=performance.now();
await fetch('https://www.cloudflare.com/cdn-cgi/trace');
$('toolResult').textContent='HTTPS OK: '+(performance.now()-t).toFixed(1)+' ms';
}catch(e){$('toolResult').textContent='HTTPS failed';}
}

async function stabilityTest(){
let ok=0,fail=0;
$('toolResult').textContent='Testing stability...';
for(let i=0;i<10;i++){
try{
let r=await fetch('https://www.cloudflare.com/cdn-cgi/trace?t='+Math.random());
r.ok?ok++:fail++;
}catch(e){fail++;}
}
$('toolResult').textContent=`Success ${ok}/10 | Failed ${fail}/10`;
}

function copyHistory(){
navigator.clipboard.writeText(historyData.join('\n')).then(()=>alert('Copied!')).catch(()=>alert('Clipboard unavailable'));
}

function downloadReport(){
let blob=new Blob([historyData.join('\n')],{type:'text/plain'});
let a=document.createElement('a');
a.href=URL.createObjectURL(blob);
a.download='NETX-report.txt';
a.click();
URL.revokeObjectURL(a.href);
}

function clearHistory(){
historyData=[];
$('history').textContent='No history.';
}

network();
getIP();
</script>
</body>
</html>

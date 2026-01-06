<!DOCTYPE html>
<html lang="it">
<head>
<meta charset="UTF-8">
<title>GreenTube 🌱</title>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
body{margin:0;font-family:Arial,sans-serif;background:linear-gradient(#0f3d2e,#1e7f4f);color:white;}
header{background:#08271c;padding:10px;display:flex;justify-content:space-between;align-items:center;}
nav button{margin:2px;padding:6px 10px;border:none;border-radius:6px;background:#2ecc71;font-weight:bold;cursor:pointer;}
.section{display:none;padding:15px;}
.box{background:rgba(0,0,0,.35);padding:15px;border-radius:12px;margin-bottom:15px;}
.video{background:rgba(0,0,0,.45);padding:10px;border-radius:10px;margin-bottom:10px;}
iframe.vertical{aspect-ratio:9/16;width:100%;height:400px;}
iframe.normal{width:100%;height:200px;}
input,textarea{width:100%;padding:7px;border-radius:6px;border:none;margin:5px 0;}
textarea{height:80px;}
button{cursor:pointer;}
img.commentImg{width:100%;max-height:300px;border-radius:10px;}
</style>
</head>
<body>

<header>
<h2 id="channelName">🌱 GreenTube</h2>
<nav id="menu">
<button onclick="showSection('home')">🎥 Home</button>
<button onclick="showSection('upload')">📤 Upload</button>
<button onclick="showSection('creator')">👤 Creator</button>
<button onclick="showSection('rank')">🏆 Classifica</button>
<button onclick="showSection('community')">💬 Community</button>
<button onclick="showSection('channel')">📊 Canale</button>
</nav>
</header>

<div id="home" class="section">
<h3>🎥 Video</h3>
<div id="videosBox"></div>
</div>

<div id="upload" class="section box">
<h3>📤 Aggiungi video</h3>
<input id="title" placeholder="Titolo video">
<input id="link" placeholder="Link YouTube">
<label><input type="checkbox" id="verticalCheck"> Video verticale (EcoSpeedy max 1 min)</label>
<button id="addBtn">Aggiungi</button>
</div>

<div id="creator" class="section box">
<h3>👤 Creator Studio</h3>
<div id="creatorStats"></div>
<h4>✏️ Nome canale</h4>
<input id="channelInput" placeholder="Nome del tuo canale">
<button onclick="saveMyChannelName(current)">Salva</button>
</div>

<div id="rank" class="section box">
<h3>🏆 Classifica</h3>
<ol id="leader"></ol>
</div>

<div id="community" class="section box">
<h3>💬 Community Live</h3>
<textarea id="communityInput" placeholder="Scrivi un messaggio..."></textarea>
<input type="text" id="communityMedia" placeholder="Link immagine o video (opzionale)">
<button id="sendMsg">Invia</button>
<div id="communityFeed" style="margin-top:10px;"></div>
</div>

<div id="channel" class="section box">
<h3>📊 Il tuo Canale</h3>
<div id="channelStats"></div>
</div>

<script>
let videos = JSON.parse(localStorage.getItem("videos"))||[];
let users = JSON.parse(localStorage.getItem("users"))||{};
let channels = JSON.parse(localStorage.getItem("channels"))||{};
let current = localStorage.getItem("current")||"";

// Inizializza proprietà video
videos = videos.map(v=>{
  if(v.views===undefined)v.views=0;
  if(v.likes===undefined)v.likes=[];
  if(v.shares===undefined)v.shares=0;
  if(v.comments===undefined)v.comments=[];
  if(v.eco===undefined)v.eco=0;
  if(v.ecoHistory===undefined)v.ecoHistory={};
  if(!v.timestamp)v.timestamp=new Date().getTime();
  return v;
});

// Imposta utente predefinito se primo accesso
if(!current){
  current="TestUser";
  users[current] = users[current]||{points:0,subscriptions:[]};
  channels[current] = channels[current]||{name:current,subscribers:[]};
  localStorage.setItem("users",JSON.stringify(users));
  localStorage.setItem("channels",JSON.stringify(channels));
  localStorage.setItem("current",current);
}

/* ===== SEZIONI ===== */
function showSection(id){
  document.querySelectorAll(".section").forEach(s=>s.style.display="none");
  document.getElementById(id).style.display="block";
  if(id==="creator") loadCreatorStudio(current);
  if(id==="community") renderCommunity();
  if(id==="rank") renderRank();
  if(id==="channel") renderChannel();
  if(id==="home") renderVideos();
}

/* ===== VIDEO ===== */
document.getElementById("addBtn").onclick=()=>{
  const t=document.getElementById("title").value.trim();
  const l=document.getElementById("link").value.trim();
  const vertical=document.getElementById("verticalCheck").checked;
  if(!t||!l)return;
  const id=Date.now();
  videos.push({id,title:t,link:toEmbed(l),user:current,eco:0,ecoHistory:{},vertical:vertical,timestamp:Date.now(),comments:[],views:0,likes:[],shares:0});
  localStorage.setItem("videos",JSON.stringify(videos));
  document.getElementById("title").value="";
  document.getElementById("link").value="";
  document.getElementById("verticalCheck").checked=false;
  renderVideos();
};

function toEmbed(url){
  if(url.includes("embed")) return url;
  if(url.includes("watch?v=")) return "https://www.youtube.com/embed/"+url.split("watch?v=")[1].split("&")[0];
  if(url.includes("youtu.be/")) return "https://www.youtube.com/embed/"+url.split("youtu.be/")[1];
  return url;
}

function addView(id){const v=videos.find(x=>x.id===id); if(v){v.views++; saveVideos();}}
function addLike(id){const v=videos.find(x=>x.id===id); if(v && !v.likes.includes(current)){v.likes.push(current); saveVideos();} else alert("Hai già messo like");}
function addEco(id){const v=videos.find(x=>x.id===id); if(!v)return; let week=new Date().toISOString().slice(0,10); if(v.ecoHistory[current]===week){alert("EcoPoint già dato questa settimana");return;} v.ecoHistory[current]=week; v.eco++; saveVideos();}
function shareVideo(id){const v=videos.find(x=>x.id===id); if(v){v.shares++; saveVideos(); alert("Video condiviso!");}}
function addComment(id){const v=videos.find(x=>x.id===id); const t=document.getElementById("comment-"+id); if(v && t.value.trim()!==""){v.comments.push({user:current,text:t.value,time:new Date().toLocaleString()}); t.value=""; saveVideos();}}

function subscribeChannel(user){
  if(!channels[user].subscribers) channels[user].subscribers=[];
  if(!channels[user].subscribers.includes(current)){
    channels[user].subscribers.push(current);
    alert("Iscritto a "+user);
    saveChannels();
    renderChannel();
  } else alert("Sei già iscritto");
}

/* ===== RENDER VIDEO ===== */
function renderVideos(){
  const box=document.getElementById("videosBox"); box.innerHTML="";
  videos.forEach(v=>{
    const div=document.createElement("div"); div.className="video";
    const date=new Date(v.timestamp);
    div.innerHTML=`
      <iframe class="${v.vertical?'vertical':'normal'}" src="${v.link}" allowfullscreen></iframe>
      <b>${v.title}</b><br>
      <small>👤 ${v.user} | 📅 ${date.toLocaleString()}</small><br>
      <button onclick="addView(${v.id})">👀 Visualizza (${v.views})</button>
      <button onclick="addLike(${v.id})">👍 Like (${v.likes.length})</button>
      <button onclick="addEco(${v.id})">🌍 EcoPoint (${v.eco})</button>
      <button onclick="subscribeChannel('${v.user}')">🔔 Iscriviti</button>
      <button onclick="shareVideo(${v.id})">🔗 Condividi (${v.shares})</button>
      <div>
        <h4>Commenti (${v.comments.length})</h4>
        <textarea id="comment-${v.id}" placeholder="Scrivi un commento"></textarea>
        <button onclick="addComment(${v.id})">Invia</button>
        <div id="commentsFeed-${v.id}"></div>
      </div>
    `;
    box.appendChild(div);
    const feed=document.getElementById(`commentsFeed-${v.id}`);
    v.comments.slice().reverse().forEach(c=>{
      const cDiv=document.createElement("div");
      cDiv.innerHTML=`<b>${c.user}</b> (${c.time}): ${c.text}`;
      feed.appendChild(cDiv);
    });
  });
}

/* ===== CREATOR ===== */
function loadCreatorStudio(user){
  document.getElementById("channelInput").value = channels[user]?.name || user;
  const myVideos = videos.filter(v=>v.user===user);
  document.getElementById("creatorStats").innerHTML=`🎥 Video: ${myVideos.length}<br>🌍 EcoPoints: ${myVideos.reduce((a,b)=>a+b.eco,0)}<br>📊 Iscritti: ${channels[user]?.subscribers.length || 0}`;
}

/* ===== CLASSIFICA ===== */
function renderRank(){
  const leader=document.getElementById("leader"); leader.innerHTML="";
  Object.entries(channels).sort((a,b)=>b[1].subscribers.length - a[1].subscribers.length)
    .forEach(([user,channel])=>{
      leader.innerHTML+=`<li>${channel.name} – ${channel.subscribers.length} iscritti</li>`;
    });
}

/* ===== CANALE ===== */
function renderChannel(){
  const stats=document.getElementById("channelStats");
  stats.innerHTML=`Nome canale: ${channels[current].name}<br>Iscritti: ${channels[current].subscribers.length}<br>`;
  stats.innerHTML+=`Lista iscritti:<br>${channels[current].subscribers.join(", ")}`;
}

/* ===== COMMUNITY ===== */
document.getElementById("sendMsg").onclick=()=>{
  const txt=document.getElementById("communityInput").value.trim();
  const media=document.getElementById("communityMedia").value.trim();
  if(!txt && !media) return;
  let communityMessages = JSON.parse(localStorage.getItem("community"))||[];
  communityMessages.push({user:current,msg:txt,media:media,time:new Date().toLocaleString()});
  localStorage.setItem("community",JSON.stringify(communityMessages));
  document.getElementById("communityInput").value="";
  document.getElementById("communityMedia").value="";
  renderCommunity();
};

function renderCommunity(){
  const feed=document.getElementById("communityFeed"); feed.innerHTML="";
  let communityMessages = JSON.parse(localStorage.getItem("community"))||[];
  communityMessages.slice().reverse().forEach(m=>{
    const div=document.createElement("div"); div.className="box";
    let mediaHtml="";
    if(m.media){
      if(m.media.match(/\.(jpeg|jpg|png|gif)$/i)){mediaHtml=`<img src="${m.media}" class="commentImg">`;}
      else { mediaHtml=`<iframe width="100%" height="200" src="${toEmbed(m.media)}" allowfullscreen></iframe>`;}
    }
    div.innerHTML=`<b>${m.user}</b> (${m.time}): ${m.msg}<br>${mediaHtml}`;
    feed.appendChild(div);
  });
}

/* ===== SALVATAGGIO ===== */
function saveVideos(){localStorage.setItem("videos",JSON.stringify(videos)); renderVideos(); loadCreatorStudio(current);}
function saveChannels(){localStorage.setItem("channels",JSON.stringify(channels)); renderChannel(); renderRank(); loadCreatorStudio(current);}
function saveMyChannelName(user){
  const input=document.getElementById("channelInput");
  if(!input)return;
  const name=input.value.trim();
  if(!name)return;
  channels[user].name=name;
  saveChannels();
  alert("Nome canale aggiornato!");
}

/* ===== AVVIO ===== */
showSection("home");
renderVideos();
renderRank();
renderChannel();
loadCreatorStudio(current);
</script>

</body>
</html>

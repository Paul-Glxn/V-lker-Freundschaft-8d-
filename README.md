<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<title>Klassenportal – Völkerfreundschaft Köthen</title>
<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background-color: #f4f6f8; margin:0; padding:0; }
header { background-color: #004d40; color: #fff; text-align: center; padding: 40px 0; font-size: 28px; font-weight: bold;
         background-image: url('https://upload.wikimedia.org/wikipedia/commons/thumb/5/5c/Schule_K%C3%B6then.jpg/800px-Schule_K%C3%B6then.jpg');
         background-size: cover; background-position: center; text-shadow: 2px 2px 5px rgba(0,0,0,0.6); }
#login, #content { max-width: 1000px; margin: 30px auto; padding: 25px; background:white; border-radius: 12px; box-shadow:0 4px 12px rgba(0,0,0,0.15); }
h2 { border-bottom:3px solid #004d40; padding-bottom:8px; margin-bottom:20px; color:#004d40; }
.section { margin-bottom:35px; padding:20px; border-radius:12px; background:#ffffff; box-shadow:0 2px 8px rgba(0,0,0,0.1); }
label { display:block; margin-top:10px; font-weight:bold; }
input, select, textarea { margin-top:5px; padding:8px; width:95%; border-radius:6px; border:1px solid #ccc; }
button { margin-top:12px; padding:10px 20px; border:none; border-radius:6px; cursor:pointer; font-weight:bold; }
button:hover { opacity:0.9; }
.btn-primary { background-color:#004d40; color:white; }
.btn-danger { background-color:#c62828; color:white; margin-left:10px; }
.hidden { display:none; }
.item { padding:12px; margin:6px 0; background:#f9f9f9; border-radius:8px; box-shadow:0 1px 3px rgba(0,0,0,0.1); display:flex; justify-content:space-between; align-items:center; flex-wrap:wrap; }
.item span { flex:1 1 auto; }
.item .note { width:100%; margin-top:6px; font-size:0.9em; color:#555; }
img { max-width:200px; margin-top:10px; border-radius:6px; }
.top-bar { display:flex; justify-content:space-between; align-items:center; margin-bottom:20px; }
.top-bar select { width:auto; }
.status { font-weight:bold; margin-left:10px; padding:3px 6px; border-radius:4px; color:white; }
.status.pending { background:#f39c12; }
.status.done { background:#27ae60; }
</style>
</head>
<body>

<header>Klassenportal – Völkerfreundschaft Köthen</header>

<div id="login">
  <h2># ANMELDEN</h2>
  <label>Geheime ID:</label>
  <input type="text" id="userid">
  <button class="btn-primary" onclick="login()">Anmelden</button>
  <p id="loginError" style="color:red;"></p>
</div>

<div id="content" class="hidden">

  <div class="top-bar">
    <div id="welcome"></div>
    <div>
      <label>Filter Fach:</label>
      <select id="fachFilter" onchange="loadData()">
        <option value="">Alle</option>
        <option>Mathe</option>
        <option>Deutsch</option>
        <option>Englisch</option>
        <option>Biologie</option>
        <option>Geschichte</option>
      </select>
    </div>
  </div>

  <!-- Hausaufgaben -->
  <div class="section">
    <h2>Hausaufgaben</h2>
    <div id="hausaufgabenList"></div>
    <div id="hausaufgabenForm" class="hidden">
      <label>Fach:</label>
      <select id="fach">
        <option>Mathe</option>
        <option>Deutsch</option>
        <option>Englisch</option>
        <option>Biologie</option>
        <option>Geschichte</option>
      </select>
      <label>Typ:</label>
      <select id="typ">
        <option>Buch</option>
        <option>Arbeitsheft</option>
      </select>
      <label>Seite:</label>
      <input id="seite">
      <label>Nummer:</label>
      <input id="nummer">
      <label>Abgabe-Datum:</label>
      <input type="date" id="deadline">
      <label>Notiz:</label>
      <textarea id="note" placeholder="Optional..."></textarea>
      <button class="btn-primary" onclick="saveHomework()">Speichern</button>
    </div>
  </div>

  <!-- Elternsprechtage -->
  <div class="section">
    <h2>Elternsprechtage</h2>
    <div id="elternList"></div>
    <div id="elternForm" class="hidden">
      <label>Datum:</label>
      <input type="date" id="elternDatum">
      <label>Uhrzeit:</label>
      <input type="time" id="elternZeit">
      <label>Lehrer:</label>
      <input id="elternLehrer">
      <label>Notiz:</label>
      <textarea id="elternNote" placeholder="Optional..."></textarea>
      <button class="btn-primary" onclick="saveParent()">Speichern</button>
    </div>
  </div>

  <!-- Pläne -->
  <div class="section">
    <h2>Pläne</h2>
    <div id="plaeneList"></div>
    <div id="plaeneForm" class="hidden">
      <label>Bild-URL:</label>
      <input id="plaeneUrl">
      <label>Notiz:</label>
      <textarea id="plaeneNote" placeholder="Optional..."></textarea>
      <button class="btn-primary" onclick="savePlan()">Hinzufügen</button>
    </div>
  </div>

</div>

<script>
const ids = { "025":"admin", "024":"viewer" };
let currentRole = null;

function login() {
  const userid = document.getElementById("userid").value;
  if(!ids[userid]) {
    document.getElementById("loginError").textContent="Ungültige ID!";
    return;
  }
  currentRole = ids[userid];
  document.getElementById("login").classList.add("hidden");
  document.getElementById("content").classList.remove("hidden");
  document.getElementById("welcome").textContent = currentRole==="admin"?"Hallo, Admin!":"Hallo, Viewer!";
  if(currentRole==="admin"){
    document.getElementById("hausaufgabenForm").classList.remove("hidden");
    document.getElementById("elternForm").classList.remove("hidden");
    document.getElementById("plaeneForm").classList.remove("hidden");
  }
  loadData();
}

function checkAdmin(){ if(currentRole!=="admin"){ alert("Keine Berechtigung!"); return false; } return true; }

// Hausaufgaben
function saveHomework(){
  if(!checkAdmin()) return;
  const hw={fach:document.getElementById("fach").value, typ:document.getElementById("typ").value, seite:document.getElementById("seite").value, nummer:document.getElementById("nummer").value, deadline:document.getElementById("deadline").value, note:document.getElementById("note").value, status:"pending"};
  let list=JSON.parse(localStorage.getItem("hausaufgaben"))||[];
  list.push(hw); localStorage.setItem("hausaufgaben",JSON.stringify(list)); loadData();
}
function deleteHomework(i){ if(!checkAdmin()) return; let list=JSON.parse(localStorage.getItem("hausaufgaben"))||[]; list.splice(i,1); localStorage.setItem("hausaufgaben",JSON.stringify(list)); loadData(); }
function toggleStatusHW(i){ let list=JSON.parse(localStorage.getItem("hausaufgaben"))||[]; list[i].status=list[i].status==="pending"?"done":"pending"; localStorage.setItem("hausaufgaben",JSON.stringify(list)); loadData(); }

// Eltern
function saveParent(){ if(!checkAdmin()) return; const parent={datum:document.getElementById("elternDatum").value, zeit:document.getElementById("elternZeit").value, lehrer:document.getElementById("elternLehrer").value, note:document.getElementById("elternNote").value}; let list=JSON.parse(localStorage.getItem("eltern"))||[]; list.push(parent); localStorage.setItem("eltern",JSON.stringify(list)); loadData(); }
function deleteParent(i){ if(!checkAdmin()) return; let list=JSON.parse(localStorage.getItem("eltern"))||[]; list.splice(i,1); localStorage.setItem("eltern",JSON.stringify(list)); loadData(); }

// Pläne
function savePlan(){ if(!checkAdmin()) return; const plan={url:document.getElementById("plaeneUrl").value,note:document.getElementById("plaeneNote").value}; let list=JSON.parse(localStorage.getItem("plaene"))||[]; list.push(plan); localStorage.setItem("plaene",JSON.stringify(list)); loadData(); }
function deletePlan(i){ if(!checkAdmin()) return; let list=JSON.parse(localStorage.getItem("plaene"))||[]; list.splice(i,1); localStorage.setItem("plaene",JSON.stringify(list)); loadData(); }

function loadData(){
  // Hausaufgaben
  const fachFilter=document.getElementById("fachFilter").value;
  const hausaufgaben=JSON.parse(localStorage.getItem("hausaufgaben"))||[];
  const hwDiv=document.getElementById("hausaufgabenList"); hwDiv.innerHTML="";
  hausaufgaben.forEach((hw,i)=>{
    if(fachFilter && hw.fach!==fachFilter) return;
    const line=document.createElement("div"); line.className="item";
    const span=document.createElement("span");
    span.innerHTML=`${hw.fach} | ${hw.typ} Seite ${hw.seite} Nr.${hw.nummer} <span class="status ${hw.status}">${hw.status==="pending"?"Offen":"Erledigt"}</span>`;
    line.appendChild(span);
    if(hw.note) { const note=document.createElement("div"); note.className="note"; note.textContent=hw.note; line.appendChild(note); }
    if(currentRole==="admin"){
      const del=document.createElement("button"); del.className="btn-danger"; del.textContent="Löschen"; del.onclick=()=>deleteHomework(i); line.appendChild(del);
      const toggle=document.createElement("button"); toggle.className="btn-primary"; toggle.textContent="Status"; toggle.onclick=()=>toggleStatusHW(i); line.appendChild(toggle);
    }
    hwDiv.appendChild(line);
  });

  // Eltern
  const eltern=JSON.parse(localStorage.getItem("eltern"))||[];
  const elDiv=document.getElementById("elternList"); elDiv.innerHTML="";
  eltern.forEach((e,i)=>{
    const line=document.createElement("div"); line.className="item";
    const span=document.createElement("span"); span.textContent=`${e.datum} um ${e.zeit} bei ${e.lehrer}`; line.appendChild(span);
    if(e.note) { const note=document.createElement("div"); note.className="note"; note.textContent=e.note; line.appendChild(note); }
    if(currentRole==="admin"){ const del=document.createElement("button"); del.className="btn-danger"; del.textContent="Löschen"; del.onclick=()=>deleteParent(i); line.appendChild(del); }
    elDiv.appendChild(line);
  });

  // Pläne
  const plaene=JSON.parse(localStorage.getItem("plaene"))||[];
  const plDiv=document.getElementById("plaeneList"); plDiv.innerHTML="";
  plaene.forEach((p,i)=>{
    const line=document.createElement("div"); line.className="item";
    const img=document.createElement("img"); img.src=p.url; line.appendChild(img);
    if(p.note) { const note=document.createElement("div"); note.className="note"; note.textContent=p.note; line.appendChild(note); }
    if(currentRole==="admin"){ const del=document.createElement("button"); del.className="btn-danger"; del.textContent="Löschen"; del.onclick=()=>deletePlan(i); line.appendChild(del); }
    plDiv.appendChild(line);
  });
}
</script>

</body>
</html>

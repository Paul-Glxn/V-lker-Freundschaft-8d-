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
  #login, #content { max-width: 900px; margin: 30px auto; padding: 25px; background:white; border-radius: 12px; box-shadow:0 4px 12px rgba(0,0,0,0.15); }
  h2 { border-bottom:3px solid #004d40; padding-bottom:8px; margin-bottom:20px; color:#004d40; }
  .section { margin-bottom:35px; padding:20px; border-radius:12px; background:#ffffff; box-shadow:0 2px 8px rgba(0,0,0,0.1); }
  label { display:block; margin-top:10px; font-weight:bold; }
  input, select { margin-top:5px; padding:8px; width:95%; border-radius:6px; border:1px solid #ccc; }
  button { margin-top:12px; padding:10px 20px; border:none; border-radius:6px; cursor:pointer; font-weight:bold; }
  button:hover { opacity:0.9; }
  .btn-primary { background-color:#004d40; color:white; }
  .btn-danger { background-color:#c62828; color:white; margin-left:10px; }
  .hidden { display:none; }
  .item { padding:12px; margin:6px 0; background:#f9f9f9; border-radius:8px; box-shadow:0 1px 3px rgba(0,0,0,0.1); display:flex; justify-content:space-between; align-items:center; }
  img { max-width:200px; margin-top:10px; border-radius:6px; }
</style>
</head>
<body>

<header>Klassenportal – Völkerfreundschaft Köthen</header>

<!-- Login -->
<div id="login">
  <h2># ANMELDEN</h2>
  <label>Geheime ID:</label>
  <input type="text" id="userid">
  <button class="btn-primary" onclick="login()">Anmelden</button>
  <p id="loginError" style="color:red;"></p>
</div>

<!-- Content -->
<div id="content" class="hidden">

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
      <button class="btn-primary" onclick="savePlan()">Hinzufügen</button>
    </div>
  </div>

</div>

<script>
  // IDs und Rollen
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
    if(currentRole==="admin"){
      document.getElementById("hausaufgabenForm").classList.remove("hidden");
      document.getElementById("elternForm").classList.remove("hidden");
      document.getElementById("plaeneForm").classList.remove("hidden");
    }
    loadData();
  }

  function checkAdmin(){
    if(currentRole!=="admin"){ alert("Keine Berechtigung!"); return false; }
    return true;
  }

  // Hausaufgaben
  function saveHomework(){
    if(!checkAdmin()) return;
    const hw={fach:document.getElementById("fach").value, typ:document.getElementById("typ").value, seite:document.getElementById("seite").value, nummer:document.getElementById("nummer").value, deadline:document.getElementById("deadline").value};
    let list=JSON.parse(localStorage.getItem("hausaufgaben"))||[];
    list.push(hw); localStorage.setItem("hausaufgaben",JSON.stringify(list)); loadData();
  }
  function deleteHomework(i){ if(!checkAdmin()) return; let list=JSON.parse(localStorage.getItem("hausaufgaben"))||[]; list.splice(i,1); localStorage.setItem("hausaufgaben",JSON.stringify(list)); loadData(); }

  // Eltern
  function saveParent(){ if(!checkAdmin()) return; const parent={datum:document.getElementById("elternDatum").value, zeit:document.getElementById("elternZeit").value, lehrer:document.getElementById("elternLehrer").value}; let list=JSON.parse(localStorage.getItem("eltern"))||[]; list.push(parent); localStorage.setItem("eltern",JSON.stringify(list)); loadData(); }
  function deleteParent(i){ if(!checkAdmin()) return; let list=JSON.parse(localStorage.getItem("eltern"))||[]; list.splice(i,1); localStorage.setItem("eltern",JSON.stringify(list)); loadData(); }

  // Pläne
  function savePlan(){ if(!checkAdmin()) return; const url=document.getElementById("plaeneUrl").value; let list=JSON.parse(localStorage.getItem("plaene"))||[]; list.push(url); localStorage.setItem("plaene",JSON.stringify(list)); loadData(); }
  function deletePlan(i){ if(!checkAdmin()) return; let list=JSON.parse(localStorage.getItem("plaene"))||[]; list.splice(i,1); localStorage.setItem("plaene",JSON.stringify(list)); loadData(); }

  function loadData(){
    // Hausaufgaben
    const hwList=JSON.parse(localStorage.getItem("hausaufgaben"))||[];
    const hwDiv=document.getElementById("hausaufgabenList"); hwDiv.innerHTML="";
    hwList.forEach((h,i)=>{ const line=document.createElement("div"); line.className="item"; line.textContent=`${h.fach} – ${h.typ}, Seite ${h.seite}, Nr. ${h.nummer} (bis ${h.deadline})`; if(currentRole==="admin"){ const del=document.createElement("button"); del.className="btn-danger"; del.textContent="Löschen"; del.onclick=()=>deleteHomework(i); line.appendChild(del); } hwDiv.appendChild(line); });

    // Eltern
    const eltern=JSON.parse(localStorage.getItem("eltern"))||[];
    const elDiv=document.getElementById("elternList"); elDiv.innerHTML="";
    eltern.forEach((e,i)=>{ const line=document.createElement("div"); line.className="item"; line.textContent=`${e.datum} um ${e.zeit} bei ${e.lehrer}`; if(currentRole==="admin"){ const del=document.createElement("button"); del.className="btn-danger"; del.textContent="Löschen"; del.onclick=()=>deleteParent(i); line.appendChild(del); } elDiv.appendChild(line); });

    // Pläne
    const plaene=JSON.parse(localStorage.getItem("plaene"))||[];
    const plDiv=document.getElementById("plaeneList"); plDiv.innerHTML="";
    plaene.forEach((p,i)=>{ const line=document.createElement("div"); line.className="item"; const img=document.createElement("img"); img.src=p; line.appendChild(img); if(currentRole==="admin"){ const del=document.createElement("button"); del.className="btn-danger"; del.textContent="Löschen"; del.onclick=()=>deletePlan(i); line.appendChild(del); } plDiv.appendChild(line); });
  }
</script>

</body>
</html>

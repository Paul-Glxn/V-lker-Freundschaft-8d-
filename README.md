 
<html lang="de">
<head>
<meta charset="UTF-8">
<title>Klasseninfo</title>
<style>
  body { font-family: Arial, sans-serif; background: #f0f2f5; margin: 0; padding: 20px;}
  h1 { text-align: center; color: #333; }
  #content { display: none; }

  /* Karten */
  .section { background: #fff; padding: 20px; margin: 15px 0; border-radius: 10px; box-shadow: 0 2px 6px rgba(0,0,0,0.1);}
  .section h2 { cursor: pointer; margin: 0; padding-bottom: 10px; color: #007bff; }
  .section.collapsed div { display: none; }

  /* Buttons */
  .btn { padding: 8px 12px; margin: 5px; cursor: pointer; border: none; border-radius: 5px; color: #fff; }
  .btn:hover { opacity: 0.9; }
  .btn-add { background-color: #007bff; }
  .btn-delete { background-color: #dc3545; }

  .hidden { display: none; }
  img { border-radius: 8px; margin: 5px 0; max-width: 200px; box-shadow: 0 2px 4px rgba(0,0,0,0.2);}
  input, select { padding: 5px; margin: 5px 0; width: 90%; border-radius: 5px; border: 1px solid #ccc; }
</style>
</head>
<body>
<h1>Klasseninfo</h1>

<!-- Login -->
<div id="login">
  <h2>Anmelden</h2>
  Benutzername: <input id="username"><br>
  Passwort: <input type="password" id="password"><br>
  Geheime ID: <input id="userid"><br>
  <button class="btn btn-add" onclick="login()">Login</button>
</div>

<!-- Hauptinhalt -->
<div id="content">

  <!-- Hausaufgaben -->
  <div class="section collapsed">
    <h2 onclick="toggleSection(this)">Hausaufgaben</h2>
    <div>
      <div id="hausaufgabenList"></div>
      <div id="btnHausaufgaben" class="hidden">
        <h3>Neue Hausaufgabe hinzufügen</h3>
        Fach: 
        <select id="fach">
          <option>Mathe</option>
          <option>Deutsch</option>
          <option>Englisch</option>
          <option>Biologie</option>
        </select><br>
        Typ: 
        <select id="typ">
          <option>Buch</option>
          <option>Arbeitsheft</option>
          <option>Übung</option>
        </select><br>
        Seite: <input id="seite"><br>
        Nummer: <input id="nummer"><br>
        Deadline: <input id="deadline" type="date"><br>
        <button class="btn btn-add" onclick="saveHomework()">Hinzufügen</button>
      </div>
    </div>
  </div>

  <!-- Elternsprechtage -->
  <div class="section collapsed">
    <h2 onclick="toggleSection(this)">Elternsprechtage</h2>
    <div>
      <div id="elternList"></div>
      <div id="btnEltern" class="hidden">
        <h3>Neuen Termin hinzufügen</h3>
        Name: <input id="elternName"><br>
        Datum: <input id="elternDatum" type="date"><br>
        Uhrzeit: <input id="elternUhrzeit" type="time"><br>
        <button class="btn btn-add" onclick="saveParent()">Hinzufügen</button>
      </div>
    </div>
  </div>

  <!-- Pläne -->
  <div class="section collapsed">
    <h2 onclick="toggleSection(this)">Pläne</h2>
    <div>
      <div id="plaeneList"></div>
      <div id="btnPlaene" class="hidden">
        <h3>Neuen Plan hinzufügen (Bild-URL)</h3>
        Bild-URL: <input id="plaeneUrl"><br>
        <button class="btn btn-add" onclick="savePlan()">Hinzufügen</button>
      </div>
    </div>
  </div>

</div>

<script>
// Benutzer und Rollen
const users = [
  { username: "Max", password: "123", id: "025", role: "admin" },
  { username: "Lisa", password: "456", id: "024", role: "viewer" }
];

// Login Funktion
function login() {
  const username = document.getElementById("username").value;
  const password = document.getElementById("password").value;
  const userid = document.getElementById("userid").value;

  const user = users.find(u => u.username === username && u.password === password && u.id === userid);
  if (!user) { alert("Falsche Daten!"); return; }

  localStorage.setItem("role", user.role);
  localStorage.setItem("username", user.username);

  document.getElementById("login").style.display = "none";
  document.getElementById("content").style.display = "block";

  if (user.role === "admin") {
    document.getElementById("btnHausaufgaben").classList.remove("hidden");
    document.getElementById("btnEltern").classList.remove("hidden");
    document.getElementById("btnPlaene").classList.remove("hidden");
  }

  loadData();
}

// Sektion einklappen/ausklappen
function toggleSection(el) {
  el.parentElement.classList.toggle("collapsed");
}

// Daten laden
function loadData() {
  // Hausaufgaben
  const hausaufgaben = JSON.parse(localStorage.getItem("hausaufgaben")) || [];
  const hwDiv = document.getElementById("hausaufgabenList");
  hwDiv.innerHTML = "";
  hausaufgaben.forEach((hw,i)=>{
    const line = document.createElement("div");
    line.textContent = `${hw.fach} - ${hw.typ} Seite: ${hw.seite} Nr: ${hw.nummer} Deadline: ${hw.deadline}`;
    if(localStorage.getItem("role")==="admin"){
      const del=document.createElement("button");
      del.textContent="Löschen";
      del.className="btn btn-delete";
      del.onclick=()=>deleteHomework(i);
      line.appendChild(del);
    }
    hwDiv.appendChild(line);
  });

  // Eltern
  const eltern = JSON.parse(localStorage.getItem("eltern")) || [];
  const elDiv = document.getElementById("elternList");
  elDiv.innerHTML = "";
  eltern.forEach((e,i)=>{
    const line=document.createElement("div");
    line.textContent = `${e.name} - ${e.datum} ${e.uhrzeit}`;
    if(localStorage.getItem("role")==="admin"){
      const del=document.createElement("button");
      del.textContent="Löschen";
      del.className="btn btn-delete";
      del.onclick=()=>deleteParent(i);
      line.appendChild(del);
    }
    elDiv.appendChild(line);
  });

  // Pläne
  const plaene = JSON.parse(localStorage.getItem("plaene")) || [];
  const plDiv = document.getElementById("plaeneList");
  plDiv.innerHTML = "";
  plaene.forEach((p,i)=>{
    const line=document.createElement("div");
    const img=document.createElement("img");
    img.src=p;
    line.appendChild(img);
    if(localStorage.getItem("role")==="admin"){
      const del=document.createElement("button");
      del.textContent="Löschen";
      del.className="btn btn-delete";
      del.onclick=()=>deletePlan(i);
      line.appendChild(del);
    }
    plDiv.appendChild(line);
  });
}

// Hausaufgaben speichern
function saveHomework() {
  if(localStorage.getItem("role")!=="admin"){ alert("Keine Berechtigung!"); return; }
  const hw = {
    fach: document.getElementById("fach").value,
    typ: document.getElementById("typ").value,
    seite: document.getElementById("seite").value,
    nummer: document.getElementById("nummer").value,
    deadline: document.getElementById("deadline").value
  };
  let list=JSON.parse(localStorage.getItem("hausaufgaben"))||[];
  list.push(hw);
  localStorage.setItem("hausaufgaben",JSON.stringify(list));
  loadData();
}

// Eltern speichern
function saveParent() {
  if(localStorage.getItem("role")!=="admin"){ alert("Keine Berechtigung!"); return; }
  const e = {
    name: document.getElementById("elternName").value,
    datum: document.getElementById("elternDatum").value,
    uhrzeit: document.getElementById("elternUhrzeit").value
  };
  let list=JSON.parse(localStorage.getItem("eltern"))||[];
  list.push(e);
  localStorage.setItem("eltern",JSON.stringify(list));
  loadData();
}

// Pläne speichern
function savePlan() {
  if(localStorage.getItem("role")!=="admin"){ alert("Keine Berechtigung!"); return; }
  const url=document.getElementById("plaeneUrl").value;
  let list=JSON.parse(localStorage.getItem("plaene"))||[];
  list.push(url);
  localStorage.setItem("plaene",JSON.stringify(list));
  loadData();
}

// Löschen
function deleteHomework(i){let list=JSON.parse(localStorage.getItem("hausaufgaben"))||[];list.splice(i,1);localStorage.setItem("hausaufgaben",JSON.stringify(list));loadData();}
function deleteParent(i){let list=JSON.parse(localStorage.getItem("eltern"))||[];list.splice(i,1);localStorage.setItem("eltern",JSON.stringify(list));loadData();}
function deletePlan(i){let list=JSON.parse(localStorage.getItem("plaene"))||[];list.splice(i,1);localStorage.setItem("plaene",JSON.stringify(list));loadData();}
</script>
</body>
</html>

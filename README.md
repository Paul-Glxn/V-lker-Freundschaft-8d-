 
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>Klassen-Website</title>
  <style>
    body { font-family: Arial; margin: 20px; }
    .hidden { display: none; }
    h2 { margin-top: 30px; }
    .section { border: 1px solid #ccc; padding: 10px; margin-bottom: 20px; }
    button { margin-top: 5px; }
    .item { margin: 5px 0; }
    .item button { margin-left: 10px; }
  </style>
</head>
<body>

  <!-- LOGIN -->
  <div id="login">
    <h1># ANMELDEN</h1>
    <input type="text" id="username" placeholder="Benutzername"><br><br>
    <input type="password" id="password" placeholder="Passwort"><br><br>
    <input type="text" id="secret" placeholder="Geheime ID"><br><br>
    <button onclick="login()">Login</button>
    <p id="error" style="color:red"></p>
  </div>

  <!-- INHALTE -->
  <div id="content" class="hidden">

    <div class="section">
      <h2>Hausaufgaben</h2>
      <button id="btnHausaufgaben" class="hidden" onclick="showHomeworkForm()">Neue Hausaufgabe</button>
      <div id="hausaufgaben"></div>
    </div>

    <div class="section">
      <h2>Elternsprechtage</h2>
      <button id="btnEltern" class="hidden" onclick="showParentForm()">Neuen Termin</button>
      <div id="eltern"></div>
    </div>

    <div class="section">
      <h2>Pläne</h2>
      <button id="btnPlaene" class="hidden" onclick="addPlan()">Bild hinzufügen</button>
      <div id="plaene"></div>
    </div>

  </div>

  <script>
    // Nutzer mit IDs
    const users = [
      {username:"admin", password:"1234", secret:"025", role:"admin"},
      {username:"user", password:"abcd", secret:"024", role:"viewer"}
    ];

    // Login
    function login() {
      const uname = document.getElementById("username").value;
      const pass = document.getElementById("password").value;
      const sec = document.getElementById("secret").value;

      const user = users.find(u => u.username===uname && u.password===pass && u.secret===sec);

      if(user){
        localStorage.setItem("role", user.role);
        showContent(user.role);
        document.getElementById("login").style.display="none";
      } else {
        document.getElementById("error").innerText="Login fehlgeschlagen!";
      }
    }

    // Inhalte anzeigen je nach Rolle
    function showContent(role){
      document.getElementById("content").classList.remove("hidden");
      if(role==="admin"){
        document.getElementById("btnHausaufgaben").classList.remove("hidden");
        document.getElementById("btnEltern").classList.remove("hidden");
        document.getElementById("btnPlaene").classList.remove("hidden");
      }
      loadData();
    }

    // ---------- HAUSAUFGABEN ----------
    function showHomeworkForm(){
      const fach = prompt("Fach (z.B. Mathe, Deutsch, Englisch):");
      if(!fach) return;
      const typ = prompt("Buch oder Arbeitsheft?");
      if(!typ) return;
      const seite = prompt("Seite:");
      if(!seite) return;
      const nummer = prompt("Nummer/Aufgabe:");
      if(!nummer) return;

      const hw = {fach, typ, seite, nummer};
      let list = JSON.parse(localStorage.getItem("hausaufgaben")) || [];
      list.push(hw);
      localStorage.setItem("hausaufgaben", JSON.stringify(list));
      loadData();
    }

    // ---------- ELTERNSPRECHTAGE ----------
    function showParentForm(){
      const datum = prompt("Datum (z.B. 12.10.2025):");
      if(!datum) return;
      const uhrzeit = prompt("Uhrzeit:");
      if(!uhrzeit) return;
      const info = prompt("Bemerkung:");
      if(!info) return;

      const termin = {datum, uhrzeit, info};
      let list = JSON.parse(localStorage.getItem("eltern")) || [];
      list.push(termin);
      localStorage.setItem("eltern", JSON.stringify(list));
      loadData();
    }

    // ---------- PLÄNE ----------
    function addPlan(){
      const url = prompt("Bild-URL eingeben:");
      if(url){
        let items = JSON.parse(localStorage.getItem("plaene")) || [];
        items.push(url);
        localStorage.setItem("plaene", JSON.stringify(items));
        loadData();
      }
    }

    // ---------- LÖSCHEN ----------
    function deleteItem(type, index){
      let list = JSON.parse(localStorage.getItem(type)) || [];
      list.splice(index,1);
      localStorage.setItem(type, JSON.stringify(list));
      loadData();
    }

    // ---------- DATEN LADEN ----------
    function loadData(){
      const role = localStorage.getItem("role");

      // Hausaufgaben
      const hwDiv = document.getElementById("hausaufgaben");
      hwDiv.innerHTML="";
      let hausaufgaben = JSON.parse(localStorage.getItem("hausaufgaben")) || [];
      hausaufgaben.forEach((hw,i)=>{
        const div = document.createElement("div");
        div.className="item";
        div.innerHTML = `<b>${hw.fach}</b> - ${hw.typ}, Seite ${hw.seite}, Nr. ${hw.nummer}`;
        if(role==="admin") div.innerHTML += ` <button onclick="deleteItem('hausaufgaben',${i})">Löschen</button>`;
        hwDiv.appendChild(div);
      });

      // Elternsprechtage
      const elternDiv = document.getElementById("eltern");
      elternDiv.innerHTML="";
      let eltern = JSON.parse(localStorage.getItem("eltern")) || [];
      eltern.forEach((t,i)=>{
        const div = document.createElement("div");
        div.className="item";
        div.innerHTML = `${t.datum} - ${t.uhrzeit} (${t.info})`;
        if(role==="admin") div.innerHTML += ` <button onclick="deleteItem('eltern',${i})">Löschen</button>`;
        elternDiv.appendChild(div);
      });

      // Pläne
      const plaeneDiv = document.getElementById("plaene");
      plaeneDiv.innerHTML="";
      let plaene = JSON.parse(localStorage.getItem("plaene")) || [];
      plaene.forEach((url,i)=>{
        const div = document.createElement("div");
        div.className="item";
        div.innerHTML = `<img src="${url}" width="200">`;
        if(role==="admin") div.innerHTML += ` <button onclick="deleteItem('plaene',${i})">Löschen</button>`;
        plaeneDiv.appendChild(div);
      });
    }

    // Schon eingeloggt?
    window.onload = () => {
      const role = localStorage.getItem("role");
      if(role){
        showContent(role);
        document.getElementById("login").style.display="none";
      }
    }
  </script>

</body>
</html>

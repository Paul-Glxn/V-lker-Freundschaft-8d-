 
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <title>Klassen-Website</title>
  <style>
    body { font-family: Arial, sans-serif; margin: 0; padding: 0; }
    header {
      position: fixed; top: 0; left: 0; right: 0;
      background: #333; color: white; display: flex; justify-content: space-between;
      padding: 10px 20px; z-index: 1000;
    }
    header nav a { color: white; margin: 0 10px; text-decoration: none; }
    header nav a:hover { text-decoration: underline; }
    .dark body { background: #121212; color: #eee; }
    .dark .card { background: #1e1e1e; }
    .container { margin-top: 70px; padding: 20px; }
    .card {
      background: white; border-radius: 8px; padding: 20px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.2); margin-bottom: 30px;
    }
    .card h2 { margin-top: 0; }
    button {
      padding: 6px 12px; border: none; border-radius: 5px;
      cursor: pointer; margin-top: 10px;
    }
    button:hover { opacity: 0.9; }
    .btn-blue { background: #2196f3; color: white; }
    .btn-green { background: #4caf50; color: white; }
    .btn-orange { background: #ff9800; color: white; }
    .btn-red { background: #f44336; color: white; }
    input, select {
      padding: 6px; margin: 5px 0; width: 100%; border-radius: 5px; border: 1px solid #ccc;
    }
    .item { margin: 8px 0; padding: 10px; border-radius: 6px; background: #f9f9f9; }
    .dark .item { background: #2a2a2a; }
    .form { margin: 10px 0; }
  </style>
</head>
<body>

  <!-- NAVBAR -->
  <header>
    <nav>
      <a href="#hausaufgaben">Hausaufgaben</a>
      <a href="#eltern">Elternsprechtage</a>
      <a href="#plaene">Pläne</a>
    </nav>
    <div>
      <button onclick="toggleTheme()">🌙/☀️</button>
      <button onclick="logout()" class="btn-red">Logout</button>
    </div>
  </header>

  <!-- LOGIN -->
  <div id="login" class="container">
    <h1># ANMELDEN</h1>
    <input type="text" id="username" placeholder="Benutzername"><br>
    <input type="password" id="password" placeholder="Passwort"><br>
    <input type="text" id="secret" placeholder="Geheime ID"><br>
    <button onclick="login()" class="btn-blue">Login</button>
    <p id="error" style="color:red"></p>
  </div>

  <!-- INHALTE -->
  <div id="content" class="container hidden">

    <!-- HAUSAUFGABEN -->
    <div id="hausaufgaben" class="card">
      <h2>📘 Hausaufgaben</h2>
      <div id="formHausaufgaben" class="form hidden">
        <select id="fach">
          <option>Mathe</option>
          <option>Deutsch</option>
          <option>Englisch</option>
          <option>Bio</option>
          <option>Chemie</option>
        </select>
        <select id="typ">
          <option>Buch</option>
          <option>Arbeitsheft</option>
        </select>
        <input type="text" id="seite" placeholder="Seite">
        <input type="text" id="nummer" placeholder="Nummer">
        <input type="date" id="deadline">
        <button onclick="saveHomework()" class="btn-blue">Speichern</button>
      </div>
      <button id="btnHausaufgaben" class="btn-blue hidden" onclick="toggleForm('formHausaufgaben')">Neue Aufgabe</button>
      <div id="listHausaufgaben"></div>
    </div>

    <!-- ELTERNSPRECHTAGE -->
    <div id="eltern" class="card">
      <h2>👨‍👩‍👧 Elternsprechtage</h2>
      <div id="formEltern" class="form hidden">
        <input type="date" id="elternDatum">
        <input type="time" id="elternZeit">
        <input type="text" id="elternInfo" placeholder="Bemerkung">
        <button onclick="saveParent()" class="btn-green">Speichern</button>
      </div>
      <button id="btnEltern" class="btn-green hidden" onclick="toggleForm('formEltern')">Neuer Termin</button>
      <div id="listEltern"></div>
    </div>

    <!-- PLÄNE -->
    <div id="plaene" class="card">
      <h2>📅 Pläne</h2>
      <div id="formPlaene" class="form hidden">
        <input type="text" id="planUrl" placeholder="Bild-URL">
        <button onclick="savePlan()" class="btn-orange">Speichern</button>
      </div>
      <button id="btnPlaene" class="btn-orange hidden" onclick="toggleForm('formPlaene')">Neuer Plan</button>
      <div id="listPlaene"></div>
    </div>

  </div>

  <script>
    // User-Daten
    const users = [
      {username:"admin", password:"1234", secret:"025", role:"admin"},
      {username:"user", password:"abcd", secret:"024", role:"viewer"}
    ];

    // Login
    function login(){
      const uname=document.getElementById("username").value;
      const pass=document.getElementById("password").value;
      const sec=document.getElementById("secret").value;
      const user=users.find(u=>u.username===uname&&u.password===pass&&u.secret===sec);

      if(user){
        localStorage.setItem("role",user.role);
        showContent(user.role);
        document.getElementById("login").style.display="none";
      } else {
        document.getElementById("error").innerText="Login fehlgeschlagen!";
      }
    }

    // Logout
    function logout(){
      localStorage.clear();
      location.reload();
    }

    // Inhalt anzeigen
    function showContent(role){
      document.getElementById("content").classList.remove("hidden");
      if(role==="admin"){
        document.getElementById("btnHausaufgaben").classList.remove("hidden");
        document.getElementById("btnEltern").classList.remove("hidden");
        document.getElementById("btnPlaene").classList.remove("hidden");
      }
      loadData();
    }

    function toggleForm(id){
      document.getElementById(id).classList.toggle("hidden");
    }

    // --- Hausaufgaben ---
    function saveHomework(){
      const hw={
        fach:document.getElementById("fach").value,
        typ:document.getElementById("typ").value,
        seite:document.getElementById("seite").value,
        nummer:document.getElementById("nummer").value,
        deadline:document.getElementById("deadline").value
      };
      let list=JSON.parse(localStorage.getItem("hausaufgaben"))||[];
      list.push(hw);
      localStorage.setItem("hausaufgaben",JSON.stringify(list));
      loadData();
    }

    // --- Eltern ---
    function saveParent(){
      const termin={
        datum:document.getElementById("elternDatum").value,
        zeit:document.getElementById("elternZeit").value,
        info:document.getElementById("elternInfo").value
      };
      let list=JSON.parse(localStorage.getItem("eltern"))||[];
      list.push(termin);
      localStorage.setItem("eltern",JSON.stringify(list));
      loadData();
    }

    // --- Pläne ---
    function savePlan(){
      const url=document.getElementById("planUrl").value;
      if(url){
        let list=JSON.parse(localStorage.getItem("plaene"))||[];
        list.push(url);
        localStorage.setItem("plaene",JSON.stringify(list));
        loadData();
      }
    }

    // --- Löschen ---
    function deleteItem(type,index){
      let list=JSON.parse(localStorage.getItem(type))||[];
      list.splice(index,1);
      localStorage.setItem(type,JSON.stringify(list));
      loadData();
    }

    // --- Laden ---
    function loadData(){
      const role=localStorage.getItem("role");

      // Hausaufgaben
      const hwDiv=document.getElementById("listHausaufgaben");
      hwDiv.innerHTML="";
      let hws=JSON.parse(localStorage.getItem("hausaufgaben"))||[];
      hws.forEach((hw,i)=>{
        const div=document.createElement("div");
        div.className="item";
        let deadlineText=hw.deadline?` (bis ${hw.deadline})`:"";
        div.innerHTML=`<b>${hw.fach}</b> - ${hw.typ}, Seite ${hw.seite}, Nr. ${hw.nummer}${deadlineText}`;
        if(role==="admin") div.innerHTML+=` <button class="btn-red" onclick="deleteItem('hausaufgaben',${i})">Löschen</button>`;
        hwDiv.appendChild(div);
      });

      // Eltern
      const elternDiv=document.getElementById("listEltern");
      elternDiv.innerHTML="";
      let eltern=JSON.parse(localStorage.getItem("eltern"))||[];
      eltern.forEach((t,i)=>{
        const div=document.createElement("div");
        div.className="item";
        div.innerHTML=`${t.datum} - ${t.zeit} (${t.info})`;
        if(role==="admin") div.innerHTML+=` <button class="btn-red" onclick="deleteItem('eltern',${i})">Löschen</button>`;
        elternDiv.appendChild(div);
      });

      // Pläne
      const plaeneDiv=document.getElementById("listPlaene");
      plaeneDiv.innerHTML="";
      let plaene=JSON.parse(localStorage.getItem("plaene"))||[];
      plaene.forEach((url,i)=>{
        const div=document.createElement("div");
        div.className="item";
        div.innerHTML=`<img src="${url}" width="200">`;
        if(role==="admin") div.innerHTML+=` <button class="btn-red" onclick="deleteItem('plaene',${i})">Löschen</button>`;
        plaeneDiv.appendChild(div);
      });
    }

    // Dark Mode
    function toggleTheme(){
      document.body.classList.toggle("dark");
    }

    // Schon eingeloggt?
    window.onload=()=>{
      const role=localStorage.getItem("role");
      if(role){
        showContent(role);
        document.getElementById("login").style.display="none";
      }
    }
  </script>

</body>
</html>

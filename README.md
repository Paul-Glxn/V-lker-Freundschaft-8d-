 <!DOCTYPE html>
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
      <button id="btnHausaufgaben" class="hidden" onclick="addItem('hausaufgaben')">Hinzufügen</button>
      <ul id="hausaufgaben"></ul>
    </div>

    <div class="section">
      <h2>Elternsprechtage</h2>
      <button id="btnEltern" class="hidden" onclick="addItem('eltern')">Hinzufügen</button>
      <ul id="eltern"></ul>
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

      // Buttons nur für Admin sichtbar machen
      if(role==="admin"){
        document.getElementById("btnHausaufgaben").classList.remove("hidden");
        document.getElementById("btnEltern").classList.remove("hidden");
        document.getElementById("btnPlaene").classList.remove("hidden");
      }

      loadData();
    }

    // Daten hinzufügen (Text)
    function addItem(type){
      const text = prompt("Bitte neuen Eintrag hinzufügen:");
      if(text){
        let items = JSON.parse(localStorage.getItem(type)) || [];
        items.push(text);
        localStorage.setItem(type, JSON.stringify(items));
        loadData();
      }
    }

    // Plan (Bild) hinzufügen
    function addPlan(){
      const url = prompt("Bild-URL eingeben:");
      if(url){
        let items = JSON.parse(localStorage.getItem("plaene")) || [];
        items.push(url);
        localStorage.setItem("plaene", JSON.stringify(items));
        loadData();
      }
    }

    // Daten laden
    function loadData(){
      ["hausaufgaben","eltern"].forEach(type => {
        const list = document.getElementById(type);
        list.innerHTML="";
        let items = JSON.parse(localStorage.getItem(type)) || [];
        items.forEach(i => {
          const li = document.createElement("li");
          li.innerText = i;
          list.appendChild(li);
        });
      });

      const plaeneDiv = document.getElementById("plaene");
      plaeneDiv.innerHTML="";
      let plaene = JSON.parse(localStorage.getItem("plaene")) || [];
      plaene.forEach(url => {
        const img = document.createElement("img");
        img.src = url;
        img.width = 200;
        img.style.margin="5px";
        plaeneDiv.appendChild(img);
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


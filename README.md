# number-of-people-who-support-Palestine
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Compteur avec utilisateurs et tableau</title>
  <style>
    body, html {
      margin: 0; padding: 0; height: 100%;
      font-family: Arial, sans-serif;
      background-image: url('https://upload.wikimedia.org/wikipedia/commons/thumb/0/00/Flag_of_Palestine.svg/2560px-Flag_of_Palestine.svg.png');
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
      color: white;
      text-align: center;
      padding: 50px;
      position: relative;
      height: 100vh;
      overflow: auto;
    }
    body::before {
      content: "";
      position: absolute;
      top: 0; left: 0; right: 0; bottom: 0;
      background-color: rgba(0, 0, 0, 0.6);
      z-index: 0;
    }
    .contenu {
      position: relative;
      z-index: 1;
      max-width: 450px;
      margin: auto;
      padding: 30px;
      border-radius: 10px;
      background-color: rgba(0,0,0,0.5);
    }
    h1 {
      font-size: 2.5em;
      text-shadow: 2px 2px 5px black;
      margin-bottom: 20px;
    }
    #compteur {
      font-size: 48px;
      margin: 20px 0;
      font-weight: bold;
      text-shadow: 2px 2px 5px black;
    }
    button {
      background-color: rgba(255, 255, 255, 0.2);
      color: white;
      padding: 10px 20px;
      font-size: 20px;
      border: 2px solid white;
      border-radius: 5px;
      cursor: pointer;
      transition: background-color 0.3s;
      text-shadow: 1px 1px 2px black;
      margin-top: 10px;
    }
    button:hover {
      background-color: rgba(255, 255, 255, 0.4);
    }
    input[type="text"], input[type="password"] {
      width: 80%;
      padding: 10px;
      margin: 10px 0;
      font-size: 16px;
      border-radius: 5px;
      border: none;
    }
    #message {
      margin: 10px 0;
      height: 24px;
      color: #ffbbbb;
      text-shadow: none;
    }
    table {
      margin: 20px auto 0;
      border-collapse: collapse;
      width: 90%;
      background-color: rgba(0,0,0,0.7);
      color: white;
      border: 2px solid #111;
      border-radius: 8px;
      overflow: hidden;
    }
    th, td {
      padding: 10px;
      border: 1px solid #222;
      text-align: center;
    }
    th {
      background-color: rgba(255,255,255,0.1);
      font-weight: bold;
    }
  </style>
</head>
<body>

  <!-- Section inscription -->
  <div class="contenu" id="signupSection">
    <h1>Inscription</h1>
    <form id="signupForm">
      <input type="text" id="signupUsername" placeholder="Nom d'utilisateur" required />
      <input type="password" id="signupPassword" placeholder="Mot de passe" required />
      <button type="submit">S'inscrire</button>
    </form>
    <div id="signupMessage"></div>
    <p>Tu as déjà un compte ? <button id="showLoginBtn" style="font-size:14px; background:none; border:none; color:#aad; cursor:pointer;">Se connecter</button></p>
  </div>

  <!-- Section connexion -->
  <div class="contenu" id="loginSection" style="display:none;">
    <h1>Connexion</h1>
    <form id="loginForm">
      <input type="text" id="loginUsername" placeholder="Nom d'utilisateur" required />
      <input type="password" id="loginPassword" placeholder="Mot de passe" required />
      <button type="submit">Se connecter</button>
    </form>
    <div id="loginMessage"></div>
    <p>Pas encore inscrit ? <button id="showSignupBtn" style="font-size:14px; background:none; border:none; color:#aad; cursor:pointer;">S'inscrire</button></p>
  </div>

  <!-- Section compteur & tableau -->
  <div class="contenu" id="compteurSection" style="display:none;">
    <h1>Bonjour <span id="userDisplay"></span></h1>
    <div id="compteur">0</div>
    <button id="btnIncrement">Clique ici pour soutenir</button>
    <button id="btnLogout" style="background-color:rgba(200,50,50,0.7); margin-left:10px;">Déconnexion</button>

    <h2>Statistiques des utilisateurs</h2>
    <table id="userTable">
      <thead>
        <tr>
          <th>Nom d'utilisateur</th>
          <th>Connexions</th>
          <th>Soutiens</th>
        </tr>
      </thead>
      <tbody>
        <!-- Rempli par JS -->
      </tbody>
    </table>
  </div>

<script>
  let users = JSON.parse(localStorage.getItem('users')) || {};
  let currentUser = null;

  const signupSection = document.getElementById('signupSection');
  const loginSection = document.getElementById('loginSection');
  const compteurSection = document.getElementById('compteurSection');
  const signupForm = document.getElementById('signupForm');
  const loginForm = document.getElementById('loginForm');
  const signupMessage = document.getElementById('signupMessage');
  const loginMessage = document.getElementById('loginMessage');
  const userDisplay = document.getElementById('userDisplay');
  const compteurDisplay = document.getElementById('compteur');
  const btnIncrement = document.getElementById('btnIncrement');
  const btnLogout = document.getElementById('btnLogout');
  const userTableBody = document.querySelector('#userTable tbody');
  const showLoginBtn = document.getElementById('showLoginBtn');
  const showSignupBtn = document.getElementById('showSignupBtn');

  showLoginBtn.addEventListener('click', () => {
    signupSection.style.display = 'none';
    loginSection.style.display = 'block';
    clearMessages();
  });

  showSignupBtn.addEventListener('click', () => {
    loginSection.style.display = 'none';
    signupSection.style.display = 'block';
    clearMessages();
  });

  function clearMessages() {
    signupMessage.textContent = '';
    loginMessage.textContent = '';
  }

  signupForm.addEventListener('submit', e => {
    e.preventDefault();
    const username = document.getElementById('signupUsername').value.trim();
    const password = document.getElementById('signupPassword').value;

    if(users[username]) {
      signupMessage.style.color = '#ffbbbb';
      signupMessage.textContent = 'Ce nom d\'utilisateur existe déjà.';
      return;
    }
    users[username] = {password: password, connexions: 0, soutiens: 0, lastSupportDate: null};
    saveUsers();
    signupMessage.style.color = 'lightgreen';
    signupMessage.textContent = 'Inscription réussie ! Connecte-toi.';
    signupForm.reset();
  });

  loginForm.addEventListener('submit', e => {
    e.preventDefault();
    const username = document.getElementById('loginUsername').value.trim();
    const password = document.getElementById('loginPassword').value;

    if(!users[username] || users[username].password !== password) {
      loginMessage.style.color = '#ffbbbb';
      loginMessage.textContent = 'Nom d\'utilisateur ou mot de passe incorrect.';
      return;
    }
    currentUser = username;
    users[currentUser].connexions++;
    saveUsers();
    showCompteurSection();
  });

  function showCompteurSection() {
    signupSection.style.display = 'none';
    loginSection.style.display = 'none';
    compteurSection.style.display = 'block';
    userDisplay.textContent = currentUser;
    updateCompteur();
    updateTable();
    clearMessages();
  }

  function updateCompteur() {
    if(currentUser && users[currentUser]) {
      compteurDisplay.textContent = users[currentUser].soutiens || 0;
    } else {
      compteurDisplay.textContent = 0;
    }
  }

  function saveUsers() {
    localStorage.setItem('users', JSON.stringify(users));
  }

  // Fonction pour comparer dates (année, mois, jour)
  function isSameDay(date1, date2) {
    return date1.getFullYear() === date2.getFullYear() &&
           date1.getMonth() === date2.getMonth() &&
           date1.getDate() === date2.getDate();
  }

  btnIncrement.addEventListener('click', () => {
    if(currentUser && users[currentUser]) {
      const today = new Date();
      const lastDateStr = users[currentUser].lastSupportDate;
      const lastDate = lastDateStr ? new Date(lastDateStr) : null;

      if(lastDate && isSameDay(today, lastDate)) {
        alert('Tu as déjà soutenu aujourd\'hui. Reviens demain !');
        return;
      }
      users[currentUser].soutiens = (users[currentUser].soutiens || 0) + 1;
      users[currentUser].lastSupportDate = today.toISOString();
      saveUsers();
      updateCompteur();
      updateTable();
    }
  });

  btnLogout.addEventListener('click', () => {
    currentUser = null;
    compteurSection.style.display = 'none';
    loginSection.style.display = 'block';
    clearMessages();
  });

  function updateTable() {
    userTableBody.innerHTML = '';
    for(const user in users) {
      const tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${user}</td>
        <td>${users[user].connexions}</td>
        <td>${users[user].soutiens || 0}</td>
      `;
      userTableBody.appendChild(tr);
    }
  }

  // Au chargement, affichage inscription
  signupSection.style.display = 'block';
  loginSection.style.display = 'none';
  compteurSection.style.display = 'none';
</script>

</body>
</html>

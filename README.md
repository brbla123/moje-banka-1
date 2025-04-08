<!DOCTYPE html>
<html lang="cs">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Moje Banka</title>
  <link rel="icon" href="https://cdn-icons-png.flaticon.com/512/190/190411.png" type="image/png">
  <style>
    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }
    body {
      font-family: 'Arial', sans-serif;
      background: #f0f4f8;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      flex-direction: column;
    }
    .container {
      background: #fff;
      border-radius: 15px;
      padding: 30px;
      box-shadow: 0 10px 20px rgba(0,0,0,0.1);
      width: 350px;
      animation: fadeIn 0.5s ease-in-out;
      margin: 10px;
    }
    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(-20px); }
      to { opacity: 1; transform: translateY(0); }
    }
    h2 {
      text-align: center;
      color: #333;
      margin-bottom: 20px;
    }
    input, button, select {
      width: 100%;
      padding: 12px;
      margin: 10px 0;
      border-radius: 8px;
      border: 1px solid #ddd;
      font-size: 1em;
    }
    input:focus, select:focus {
      outline: none;
      border-color: #3498db;
    }
    button {
      background-color: #3498db;
      color: white;
      border: none;
      cursor: pointer;
      font-weight: bold;
      transition: background 0.3s;
    }
    button:hover {
      background-color: #2980b9;
    }
    .hidden {
      display: none;
    }
    .balance {
      font-size: 1.5em;
      margin: 20px 0;
      color: #333;
      font-weight: bold;
    }
    .history {
      text-align: left;
      max-height: 150px;
      overflow-y: auto;
      margin-top: 20px;
    }
    .history ul {
      list-style-type: none;
    }
    .history li {
      margin: 5px 0;
      font-size: 0.9em;
      color: #555;
    }
    .dark-mode {
      background: #2c3e50;
      color: white;
    }
    .dark-mode button {
      background-color: #2ecc71;
    }
    .dark-mode input {
      background-color: #34495e;
      color: white;
    }
    .admin-section {
      background-color: #ecf0f1;
      padding: 20px;
      border-radius: 10px;
      margin-top: 20px;
    }
    .slide {
      display: none;
    }
    .slide.active {
      display: block;
    }
  </style>
</head>
<body>

<!-- Login Page -->
<div class="container" id="loginPage">
  <h2>Přihlášení</h2>
  <input type="text" id="username" placeholder="Uživatelské jméno" />
  <input type="password" id="password" placeholder="Heslo" />
  <button onclick="login()">Přihlásit se</button>
  <button onclick="showRegisterPage()">Registrace</button>
</div>

<!-- Bank Page -->
<div class="container hidden" id="bankPage">
  <h2>Moje Banka</h2>
  <div>
    <label for="accountSelect">Účet:</label>
    <select id="accountSelect" onchange="switchAccount()">
      <!-- Dynamicky se zde budou zobrazovat účty -->
    </select>
  </div>
  <div class="balance" id="balance">Zůstatek: 10 000 Kč</div>
  <input type="number" id="amount" placeholder="Částka k odeslání" />
  <input type="text" id="recipient" placeholder="Příjemce" />
  <button onclick="sendMoney()">Odeslat</button>
  <button onclick="toggleDarkMode()">Přepnout režim</button>
  <button onclick="logout()">Odhlásit se</button>
  <div class="history" id="history">
    <h3>Historie převodů</h3>
    <ul id="historyList"></ul>
  </div>
</div>

<!-- Admin Page -->
<div class="container hidden" id="adminPage">
  <h2>Admin Menu</h2>
  <div class="admin-section">
    <h3>Přidat peníze na účet</h3>
    <input type="text" id="adminAccountName" placeholder="Jméno účtu" />
    <input type="number" id="adminAmount" placeholder="Částka k přidání" />
    <button onclick="addMoneyToAccount()">Přidat peníze</button>
  </div>
  <div class="admin-section">
    <h3>Přidat informaci</h3>
    <textarea id="adminMessage" placeholder="Napište novou informaci..."></textarea>
    <button onclick="addInformation()">Přidat informaci</button>
  </div>
  <div class="admin-section">
    <h3>Spravovat slajdy</h3>
    <button onclick="showSlide('slide1')">Zobrazit Slide 1</button>
    <button onclick="showSlide('slide2')">Zobrazit Slide 2</button>
    <div class="slide" id="slide1">
      <h4>Slide 1 - Bankovní Přehled</h4>
      <p>Informace o účtech a transakcích.</p>
    </div>
    <div class="slide" id="slide2">
      <h4>Slide 2 - Admin Funkce</h4>
      <p>Správa účtů a peněz pro admina.</p>
    </div>
  </div>
  <button onclick="logout()">Odhlásit se</button>
</div>

<script>
  let users = {}; // Objekt pro uchovávání všech uživatelů
  let currentUser = null;
  
  // Funkce pro registraci
  function showRegisterPage() {
    const username = prompt("Zadejte uživatelské jméno:");
    const password = prompt("Zadejte heslo:");
    if (username && password) {
      users[username] = { password: password, balance: 10000, transactions: [] };
      alert("Registrace úspěšná! Můžete se přihlásit.");
    } else {
      alert("Zadejte platné údaje.");
    }
  }

  // Funkce pro přihlášení
  function login() {
    const username = document.getElementById('username').value.trim();
    const password = document.getElementById('password').value.trim();

    if (users[username] && users[username].password === password) {
      currentUser = username;
      document.getElementById('loginPage').classList.add('hidden');
      document.getElementById('bankPage').classList.remove('hidden');
      updateBalance();
      if (currentUser === 'admin') {
        document.getElementById('adminPage').classList.remove('hidden');
      } else {
        document.getElementById('adminPage').classList.add('hidden');
      }
    } else {
      alert("Chybný uživatel nebo heslo.");
    }
  }

  // Funkce pro odhlášení
  function logout() {
    currentUser = null;
    document.getElementById('loginPage').classList.remove('hidden');
    document.getElementById('bankPage').classList.add('hidden');
    document.getElementById('adminPage').classList.add('hidden');
    document.getElementById('username').value = '';
    document.getElementById('password').value = '';
  }

  // Funkce pro zobrazení zůstatku
  function updateBalance() {
    if (currentUser) {
      const balance = users[currentUser].balance;
      document.getElementById('balance').innerText = `Zůstatek: ${balance.toLocaleString('cs-CZ')} Kč`;
    }
  }

  // Funkce pro odeslání peněz
  function sendMoney() {
    const amount = parseFloat(document.getElementById('amount').value);
    const recipient = document.getElementById('recipient').value.trim();

    if (isNaN(amount) || amount <= 0 || recipient === "") {
      alert('Zadejte správnou částku a příjemce.');
      return;
    }

    if (!users[recipient]) {
      alert('Příjemce neexistuje.');
      return;
    }

    if (amount > users[currentUser].balance) {
      alert('Nemáte dostatek peněz.');
      return;
    }

    users[currentUser].balance -= amount;
    users[recipient].balance += amount;
    addHistory(`Odesláno ${amount.toLocaleString('cs-CZ')} Kč příjemci ${recipient}`);
    updateBalance();
    document.getElementById('amount').value = '';
    document.getElementById('recipient').value = '';
  }

  // Funkce pro přidání historie transakce
  function addHistory(transaction) {
    const historyList = document.getElementById('historyList');
    const li = document.createElement('li');
    li.textContent = transaction;
    historyList.appendChild(li);
  }

  // Admin funkce - přidání peněz na účet
  function addMoneyToAccount() {
    const accountName = document.getElementById('adminAccountName').value.trim();
    const amount = parseFloat(document.getElementById('adminAmount').value);

    if (accountName && amount > 0 && users[accountName]) {
      users[accountName].balance += amount;
      addHistory(`Admin přidal ${amount.toLocaleString('cs-CZ')} Kč na účet ${accountName}`);
      updateBalance();
      document.getElementById('adminAmount').value = '';
    } else {
      alert('Zadejte platné jméno účtu a částku.');
    }
  }

  // Funkce pro přidání informace
  function addInformation() {
    const message = document.getElementById('adminMessage').value.trim();
    if (message !== '') {
      alert(`Informace přidána: ${message}`);
      document.getElementById('adminMessage').value = '';
    }
  }

  // Funkce pro zobrazení slajdů
  function showSlide(slideId) {
    const slides = document.querySelectorAll('.slide');
    slides.forEach(slide => slide.classList.remove('active'));
    document.getElementById(slideId).classList.add('active');
  }
</script>

</body>
</html>



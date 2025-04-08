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
</div>

<!-- Bank Page -->
<div class="container hidden" id="bankPage">
  <h2>Moje Banka</h2>
  <div>
    <label for="accountSelect">Účet:</label>
    <select id="accountSelect" onchange="switchAccount()">
      <!-- Dynamicky se zde budou zobrazovat účty -->
    </select>
    <button onclick="addAccount()">Přidat nový účet</button>
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
    <select id="adminAccountSelect">
      <!-- Dynamicky se zde budou zobrazovat účty pro admina -->
    </select>
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
  let accounts = {
    current: { name: 'Běžný účet', balance: 10000 },
    savings: { name: 'Spořící účet', balance: 50000 }
  };
  let currentAccount = "current";
  let isAdmin = false;

  function login() {
    const username = document.getElementById('username').value.trim();
    const password = document.getElementById('password').value.trim();
    
    // Admin login (tady si můžeš přidat svou logiku pro admin přihlášení)
    if (username === 'admin' && password === 'admin123') {
      isAdmin = true;
      document.getElementById('adminPage').classList.remove('hidden');
      document.getElementById('loginPage').classList.add('hidden');
      updateAccountList();
      return;
    }
    
    if (username && password) {
      document.getElementById('bankPage').classList.remove('hidden');
      document.getElementById('loginPage').classList.add('hidden');
      updateAccountList();
      updateBalance();
    } else {
      alert('Zadejte uživatelské jméno a heslo!');
    }
  }

  function logout() {
    document.getElementById('loginPage').classList.remove('hidden');
    document.getElementById('bankPage').classList.add('hidden');
    document.getElementById('adminPage').classList.add('hidden');
    document.getElementById('username').value = '';
    document.getElementById('password').value = '';
    document.getElementById('historyList').innerHTML = '';
  }

  function updateAccountList() {
    const accountSelect = document.getElementById('accountSelect');
    const adminAccountSelect = document.getElementById('adminAccountSelect');
    
    accountSelect.innerHTML = '';
    adminAccountSelect.innerHTML = '';
    
    for (const key in accounts) {
      const option = document.createElement('option');
      option.value = key;
      option.textContent = accounts[key].name;
      accountSelect.appendChild(option);
      
      const adminOption = document.createElement('option');
      adminOption.value = key;
      adminOption.textContent = accounts[key].name;
      adminAccountSelect.appendChild(adminOption);
    }
  }

  function updateBalance() {
    const balanceEl = document.getElementById('balance');
    balanceEl.innerText = `Zůstatek: ${accounts[currentAccount].balance.toLocaleString('cs-CZ')} Kč`;
  }

  function sendMoney() {
    const amount = parseFloat(document.getElementById('amount').value);
    const recipient = document.getElementById('recipient').value.trim();
    if (isNaN(amount) || amount <= 0 || recipient === "") {
      alert('Zade

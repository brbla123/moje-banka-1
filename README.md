<!DOCTYPE html>
<html lang="cs">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Moje Banka</title>
  <link rel="icon" href="https://cdn-icons-png.flaticon.com/512/190/190411.png" type="image/png">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body { font-family: 'Arial', sans-serif; background: #f0f4f8; display: flex; justify-content: center; align-items: center; height: 100vh; flex-direction: column; }
    .container { background: #fff; border-radius: 15px; padding: 30px; box-shadow: 0 10px 20px rgba(0,0,0,0.1); width: 350px; margin: 10px; }
    h2 { text-align: center; color: #333; margin-bottom: 20px; }
    input, button, select { width: 100%; padding: 12px; margin: 10px 0; border-radius: 8px; border: 1px solid #ddd; font-size: 1em; }
    button { background-color: #3498db; color: white; border: none; cursor: pointer; font-weight: bold; transition: background 0.3s; }
    button:hover { background-color: #2980b9; }
    .balance { font-size: 1.5em; margin: 20px 0; color: #333; font-weight: bold; }
    .hidden { display: none; }
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

<!-- Register Page -->
<div class="container hidden" id="registerPage">
  <h2>Registrace</h2>
  <input type="text" id="newUsername" placeholder="Uživatelské jméno" />
  <input type="password" id="newPassword" placeholder="Heslo" />
  <input type="password" id="confirmPassword" placeholder="Potvrďte heslo" />
  <button onclick="register()">Registrovat</button>
  <button onclick="showLoginPage()">Zpět na přihlášení</button>
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
  <button onclick="logout()">Odhlásit se</button>
</div>

<script>
  let users = {}; // Objekt pro uchovávání všech uživatelů
  let currentUser = null;

  // Načtení uživatelů z LocalStorage
  function loadUsersFromLocalStorage() {
    const storedUsers = localStorage.getItem('users');
    if (storedUsers) {
      users = JSON.parse(storedUsers);
    } else {
      // Pokud není v LocalStorage žádný uživatel, vytvoříme admina
      users['admin'] = { password: '1', balance: 100000, transactions: [] };
      saveUsersToLocalStorage();
    }
  }

  // Uložení uživatelů do LocalStorage
  function saveUsersToLocalStorage() {
    localStorage.setItem('users', JSON.stringify(users));
  }

  // Funkce pro přepnutí na přihlašovací stránku
  function showLoginPage() {
    document.getElementById('registerPage').classList.add('hidden');
    document.getElementById('loginPage').classList.remove('hidden');
  }

  // Funkce pro přepnutí na registrační stránku
  function showRegisterPage() {
    document.getElementById('loginPage').classList.add('hidden');
    document.getElementById('registerPage').classList.remove('hidden');
  }

  // Funkce pro registraci
  function register() {
    const username = document.getElementById('newUsername').value.trim();
    const password = document.getElementById('newPassword').value.trim();
    const confirmPassword = document.getElementById('confirmPassword').value.trim();

    if (username && password && confirmPassword && password === confirmPassword) {
      if (users[username]) {
        alert("Toto uživatelské jméno je již zaregistrováno.");
      } else {
        users[username] = { password: password, balance: 10000, transactions: [] };
        saveUsersToLocalStorage(); // Uložení uživatelů
        alert("Registrace úspěšná! Můžete se přihlásit.");
        showLoginPage();
      }
    } else {
      alert("Zkontrolujte, zda jsou všechna pole správně vyplněná.");
    }
  }

  // Funkce pro přihlášení
  function login() {
    const username = document.getElementById('username').value.trim();
    const password = document.getElementById('password').value.trim();

    if (users[username] && users[username].password === password) {
      currentUser = username;
      document.getElementById('loginPage').classList.add('hidden');
      document.getElementById('registerPage').classList.add('hidden');
      document.getElementById('bankPage').classList.remove('hidden');
      updateBalance();
      updateAccountSelect();
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

  // Funkce pro zobrazení seznamu účtů v selectu
  function updateAccountSelect() {
    const accountSelect = document.getElementById('accountSelect');
    accountSelect.innerHTML = ''; // Vyprázdní seznam
    for (const user in users) {
      const option = document.createElement('option');
      option.value = user;
      option.textContent = user;
      accountSelect.appendChild(option);
    }
  }

  // Funkce pro přepnutí účtu
  function switchAccount() {
    const accountName = document.getElementById('accountSelect').value;
    if (accountName) {
      currentUser = accountName;
      updateBalance();
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

    // Odeslání peněz
    users[currentUser].balance -= amount;
    users[recipient].balance += amount;
    users[currentUser].transactions.push(`Odesláno ${amount.toLocaleString('cs-CZ')} Kč na účet ${recipient}`);
    users[recipient].transactions.push(`Přijato ${amount.toLocaleString('cs-CZ')} Kč od ${currentUser}`);
    
    saveUsersToLocalStorage(); // Uložení změn do LocalStorage
    updateBalance();
    document.getElementById('amount').value = '';
    document.getElementById('recipient').value = '';
  }

  // Admin funkce - přidání peněz na účet
  function addMoneyToAccount() {
    const accountName = document.getElementById('adminAccountName').value.trim();
    const amount = parseFloat(document.getElementById('adminAmount').value);

    if (accountName && amount > 0 && users[accountName]) {
      users[accountName].balance += amount;
      saveUsersToLocalStorage(); // Uložení změn do LocalStorage
      updateBalance();
      alert('Peněžitý zůstatek byl úspěšně přidán.');
      document.getElementById('adminAmount').value = '';
    } else {
      alert('Zadejte platné jméno účtu a částku.');
    }
  }

  // Funkce pro přepnutí mezi světlým a tmavým režimem
  function toggleDarkMode() {
    document.body.classList.toggle('dark-mode');
  }

  // Načtení uživatelů při načtení stránky
  loadUsersFromLocalStorage();
</script>

</body>
</html>


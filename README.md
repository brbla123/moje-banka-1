<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Bank App</title>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }
        h1 {
            color: #333;
        }
        input {
            margin: 5px;
        }
        button {
            margin: 5px;
            padding: 5px 10px;
        }
        ul {
            list-style-type: none;
            padding: 0;
        }
        li {
            margin: 10px 0;
            padding: 10px;
            border: 1px solid #ddd;
        }
    </style>
</head>
<body>
    <h1>Bank App</h1>
    <div>
        <input id="name" type="text" placeholder="Name">
        <input id="email" type="email" placeholder="Email">
        <input id="balance" type="number" placeholder="Balance">
        <button onclick="addUser()">Add User</button>
    </div>
    <ul id="userList"></ul>

    <script>
        async function fetchUsers() {
            const response = await axios.get('/users');
            const users = response.data;
            const userList = document.getElementById('userList');
            userList.innerHTML = '';
            users.forEach(user => {
                const li = document.createElement('li');
                li.textContent = `${user.name} - ${user.email} - $${user.balance}`;
                userList.appendChild(li);
            });
        }

        async function addUser() {
            const name = document.getElementById('name').value;
            const email = document.getElementById('email').value;
            const balance = document.getElementById('balance').value;
            const response = await axios.post('/users', { name, email, balance });
            const user = response.data;
            const userList = document.getElementById('userList');
            const li = document.createElement('li');
            li.textContent = `${user.name} - ${user.email} - $${user.balance}`;
            userList.appendChild(li);
            document.getElementById('name').value = '';
            document.getElementById('email').value = '';
            document.getElementById('balance').value = '';
        }

        fetchUsers();
    </script>
</body>
</html>

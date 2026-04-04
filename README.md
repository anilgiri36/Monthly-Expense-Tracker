<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Expense Tracker</title>
<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: #f4f7f8;
    margin: 0;
    padding: 0;
  }
  h1, h2 {
    text-align: center;
    color: #333;
  }
  .container {
    max-width: 800px;
    margin: 20px auto;
    padding: 20px;
    background: #fff;
    border-radius: 10px;
    box-shadow: 0 5px 15px rgba(0,0,0,0.1);
  }
  input, select, button {
    padding: 10px;
    margin: 5px 0;
    width: 100%;
    border-radius: 5px;
    border: 1px solid #ccc;
  }
  button {
    background: #E91E63;
    color: white;
    font-weight: bold;
    cursor: pointer;
    border: none;
    transition: .3s;
  }
  button:hover {
    background: #d81b60;
  }
  table {
    border-collapse: collapse;
    width: 100%;
    margin-top: 20px;
  }
  th, td {
    border: 1px solid #ddd;
    padding: 10px;
    text-align: center;
  }
  th {
    background-color: #E91E63;
    color: white;
  }
  tr:nth-child(even) {
    background-color: #f9f9f9;
  }
  .totals {
    display: flex;
    justify-content: space-around;
    margin-bottom: 20px;
  }
  .totals div {
    background: #E91E63;
    color: white;
    padding: 15px;
    border-radius: 8px;
    width: 30%;
    text-align: center;
    font-weight: bold;
  }
  .delete-btn {
    color: white;
    background: red;
    border-radius: 5px;
    padding: 5px 10px;
    cursor: pointer;
  }
  .section {
    background: #fafafa;
    padding: 15px;
    border-radius: 8px;
    margin-bottom: 20px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.05);
  }
  .hidden {
    display: none;
  }
</style>
</head>
<body>
<div class="container">
  <h1>Roommate Expense Tracker</h1>

  <div id="signinDiv" class="section">
    <h2>Sign In</h2>
    <form id="signinForm">
      <input type="text" id="username" placeholder="Username" required />
      <input type="password" id="password" placeholder="Password" required />
      <button type="submit">Sign In</button>
    </form>
  </div>

  <div id="adminDiv" class="hidden">
    <div class="section">
      <h2>Admin Panel - Set Roommate Accounts</h2>
      <form id="adminForm">
        <input type="text" id="roommateName1" placeholder="Roommate 1 Username" required />
        <input type="password" id="roommatePass1" placeholder="Roommate 1 Password" required />
        <input type="text" id="roommateName2" placeholder="Roommate 2 Username" required />
        <input type="password" id="roommatePass2" placeholder="Roommate 2 Password" required />
        <button type="submit">Save Roommate Accounts</button>
      </form>
    </div>

    <div class="section">
      <h2>Change Admin Password</h2>
      <form id="changePassForm">
        <input type="password" id="currentPass" placeholder="Current Password" required />
        <input type="password" id="newPass" placeholder="New Password" required />
        <input type="password" id="confirmPass" placeholder="Confirm New Password" required />
        <button type="submit">Change Password</button>
      </form>
    </div>
  </div>

  <div id="trackerDiv" class="hidden">
    <div class="totals">
      <div id="total1">Roommate 1: ₹0</div>
      <div id="total2">Roommate 2: ₹0</div>
      <div id="totalAll">Total: ₹0</div>
    </div>

    <div class="section">
      <h2>Add Expense</h2>
      <form id="expenseForm">
        <input type="date" id="date" required />
        <select id="name" required></select>
        <input type="number" id="amount" placeholder="Amount (₹)" min="1" required />
        <input type="text" id="desc" placeholder="Description (optional)" />
        <button type="submit">Add Expense</button>
      </form>
    </div>

    <table id="expenseTable">
      <thead>
        <tr>
          <th>Date</th><th>Name</th><th>Amount (₹)</th><th>Description</th><th>Action</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>
</div>

<script>
if(!localStorage.getItem('users')){
  localStorage.setItem('users', JSON.stringify([{username:'admin',password:'admin123',role:'admin'}]));
}
let users = JSON.parse(localStorage.getItem('users'));
let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
let currentUser = null;
let roommates = JSON.parse(localStorage.getItem('roommates')) || [];

const signinDiv = document.getElementById('signinDiv');
const adminDiv = document.getElementById('adminDiv');
const trackerDiv = document.getElementById('trackerDiv');
const signinForm = document.getElementById('signinForm');
const adminForm = document.getElementById('adminForm');
const changePassForm = document.getElementById('changePassForm');
const expenseForm = document.getElementById('expenseForm');
const nameSelect = document.getElementById('name');
const expenseTableBody = document.querySelector('#expenseTable tbody');
const total1P = document.getElementById('total1');
const total2P = document.getElementById('total2');
const totalAllP = document.getElementById('totalAll');

signinForm.addEventListener('submit', e => {
  e.preventDefault();
  const username = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value.trim();
  const user = users.find(u => u.username === username && u.password === password);
  if(!user){ alert('Invalid credentials'); return; }
  currentUser = user;
  signinDiv.classList.add('hidden');
  if(user.role === 'admin'){
    adminDiv.classList.remove('hidden');
    document.getElementById('roommateName1').value = roommates[0]?.username || '';
    document.getElementById('roommateName2').value = roommates[1]?.username || '';
  } else {
    trackerDiv.classList.remove('hidden');
    updateNameOptions();
    updateTable();
  }
});

adminForm.addEventListener('submit', e => {
  e.preventDefault();
  roommates = [
    {username: document.getElementById('roommateName1').value.trim(), password: document.getElementById('roommatePass1').value, role:'roommate'},
    {username: document.getElementById('roommateName2').value.trim(), password: document.getElementById('roommatePass2').value, role:'roommate'}
  ];
  users = users.filter(u => u.role !== 'roommate').concat(roommates);
  localStorage.setItem('users', JSON.stringify(users));
  localStorage.setItem('roommates', JSON.stringify(roommates));
  alert('Roommate accounts saved!');
});

changePassForm.addEventListener('submit', e => {
  e.preventDefault();
  const curr = document.getElementById('currentPass').value;
  const newP = document.getElementById('newPass').value;
  const confirmP = document.getElementById('confirmPass').value;
  if(curr !== currentUser.password){ alert('Current password incorrect'); return; }
  if(newP !== confirmP){ alert('New passwords do not match'); return; }
  currentUser.password = newP;
  const idx = users.findIndex(u => u.username === 'admin');
  users[idx].password = newP;
  localStorage.setItem('users', JSON.stringify(users));
  alert('Password changed!');
  changePassForm.reset();
});

function updateNameOptions(){
  nameSelect.innerHTML = '';
  roommates.forEach(r => {
    const opt = document.createElement('option');
    opt.value = r.username;
    opt.textContent = r.username;
    nameSelect.appendChild(opt);
  });
}

expenseForm.addEventListener('submit', e => {
  e.preventDefault();
  const date = document.getElementById('date').value;
  const name = document.getElementById('name').value;
  const amount = parseFloat(document.getElementById('amount').value);
  const desc = document.getElementById('desc').value.trim();
  if(!date || !name || !amount || amount <= 0){
    alert('Fill all fields correctly');
    return;
  }
  expenses.push({date, name, amount, desc});
  localStorage.setItem('expenses', JSON.stringify(expenses));
  updateTable();
  expenseForm.reset();
});

function deleteExpense(index){
  if(currentUser.role !== 'admin'){
    alert('Only admin can delete');
    return;
  }
  if(confirm('Are you sure to delete this expense?')){
    expenses.splice(index, 1);
    localStorage.setItem('expenses', JSON.stringify(expenses));
    updateTable();
  }
}

function updateTable(){
  expenseTableBody.innerHTML = '';
  let total = {};
  roommates.forEach(r => total[r.username] = 0);
  expenses.forEach(({date, name, amount, desc}, i) => {
    const tr = document.createElement('tr');
    const actionHTML = currentUser.role === 'admin' ? `<span class="delete-btn" onclick="deleteExpense(${i})">Delete</span>` : '';
    tr.innerHTML = `<td>${date}</td><td>${name}</td><td>₹${amount}</td><td>${desc || ''}</td><td>${actionHTML}</td>`;
    expenseTableBody.appendChild(tr);
    if(total[name] !== undefined) total[name] += amount;
  });
  total1P.textContent = `${roommates[0]?.username || 'Roommate 1'}: ₹${total[roommates[0]?.username] || 0}`;
  total2P.textContent = `${roommates[1]?.username || 'Roommate 2'}: ₹${total[roommates[1]?.username] || 0}`;
  totalAllP.textContent = `Total: ₹${(total[roommates[0]?.username] || 0) + (total[roommates[1]?.username] || 0)}`;
}
</script>
</body>
</html>

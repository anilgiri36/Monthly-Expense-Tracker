<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker</title>
<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background:#f4f7f8; margin:0; padding:0; }
h1,h2{text-align:center;color:#333;}
.container{max-width:900px;margin:20px auto;padding:20px;background:#fff;border-radius:10px;box-shadow:0 5px 15px rgba(0,0,0,0.1);}
input,select,button{padding:10px;margin:5px 0;width:100%;border-radius:5px;border:1px solid #ccc;}
button{background:#E91E63;color:white;font-weight:bold;cursor:pointer;border:none;transition:.3s;}
button:hover{background:#d81b60;}
table{border-collapse:collapse;width:100%;margin-top:20px;}
th,td{border:1px solid #ddd;padding:10px;text-align:center;}
th{background-color:#E91E63;color:white;}
tr:nth-child(even){background-color:#f9f9f9;}
.totals{display:flex;justify-content:space-around;margin-bottom:20px;flex-wrap:wrap;}
.totals div{background:#E91E63;color:white;padding:15px;border-radius:8px;width:28%;text-align:center;font-weight:bold;margin-bottom:10px;}
.delete-btn{color:white;background:red;border-radius:5px;padding:5px 10px;cursor:pointer;}
.section{background:#fafafa;padding:15px;border-radius:8px;margin-bottom:20px;box-shadow:0 2px 5px rgba(0,0,0,0.05);}
.hidden{display:none;}
.user-row{display:flex;justify-content:space-between;align-items:center;margin:5px 0;}
.user-row button{width:auto;margin-left:10px;padding:5px 10px;}
.legend{display:flex;justify-content:space-around;font-weight:bold;margin-bottom:15px;}
.legend span{padding:5px 10px;border-radius:5px;}
</style>
</head>
<body>

<div class="container">
<h1>Roommate Expense Tracker</h1>

<!-- MONTHLY BALANCE LEGEND -->
<div class="section">
  <h2>Monthly Balance Legend</h2>
  <div class="legend">
    <span style="color:green;">Owed – Paid more than your share</span>
    <span style="color:red;">Owes – Paid less than your share</span>
    <span style="color:black;">Settled – Even with your share</span>
  </div>
</div>

<!-- MONTHLY BALANCE DISPLAY -->
<div class="totals">
  <div id="monthTotal1">Roommate 1: ₹0</div>
  <div id="monthTotal2">Roommate 2: ₹0</div>
  <div id="monthTotal3">Roommate 3: ₹0</div>
  <div id="monthTotalAll">Total Paid: ₹0</div>
</div>

<!-- SIGN-IN FORM -->
<div id="signinDiv" class="section">
  <h2>Sign In</h2>
  <form id="signinForm">
    <input type="text" id="username" placeholder="Username or Mobile" required>
    <input type="password" id="password" placeholder="Password" required>
    <button type="submit">Sign In</button>
  </form>
</div>

<!-- ADMIN PANEL -->
<div id="adminDiv" class="hidden">
  <div class="section">
    <h2>Admin Panel - Roommates</h2>
    <form id="adminForm">
      <input type="text" id="roommateName1" placeholder="Roommate 1 Username">
      <input type="text" id="roommateMobile1" placeholder="Roommate 1 Mobile (optional)">
      <input type="password" id="roommatePass1" placeholder="Roommate 1 Password">
      
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username">
      <input type="text" id="roommateMobile2" placeholder="Roommate 2 Mobile (optional)">
      <input type="password" id="roommatePass2" placeholder="Roommate 2 Password">
      
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username">
      <input type="text" id="roommateMobile3" placeholder="Roommate 3 Mobile (optional)">
      <input type="password" id="roommatePass3" placeholder="Roommate 3 Password">
      
      <button type="submit">Save Roommates</button>
    </form>
    <h3>Existing Roommates</h3>
    <div id="roommateList"></div>
  </div>

  <div class="section">
    <h2>Change Admin Password</h2>
    <form id="changePassForm">
      <input type="password" id="currentPass" placeholder="Current Password" required>
      <input type="password" id="newPass" placeholder="New Password" required>
      <input type="password" id="confirmPass" placeholder="Confirm New Password" required>
      <button type="submit">Change Password</button>
    </form>
  </div>
</div>

<!-- EXPENSE TRACKER -->
<div id="trackerDiv" class="hidden">
  <div class="section">
    <h2>Add Expense</h2>
    <form id="expenseForm">
      <input type="date" id="date" required>
      <select id="name" required></select>
      <input type="number" id="amount" placeholder="Amount (₹)" min="1" required>
      <input type="text" id="desc" placeholder="Description (optional)">
      <button type="submit">Add Expense</button>
    </form>
  </div>

  <table id="expenseTable">
    <thead>
      <tr><th>Date</th><th>Name</th><th>Amount (₹)</th><th>Description</th><th>Action</th></tr>
    </thead>
    <tbody></tbody>
  </table>
</div>

<footer style="margin: 20px 0; color: gray;">
 <p>© 2026 Anil Giri. All rights reserved.</p>
</footer>

<script>
// Initialize users
if(!localStorage.getItem('users')){
  localStorage.setItem('users', JSON.stringify([{username:'admin',password:'admin123',role:'admin'}]));
}
let users = JSON.parse(localStorage.getItem('users'));
let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
let currentUser = null;
let roommates = JSON.parse(localStorage.getItem('roommates')) || [];

// DOM Elements
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
const total3P = document.getElementById('total3');
const totalAllP = document.getElementById('totalAll');
const roommateListDiv = document.getElementById('roommateList');
const monthTotal1 = document.getElementById('monthTotal1');
const monthTotal2 = document.getElementById('monthTotal2');
const monthTotal3 = document.getElementById('monthTotal3');
const monthTotalAll = document.getElementById('monthTotalAll');

// ------------------ FUNCTIONS ------------------ //

// Update name dropdown for expense entry
function updateNameOptions(){
  nameSelect.innerHTML = '';
  roommates.forEach(r=>{
    const opt = document.createElement('option');
    opt.value = r.username || r.mobile;
    opt.textContent = r.username || r.mobile;
    nameSelect.appendChild(opt);
  });
}

// ------------------ SIGN-IN ------------------ //
signinForm.addEventListener('submit', e=>{
  e.preventDefault();
  const username = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value.trim();
  const user = users.find(u=>(u.username===username || u.mobile===username) && u.password===password);
  if(!user){ alert('Invalid credentials'); return; }
  currentUser = user;
  signinDiv.classList.add('hidden');
  if(user.role==='admin'){
    adminDiv.classList.remove('hidden');
    updateRoommateList();
  } else {
    trackerDiv.classList.remove('hidden');
    updateNameOptions();
    updateTable();
  }
  updateMonthlyBalances();
});

// ------------------ ADMIN PANEL ------------------ //
adminForm.addEventListener('submit', e=>{
  e.preventDefault();
  roommates = [];
  const names = [
    document.getElementById('roommateName1').value.trim(),
    document.getElementById('roommateName2').value.trim(),
    document.getElementById('roommateName3').value.trim()
  ];
  const mobiles = [
    document.getElementById('roommateMobile1').value.trim(),
    document.getElementById('roommateMobile2').value.trim(),
    document.getElementById('roommateMobile3').value.trim()
  ];
  const passwords = [
    document.getElementById('roommatePass1').value,
    document.getElementById('roommatePass2').value,
    document.getElementById('roommatePass3').value
  ];
  
  // Check for duplicates
  let duplicateError = false;
  let allUsers = users.filter(u=>u.role!=='roommate').map(u=>u.username);
  let allMobiles = users.filter(u=>u.role!=='roommate').map(u=>u.mobile);
  
  for(let i=0;i<3;i++){
    if((names[i] && allUsers.includes(names[i])) || (mobiles[i] && allMobiles.includes(mobiles[i]))){
      duplicateError = true;
      break;
    }
  }
  if(duplicateError){ alert('Username or Mobile already exists!'); return; }

  for(let i=0;i<3;i++){
    if((names[i] || mobiles[i]) && passwords[i]){
      roommates.push({username:names[i],mobile:mobiles[i],password:passwords[i],role:'roommate'});
    }
  }

  // Update users
  users = users.filter(u=>u.role!=='roommate').concat(roommates);
  localStorage.setItem('users', JSON.stringify(users));
  localStorage.setItem('roommates', JSON.stringify(roommates));
  updateRoommateList();
  alert('Roommates saved!');
  adminForm.reset();
});

// ------------------ CHANGE PASSWORD ------------------ //
changePassForm.addEventListener('submit', e=>{
  e.preventDefault();
  const curr = document.getElementById('currentPass').value;
  const newP = document.getElementById('newPass').value;
  const confirmP = document.getElementById('confirmPass').value;
  if(curr !== currentUser.password){ alert('Current password incorrect'); return; }
  if(newP !== confirmP){ alert('New passwords do not match'); return; }
  currentUser.password = newP;
  const idx = users.findIndex(u=>u.username==='admin');
  users[idx].password = newP;
  localStorage.setItem('users', JSON.stringify(users));
  alert('Password changed!');
  changePassForm.reset();
});

// ------------------ EXPENSE ENTRY ------------------ //
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();
  const date = document.getElementById('date').value;
  const name = document.getElementById('name').value;
  const amount = parseFloat(document.getElementById('amount').value);
  const desc = document.getElementById('desc').value.trim();

  // Normal users can only add their own
  if(currentUser.role!=='admin' && name !== (currentUser.username || currentUser.mobile)){
    alert('You can only add expenses for yourself!');
    return;
  }

  if(!date || !name || !amount || amount <=0){ alert('Fill all fields correctly'); return; }
  expenses.push({date,name,amount,desc});
  localStorage.setItem('expenses', JSON.stringify(expenses));
  updateTable();
  updateMonthlyBalances();
  expenseForm.reset();
});

// ------------------ DELETE EXPENSE ------------------ //
function deleteExpense(index){
  if(currentUser.role!=='admin'){ alert('Only admin can delete'); return; }
  if(confirm('Are you sure to delete this expense?')){
    expenses.splice(index,1);
    localStorage.setItem('expenses', JSON.stringify(expenses));
    updateTable();
    updateMonthlyBalances();
  }
}

// ------------------ UPDATE TABLE ------------------ //
function updateTable(){
  expenseTableBody.innerHTML = '';
  let total = {};
  roommates.forEach(r=>total[r.username || r.mobile]=0);
  expenses.forEach(({date,name,amount,desc}, i)=>{
    const tr = document.createElement('tr');
    const actionHTML = currentUser.role==='admin' ? `<span class="delete-btn" onclick="deleteExpense(${i})">Delete</span>` : '';
    tr.innerHTML = `<td>${date}</td><td>${name}</td><td>₹${amount}</td><td>${desc||''}</td><td>${actionHTML}</td>`;
    expenseTableBody.appendChild(tr);
    if(total[name]!==undefined) total[name]+=amount;
  });
}

// ------------------ MONTHLY BALANCES ------------------ //
function updateMonthlyBalances() {
  const now = new Date();
  const month = now.getMonth();
  const year = now.getFullYear();

  let monthlyPaid = {};
  roommates.forEach(r => monthlyPaid[r.username || r.mobile] = 0);

  expenses.forEach(exp => {
    const expDate = new Date(exp.date);
    if(expDate.getMonth() === month && expDate.getFullYear() === year){
      if(monthlyPaid[exp.name] !== undefined) monthlyPaid[exp.name] += exp.amount;
    }
  });

  const totalMonth = Object.values(monthlyPaid).reduce((a,b)=>a+b,0);
  const numRoommates = roommates.length;
  const share = totalMonth / numRoommates;

  function formatBalance(nameKey){
    const diff = monthlyPaid[nameKey] - share;
    if(diff > 0) return `<span style="color:green;">Owed – ₹${diff.toFixed(0)}</span>`;
    else if(diff < 0) return `<span style="color:red;">Owes – ₹${Math.abs(diff).toFixed(0)}</span>`;
    else return `<span>Settled – ₹0</span>`;
  }

  monthTotal1.innerHTML = `${roommates[0]?.username || roommates[0]?.mobile || 'Roommate 1'}: ${formatBalance(roommates[0]?.username || roommates[0]?.mobile)}`;
  monthTotal2.innerHTML = `${roommates[1]?.username || roommates[1]?.mobile || 'Roommate 2'}: ${formatBalance(roommates[1]?.username || roommates[1]?.mobile)}`;
  monthTotal3.innerHTML = `${roommates[2]?.username || roommates[2]?.mobile || 'Roommate 3'}: ${formatBalance(roommates[2]?.username || roommates[2]?.mobile)}`;
  monthTotalAll.textContent = `Total Paid: ₹${totalMonth}`;
}

// ------------------ ROOMMATE LIST ------------------ //
function updateRoommateList(){
  roommateListDiv.innerHTML = '';
  roommates.forEach((r,i)=>{
    const div = document.createElement('div');
    div.className = 'user-row';
    div.innerHTML = `<span>${r.username || r.mobile}</span><button onclick="deleteRoommate(${i})">Delete</button>`;
    roommateListDiv.appendChild(div);
  });
  updateNameOptions();
}

// ------------------ DELETE ROOMMATE ------------------ //
function deleteRoommate(index){
  if(confirm('Are you sure to delete this roommate? This will not delete existing expenses.')){
    const removed = roommates.splice(index,1);
    users = users.filter(u=>u.username!==removed[0].username && u.mobile!==removed[0].mobile);
    localStorage.setItem('users', JSON.stringify(users));
    localStorage.setItem('roommates', JSON.stringify(roommates));
    updateRoommateList();
    alert('Roommate deleted!');
  }
}

// ------------------ INITIALIZE ------------------ //
updateNameOptions();
updateTable();
updateRoommateList();
updateMonthlyBalances();
</script>

</body>
</html>

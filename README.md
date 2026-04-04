<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker</title>
<style>
body { font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; margin:0; padding:0; background:#f7f9fc; color:#333; }
.container { max-width: 1000px; margin: 40px auto; padding:30px; background:#fff; border-radius:15px; box-shadow:0 8px 20px rgba(0,0,0,0.1); position:relative; }
h1 { text-align:center; font-size:2rem; color:#E91E63; margin-bottom:30px; }
.section { background:#f9f9f9; padding:25px; border-radius:12px; margin-bottom:25px; box-shadow:0 3px 8px rgba(0,0,0,0.05); }
input, select, button { width:100%; padding:12px; margin:10px 0; border-radius:8px; border:1px solid #ccc; font-size:1rem; }
input:focus, select:focus { border-color:#E91E63; outline:none; }
button { background:#E91E63; color:#fff; font-weight:bold; border:none; cursor:pointer; transition:.3s; box-shadow:0 4px 10px rgba(0,0,0,0.1); }
button:hover { background:#d81b60; }
#userInfo { position:absolute; top:20px; right:20px; display:flex; align-items:center; gap:10px; background:#fff; padding:10px 15px; border-radius:12px; box-shadow:0 3px 8px rgba(0,0,0,0.15); }
#userInfo img { width:45px; height:45px; border-radius:50%; border:2px solid #E91E63; }
#userInfo div { font-weight:bold; color:#E91E63; }
#userInfo button { background:none; border:none; font-size:20px; color:#E91E63; cursor:pointer; }
.totals { display:flex; gap:15px; flex-wrap:wrap; margin-bottom:20px; }
.totals div { flex:1 1 28%; background:#fff; padding:20px; border-radius:12px; text-align:center; font-weight:bold; box-shadow:0 4px 12px rgba(0,0,0,0.05); }
.owes { color:green; }
.owed { color:red; }
table { width:100%; border-collapse:collapse; border-radius:10px; overflow:hidden; box-shadow:0 4px 10px rgba(0,0,0,0.05); }
th, td { padding:12px; text-align:center; }
th { background:#E91E63; color:#fff; font-weight:600; }
tr:nth-child(even) { background:#f4f4f4; }
.delete-btn { color:#fff; background:red; border-radius:5px; padding:6px 12px; cursor:pointer; transition:.3s; }
.delete-btn:hover { background:darkred; }
@media(max-width:768px){ .totals { flex-direction:column; } }
</style>
</head>
<body>
<div class="container">
<h1>Roommate Expense Tracker</h1>

<div id="userInfo" class="hidden">
  <img id="userAvatar" src="https://i.pravatar.cc/45" alt="Avatar">
  <div id="userNameDisplay"></div>
  <button id="signOutBtn" title="Log Out">⎋</button>
</div>

<div id="signinDiv" class="section">
  <h2>Sign In</h2>
  <form id="signinForm">
    <input type="text" id="username" placeholder="Username or Mobile" required>
    <input type="password" id="password" placeholder="Password" required>
    <button type="submit">Sign In</button>
  </form>
</div>

<div id="adminDiv" class="hidden">
  <div class="section">
    <h2>Admin Panel - Roommates</h2>
    <form id="adminForm">
      <input type="text" id="roommateName1" placeholder="Roommate 1 Username">
      <input type="text" id="roommateMobile1" placeholder="Roommate 1 Mobile">
      <input type="password" id="roommatePass1" placeholder="Roommate 1 Password">
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username">
      <input type="text" id="roommateMobile2" placeholder="Roommate 2 Mobile">
      <input type="password" id="roommatePass2" placeholder="Roommate 2 Password">
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username">
      <input type="text" id="roommateMobile3" placeholder="Roommate 3 Mobile">
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

<div id="trackerDiv" class="hidden">
  <div class="totals">
    <div id="total1">Roommate 1: ₹0</div>
    <div id="total2">Roommate 2: ₹0</div>
    <div id="total3">Roommate 3: ₹0</div>
    <div id="totalAll">Total: ₹0</div>
  </div>
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

<footer style="margin:20px 0;color:gray;text-align:center;">
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
const userInfoDiv = document.getElementById('userInfo');
const userAvatar = document.getElementById('userAvatar');
const userNameDisplay = document.getElementById('userNameDisplay');
const signOutBtn = document.getElementById('signOutBtn');

// Sign-in
signinForm.addEventListener('submit', e=>{
  e.preventDefault();
  const username = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value.trim();
  const user = users.find(u=>(u.username===username || u.mobile===username) && u.password===password);
  if(!user){ alert('Invalid credentials'); return; }
  currentUser = user;
  signinDiv.classList.add('hidden');
  userInfoDiv.classList.remove('hidden');
  userNameDisplay.textContent = currentUser.username;
  userAvatar.src = `https://i.pravatar.cc/45?u=${currentUser.username}`;
  if(user.role==='admin'){
    adminDiv.classList.remove('hidden');
    updateRoommateList();
  } else {
    trackerDiv.classList.remove('hidden');
    updateNameOptions();
    updateTable();
  }
});

// Sign out
signOutBtn.addEventListener('click', ()=>{
  location.reload();
});

// Admin sets roommate accounts
adminForm.addEventListener('submit', e=>{
  e.preventDefault();
  let tempRoommates = [];
  const names = [document.getElementById('roommateName1').value.trim(), document.getElementById('roommateName2').value.trim(), document.getElementById('roommateName3').value.trim()];
  const mobiles = [document.getElementById('roommateMobile1').value.trim(), document.getElementById('roommateMobile2').value.trim(), document.getElementById('roommateMobile3').value.trim()];
  const passwords = [document.getElementById('roommatePass1').value, document.getElementById('roommatePass2').value, document.getElementById('roommatePass3').value];

  for(let i=0;i<3;i++){
    if(names[i] && passwords[i]){
      // Check duplicates
      if(users.some(u=>u.username===names[i])) { alert(`Username ${names[i]} already exists`); return; }
      if(mobiles[i] && users.some(u=>u.mobile===mobiles[i])) { alert(`Mobile ${mobiles[i]} already exists`); return; }
      tempRoommates.push({username:names[i],mobile:mobiles[i] || '',password:passwords[i],role:'roommate'});
    }
  }
  // Update users and roommates
  users = users.filter(u=>u.role!=='roommate').concat(tempRoommates);
  roommates = tempRoommates;
  localStorage.setItem('users', JSON.stringify(users));
  localStorage.setItem('roommates', JSON.stringify(roommates));
  updateRoommateList();
  alert('Roommates saved!');
  adminForm.reset();
});

// Change Admin Password
changePassForm.addEventListener('submit', e=>{
  e.preventDefault();
  const curr = document.getElementById('currentPass').value;
  const newP = document.getElementById('newPass').value;
  const confirmP = document.getElementById('confirmPass').value;
  if(curr!==currentUser.password){ alert('Current password incorrect'); return; }
  if(newP!==confirmP){ alert('New passwords do not match'); return; }
  currentUser.password = newP;
  const idx = users.findIndex(u=>u.username==='admin');
  users[idx].password = newP;
  localStorage.setItem('users', JSON.stringify(users));
  alert('Password changed!');
  changePassForm.reset();
});

// Update Name Dropdown
function updateNameOptions(){
  nameSelect.innerHTML = '';
  if(currentUser.role==='admin'){
    roommates.forEach(r=>{ const opt=document.createElement('option'); opt.value=r.username; opt.textContent=r.username; nameSelect.appendChild(opt); });
  } else { 
    const opt = document.createElement('option'); opt.value=currentUser.username; opt.textContent=currentUser.username; nameSelect.appendChild(opt);
  }
}

// Add Expense
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();
  const date = document.getElementById('date').value;
  const name = document.getElementById('name').value;
  const amount = parseFloat(document.getElementById('amount').value);
  const desc = document.getElementById('desc').value.trim();
  if(!date || !name || !amount || amount<=0){ alert('Fill all fields correctly'); return; }
  if(currentUser.role!=='admin' && name!==currentUser.username){ alert('You can only add your own expense'); return; }
  expenses.push({date,name,amount,desc});
  localStorage.setItem('expenses', JSON.stringify(expenses));
  updateTable();
  expenseForm.reset();
});

// Delete Expense
function deleteExpense(index){
  if(currentUser.role!=='admin'){ alert('Only admin can delete'); return; }
  if(confirm('Are you sure to delete this expense?')){
    expenses.splice(index,1);
    localStorage.setItem('expenses', JSON.stringify(expenses));
    updateTable();
  }
}

// Update Table & Totals
function updateTable(){
  expenseTableBody.innerHTML='';
  let total={}; roommates.forEach(r=>total[r.username]=0);
  expenses.forEach(({date,name,amount,desc},i)=>{
    const tr=document.createElement('tr');
    const actionHTML=currentUser.role==='admin'?`<span class="delete-btn" onclick="deleteExpense(${i})">Delete</span>`:'';
    tr.innerHTML=`<td>${date}</td><td>${name}</td><td>₹${amount}</td><td>${desc||''}</td><td>${actionHTML}</td>`;
    expenseTableBody.appendChild(tr);
    if(total[name]!==undefined) total[name]+=amount;
  });
  // Calculate owes/owed
  const sumAll = Object.values(total).reduce((a,b)=>a+b,0);
  const avg = sumAll/roommates.length;
  total1P.innerHTML = `${roommates[0]?.username || 'Roommate 1'}: <span class="${total[roommates[0]?.username]>=avg?'owes':'owed'}">₹${Math.abs(total[roommates[0]?.username]-avg).toFixed(0)}</span>`;
  total2P.innerHTML = `${roommates[1]?.username || 'Roommate 2'}: <span class="${total[roommates[1]?.username]>=avg?'owes':'owed'}">₹${Math.abs(total[roommates[1]?.username]-avg).toFixed(0)}</span>`;
  total3P.innerHTML = `${roommates[2]?.username || 'Roommate 3'}: <span class="${total[roommates[2]?.username]>=avg?'owes':'owed'}">₹${Math.abs(total[roommates[2]?.username]-avg).toFixed(0)}</span>`;
  totalAllP.textContent=`Total: ₹${sumAll}`;
}

// Update Roommate List
function updateRoommateList(){
  roommateListDiv.innerHTML='';
  roommates.forEach((r,i)=>{
    const div=document.createElement('div');
    div.className='user-row';
    div.innerHTML=`<span>${r.username}</span><button onclick="deleteRoommate(${i})">Delete</button>`;
    roommateListDiv.appendChild(div);
  });
  updateNameOptions();
}

// Delete Roommate
function deleteRoommate(index){
  if(confirm('Are you sure to delete this roommate? This will not delete existing expenses.')){
    const removed = roommates.splice(index,1);
    users = users.filter(u=>u.username!==removed[0].username);
    localStorage.setItem('users', JSON.stringify(users));
    localStorage.setItem('roommates', JSON.stringify(roommates));
    updateRoommateList();
    alert('Roommate deleted!');
  }
}

// Initialize
updateNameOptions();
updateTable();
updateRoommateList();
</script>
</body>
</html>

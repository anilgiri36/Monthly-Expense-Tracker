<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
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
.balance-info {font-weight:bold; margin-bottom:5px;}
.balance-owed {color: red;}
.balance-owes {color: green;}
</style>
</head>
<body>

<div class="container">
<h1>Roommate Expense Tracker</h1>

<div id="balanceSummary" class="totals" style="margin-bottom: 30px; flex-wrap: wrap;">
  <!-- Per user balance summary will be inserted here -->
</div>

<!-- SIGN-IN FORM -->
<div id="signinDiv" class="section">
  <h2>Sign In</h2>
  <form id="signinForm">
    <input type="text" id="username" placeholder="Username or Mobile" required />
    <input type="password" id="password" placeholder="Password" required />
    <button type="submit">Sign In</button>
  </form>
</div>

<!-- ADMIN PANEL -->
<div id="adminDiv" class="hidden">
  <div class="section">
    <h2>Admin Panel - Roommates</h2>
    <form id="adminForm">
      <input type="text" id="roommateName1" placeholder="Roommate 1 Username" />
      <input type="text" id="roommateMobile1" placeholder="Roommate 1 Mobile (optional)" />
      <input type="password" id="roommatePass1" placeholder="Roommate 1 Password" />
      
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username" />
      <input type="text" id="roommateMobile2" placeholder="Roommate 2 Mobile (optional)" />
      <input type="password" id="roommatePass2" placeholder="Roommate 2 Password" />
      
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username" />
      <input type="text" id="roommateMobile3" placeholder="Roommate 3 Mobile (optional)" />
      <input type="password" id="roommatePass3" placeholder="Roommate 3 Password" />
      
      <button type="submit">Save Roommates</button>
    </form>
    <h3>Existing Roommates</h3>
    <div id="roommateList"></div>
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

<!-- EXPENSE TRACKER -->
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
      <input type="date" id="date" required />
      <select id="name" required></select>
      <input type="number" id="amount" placeholder="Amount (₹)" min="1" required />
      <input type="text" id="desc" placeholder="Description (optional)" />
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
const balanceSummaryDiv = document.getElementById('balanceSummary');

// Sign-in
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
  updateBalanceSummary();
});

// Admin sets roommate accounts
adminForm.addEventListener('submit', e=>{
  e.preventDefault();
  let newRoommates = [];
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
  
  // Validate duplicates for usernames and mobiles
  const allUsernames = users.filter(u => u.role !== 'roommate').map(u => u.username);
  const allMobiles = users.filter(u => u.role !== 'roommate').map(u => u.mobile).filter(m => m);
  
  for(let i=0; i<3; i++){
    if(names[i] || mobiles[i]) {
      if(passwords[i].length === 0) {
        alert(`Password required for roommate #${i+1}`);
        return;
      }
      // Check duplicates
      if(names[i] && (allUsernames.includes(names[i]) || newRoommates.some(r=>r.username===names[i]))){
        alert(`Username "${names[i]}" already exists.`);
        return;
      }
      if(mobiles[i] && (allMobiles.includes(mobiles[i]) || newRoommates.some(r=>r.mobile===mobiles[i]))){
        alert(`Mobile "${mobiles[i]}" already exists.`);
        return;
      }
      newRoommates.push({username: names[i] || mobiles[i], mobile: mobiles[i] || '', password: passwords[i], role:'roommate'});
    }
  }
  
  // Update users and roommates
  users = users.filter(u=>u.role!=='roommate').concat(newRoommates);
  roommates = newRoommates;
  
  localStorage.setItem('users', JSON.stringify(users));
  localStorage.setItem('roommates', JSON.stringify(roommates));
  
  updateRoommateList();
  alert('Roommates saved!');
  adminForm.reset();
  updateBalanceSummary();
});

// Admin changes password
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

// Update name dropdown
function updateNameOptions(){
  nameSelect.innerHTML = '';
  roommates.forEach(r=>{
    const opt = document.createElement('option');
    opt.value = r.username; 
    opt.textContent = r.username + (r.mobile ? ` (${r.mobile})` : '');
    nameSelect.appendChild(opt);
  });
}

// Add expense
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();
  const date = document.getElementById('date').value;
  const name = document.getElementById('name').value;
  const amount = parseFloat(document.getElementById('amount').value);
  const desc = document.getElementById('desc').value.trim();
  if(!date || !name || !amount || amount <=0){ alert('Fill all fields correctly'); return; }
  // Validate that current user can only add expenses under their own username
  if(currentUser.username !== name){
    alert("You can only add expenses under your own username.");
    return;
  }
  expenses.push({date,name,amount,desc});
  localStorage.setItem('expenses', JSON.stringify(expenses));
  updateTable();
  updateBalanceSummary();
  expenseForm.reset();
});

// Delete expense - only admin
function deleteExpense(index){
  if(currentUser.role!=='admin'){ alert('Only admin can delete'); return; }
  if(confirm('Are you sure to delete this expense?')){
    expenses.splice(index,1);
    localStorage.setItem('expenses', JSON.stringify(expenses));
    updateTable();
    updateBalanceSummary();
  }
}

// Update table & totals
function updateTable(){
  expenseTableBody.innerHTML = '';
  let total = {};
  roommates.forEach(r=>total[r.username]=0);
  expenses.forEach(({date,name,amount,desc}, i)=>{
    const tr = document.createElement('tr');
    const actionHTML = currentUser.role==='admin' ? `<span class="delete-btn" onclick="deleteExpense(${i})">Delete</span>` : '';
    tr.innerHTML = `<td>${date}</td><td>${name}</td><td>₹${amount}</td><td>${desc||''}</td><td>${actionHTML}</td>`;
    expenseTableBody.appendChild(tr);
    if(total[name]!==undefined) total[name]+=amount;
  });
  total1P.textContent = `${roommates[0]?.username || 'Roommate 1'}: ₹${total[roommates[0]?.username]||0}`;
  total2P.textContent = `${roommates[1]?.username || 'Roommate 2'}: ₹${total[roommates[1]?.username]||0}`;
  total3P.textContent = `${roommates[2]?.username || 'Roommate 3'}: ₹${total[roommates[2]?.username]||0}`;
  const sumAll = Object.values(total).reduce((a,b)=>a+b,0);
  totalAllP.textContent = `Total: ₹${sumAll}`;
}

// Update roommate list for admin
function updateRoommateList(){
  roommateListDiv.innerHTML = '';
  roommates.forEach((r,i)=>{
    const div = document.createElement('div');
    div.className = 'user-row';
    div.innerHTML = `<span>${r.username}${r.mobile ? ' ('+r.mobile+')' : ''}</span><button onclick="deleteRoommate(${i})">Delete</button>`;
    roommateListDiv.appendChild(div);
  });
  updateNameOptions();
}

// Delete roommate
function deleteRoommate(index){
  if(confirm('Are you sure to delete this roommate? This will not delete existing expenses.')){
    const removed = roommates.splice(index,1);
    users = users.filter(u=>u.username!==removed[0].username);
    localStorage.setItem('users', JSON.stringify(users));
    localStorage.setItem('roommates', JSON.stringify(roommates));
    updateRoommateList();
    alert('Roommate deleted!');
    updateBalanceSummary();
  }
}

// Update balance summary for all roommates
function updateBalanceSummary(){
  if(!currentUser) return;
  balanceSummaryDiv.innerHTML = '';
  if(roommates.length === 0) return;

  // Calculate total expenses and per user totals for the current month
  let totalPaid = 0;
  let totals = {};
  roommates.forEach(r => { totals[r.username] = 0; });

  // Get current month/year
  const now = new Date();
  const currentMonth = now.getMonth();
  const currentYear = now.getFullYear();

  expenses.forEach(({date,name,amount})=>{
    const expenseDate = new Date(date);
    if(expenseDate.getMonth() === currentMonth && expenseDate.getFullYear() === currentYear){
      if(totals.hasOwnProperty(name)) totals[name] += amount;
      totalPaid += amount;
    }
  });

  // Calculate fair share per roommate
  const share = totalPaid / roommates.length;

  roommates.forEach(r => {
    const paid = totals[r.username];
    const balance = paid - share;
    const div = document.createElement('div');

    // Label and color according to balance
    let label = '';
    let colorClass = '';
    if(balance > 0){
      label = 'Owed';
      colorClass = 'balance-owed';
    } else if(balance < 0){
      label = 'Owes';
      colorClass = 'balance-owes';
    } else {
      label = 'Settled';
      colorClass = '';
    }
    div.className = `totals ${colorClass}`;
    div.innerHTML = `<div class="balance-info"><strong>${r.username}:</strong> ${label} – ₹${Math.abs(balance).toFixed(2)}</div>`;
    balanceSummaryDiv.appendChild(div);
  });

  // Total paid summary
  const totalDiv = document.createElement('div');
  totalDiv.className = 'totals';
  totalDiv.style.width = '28%';
  totalDiv.style.backgroundColor = '#E91E63';
  totalDiv.style.color = 'white';
  totalDiv.style.padding = '15px';
  totalDiv.style.borderRadius = '8px';
  totalDiv.style.textAlign = 'center';
  totalDiv.style.fontWeight = 'bold';
  totalDiv.style.marginBottom = '10px';
  totalDiv.textContent = `Total Paid: ₹${totalPaid.toFixed(2)}`;
  balanceSummaryDiv.appendChild(totalDiv);
}

// Initialize page
updateNameOptions();
updateTable();
updateRoommateList();
updateBalanceSummary();
</script>

<footer style="margin: 20px 0; color: gray; text-align:center;">
 <p>© 2026 Anil Giri. All rights reserved.</p>
</footer>

</body>
</html>

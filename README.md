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
.status-owed{color:#d32f2f; font-weight:bold; margin-right: 5px;}
.status-owes{color:#388e3c; font-weight:bold; margin-right: 5px;}
.amount-positive { color: #388e3c; font-weight: bold; }
.amount-negative { color: #d32f2f; font-weight: bold; }
</style>
</head>
<body>

<div class="container">
  <h1>Roommate Expense Tracker</h1>

  <!-- Monthly Summary -->
  <div class="totals" id="monthlySummary" style="margin-bottom:30px;"></div>

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

    <div class="totals" id="userTotals" style="margin-bottom: 20px;"></div>

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
</div>

<footer style="margin: 20px 0; color: gray; text-align:center;">
  <p>© 2026 Anil Giri. All rights reserved.</p>
</footer>

<script>
// Initialize users
if(!localStorage.getItem('users')){
  localStorage.setItem('users', JSON.stringify([{username:'admin',password:'admin123',role:'admin', mobile:''}]));
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
const roommateListDiv = document.getElementById('roommateList');
const monthlySummaryDiv = document.getElementById('monthlySummary');
const userTotalsDiv = document.getElementById('userTotals');

// Helper: Check for duplicate username or mobile
function isDuplicateUser(username, mobile, excludeIndex=-1){
  username = username.toLowerCase();
  mobile = mobile.trim();
  for(let i=0; i<roommates.length; i++){
    if(i === excludeIndex) continue;
    if(roommates[i].username.toLowerCase() === username) return 'Username already exists';
    if(mobile && roommates[i].mobile === mobile) return 'Mobile number already exists';
  }
  if(users.find(u=>u.username.toLowerCase() === username && u.role === 'admin') && excludeIndex === -1){
    return 'Username already exists';
  }
  return null;
}

// Sign-in
signinForm.addEventListener('submit', e=>{
  e.preventDefault();
  const usernameOrMobile = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value.trim();
  let user = users.find(u=>
    (u.username.toLowerCase() === usernameOrMobile.toLowerCase() || u.mobile === usernameOrMobile)
    && u.password === password);
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
    updateMonthlySummary();
  }
});

// Admin sets roommate accounts
adminForm.addEventListener('submit', e=>{
  e.preventDefault();
  roommates = [];
  let names = [
    document.getElementById('roommateName1').value.trim(),
    document.getElementById('roommateName2').value.trim(),
    document.getElementById('roommateName3').value.trim()
  ];
  let mobiles = [
    document.getElementById('roommateMobile1').value.trim(),
    document.getElementById('roommateMobile2').value.trim(),
    document.getElementById('roommateMobile3').value.trim()
  ];
  let passwords = [
    document.getElementById('roommatePass1').value,
    document.getElementById('roommatePass2').value,
    document.getElementById('roommatePass3').value
  ];

  for(let i=0;i<3;i++){
    if(names[i] && passwords[i]){
      let dup = isDuplicateUser(names[i], mobiles[i]);
      if(dup){
        alert(dup + ` at roommate ${i+1}`);
        return;
      }
      roommates.push({username:names[i], password:passwords[i], role:'roommate', mobile: mobiles[i] || ''});
    } else if(names[i] || passwords[i]) {
      alert('Please fill both username and password for roommate ' + (i+1));
      return;
    }
  }

  // Update users, keep admin intact
  users = users.filter(u=>u.role!=='roommate').concat(roommates);
  localStorage.setItem('users', JSON.stringify(users));
  localStorage.setItem('roommates', JSON.stringify(roommates));
  updateRoommateList();
  alert('Roommates saved!');
  adminForm.reset();
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

// Update name dropdown (only current user's name for roommates)
function updateNameOptions(){
  nameSelect.innerHTML = '';
  if(currentUser.role === 'admin'){
    roommates.forEach(r=>{
      const opt = document.createElement('option');
      opt.value = r.username; opt.textContent = r.username;
      nameSelect.appendChild(opt);
    });
  } else {
    // Normal user can only add expense for self
    const opt = document.createElement('option');
    opt.value = currentUser.username;
    opt.textContent = currentUser.username;
    nameSelect.appendChild(opt);
  }
}

// Add expense
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();
  const date = document.getElementById('date').value;
  const name = document.getElementById('name').value;
  const amount = parseFloat(document.getElementById('amount').value);
  const desc = document.getElementById('desc').value.trim();
  if(!date || !name || !amount || amount <=0){ alert('Fill all fields correctly'); return; }

  // Normal users can't add for others
  if(currentUser.role !== 'admin' && name !== currentUser.username){
    alert('You can only add expense for yourself.');
    return;
  }

  expenses.push({date,name,amount,desc});
  localStorage.setItem('expenses', JSON.stringify(expenses));
  updateTable();
  updateMonthlySummary();
  expenseForm.reset();
});

// Delete expense (only admin)
function deleteExpense(index){
  if(currentUser.role!=='admin'){ alert('Only admin can delete'); return; }
  if(confirm('Are you sure to delete this expense?')){
    expenses.splice(index,1);
    localStorage.setItem('expenses', JSON.stringify(expenses));
    updateTable();
    updateMonthlySummary();
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
    tr.innerHTML = `<td>${date}</td><td>${name}</td><td>₹${amount.toFixed(2)}</td><td>${desc||''}</td><td>${actionHTML}</td>`;
    expenseTableBody.appendChild(tr);
    if(total[name]!==undefined) total[name]+=amount;
  });
  // Show user totals section
  userTotalsDiv.innerHTML = '';
  roommates.forEach(r=>{
    const div = document.createElement('div');
    div.style.margin = '0 10px';
    div.style.display = 'inline-block';
    div.style.fontWeight = 'bold';
    div.textContent = `${r.username}: ₹${(total[r.username]||0).toFixed(2)}`;
    userTotalsDiv.appendChild(div);
  });
}

// Update roommate list for admin
function updateRoommateList(){
  roommateListDiv.innerHTML = '';
  roommates.forEach((r,i)=>{
    const div = document.createElement('div');
    div.className = 'user-row';
    div.innerHTML = `<span>${r.username} (${r.mobile || 'No mobile'})</span><button onclick="deleteRoommate(${i})">Delete</button>`;
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
  }
}

// Calculate and display monthly owed/owes summary on top
function updateMonthlySummary(){
  monthlySummaryDiv.innerHTML = '';
  if(roommates.length === 0) return;

  const now = new Date();
  const currentYear = now.getFullYear();
  const currentMonth = now.getMonth() + 1; // Jan = 0

  // Filter expenses for current month & year
  const currentMonthExpenses = expenses.filter(e=>{
    let [y,m] = e.date.split('-').map(Number);
    return y === currentYear && m === currentMonth;
  });

  const totalPaidPerUser = {};
  roommates.forEach(r => totalPaidPerUser[r.username] = 0);

  currentMonthExpenses.forEach(e => {
    if(totalPaidPerUser.hasOwnProperty(e.name)){
      totalPaidPerUser[e.name] += e.amount;
    }
  });

  const totalPaid = Object.values(totalPaidPerUser).reduce((a,b)=>a+b,0);
  const perUserShare = totalPaid / roommates.length;

  roommates.forEach(r=>{
    const paid = totalPaidPerUser[r.username] || 0;
    const diff = paid - perUserShare;
    const div = document.createElement('div');
    div.style.width = '28%';
    div.style.backgroundColor = '#E91E63';
    div.style.color = 'white';
    div.style.padding = '15px';
    div.style.borderRadius = '8px';
    div.style.textAlign = 'center';
    div.style.fontWeight = 'bold';
    div.style.marginBottom = '10px';

    let statusText = '';
    let amountClass = '';

    if(diff > 0){
      statusText = 'Owed';
      amountClass = 'amount-positive';
    } else if(diff < 0){
      statusText = 'Owes';
      amountClass = 'amount-negative';
    } else {
      statusText = 'Settled';
      amountClass = 'amount-positive';
    }

    div.innerHTML = `<span class="status-${statusText.toLowerCase()}">${r.username}: ${statusText} – </span><span class="${amountClass}">₹${Math.abs(diff).toFixed(2)}</span>`;
    monthlySummaryDiv.appendChild(div);
  });

  // Total Paid
  const totalDiv = document.createElement('div');
  totalDiv.style.width = '28%';
  totalDiv.style.backgroundColor = '#E91E63';
  totalDiv.style.color = 'white';
  totalDiv.style.padding = '15px';
  totalDiv.style.borderRadius = '8px';
  totalDiv.style.textAlign = 'center';
  totalDiv.style.fontWeight = 'bold';
  totalDiv.textContent = `Total Paid: ₹${totalPaid.toFixed(2)}`;
  monthlySummaryDiv.appendChild(totalDiv);
}

// Initialize on page load
updateNameOptions();
updateTable();
updateRoommateList();
updateMonthlySummary();
</script>

</body>
</html>

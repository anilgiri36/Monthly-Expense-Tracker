<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker</title>

<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background:#f4f7f8; margin:0; }
h1,h2{text-align:center;color:#333;}

.container{
  max-width:900px;margin:20px auto;padding:20px;
  background:#fff;border-radius:10px;
  box-shadow:0 5px 15px rgba(0,0,0,0.1);
}

input,select,button{
  padding:10px;margin:5px 0;width:100%;
  border-radius:5px;border:1px solid #ccc;
}

button{
  background:#E91E63;color:white;
  font-weight:bold;cursor:pointer;border:none;
}
button:hover{background:#d81b60;}

table{border-collapse:collapse;width:100%;margin-top:20px;}
th,td{border:1px solid #ddd;padding:10px;text-align:center;}
th{background-color:#E91E63;color:white;}
tr:nth-child(even){background-color:#f9f9f9;}

.totals{display:flex;justify-content:space-around;margin-bottom:20px;flex-wrap:wrap;}
.totals div{
  background:#E91E63;color:white;
  padding:15px;border-radius:8px;
  width:28%;text-align:center;
  font-weight:bold;margin-bottom:10px;
}

.delete-btn{
  color:white;background:red;
  border-radius:5px;padding:5px 10px;
  cursor:pointer;
}

.section{
  background:#fafafa;padding:15px;
  border-radius:8px;margin-bottom:20px;
}

.hidden{display:none;}

.footer{
  text-align:center;
  margin:20px 0;
  color:gray;
}
.footer p:last-child{
  color:green;
  font-weight:bold;
}
</style>
</head>

<body>

<div class="container">
<h1>Roommate Expense Tracker</h1>

<!-- SIGN-IN -->
<div id="signinDiv" class="section">
  <h2>Sign In</h2>
  <form id="signinForm">
    <input type="text" id="username" placeholder="Username" required>
    <input type="password" id="password" placeholder="Password" required>
    <button type="submit">Sign In</button>
  </form>
</div>

<!-- ADMIN -->
<div id="adminDiv" class="hidden">
  <div class="section">
    <h2>Admin Panel</h2>
    <form id="adminForm">
      <input type="text" id="roommateName1" placeholder="Roommate 1 Username">
      <input type="password" id="roommatePass1" placeholder="Password">
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username">
      <input type="password" id="roommatePass2" placeholder="Password">
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username">
      <input type="password" id="roommatePass3" placeholder="Password">
      <button type="submit">Save Roommates</button>
    </form>
    <div id="roommateList"></div>
  </div>
</div>

<!-- TRACKER -->
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
      <select id="name"></select>
      <input type="number" id="amount" placeholder="Amount (₹)" required>
      <input type="text" id="desc" placeholder="Description">
      <button type="submit">Add Expense</button>
    </form>
  </div>

  <table id="expenseTable">
    <thead>
      <tr>
        <th>Date</th>
        <th>Name</th>
        <th>Amount</th>
        <th>Description</th>
        <th id="actionHeader">Action</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

</div>
</div>

<!-- FOOTER -->
<div class="footer">
  <p>© 2026 Anil Giri. All rights reserved.</p>
  <p>A page created by Anil Giri</p>
</div>

<script>
// Default admin
if(!localStorage.getItem('users')){
  localStorage.setItem('users', JSON.stringify([
    {username:'admin',password:'admin123',role:'admin'}
  ]));
}

let users = JSON.parse(localStorage.getItem('users'));
let roommates = JSON.parse(localStorage.getItem('roommates')) || [];
let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
let currentUser = null;

// Elements
const signinForm = document.getElementById('signinForm');
const signinDiv = document.getElementById('signinDiv');
const adminDiv = document.getElementById('adminDiv');
const trackerDiv = document.getElementById('trackerDiv');
const nameSelect = document.getElementById('name');
const tableBody = document.querySelector('#expenseTable tbody');

// LOGIN
signinForm.addEventListener('submit', e=>{
  e.preventDefault();
  const u = username.value.trim();
  const p = password.value.trim();

  const user = users.find(x=>x.username===u && x.password===p);
  if(!user){ alert("Invalid login"); return; }

  currentUser = user;
  signinDiv.classList.add('hidden');

  // Hide action column for normal users
  document.getElementById('actionHeader').style.display =
    currentUser.role === 'admin' ? '' : 'none';

  if(user.role==='admin'){
    adminDiv.classList.remove('hidden');
  } else {
    trackerDiv.classList.remove('hidden');
    loadNames();
    render();
  }
});

// Load dropdown
function loadNames(){
  nameSelect.innerHTML='';

  if(currentUser.role === 'admin'){
    roommates.forEach(r=>{
      let o=document.createElement('option');
      o.value = r.username;
      o.textContent = r.username;
      nameSelect.appendChild(o);
    });
  } else {
    let o=document.createElement('option');
    o.value = currentUser.username;
    o.textContent = currentUser.username;
    nameSelect.appendChild(o);
    nameSelect.disabled = true;
  }
}

// Add expense
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();

  const selectedName = name.value;

  // 🔒 Restrict user
  if(currentUser.role !== 'admin' && selectedName !== currentUser.username){
    alert("You can only add your own expense!");
    return;
  }

  expenses.push({
    date:date.value,
    name:selectedName,
    amount:+amount.value,
    desc:desc.value
  });

  localStorage.setItem('expenses',JSON.stringify(expenses));
  render();
  expenseForm.reset();
});

// Render table
function render(){
  tableBody.innerHTML='';

  expenses.forEach((e,i)=>{
    let actionBtn = '';

    if(currentUser.role === 'admin'){
      actionBtn = `<span class="delete-btn" onclick="deleteExpense(${i})">Delete</span>`;
    }

    let tr=document.createElement('tr');
    tr.innerHTML=`
      <td>${e.date}</td>
      <td>${e.name}</td>
      <td>₹${e.amount}</td>
      <td>${e.desc || ''}</td>
      <td>${actionBtn}</td>
    `;
    tableBody.appendChild(tr);
  });
}

// Delete (admin only)
function deleteExpense(i){
  if(!currentUser || currentUser.role !== 'admin'){
    alert("Only admin can delete!");
    return;
  }

  if(confirm("Delete this expense?")){
    expenses.splice(i,1);
    localStorage.setItem('expenses',JSON.stringify(expenses));
    render();
  }
}
</script>

</body>
</html>

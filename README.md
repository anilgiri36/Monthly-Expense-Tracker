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
.status-info{text-align:center;margin-bottom:15px;font-weight:bold;}
.owed{color:#d32f2f;}
.owes{color:#388e3c;}
.settled{color:#555;}
footer{text-align:center;margin:20px 0;color:gray;}
marquee b{color:green;}
</style>
</head>
<body>

<div class="container">
<h1>Roommate Expense Tracker</h1>

<!-- Top balances display -->
<div id="balancesDiv" class="totals" style="margin-bottom:30px;"></div>

<!-- Sign-In -->
<div id="signinDiv" class="section">
  <h2>Sign In</h2>
  <form id="signinForm">
    <input type="text" id="username" placeholder="Username or Mobile" required>
    <input type="password" id="password" placeholder="Password" required>
    <button type="submit">Sign In</button>
  </form>
</div>

<!-- Admin Panel -->
<div id="adminDiv" class="hidden">
  <div class="section">
    <h2>Admin Panel - Roommates</h2>
    <form id="adminForm">
      <input type="text" id="roommateName1" placeholder="Roommate 1 Username">
      <input type="password" id="roommatePass1" placeholder="Roommate 1 Password">
      <input type="text" id="roommateMobile1" placeholder="Roommate 1 Mobile (optional)">
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username">
      <input type="password" id="roommatePass2" placeholder="Roommate 2 Password">
      <input type="text" id="roommateMobile2" placeholder="Roommate 2 Mobile (optional)">
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username">
      <input type="password" id="roommatePass3" placeholder="Roommate 3 Password">
      <input type="text" id="roommateMobile3" placeholder="Roommate 3 Mobile (optional)">
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

<!-- Expense Tracker -->
<div id="trackerDiv" class="hidden">
  <div class="totals">
    <div id="total1"></div>
    <div id="total2"></div>
    <div id="total3"></div>
    <div id="totalAll"></div>
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

<footer>
  <p>© 2026 Anil Giri. All rights reserved.</p>
</footer>
<marquee><b>A page created by Anil Giri</b></marquee>

<script>
// Initialize users and roommates
if(!localStorage.getItem('users')){
  localStorage.setItem('users', JSON.stringify([{username:'admin', password:'admin123', role:'admin'}]));
}
if(!localStorage.getItem('roommates')){
  localStorage.setItem('roommates', JSON.stringify([]));
}

let users = JSON.parse(localStorage.getItem('users'));
let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
let roommates = JSON.parse(localStorage.getItem('roommates')) || [];
let currentUser = null;

// DOM Elements
const signinDiv = document.getElementById('signinDiv');
const adminDiv = document.getElementById('adminDiv');
const trackerDiv = document.getElementById('trackerDiv');
const balancesDiv = document.getElementById('balancesDiv');
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

function saveUsers(){ localStorage.setItem('users', JSON.stringify(users)); }
function saveRoommates(){ localStorage.setItem('roommates', JSON.stringify(roommates)); }
function saveExpenses(){ localStorage.setItem('expenses', JSON.stringify(expenses)); }

// Update balances summary
function updateBalancesSummary(){
  if(roommates.length === 0){ balancesDiv.innerHTML = '<div>No roommates added yet.</div>'; return; }
  let totalPaid = 0, totalByUser = {};
  roommates.forEach(r => totalByUser[r.username]=0);
  expenses.forEach(({name,amount}) => { if(totalByUser[name]!==undefined){ totalByUser[name]+=amount; totalPaid+=amount; } });
  const equalShare = totalPaid/roommates.length;
  balancesDiv.innerHTML = '';
  roommates.forEach(r=>{
    const diff = totalByUser[r.username]-equalShare;
    let statusText='', className='';
    if(diff>0){statusText='Owed';className='owed';}
    else if(diff<0){statusText='Owes';className='owes';}
    else{statusText='Settled';className='settled';}
    const amountText=`₹${Math.abs(diff).toFixed(2)}`;
    const div=document.createElement('div');
    div.className=className;
    div.style.width='28%';
    div.style.background='#E91E63';
    div.style.color='white';
    div.style.padding='15px';
    div.style.borderRadius='8px';
    div.style.textAlign='center';
    div.style.fontWeight='bold';
    div.style.marginBottom='10px';
    div.textContent=`${r.username}: ${statusText} – ${amountText}`;
    balancesDiv.appendChild(div);
  });
  const totalDiv=document.createElement('div');
  totalDiv.style.width='28%'; totalDiv.style.background='#E91E63'; totalDiv.style.color='white';
  totalDiv.style.padding='15px'; totalDiv.style.borderRadius='8px'; totalDiv.style.textAlign='center';
  totalDiv.style.fontWeight='bold'; totalDiv.style.marginBottom='10px';
  totalDiv.textContent=`Total Paid: ₹${totalPaid.toFixed(2)}`;
  balancesDiv.appendChild(totalDiv);
}

// Update roommate list in admin panel
function updateRoommateList(){
  roommateListDiv.innerHTML='';
  roommates.forEach((r,i)=>{
    const div=document.createElement('div');
    div.className='user-row';
    div.innerHTML=`<span>${r.username} (${r.mobile||'No mobile'})</span> <button onclick="deleteRoommate(${i})">Delete</button>`;
    roommateListDiv.appendChild(div);
  });
  updateNameOptions();
  updateBalancesSummary();
}

// Delete roommate
window.deleteRoommate=function(index){
  if(confirm('Are you sure to delete this roommate? This will not delete existing expenses.')){
    const removed=roommates.splice(index,1);
    users=users.filter(u=>u.username!==removed[0].username);
    saveUsers(); saveRoommates();
    updateRoommateList();
    alert('Roommate deleted!');
  }
}

// Update name dropdown for expense
function updateNameOptions(){
  nameSelect.innerHTML='';
  roommates.forEach(r=>{
    const opt=document.createElement('option');
    opt.value=r.username;
    opt.textContent=r.username;
    nameSelect.appendChild(opt);
  });
}

// Sign-in
signinForm.addEventListener('submit', e=>{
  e.preventDefault();
  const inputVal=document.getElementById('username').value.trim();
  const password=document.getElementById('password').value.trim();
  const user=users.find(u=>(u.username===inputVal||u.mobile===inputVal)&&u.password===password);
  if(!user){alert('Invalid credentials'); return;}
  currentUser=user; signinDiv.classList.add('hidden'); balancesDiv.style.display='none';
  if(user.role==='admin'){adminDiv.classList.remove('hidden');}
  else{trackerDiv.classList.remove('hidden'); updateNameOptions(); updateTable();}
});

// Admin sets roommates
adminForm.addEventListener('submit', e=>{
  e.preventDefault();
  const names=[document.getElementById('roommateName1').value.trim(),document.getElementById('roommateName2').value.trim(),document.getElementById('roommateName3').value.trim()];
  const passwords=[document.getElementById('roommatePass1').value,document.getElementById('roommatePass2').value,document.getElementById('roommatePass3').value];
  const mobiles=[document.getElementById('roommateMobile1').value.trim(),document.getElementById('roommateMobile2').value.trim(),document.getElementById('roommateMobile3').value.trim()];
  let errorMsg='', newRoommates=[];
  for(let i=0;i<3;i++){
    if(names[i]||mobiles[i]){
      if(!passwords[i]){errorMsg='Password required for every roommate with username or mobile.'; break;}
      if(names[i]&&newRoommates.some(r=>r.username===names[i])){errorMsg=`Duplicate roommate username: ${names[i]}`; break;}
      if(mobiles[i]&&newRoommates.some(r=>r.mobile===mobiles[i])){errorMsg=`Duplicate roommate mobile: ${mobiles[i]}`; break;}
      if(users.some(u=>u.username===names[i]&&!roommates.some(r=>r.username===u.username))){errorMsg=`Username exists: ${names[i]}`; break;}
      if(mobiles[i]&&users.some(u=>u.mobile===mobiles[i]&&!roommates.some(r=>r.mobile===u.mobile))){errorMsg=`Mobile exists: ${mobiles[i]}`; break;}
      newRoommates.push({username:names[i]||'',password:passwords[i],mobile:mobiles[i]||'',role:'roommate'});
    }
  }
  if(errorMsg){alert(errorMsg);return;}
  roommates=newRoommates;
  users=users.filter(u=>u.role!=='roommate').concat(roommates);
  saveUsers(); saveRoommates();
  updateRoommateList();
  alert('Roommates saved!');
  adminForm.reset();
});

// Admin password change
changePassForm.addEventListener('submit', e=>{
  e.preventDefault();
  const curr=document.getElementById('currentPass').value;
  const newP=document.getElementById('newPass').value;
  const confirmP=document.getElementById('confirmPass').value;
  if(curr!==currentUser.password){alert('Current password incorrect'); return;}
  if(newP!==confirmP){alert('New passwords do not match'); return;}
  currentUser.password=newP;
  const idx=users.findIndex(u=>u.username==='admin');
  users[idx].password=newP;
  saveUsers();
  alert('Password changed!');
  changePassForm.reset();
});

// Add expense
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();
  const date=document.getElementById('date').value;
  const name=document.getElementById('name').value;
  const amount=parseFloat(document.getElementById('amount').value);
  const desc=document.getElementById('desc').value.trim();
  if(currentUser.role!=='admin'&&name!==currentUser.username){alert('You can only add expenses under your own username.'); return;}
  if(!date||!name||!amount||amount<=0){alert('Fill all fields correctly'); return;}
  expenses.push({date,name,amount,desc});
  saveExpenses(); updateTable(); updateBalancesSummary();
  expenseForm.reset();
});

// Delete expense only admin
window.deleteExpense=function(index){
  if(currentUser.role!=='admin'){alert('Only admin can delete expenses.'); return;}
  if(confirm('Are you sure to delete this expense?')){expenses.splice(index,1); saveExpenses(); updateTable(); updateBalancesSummary();}
};

// Update expense table
function updateTable(){
  expenseTableBody.innerHTML='';
  let total={};
  roommates.forEach(r=>total[r.username]=0);
  expenses.forEach(({date,name,amount,desc},i)=>{
    const tr=document.createElement('tr');
    const actionHTML=currentUser.role==='admin'?`<span class="delete-btn" onclick="deleteExpense(${i})">Delete</span>`:'';
    tr.innerHTML=`<td>${date}</td><td>${name}</td><td>₹${amount.toFixed(2)}</td><td>${desc||''}</td><td>${actionHTML}</td>`;
    expenseTableBody.appendChild(tr);
    if(total[name]!==undefined) total[name]+=amount;
  });
  total1P.textContent=`${roommates[0]?.username||'Roommate 1'}: ₹${total[roommates[0]?.username]||0}`;
  total2P.textContent=`${roommates[1]?.username||'Roommate 2'}: ₹${total[roommates[1]?.username]||0}`;
  total3P.textContent=`${roommates[2]?.username||'Roommate 3'}: ₹${total[roommates[2]?.username]||0}`;
  const sumAll=Object.values(total).reduce((a,b)=>a+b,0);
  totalAllP.textContent=`Total: ₹${sumAll.toFixed(2)}`;
}

// Initialize
updateBalancesSummary();
updateRoommateList();
</script>

</body>
</html>

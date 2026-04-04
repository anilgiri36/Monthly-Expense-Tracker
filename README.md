<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker</title>

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background:#f4f7f8; margin:0; padding:0; }
h1,h2,h3{text-align:center;color:#333;}
.container{max-width:900px;margin:20px auto;padding:20px;background:#fff;border-radius:10px;box-shadow:0 5px 15px rgba(0,0,0,0.1);}
input,select,button{padding:10px;margin:5px 0;width:100%;border-radius:5px;border:1px solid #ccc;}
button{background:#E91E63;color:white;font-weight:bold;cursor:pointer;border:none; transition:.3s;}
button:hover{background:#d81b60;}
table{border-collapse:collapse;width:100%;margin-top:20px;}
th,td{border:1px solid #ddd;padding:10px;text-align:center;}
th{background-color:#E91E63;color:white;}
tr:nth-child(even){background-color:#f9f9f9;}
.totals{display:flex;justify-content:space-around;margin-bottom:20px;flex-wrap:wrap;}
.totals div{background:#E91E63;color:white;padding:15px;border-radius:8px;width:28%;text-align:center;font-weight:bold;margin-bottom:10px;}
.section{background:#fafafa;padding:15px;border-radius:8px;margin-bottom:20px;}
.hidden{display:none;}
.user-row{display:flex;justify-content:space-between;margin:5px 0;}
#userInfo { position:absolute; top:20px; right:20px; font-weight:bold; color:#E91E63; }
.owes {color:red;} .owed {color:green;}
</style>
</head>
<body>

<div id="userInfo">
  <a href="#signinDiv" id="adminLoginLink">Sign in as Admin</a>
  <span id="userNameDisplay" class="hidden"></span>
  <a href="#" id="signOutLink" class="hidden">Log Out</a>
</div>

<div id="homePage" class="section">
  <h1>Welcome to Roommate Expense Tracker</h1>
  <p style="text-align:center;">Track shared expenses easily.</p>
</div>

<div class="container">

<!-- SIGN-IN FORM -->
<div id="signinDiv" class="section hidden">
  <h2>Sign In</h2>
  <form id="signinForm">
    <input type="email" id="username" placeholder="Email" required>
    <input type="password" id="password" placeholder="Password" required>
    <button type="submit">Sign In</button>
  </form>
</div>

<!-- ADMIN PANEL -->
<div id="adminDiv" class="hidden">
  <div class="section">
    <h2>Admin Panel</h2>
    <form id="adminForm">
      <h3>Add Roommate</h3>
      <input type="email" id="roommateEmail" placeholder="Roommate Email" required>
      <input type="password" id="roommatePassword" placeholder="Set Password" required>
      <button type="submit">Add Roommate</button>
    </form>
    <div id="roommateList"></div>
  </div>
</div>

<!-- EXPENSE TRACKER -->
<div id="trackerDiv" class="hidden">
  <div class="totals">
    <div id="total1">₹0</div>
    <div id="total2">₹0</div>
    <div id="total3">₹0</div>
    <div id="totalAll">₹0</div>
  </div>

  <div class="section">
    <form id="expenseForm">
      <input type="date" id="date" required>
      <select id="name" required></select>
      <input type="number" id="amount" placeholder="Amount" min="1" required>
      <button type="submit">Add Expense</button>
    </form>
  </div>

  <table id="expenseTable">
    <thead>
      <tr><th>Date</th><th>Name</th><th>Amount</th><th>Status</th><th>Action</th></tr>
    </thead>
    <tbody></tbody>
  </table>
</div>

</div>

<footer style="text-align:center; margin:20px; color:gray;">
  <p>© 2026 Anil Giri. All rights reserved.</p>
  <p style="color:green; font-weight:bold;">A page created by Anil Giri</p>
</footer>

<script>
// Firebase config
const firebaseConfig = {
  apiKey: "AIzaSyDe_RpVPea__Zjsxjy1BqX4xL56pSssAeY",
  authDomain: "roommateexpensetracker.firebaseapp.com",
  databaseURL: "https://roommateexpensetracker-default-rtdb.firebaseio.com",
  projectId: "roommateexpensetracker",
  storageBucket: "roommateexpensetracker.firebasestorage.app",
  messagingSenderId: "460996999829",
  appId: "1:460996999829:web:c1670e4e7558876a55f99c",
  measurementId: "G-Q8LEPT72XW"
};
firebase.initializeApp(firebaseConfig);
const auth = firebase.auth();
const database = firebase.database();

// DOM
const homePage = document.getElementById('homePage');
const signinDiv = document.getElementById('signinDiv');
const adminDiv = document.getElementById('adminDiv');
const trackerDiv = document.getElementById('trackerDiv');
const userInfo = document.getElementById('userInfo');
const adminLoginLink = document.getElementById('adminLoginLink');
const userNameDisplay = document.getElementById('userNameDisplay');
const signOutLink = document.getElementById('signOutLink');
const signinForm = document.getElementById('signinForm');
const adminForm = document.getElementById('adminForm');
const roommateEmailInput = document.getElementById('roommateEmail');
const roommatePasswordInput = document.getElementById('roommatePassword');
const roommateListDiv = document.getElementById('roommateList');
const expenseForm = document.getElementById('expenseForm');
const nameSelect = document.getElementById('name');
const expenseTableBody = document.querySelector('#expenseTable tbody');
const total1P = document.getElementById('total1');
const total2P = document.getElementById('total2');
const total3P = document.getElementById('total3');
const totalAllP = document.getElementById('totalAll');

let currentUser = null;
let roommates = [];

// Show Sign in form
adminLoginLink.addEventListener('click', e=>{
  e.preventDefault();
  signinDiv.classList.remove('hidden');
  homePage.classList.add('hidden');
});

// Sign out
signOutLink.addEventListener('click', e=>{
  e.preventDefault();
  auth.signOut();
  currentUser=null;
  userNameDisplay.classList.add('hidden');
  signOutLink.classList.add('hidden');
  adminLoginLink.classList.remove('hidden');
  homePage.classList.remove('hidden');
  trackerDiv.classList.add('hidden');
  adminDiv.classList.add('hidden');
});

// Listen auth state
auth.onAuthStateChanged(user=>{
  if(user){
    currentUser = user;
    userNameDisplay.textContent = user.email;
    userNameDisplay.classList.remove('hidden');
    signOutLink.classList.remove('hidden');
    adminLoginLink.classList.add('hidden');
    signinDiv.classList.add('hidden');
    homePage.classList.add('hidden');
    // check if admin
    if(user.email === 'admin@example.com'){
      adminDiv.classList.remove('hidden');
    } else {
      trackerDiv.classList.remove('hidden');
    }
    loadRoommates();
    loadExpenses();
  } else {
    currentUser=null;
  }
});

// Admin add roommate
adminForm.addEventListener('submit', e=>{
  e.preventDefault();
  const email = roommateEmailInput.value.trim();
  const password = roommatePasswordInput.value.trim();
  if(!email || !password) return alert("Email and password required.");

  // Check duplicate in DB
  database.ref('roommates').orderByChild('username').equalTo(email).once('value', snapshot=>{
    if(snapshot.exists()) return alert("User already exists!");
    
    // Create Firebase user
    auth.createUserWithEmailAndPassword(email,password)
      .then(cred=>{
        database.ref('roommates').push({username: email});
        alert("Roommate added!");
        roommateEmailInput.value=''; roommatePasswordInput.value='';
        loadRoommates();
      })
      .catch(err=>alert(err.message));
  });
});

// Load roommates into select and list
function loadRoommates(){
  database.ref('roommates').once('value', snap=>{
    roommates=[];
    nameSelect.innerHTML='';
    roommateListDiv.innerHTML='';
    snap.forEach(child=>{
      const r = child.val(); roommates.push(r.username);
      // dropdown
      if(currentUser.email===r.username || currentUser.email==='admin@example.com'){
        const opt = document.createElement('option'); opt.value=r.username; opt.textContent=r.username;
        nameSelect.appendChild(opt);
      }
      // list
      const div = document.createElement('div'); div.className='user-row';
      div.textContent=r.username;
      roommateListDiv.appendChild(div);
    });
  });
}

// Expenses
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();
  const date = document.getElementById('date').value;
  const name = nameSelect.value;
  const amount = parseFloat(document.getElementById('amount').value);
  if(!date||!name||!amount) return;
  if(currentUser.email !== name && currentUser.email!=='admin@example.com') return alert("Cannot add expense for others.");

  database.ref('expenses').push({date,name,amount});
  expenseForm.reset();
  loadExpenses();
});

function loadExpenses(){
  database.ref('expenses').once('value', snap=>{
    expenseTableBody.innerHTML='';
    const totals = {};
    roommates.forEach(r=>totals[r]=0);
    snap.forEach((child,i)=>{
      const e = child.val();
      if(totals[e.name]!==undefined) totals[e.name]+=e.amount;

      const tr = document.createElement('tr');
      let status='', colorClass='';
      const avg = Object.values(totals).reduce((a,b)=>a+b,0)/roommates.length;
      const diff = totals[e.name]-avg;
      if(diff>0){ status='owed'; colorClass='owed'; }
      else if(diff<0){ status='owes'; colorClass='owes'; }
      else { status='settled'; colorClass=''; }

      tr.innerHTML = `<td>${e.date}</td><td>${e.name}</td><td>₹${e.amount}</td>
      <td class="${colorClass}">${status}</td>
      <td>${currentUser.email==='admin@example.com'?'<span class="delete-btn" onclick="deleteExpense(\''+child.key+'\')">Delete</span>':''}</td>`;
      expenseTableBody.appendChild(tr);
    });
    // Update totals top
    total1P.textContent = `${roommates[0]||''}: ₹${totals[roommates[0]]||0}`;
    total2P.textContent = `${roommates[1]||''}: ₹${totals[roommates[1]]||0}`;
    total3P.textContent = `${roommates[2]||''}: ₹${totals[roommates[2]]||0}`;
    totalAllP.textContent = `Total: ₹${Object.values(totals).reduce((a,b)=>a+b,0)}`;
  });
}

// Delete expense
function deleteExpense(key){
  if(currentUser.email!=='admin@example.com') return;
  database.ref('expenses/'+key).remove();
  loadExpenses();
}

</script>
</body>
</html>

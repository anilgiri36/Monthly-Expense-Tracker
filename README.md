<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker</title>

<!-- Firebase SDKs -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background:#f4f7f8; margin:0; padding:0; }
h1,h2,h3{text-align:center;color:#333;}
.container{max-width:900px;margin:20px auto;padding:20px;background:#fff;border-radius:10px;box-shadow:0 5px 15px rgba(0,0,0,0.1);}
input,select,button{padding:10px;margin:5px 0;width:100%;border-radius:5px;border:1px solid #ccc;}
button{background:#E91E63;color:white;font-weight:bold;cursor:pointer;border:none;transition:.3s;}
button:hover{background:#d81b60;}
table{border-collapse:collapse;width:100%;margin-top:20px;}
th,td{border:1px solid #ddd;padding:10px;text-align:center;}
th{background-color:#E91E63;color:white;}
tr:nth-child(even){background-color:#f9f9f9;}
.totals{display:flex;justify-content:space-around;margin-bottom:20px;flex-wrap:wrap;}
.totals div{padding:15px;border-radius:8px;width:28%;text-align:center;font-weight:bold;margin-bottom:10px;}
.totals .owed {background:#4caf50;color:white;}
.totals .owes {background:#f44336;color:white;}
.section{background:#fafafa;padding:15px;border-radius:8px;margin-bottom:20px;}
.hidden{display:none;}
.user-row{display:flex;justify-content:space-between;margin:5px 0;}
#userInfo { position:absolute; top:20px; right:20px; font-weight:bold; color:#E91E63; }
#userInfo a{margin-left:10px; color:#E91E63; text-decoration:none;}
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

<h1>Roommate Expense Tracker</h1>

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
    <h2>Admin Panel</h2>
    <form id="adminForm">
      <input type="text" id="roommateName1" placeholder="Roommate 1 Username or Mobile">
      <input type="password" id="roommatePass1" placeholder="Password">
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username or Mobile">
      <input type="password" id="roommatePass2" placeholder="Password">
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username or Mobile">
      <input type="password" id="roommatePass3" placeholder="Password">
      <button type="submit">Save Roommates</button>
    </form>
    <h3>Existing Roommates</h3>
    <div id="roommateList"></div>
  </div>
</div>

<div id="trackerDiv" class="hidden">

  <div class="totals">
    <div id="total1" class="owed">₹0</div>
    <div id="total2" class="owed">₹0</div>
    <div id="total3" class="owed">₹0</div>
    <div id="totalAll" class="owed">₹0</div>
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

  <!-- Edit section for Admin -->
  <div id="editSection" class="section hidden">
    <h3>Edit Expense</h3>
    <input type="date" id="editDate">
    <input type="text" id="editName" disabled>
    <input type="number" id="editAmount">
    <input type="text" id="editDesc">
    <button onclick="updateExpense()">Update</button>
  </div>

  <table id="expenseTable">
    <thead>
      <tr><th>Date</th><th>Name</th><th>Amount (₹)</th><th>Description</th><th>Action</th></tr>
    </thead>
    <tbody></tbody>
  </table>

</div>

</div> <!-- container -->

<footer style="text-align:center; margin:20px; color:gray;">
  <p>© 2026 Anil Giri. All rights reserved.</p>
  <marquee><p style="color:green; font-weight:bold;">A page created by Anil Giri</p></marquee>
</footer>

<script>
// ---------- Firebase Config ----------
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
const db = firebase.database();

// ---------- DOM Elements ----------
const signinDiv = document.getElementById('signinDiv');
const adminDiv = document.getElementById('adminDiv');
const trackerDiv = document.getElementById('trackerDiv');
const signinForm = document.getElementById('signinForm');
const adminForm = document.getElementById('adminForm');
const expenseForm = document.getElementById('expenseForm');
const nameSelect = document.getElementById('name');
const expenseTableBody = document.querySelector('#expenseTable tbody');
const total1P = document.getElementById('total1');
const total2P = document.getElementById('total2');
const total3P = document.getElementById('total3');
const totalAllP = document.getElementById('totalAll');
const roommateListDiv = document.getElementById('roommateList');
const userNameDisplay = document.getElementById('userNameDisplay');
const signOutLink = document.getElementById('signOutLink');
const adminLoginLink = document.getElementById('adminLoginLink');
const editSection = document.getElementById('editSection');
const editDate = document.getElementById('editDate');
const editName = document.getElementById('editName');
const editAmount = document.getElementById('editAmount');
const editDesc = document.getElementById('editDesc');

let currentUser = null;
let roommates = [];
let expenses = [];
let userRole = "";
let editId = null;

// ---------- Firebase Auth ----------
signinForm.addEventListener('submit', e=>{
  e.preventDefault();
  const username = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value;

  auth.signInWithEmailAndPassword(username, password)
  .then(cred=>{
    currentUser = cred.user;
    userNameDisplay.textContent = username;
    userNameDisplay.classList.remove('hidden');
    signOutLink.classList.remove('hidden');
    adminLoginLink.classList.add('hidden');
    signinDiv.classList.add('hidden');
    homePage.classList.add('hidden');

    // Fetch user role
    db.ref('users/' + currentUser.uid).once('value').then(snapshot=>{
      const data = snapshot.val();
      if(!data) { alert('User role not found'); return; }
      if(data.role==='admin'){
        userRole = 'admin';
        adminDiv.classList.remove('hidden');
      } else {
        userRole = 'roommate';
        trackerDiv.classList.remove('hidden');
      }
      loadRoommates();
      loadExpenses();
    });
  })
  .catch(err=>alert(err.message));
});

// ---------- Sign Out ----------
signOutLink.addEventListener('click', e=>{
  e.preventDefault();
  auth.signOut().then(()=>location.reload());
});

// ---------- Admin Adds Roommates ----------
adminForm.addEventListener('submit', e=>{
  e.preventDefault();
  const names = [
    document.getElementById('roommateName1').value.trim(),
    document.getElementById('roommateName2').value.trim(),
    document.getElementById('roommateName3').value.trim()
  ];
  const passwords = [
    document.getElementById('roommatePass1').value,
    document.getElementById('roommatePass2').value,
    document.getElementById('roommatePass3').value
  ];

  names.forEach((name,i)=>{
    if(name && passwords[i]){
      // Check duplicates
      db.ref('users').orderByChild('username').equalTo(name).once('value',snap=>{
        if(snap.exists()){ alert('Username exists: ' + name); return; }
        auth.createUserWithEmailAndPassword(name,passwords[i])
        .then(userCred=>{
          db.ref('users/' + userCred.user.uid).set({username:name,role:'roommate'});
          loadRoommates();
        })
        .catch(err=>alert(err.message));
      });
    }
  });
  adminForm.reset();
});

// ---------- Load Roommates ----------
function loadRoommates(){
  db.ref('users').once('value').then(snap=>{
    roommates=[];
    roommateListDiv.innerHTML='';
    nameSelect.innerHTML='';
    snap.forEach(child=>{
      const u = child.val();
      if(u.role==='roommate'){
        roommates.push({uid:child.key, username:u.username});
        const opt = document.createElement('option');
        opt.value = u.username; opt.textContent = u.username;
        nameSelect.appendChild(opt);
        // show in admin list
        const div = document.createElement('div');
        div.className='user-row';
        div.innerHTML=`<span>${u.username}</span>`;
        roommateListDiv.appendChild(div);
      }
    });
  });
}

// ---------- Add Expense ----------
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();
  const date = document.getElementById('date').value;
  const name = document.getElementById('name').value;
  const amount = parseFloat(document.getElementById('amount').value);
  const desc = document.getElementById('desc').value.trim();

  if(userRole !== 'admin' && name !== currentUser.email){
    alert('You can only add your own expense!');
    return;
  }

  const newExpense = {date,name,amount,desc};
  db.ref('expenses').push(newExpense).then(()=>{
    loadExpenses();
    expenseForm.reset();
  });
});

// ---------- Load Expenses & Totals ----------
function loadExpenses(){
  db.ref('expenses').once('value').then(snap=>{
    expenses=[];
    expenseTableBody.innerHTML='';
    let totalMap={};
    roommates.forEach(r=>totalMap[r.username]=0);
    snap.forEach(child=>{
      const exp = child.val();
      expenses.push(exp);
      const tr=document.createElement('tr');

      const actionHTML = (userRole==='admin') 
        ? `<button onclick="editExpense('${child.key}')">Edit</button>
           <button onclick="deleteExpense('${child.key}')">Delete</button>` : '';

      tr.innerHTML=`<td>${exp.date}</td><td>${exp.name}</td><td>₹${exp.amount}</td><td>${exp.desc||''}</td><td>${actionHTML}</td>`;
      expenseTableBody.appendChild(tr);

      if(totalMap[exp.name]!==undefined) totalMap[exp.name]+=exp.amount;
    });

    // Update totals with owes/owed colors
    let sumAll = Object.values(totalMap).reduce((a,b)=>a+b,0);
    [total1P,total2P,total3P].forEach((el,i)=>{
      const uname = roommates[i]?.username||'Roommate '+(i+1);
      const amt = totalMap[uname]||0;
      el.textContent = amt>0 ? `Owed ₹${amt}` : `Owes ₹0`;
      el.className = amt>0 ? 'totals owed' : 'totals owes';
    });
    totalAllP.textContent=`Total: ₹${sumAll}`;
  });
}

// ---------- Delete Expense ----------
function deleteExpense(id){
  if(confirm("Delete this expense?")){
    db.ref('expenses/' + id).remove().then(()=>loadExpenses());
  }
}

// ---------- Edit Expense ----------
function editExpense(id){
  db.ref('expenses/' + id).once('value').then(snap=>{
    const e = snap.val();
    editDate.value = e.date;
    editName.value = e.name;
    editAmount.value = e.amount;
    editDesc.value = e.desc||'';
    editId = id;
    editSection.classList.remove('hidden');
  });
}

// ---------- Update Expense ----------
function updateExpense(){
  if(!editId) return;
  db.ref('expenses/' + editId).update({
    date: editDate.value,
    amount: parseFloat(editAmount.value),
    desc: editDesc.value
  })
  .then(()=>{
    alert("Expense updated");
    editSection.classList.add('hidden');
    loadExpenses();
  });
}

// ---------- Load on page start ----------
loadRoommates();
loadExpenses();

</script>
</body>
</html>

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
.totals-container{display:flex;justify-content:space-around;margin-bottom:20px;flex-wrap:wrap;}
.totals-card{padding:15px;border-radius:8px;text-align:center;font-weight:bold;margin-bottom:10px;width:28%;}
.totals.owed{background:#4caf50;color:white;}
.totals.owes{background:#f44336;color:white;}
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

  <!-- Top Totals -->
  <div class="totals-container">
    <div id="total1" class="totals-card totals owed">Roommate 1: ₹0</div>
    <div id="total2" class="totals-card totals owed">Roommate 2: ₹0</div>
    <div id="total3" class="totals-card totals owed">Roommate 3: ₹0</div>
    <div id="totalAll" class="totals-card totals owed">Total Expenses: ₹0</div>
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

// ---------- DOM ----------
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
const homePage = document.getElementById('homePage');

let currentUser = null;
let roommates = [];
let currentUsername = '';
let isAdmin = false;

// ---------- Sign In ----------
signinForm.addEventListener('submit', async e=>{
  e.preventDefault();
  const username = document.getElementById('username').value.trim();
  const password = document.getElementById('password').value;

  try{
    const email = username.includes('@') ? username : username + '@example.com';
    const cred = await auth.signInWithEmailAndPassword(email, password);
    currentUser = cred.user;

    const snapshot = await db.ref('users/' + currentUser.uid).once('value');
    const data = snapshot.val();
    if(!data){ alert('User data not found'); return; }

    currentUsername = data.username;
    userNameDisplay.textContent = currentUsername;
    userNameDisplay.classList.remove('hidden');
    signOutLink.classList.remove('hidden');
    adminLoginLink.classList.add('hidden');
    signinDiv.classList.add('hidden');
    homePage.classList.add('hidden');

    isAdmin = data.role==='admin';
    if(isAdmin) adminDiv.classList.remove('hidden');
    trackerDiv.classList.remove('hidden');

    await loadRoommates();
    await loadExpenses();
  }catch(err){ alert(err.message); }
});

// ---------- Sign Out ----------
signOutLink.addEventListener('click', e=>{
  e.preventDefault();
  auth.signOut().then(()=> location.reload());
});

// ---------- Admin Adds Roommates ----------
adminForm.addEventListener('submit', async e=>{
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

  for(let i=0;i<names.length;i++){
    if(names[i] && passwords[i]){
      const snap = await db.ref('users').orderByChild('username').equalTo(names[i]).once('value');
      if(snap.exists()){ alert('Username exists: ' + names[i]); continue; }
      const email = names[i].includes('@') ? names[i] : names[i] + '@example.com';
      try{
        const userCred = await auth.createUserWithEmailAndPassword(email,passwords[i]);
        await db.ref('users/' + userCred.user.uid).set({username:names[i],role:'roommate'});
      }catch(err){ alert(err.message); }
    }
  }
  adminForm.reset();
  await loadRoommates();
});

// ---------- Load Roommates ----------
async function loadRoommates(){
  const snap = await db.ref('users').once('value');
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

      const div = document.createElement('div');
      div.className='user-row';
      div.innerHTML=`<span>${u.username}</span>`;
      roommateListDiv.appendChild(div);
    }
  });
}

// ---------- Add Expense ----------
expenseForm.addEventListener('submit', async e=>{
  e.preventDefault();
  const date = document.getElementById('date').value;
  const name = document.getElementById('name').value;
  const amount = parseFloat(document.getElementById('amount').value);
  const desc = document.getElementById('desc').value.trim();

  if(!isAdmin && name!==currentUsername){
    alert('You can only add expense for yourself!');
    return;
  }

  const newExpense = {date,name,amount,desc,uid:currentUser.uid};
  await db.ref('expenses').push(newExpense);
  expenseForm.reset();
  loadExpenses();
});

// ---------- Load Expenses ----------
async function loadExpenses(){
  const snap = await db.ref('expenses').once('value');
  const totalMap = {};
  roommates.forEach(r=> totalMap[r.username]=0);
  expenseTableBody.innerHTML='';

  snap.forEach(child=>{
    const exp = child.val();
    if(totalMap[exp.name]!==undefined) totalMap[exp.name]+=exp.amount;

    const tr = document.createElement('tr');
    let actionHTML = '';
    if(isAdmin || exp.uid===currentUser.uid){
      actionHTML = `
        <button onclick="editExpense('${child.key}')">Edit</button>
        <button onclick="deleteExpense('${child.key}')">Delete</button>
      `;
    }
    tr.innerHTML = `<td>${exp.date}</td><td>${exp.name}</td><td>₹${exp.amount}</td><td>${exp.desc||''}</td><td>${actionHTML}</td>`;
    expenseTableBody.appendChild(tr);
  });

  [total1P,total2P,total3P].forEach((el,i)=>{
    const uname = roommates[i]?.username || 'Roommate '+(i+1);
    const amt = totalMap[uname] || 0;
    el.textContent = `${uname}: ₹${amt}`;
    el.className = amt>0 ? 'totals-card totals owed' : 'totals-card totals owes';
  });

  const sumAll = Object.values(totalMap).reduce((a,b)=>a+b,0);
  totalAllP.textContent = `Total Expenses: ₹${sumAll}`;
}

// ---------- Delete & Edit ----------
window.deleteExpense = async (key)=>{
  if(confirm('Delete this expense?')){ await db.ref('expenses/'+key).remove(); loadExpenses(); }
}

window.editExpense = async (key)=>{
  const snap = await db.ref('expenses/'+key).once('value');
  const exp = snap.val();
  const newDate = prompt('Edit Date:', exp.date);
  if(!newDate) return;
  const newName = isAdmin ? prompt('Edit Name:', exp.name) : exp.name;
  const newAmount = parseFloat(prompt('Edit Amount:', exp.amount));
  const newDesc = prompt('Edit Description:', exp.desc||'');
  await db.ref('expenses/'+key).update({date:newDate,name:newName,amount:newAmount,desc:newDesc});
  loadExpenses();
}

// ---------- Initial Load ----------
loadRoommates();
loadExpenses();
</script>

</body>
</html>

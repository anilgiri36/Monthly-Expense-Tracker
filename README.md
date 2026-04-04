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
body { font-family:'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background:#f4f7f8; margin:0; padding:0; }
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
#userInfo { position:absolute; top:20px; right:20px; font-weight:bold; color:#E91E63; }
#userInfo a{margin-left:10px;color:#E91E63;text-decoration:none;}
.roommateTotals{display:flex; justify-content:space-around; margin-bottom:20px; flex-wrap:wrap;}
.roommateTotals div{background:#e0e0e0; padding:10px 20px; border-radius:8px; font-weight:bold;}
</style>
</head>
<body>

<div id="userInfo">
  <a href="#signinDiv" id="adminLoginLink">Sign in</a>
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
    <input type="text" id="username" placeholder="Username or Email" required>
    <input type="password" id="password" placeholder="Password" required>
    <button type="submit">Sign In</button>
  </form>
</div>

<div id="adminDiv" class="hidden">
  <div class="section">
    <h2>Admin Panel - Manage Roommates</h2>
    <form id="adminForm">
      <input type="text" id="roommateName1" placeholder="Roommate 1 Username/Email">
      <input type="password" id="roommatePass1" placeholder="Password">
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username/Email">
      <input type="password" id="roommatePass2" placeholder="Password">
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username/Email">
      <input type="password" id="roommatePass3" placeholder="Password">
      <button type="submit">Save Roommates</button>
    </form>
    <h3>Existing Roommates</h3>
    <div id="roommateList"></div>
  </div>
</div>

<div id="trackerDiv" class="hidden">

  <!-- Top roommate totals -->
  <div class="roommateTotals" id="roommateTotalsDiv"></div>

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

</div> <!-- container -->

<footer style="text-align:center; margin:20px; color:gray;">
  <p>© 2026 Anil Giri. All rights reserved.</p>
</footer>

<script>
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

// DOM Elements
const signinDiv=document.getElementById('signinDiv'),
      adminDiv=document.getElementById('adminDiv'),
      trackerDiv=document.getElementById('trackerDiv'),
      signinForm=document.getElementById('signinForm'),
      adminForm=document.getElementById('adminForm'),
      expenseForm=document.getElementById('expenseForm'),
      nameSelect=document.getElementById('name'),
      expenseTableBody=document.querySelector('#expenseTable tbody'),
      roommateListDiv=document.getElementById('roommateList'),
      roommateTotalsDiv=document.getElementById('roommateTotalsDiv'),
      userNameDisplay=document.getElementById('userNameDisplay'),
      signOutLink=document.getElementById('signOutLink'),
      adminLoginLink=document.getElementById('adminLoginLink'),
      homePage=document.getElementById('homePage');

let currentUser=null, roommates=[], expenses=[];

// Sign In
signinForm.addEventListener('submit', e=>{
  e.preventDefault();
  const username=document.getElementById('username').value.trim();
  const password=document.getElementById('password').value;
  auth.signInWithEmailAndPassword(username,password).then(cred=>{
    currentUser=cred.user;
    userNameDisplay.textContent=username;
    userNameDisplay.classList.remove('hidden');
    signOutLink.classList.remove('hidden');
    adminLoginLink.classList.add('hidden');
    signinDiv.classList.add('hidden');
    homePage.classList.add('hidden');
    db.ref('users/'+currentUser.uid).once('value').then(snapshot=>{
      const data=snapshot.val();
      currentUser.role=data.role;
      if(data.role==='admin') adminDiv.classList.remove('hidden');
      else trackerDiv.classList.remove('hidden');
      loadRoommates();
      loadExpenses();
    });
  }).catch(err=>alert(err.message));
});

// Sign Out
signOutLink.addEventListener('click', e=>{
  e.preventDefault(); auth.signOut().then(()=>location.reload());
});

// Admin Add Roommates
adminForm.addEventListener('submit', e=>{
  e.preventDefault();
  const names=[
    document.getElementById('roommateName1').value.trim(),
    document.getElementById('roommateName2').value.trim(),
    document.getElementById('roommateName3').value.trim()
  ];
  const passwords=[
    document.getElementById('roommatePass1').value,
    document.getElementById('roommatePass2').value,
    document.getElementById('roommatePass3').value
  ];
  names.forEach((name,i)=>{
    if(name && passwords[i]){
      db.ref('users').orderByChild('username').equalTo(name).once('value',snap=>{
        if(snap.exists()){ alert('Username exists: '+name); return; }
        auth.createUserWithEmailAndPassword(name,passwords[i]).then(userCred=>{
          db.ref('users/'+userCred.user.uid).set({username:name,role:'roommate'});
          loadRoommates();
        }).catch(err=>alert(err.message));
      });
    }
  });
  adminForm.reset();
});

// Load Roommates
function loadRoommates(){
  db.ref('users').once('value').then(snap=>{
    roommates=[]; roommateListDiv.innerHTML=''; nameSelect.innerHTML=''; roommateTotalsDiv.innerHTML='';
    snap.forEach(child=>{
      const u=child.val();
      if(u.role==='roommate'){
        roommates.push({uid:child.key, username:u.username});
        const opt=document.createElement('option'); opt.value=u.username; opt.textContent=u.username;
        nameSelect.appendChild(opt);
        const div=document.createElement('div'); div.className='user-row'; div.textContent=u.username;
        roommateListDiv.appendChild(div);
        const totalDiv=document.createElement('div'); totalDiv.id='total_'+u.username.replace(/[@.]/g,'_'); totalDiv.textContent=`${u.username}: ₹0`;
        roommateTotalsDiv.appendChild(totalDiv);
      }
    });
  });
}

// Add Expense
expenseForm.addEventListener('submit', e=>{
  e.preventDefault();
  const date=document.getElementById('date').value;
  const name=document.getElementById('name').value;
  const amount=parseFloat(document.getElementById('amount').value);
  const desc=document.getElementById('desc').value.trim();
  if(currentUser.role!=='admin' && currentUser.email!==name && currentUser.phoneNumber!==name){ 
    alert('You can only add expense for yourself!'); return;
  }
  const newExpense={date,name,amount,desc,uid:currentUser.uid};
  db.ref('expenses').push(newExpense).then(()=>{ loadExpenses(); expenseForm.reset(); });
});

// Load Expenses
function loadExpenses(){
  db.ref('expenses').once('value').then(snap=>{
    expenses=[]; expenseTableBody.innerHTML='';
    let totalMap={}; roommates.forEach(r=>totalMap[r.username]=0);
    snap.forEach(child=>{
      const exp=child.val(); exp.key=child.key; expenses.push(exp);
      let actionHTML='';
      if(currentUser.role==='admin' || currentUser.uid===exp.uid){
        actionHTML=`<button onclick="editExpense('${exp.key}')">Edit</button> <button onclick="deleteExpense('${exp.key}')">Delete</button>`;
      }
      const tr=document.createElement('tr');
      tr.innerHTML=`<td>${exp.date}</td><td>${exp.name}</td><td>₹${exp.amount}</td><td>${exp.desc||''}</td><td>${actionHTML}</td>`;
      expenseTableBody.appendChild(tr);
      if(totalMap[exp.name]!==undefined) totalMap[exp.name]+=exp.amount;
    });

    // Update roommate totals at top
    roommates.forEach(r=>{
      const div=document.getElementById('total_'+r.username.replace(/[@.]/g,'_'));
      if(div) div.textContent=`${r.username}: ₹${totalMap[r.username]||0}`;
    });
  });
}

// Edit/Delete
function deleteExpense(key){ if(confirm('Delete this expense?')){ db.ref('expenses/'+key).remove().then(()=>loadExpenses()); } }
function editExpense(key){
  const exp=expenses.find(e=>e.key===key); if(!exp) return;
  document.getElementById('date').value=exp.date;
  document.getElementById('name').value=exp.name;
  document.getElementById('amount').value=exp.amount;
  document.getElementById('desc').value=exp.desc;
  deleteExpense(key); // will save as new on submit
}

// Initial Load
loadRoommates();
loadExpenses();
</script>
</body>
</html>

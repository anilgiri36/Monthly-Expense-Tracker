<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker</title>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

<style>
body { font-family:Segoe UI; background:#f4f7f8; margin:0; }
.container{max-width:900px;margin:20px auto;padding:20px;background:#fff;border-radius:10px;}
input,select,button{padding:10px;margin:5px 0;width:100%;}
button{background:#E91E63;color:#fff;border:none;cursor:pointer;}
button:hover{background:#d81b60;}
.hidden{display:none;}
#userInfo{position:absolute;top:15px;right:20px;color:#E91E63;font-weight:bold;}
.owed{background:#4caf50;color:white;padding:10px;margin:5px;border-radius:6px;}
.owes{background:#f44336;color:white;padding:10px;margin:5px;border-radius:6px;}
table{width:100%;border-collapse:collapse;margin-top:20px;}
th,td{border:1px solid #ddd;padding:8px;text-align:center;}
th{background:#E91E63;color:white;}
.section{margin-top:20px;}
</style>
</head>

<body>

<div id="userInfo">
  <span id="userName"></span>
  <a href="#" id="logout" class="hidden"> | Logout</a>
</div>

<div class="container">

<h2>Roommate Expense Tracker</h2>

<!-- LOGIN -->
<div id="loginDiv">
  <input type="text" id="email" placeholder="Email">
  <input type="password" id="password" placeholder="Password">
  <button onclick="login()">Login</button>
</div>

<!-- ADMIN -->
<div id="adminDiv" class="hidden">
  <h3>Admin Panel</h3>
  <input id="rEmail" placeholder="Roommate Email">
  <input id="rPass" placeholder="Password">
  <button onclick="addRoommate()">Add Roommate</button>
</div>

<!-- TRACKER -->
<div id="trackerDiv" class="hidden">

  <h3>Add Expense</h3>
  <input type="date" id="date">
  <select id="name"></select>
  <input type="number" id="amount" placeholder="Amount">
  <input type="text" id="desc" placeholder="Description">
  <button onclick="addExpense()">Add Expense</button>

  <!-- EDIT -->
  <div id="editSection" class="section hidden">
    <h3>Edit Expense</h3>
    <input type="date" id="editDate">
    <input id="editName" disabled>
    <input type="number" id="editAmount">
    <input id="editDesc">
    <button onclick="updateExpense()">Update</button>
  </div>

  <h3>Totals</h3>
  <div id="totals"></div>

  <h3>Settlement</h3>
  <div id="settlementList"></div>

  <h3>Expenses</h3>
  <table>
    <thead>
      <tr><th>Date</th><th>Name</th><th>Amount</th><th>Desc</th><th>Action</th></tr>
    </thead>
    <tbody id="tableBody"></tbody>
  </table>

</div>

</div>

<footer style="text-align:center; margin:20px; color:gray;">
  <p>© 2026 Anil Giri. All rights reserved.</p>
  <marquee><b style="color: green;">A page created by Anil Giri</b></marquee>
</footer>

<script>
// Firebase config
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "roommateexpensetracker.firebaseapp.com",
  databaseURL: "https://roommateexpensetracker-default-rtdb.firebaseio.com",
  projectId: "roommateexpensetracker"
};
firebase.initializeApp(firebaseConfig);

const auth = firebase.auth();
const db = firebase.database();

let currentUser=null, role=null, editId=null;

// LOGIN
function login(){
  const email=document.getElementById('email').value;
  const pass=document.getElementById('password').value;

  auth.signInWithEmailAndPassword(email,pass)
  .then(res=>{
    currentUser=res.user;
    document.getElementById('userName').innerText=email;
    document.getElementById('logout').classList.remove('hidden');
    document.getElementById('loginDiv').classList.add('hidden');

    db.ref('users/'+currentUser.uid).once('value').then(s=>{
      role=s.val().role;

      if(role==='admin'){
        document.getElementById('adminDiv').classList.remove('hidden');
      }

      document.getElementById('trackerDiv').classList.remove('hidden');
      loadUsers();
      loadExpenses();
    });
  }).catch(err=>alert(err.message));
}

// LOGOUT
document.getElementById('logout').onclick=()=>location.reload();

// ADD ROOMMATE
function addRoommate(){
  const email=rEmail.value;
  const pass=rPass.value;

  auth.createUserWithEmailAndPassword(email,pass)
  .then(res=>{
    db.ref('users/'+res.user.uid).set({username:email,role:'roommate'});
    alert("Roommate added");
    loadUsers();
  });
}

// LOAD USERS
function loadUsers(){
  db.ref('users').once('value').then(s=>{
    const select=name;
    select.innerHTML='';
    s.forEach(c=>{
      if(c.val().role==='roommate'){
        let opt=document.createElement('option');
        opt.value=c.val().username;
        opt.text=c.val().username;
        select.appendChild(opt);
      }
    });
  });
}

// ADD EXPENSE
function addExpense(){
  const uname=name.value;

  if(role!=='admin' && uname!==currentUser.email){
    alert("Add only your expense");
    return;
  }

  db.ref('expenses').push({
    date:date.value,
    name:uname,
    amount:parseFloat(amount.value),
    desc:desc.value
  }).then(loadExpenses);
}

// LOAD EXPENSES
function loadExpenses(){
  db.ref('expenses').once('value').then(s=>{
    tableBody.innerHTML='';
    let totalMap={};

    s.forEach(c=>{
      const e=c.val();
      totalMap[e.name]=(totalMap[e.name]||0)+e.amount;

      let actions = role==='admin'
        ? `<button onclick="editExpense('${c.key}')">Edit</button>
           <button onclick="deleteExpense('${c.key}')">Delete</button>`
        : '';

      let tr=`<tr>
        <td>${e.date}</td>
        <td>${e.name}</td>
        <td>₹${e.amount}</td>
        <td>${e.desc||''}</td>
        <td>${actions}</td>
      </tr>`;

      tableBody.innerHTML+=tr;
    });

    showTotals(totalMap);
    calculateSettlement(totalMap);
  });
}

// DELETE
function deleteExpense(id){
  if(confirm("Delete?")){
    db.ref('expenses/'+id).remove().then(loadExpenses);
  }
}

// EDIT
function editExpense(id){
  db.ref('expenses/'+id).once('value').then(s=>{
    let e=s.val();
    editDate.value=e.date;
    editName.value=e.name;
    editAmount.value=e.amount;
    editDesc.value=e.desc||'';
    editId=id;
    editSection.classList.remove('hidden');
  });
}

// UPDATE
function updateExpense(){
  db.ref('expenses/'+editId).update({
    date:editDate.value,
    amount:parseFloat(editAmount.value),
    desc:editDesc.value
  }).then(()=>{
    editSection.classList.add('hidden');
    loadExpenses();
  });
}

// TOTALS
function showTotals(t){
  totals.innerHTML='';
  let sum=Object.values(t).reduce((a,b)=>a+b,0);
  let avg=sum/Object.keys(t).length;

  Object.keys(t).forEach(u=>{
    let bal=t[u]-avg;
    let div=document.createElement('div');
    div.className=bal>0?'owed':'owes';
    div.innerText=`${u}: ${bal>0?'Gets':'Owes'} ₹${Math.abs(bal).toFixed(2)}`;
    totals.appendChild(div);
  });
}

// SETTLEMENT
function calculateSettlement(t){
  settlementList.innerHTML='';
  let users=Object.keys(t);
  let sum=Object.values(t).reduce((a,b)=>a+b,0);
  let avg=sum/users.length;

  let debt=[],cred=[];

  users.forEach(u=>{
    let b=t[u]-avg;
    if(b>0) cred.push({u,a:b});
    else if(b<0) debt.push({u,a:-b});
  });

  while(debt.length && cred.length){
    let d=debt[0], c=cred[0];
    let amt=Math.min(d.a,c.a);

    settlementList.innerHTML+=`${d.u} pays ₹${amt.toFixed(2)} to ${c.u}<br>`;

    d.a-=amt; c.a-=amt;
    if(!d.a) debt.shift();
    if(!c.a) cred.shift();
  }
}
</script>

</body>
</html>

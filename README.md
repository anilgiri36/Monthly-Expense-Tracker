<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker with Admin</title>
<style>
  body { font-family: Arial, sans-serif; margin: 20px; max-width: 700px; }
  input, select, button { padding: 6px; margin: 4px; }
  table { border-collapse: collapse; width: 100%; margin-top: 20px; }
  th, td { border: 1px solid #ccc; padding: 8px; text-align: left; }
  th { background: #f0f0f0; }
  #totals { margin-bottom: 20px; font-weight: bold; }
  .hidden { display: none; }
  .delete-btn { color: red; cursor: pointer; }
</style>
</head>
<body>

<h1>Roommate Expense Tracker</h1>

<!-- SIGN-IN FORM -->
<div id="signinDiv">
  <h2>Sign In</h2>
  <form id="signinForm">
    <label>Username: <input type="text" id="username" required></label><br>
    <label>Password: <input type="password" id="password" required></label><br>
    <button type="submit">Sign In</button>
  </form>
</div>

<!-- ADMIN PANEL -->
<div id="adminDiv" class="hidden">
  <h2>Admin Panel - Set Roommates</h2>
  <form id="adminForm">
    <label>Roommate 1 Name: <input type="text" id="roommate1" required></label><br>
    <label>Roommate 2 Name: <input type="text" id="roommate2" required></label><br>
    <button type="submit">Save Roommates</button>
  </form>
</div>

<!-- EXPENSE TRACKER -->
<div id="trackerDiv" class="hidden">

  <!-- Totals on top -->
  <div id="totals">
    <p id="total1">Roommate 1: ₹0</p>
    <p id="total2">Roommate 2: ₹0</p>
    <p><strong id="totalAll">Total: ₹0</strong></p>
  </div>

  <!-- Expense form -->
  <form id="expenseForm">
    <label>Date: <input type="date" id="date" required></label><br>
    <label>Name: 
      <select id="name" required></select>
    </label><br>
    <label>Amount (₹): <input type="number" id="amount" min="1" required></label><br>
    <label>Description: <input type="text" id="desc" placeholder="Optional"></label><br>
    <button type="submit">Add Expense</button>
  </form>

  <table id="expenseTable">
    <thead>
      <tr><th>Date</th><th>Name</th><th>Amount (₹)</th><th>Description</th><th>Action</th></tr>
    </thead>
    <tbody></tbody>
  </table>

</div>

<script>
  // Initialize default admin credentials
  if (!localStorage.getItem('users')) {
    localStorage.setItem('users', JSON.stringify([
      {username:'admin', password:'admin123', role:'admin'}
    ]));
  }

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
  const totalAllP = document.getElementById('totalAll');

  let roommates = JSON.parse(localStorage.getItem('roommates')) || ['Roommate1','Roommate2'];
  let expenses = JSON.parse(localStorage.getItem('expenses')) || [];
  let currentUser = null;

  // Sign-in process
  signinForm.addEventListener('submit', e=>{
    e.preventDefault();
    const username = document.getElementById('username').value.trim();
    const password = document.getElementById('password').value.trim();
    const users = JSON.parse(localStorage.getItem('users'));

    const user = users.find(u=>u.username===username && u.password===password);
    if(!user) { alert('Invalid credentials'); return; }
    currentUser = user;
    signinDiv.classList.add('hidden');
    if(user.role==='admin'){
      adminDiv.classList.remove('hidden');
      document.getElementById('roommate1').value = roommates[0];
      document.getElementById('roommate2').value = roommates[1];
    } else {
      trackerDiv.classList.remove('hidden');
      updateNameOptions();
      updateTable();
    }
  });

  // Admin saves roommate names
  adminForm.addEventListener('submit', e=>{
    e.preventDefault();
    roommates[0] = document.getElementById('roommate1').value.trim();
    roommates[1] = document.getElementById('roommate2').value.trim();
    localStorage.setItem('roommates', JSON.stringify(roommates));
    alert('Roommates saved! Now normal users can log in.');
    adminDiv.classList.add('hidden');
    signinDiv.classList.remove('hidden');
  });

  function updateNameOptions(){
    nameSelect.innerHTML = '';
    roommates.forEach(r=>{
      const opt = document.createElement('option');
      opt.value = r; opt.textContent = r;
      nameSelect.appendChild(opt);
    });
  }

  // Expense form submission
  expenseForm.addEventListener('submit', e=>{
    e.preventDefault();
    const date = document.getElementById('date').value;
    const name = document.getElementById('name').value;
    const amount = parseFloat(document.getElementById('amount').value);
    const desc = document.getElementById('desc').value.trim();
    if(!date||!name||!amount||amount<=0){ alert('Fill all fields correctly'); return; }

    expenses.push({date,name,amount,desc});
    localStorage.setItem('expenses', JSON.stringify(expenses));
    updateTable();
    expenseForm.reset();
  });

  // Delete expense (only admin)
  function deleteExpense(index){
    if(currentUser.role !== 'admin') { alert('Only admin can delete expenses'); return; }
    if(confirm('Are you sure you want to delete this expense?')){
      expenses.splice(index,1);
      localStorage.setItem('expenses', JSON.stringify(expenses));
      updateTable();
    }
  }

  function updateTable(){
    expenseTableBody.innerHTML = '';
    let total1 = 0, total2 = 0;
    expenses.forEach(({date,name,amount,desc}, index)=>{
      const tr = document.createElement('tr');
      let actionHTML = currentUser && currentUser.role==='admin' ? `<span class="delete-btn" onclick="deleteExpense(${index})">Delete</span>` : '';
      tr.innerHTML = `<td>${date}</td><td>${name}</td><td>₹${amount}</td><td>${desc||''}</td><td>${actionHTML}</td>`;
      expenseTableBody.appendChild(tr);
      if(name===roommates[0]) total1 += amount;
      if(name===roommates[1]) total2 += amount;
    });
    total1P.textContent = `${roommates[0]}: ₹${total1}`;
    total2P.textContent = `${roommates[1]}: ₹${total2}`;
    totalAllP.textContent = `Total: ₹${total1+total2}`;
  }

</script>
</body>
</html>

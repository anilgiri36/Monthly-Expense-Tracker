<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker</title>
<style>
/* General Body & Layout */
body { 
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; 
    background: #f0f2f5; 
    margin: 0; 
    padding: 0; 
    display: flex; 
    justify-content: center; 
    align-items: flex-start; 
    min-height: 100vh;
}
.container { 
    width: 95%; 
    max-width: 1000px; 
    margin: 30px auto; 
    background: #fff; 
    border-radius: 15px; 
    box-shadow: 0 8px 20px rgba(0,0,0,0.15); 
    padding: 25px; 
}

/* Header */
h1 { 
    text-align: center; 
    background: linear-gradient(90deg, #E91E63, #FF4081); 
    color: white; 
    padding: 20px; 
    border-radius: 12px; 
    margin-top: 0; 
}
h2 { 
    text-align: center; 
    color: #333; 
    margin-bottom: 15px;
}

/* Sections */
.section { 
    background: #fafafa; 
    padding: 20px; 
    border-radius: 10px; 
    margin-bottom: 25px; 
    box-shadow: 0 3px 8px rgba(0,0,0,0.05); 
}

/* Forms */
input, select, button { 
    padding: 10px; 
    margin: 8px 0; 
    width: 100%; 
    border-radius: 8px; 
    border: 1px solid #ccc; 
}
button { 
    background:#E91E63; 
    color:white; 
    font-weight:bold; 
    cursor:pointer; 
    border:none; 
    transition:.3s; 
}
button:hover { 
    background:#d81b60; 
}

/* User Info Panel */
#userInfo { 
    display: flex; 
    align-items: center; 
    justify-content: space-between; 
    background: #fce4ec; 
    padding: 15px 20px; 
    border-radius: 12px; 
    margin-bottom: 25px; 
    box-shadow: 0 3px 8px rgba(0,0,0,0.1); 
}
#userInfo img { 
    width: 70px; 
    height: 70px; 
    border-radius: 50%; 
    border: 2px solid #E91E63; 
}
#userInfo div { 
    font-weight: bold; 
    color: #E91E63; 
    font-size: 18px; 
    flex-grow: 1; 
    margin-left: 15px; 
}
#userInfo button { 
    padding: 10px 20px; 
    border-radius: 8px; 
}

/* Totals Cards */
.totals { 
    display: flex; 
    justify-content: space-around; 
    flex-wrap: wrap; 
    margin-bottom: 20px; 
}
.totals div { 
    flex: 1 1 28%; 
    background: #E91E63; 
    color: white; 
    padding: 15px; 
    border-radius: 10px; 
    text-align: center; 
    font-weight: bold; 
    margin: 10px; 
}
.owes { color: green; font-weight: bold; }
.owed { color: red; font-weight: bold; }

/* Table */
table { 
    border-collapse: collapse; 
    width: 100%; 
    margin-top: 20px; 
    border-radius: 10px; 
    overflow: hidden; 
}
th, td { 
    border: 1px solid #ddd; 
    padding: 10px; 
    text-align: center; 
}
th { 
    background-color: #E91E63; 
    color: white; 
}
tr:nth-child(even){ background-color:#f9f9f9; }
.delete-btn { 
    color:white; 
    background:red; 
    border-radius:5px; 
    padding:5px 10px; 
    cursor:pointer; 
}

/* Responsive */
@media(max-width:768px){
    .totals { flex-direction: column; }
}
</style>
</head>
<body>
<div class="container">
<h1>Roommate Expense Tracker</h1>

<!-- User Info -->
<div id="userInfo" class="hidden">
  <img id="userAvatar" src="" alt="Avatar">
  <div id="userNameDisplay"></div>
  <button id="signOutBtn">Sign Out</button>
</div>

<!-- Sign-in -->
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
      <input type="text" id="roommateMobile1" placeholder="Roommate 1 Mobile">
      <input type="password" id="roommatePass1" placeholder="Roommate 1 Password">
      
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username">
      <input type="text" id="roommateMobile2" placeholder="Roommate 2 Mobile">
      <input type="password" id="roommatePass2" placeholder="Roommate 2 Password">
      
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username">
      <input type="text" id="roommateMobile3" placeholder="Roommate 3 Mobile">
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

<!-- Expense Tracker -->
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

<footer style="margin: 20px 0; color: gray; text-align:center;">
 <p>© 2026 Anil Giri. All rights reserved.</p>
</footer>

<script>
// (All previous JS logic for sign-in, avatar, admin, expenses, owes/owed, totals, etc. remains unchanged)
// You can copy the JS from the previous revised version here
</script>
</body>
</html>

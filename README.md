<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Roommate Expense Tracker</title>

<style>
body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background:#f4f7f8; margin:0; padding:0; position:relative; }
h1,h2,h3{text-align:center;color:#333;}
.container{max-width:900px;margin:20px auto;padding:20px;background:#fff;border-radius:10px;box-shadow:0 5px 15px rgba(0,0,0,0.1);}
input,select,button{padding:10px;margin:5px 0;width:100%;border-radius:5px;border:1px solid #ccc;}
button{background:#E91E63;color:white;font-weight:bold;cursor:pointer;border:none;}
button:hover{background:#d81b60;}
table{border-collapse:collapse;width:100%;margin-top:20px;}
th,td{border:1px solid #ddd;padding:10px;text-align:center;}
th{background-color:#E91E63;color:white;}
tr:nth-child(even){background-color:#f9f9f9;}
.totals{display:flex;justify-content:space-around;margin-bottom:20px;flex-wrap:wrap;}
.totals div{background:#E91E63;color:white;padding:15px;border-radius:8px;width:28%;text-align:center;font-weight:bold;margin-bottom:10px;}
.delete-btn{color:white;background:red;border-radius:5px;padding:5px 10px;cursor:pointer;}
.section{background:#fafafa;padding:15px;border-radius:8px;margin-bottom:20px;}
.hidden{display:none;}
.user-row{display:flex;justify-content:space-between;margin:5px 0;}
#userInfo { position:absolute; top:20px; right:20px; font-weight:bold; color:#E91E63; }
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
      <input type="text" id="roommateName1" placeholder="Roommate 1 Username">
      <input type="password" id="roommatePass1" placeholder="Password">
      <input type="text" id="roommateName2" placeholder="Roommate 2 Username">
      <input type="password" id="roommatePass2" placeholder="Password">
      <input type="text" id="roommateName3" placeholder="Roommate 3 Username">
      <input type="password" id="roommatePass3" placeholder="Password">
      <button type="submit">Save</button>
    </form>
    <div id="roommateList"></div>
  </div>

</div>

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
      <select id="name"></select>
      <input type="number" id="amount" placeholder="Amount" required>
      <button>Add Expense</button>
    </form>
  </div>

  <table id="expenseTable">
    <thead>
      <tr><th>Date</th><th>Name</th><th>Amount</th><th>Action</th></tr>
    </thead>
    <tbody></tbody>
  </table>

</div>

</div> <!-- ✅ FIXED: container CLOSED -->

<script>
// your same JS (unchanged)
</script>

<footer style="text-align:center; margin:20px; color:gray;">
  <p>© 2026 Anil Giri. All rights reserved.</p>
  <p style="color:green; font-weight:bold;">A page created by Anil Giri</p>
</footer>

</body>
</html>

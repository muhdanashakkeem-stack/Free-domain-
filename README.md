<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Aflaha Traders Billing</title>

<style>
body {
    font-family: Arial;
    background: #0a192f;
    color: white;
    margin: 0;
    padding: 10px;
}
.container {
    background: #112240;
    padding: 15px;
    border-radius: 10px;
    max-width: 1000px;
    margin: auto;
}
h2,h3 { text-align:center; color:#64ffda; }

#datetime {
    text-align:center;
    margin-bottom:10px;
    font-size:14px;
    color:#ccc;
}

input, select {
    padding:10px;
    margin:5px;
    border-radius:5px;
    border:1px solid #233554;
    background:#0a192f;
    color:white;
}

button {
    padding:10px;
    border:none;
    border-radius:5px;
    background:#64ffda;
    cursor:pointer;
    font-weight:bold;
}
button:hover { background:#52e0c4; }

.error { color:red; display:none; }

table {
    width:100%;
    border-collapse:collapse;
}
th,td {
    padding:10px;
    border-bottom:1px solid #233554;
    text-align:center;
}
th { background:#1f4068; color:#64ffda; }

.total {
    text-align:right;
    font-size:18px;
    margin-top:10px;
}
</style>
</head>

<body>

<!-- LOGIN -->
<div class="container" id="login">
    <h2>🔒 Aflaha Traders</h2>
    <input type="password" id="pass" placeholder="Password">
    <br>
    <button onclick="checkLogin()">Login</button>
    <p id="err" class="error">Wrong Password</p>
</div>

<!-- APP -->
<div id="app" style="display:none;">
<div class="container">

<h2>Aflaha Traders Billing</h2>
<div id="datetime"></div>
<h3 id="shopTitle">Shop:</h3>

<input id="shop" placeholder="Shop Name">
<input id="product" list="plist" placeholder="Product">
<datalist id="plist"></datalist>

<input id="qty" type="number" placeholder="Qty">
<select id="unit">
    <option value="kg">KG</option>
    <option value="g">GRAM</option>
    <option value="l">LITRE</option>
</select>
<input id="price" type="number" placeholder="Rate">

<button onclick="addItem()">➕ Add</button>

<!-- GST -->
<div style="margin-top:10px;">
    GST %: <input id="gstPercent" type="number" value="0" style="width:80px;">
    <button onclick="toggleGST()" id="gstBtn" style="background:gray;color:white;">
        GST OFF
    </button>
</div>

<table>
<thead>
<tr>
<th>Product</th>
<th>Qty</th>
<th>Unit</th>
<th>Rate</th>
<th>Total</th>
<th>Delete</th>
</tr>
</thead>
<tbody id="body"></tbody>
</table>

<div class="total">
Subtotal ₹ <span id="subtotal">0.00</span><br>
GST ₹ <span id="gstAmount">0.00</span><br>
<b>Final ₹ <span id="total">0.00</span></b>
</div>

<br>
<button onclick="prev()">⬅ Prev</button>
<button onclick="next()">Next ➡</button>
<br><br>

<button onclick="window.print()">📄 Download Bill</button>
<button onclick="report()">📚 Daily Report</button>
<button onclick="logout()" style="background:red;color:white;">Logout</button>

</div>
</div>

<script>

// LOGIN
const PASSWORD = "12345678";

function checkLogin(){
    let p = document.getElementById("pass").value;
    if(p === PASSWORD){
        document.getElementById("login").style.display="none";
        document.getElementById("app").style.display="block";

        updateTime();
        setInterval(updateTime,1000);
    } else {
        document.getElementById("err").style.display="block";
    }
}

function logout(){
    location.reload();
}

// TIME
function updateTime(){
    let el = document.getElementById("datetime");
    if(!el) return;

    let now = new Date();
    let date = now.toLocaleDateString();
    let time = now.toLocaleTimeString();
    el.innerText = date + " | " + time;
}

// PRODUCTS
let products = [
"MASALA KADALA","ROSTED KADALA","GREEN PEECE",
"MULAKU MIXTURE","GARLIC MIXTURE","BOMBAY MIXTURE"
];

function loadProducts(){
    let d=document.getElementById("plist");
    d.innerHTML="";
    products.forEach(p=>{
        let o=document.createElement("option");
        o.value=p;
        d.appendChild(o);
    });
}
loadProducts();

// BILL SYSTEM
let bills=[{shop:"",items:[],total:0}];
let i=0;
let gstEnabled=false;

function addItem(){
    let shop=document.getElementById("shop").value;
    let product=document.getElementById("product").value.toUpperCase();
    let qty=parseFloat(document.getElementById("qty").value);
    let price=parseFloat(document.getElementById("price").value);
    let unit=document.getElementById("unit").value;

    if(!shop||!product||!qty||!price){
        alert("Fill all fields");
        return;
    }

    if(!products.includes(product)){
        products.push(product);
        loadProducts();
    }

    let total=qty*price;

    let b=bills[i];
    b.shop=shop;
    b.items.push({product,qty,unit,price,total});
    b.total+=total;

    render();

    document.getElementById("product").value="";
    document.getElementById("qty").value="";
    document.getElementById("price").value="";
}

function render(){
    let b=bills[i];
    let body=document.getElementById("body");
    if(!body) return;

    body.innerHTML="";
    let subtotal = 0;

    b.items.forEach((x,index)=>{
        subtotal += x.total;

        body.innerHTML+=`
        <tr>
        <td>${x.product}</td>
        <td>${x.qty}</td>
        <td>${x.unit}</td>
        <td>₹${x.price}</td>
        <td>₹${x.total.toFixed(2)}</td>
        <td><button onclick="deleteItem(${index})" style="background:red;color:white;">❌</button></td>
        </tr>`;
    });

    let gstPercent = parseFloat(document.getElementById("gstPercent").value) || 0;
    let gstAmount = gstEnabled ? (subtotal * gstPercent / 100) : 0;
    let finalTotal = subtotal + gstAmount;

    document.getElementById("subtotal").innerText = subtotal.toFixed(2);
    document.getElementById("gstAmount").innerText = gstAmount.toFixed(2);
    document.getElementById("total").innerText = finalTotal.toFixed(2);

    document.getElementById("shopTitle").innerText="Shop: "+b.shop;
}

function deleteItem(index){
    let b = bills[i];
    b.total -= b.items[index].total;
    b.items.splice(index, 1);
    render();
}

function toggleGST(){
    gstEnabled = !gstEnabled;
    let btn=document.getElementById("gstBtn");

    if(gstEnabled){
        btn.innerText="GST ON";
        btn.style.background="green";
    } else {
        btn.innerText="GST OFF";
        btn.style.background="gray";
    }
    render();
}

function next(){
    i++;
    if(!bills[i]) bills.push({shop:"",items:[],total:0});
    render();
}

function prev(){
    if(i>0){ i--; render(); }
}

// REPORT
function report(){

    let totalSales = 0;
    let allHTML = "";

    bills.forEach((b,index)=>{
        if(b.items.length===0) return;

        allHTML += `<h3>Bill ${index+1} - ${b.shop}</h3>`;
        allHTML += `<table border="1" style="width:100%;border-collapse:collapse;text-align:center;margin-bottom:20px;">
        <tr>
        <th>Product</th><th>Qty</th><th>Unit</th><th>Rate</th><th>Total</th>
        </tr>`;

        b.items.forEach(x=>{
            totalSales += x.total;

            allHTML += `
            <tr>
            <td>${x.product}</td>
            <td>${x.qty}</td>
            <td>${x.unit}</td>
            <td>₹${x.price}</td>
            <td>₹${x.total.toFixed(2)}</td>
            </tr>`;
        });

        allHTML += `</table>`;
    });

    let win = window.open("", "_blank");

    win.document.write(`
    <html>
    <head>
    <title>Daily Report</title>
    </head>
    <body style="font-family:Arial;padding:20px;">

    <h2 style="text-align:center;">Aflaha Traders - Daily Report</h2>
    <h3>Total Sales: ₹${totalSales.toFixed(2)}</h3>

    <h3>Expenses</h3>
    Driver: <input id="d" type="number" value="0"><br>
    Salesman: <input id="s" type="number" value="0"><br>
    Petrol: <input id="p" type="number" value="0"><br>
    Food: <input id="f" type="number" value="0"><br>

    <button onclick="calc()">Calculate Profit</button>
    <button onclick="window.print()">Download</button>

    <h3 id="result">
    Expense ₹0 <br>
    Profit ₹${totalSales.toFixed(2)}
    </h3>

    <hr>
    <h2>All Orders</h2>
    ${allHTML}

    <script>
    function calc(){
        let d=+document.getElementById('d').value||0;
        let s=+document.getElementById('s').value||0;
        let p=+document.getElementById('p').value||0;
        let f=+document.getElementById('f').value||0;

        let exp=d+s+p+f;
        let sales=${totalSales};
        let profit=sales-exp;

        document.getElementById("result").innerHTML=
        "Expense ₹"+exp.toFixed(2)+"<br>Profit ₹"+profit.toFixed(2);
    }
    <\/script>

    </body>
    </html>
    `);

    win.document.close();
}

</script>

</body>
</html>

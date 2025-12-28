<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Anna-Lynn Collections</title>
<style>
:root{--accent:#e91e63;--card:#fff;--shadow:0 6px 20px rgba(0,0,0,.08);}
body{margin:0;font-family:Arial;background:#fafafa;}
header{background:#111;color:#fff;padding:18px;text-align:center;font-size:22px;}
.container{max-width:1100px;margin:auto;padding:16px;}
nav{display:flex;gap:10px;flex-wrap:wrap;justify-content:center;margin-bottom:14px;}
.nav-btn{padding:8px 14px;border-radius:20px;border:0;cursor:pointer;box-shadow:var(--shadow);}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(180px,1fr));gap:16px;}
.card{background:var(--card);padding:10px;border-radius:12px;box-shadow:var(--shadow);}
.card img{width:100%;height:160px;object-fit:cover;border-radius:8px;}
.price{color:var(--accent);font-weight:bold;}
.btn{padding:6px 10px;border:0;border-radius:6px;cursor:pointer;}
.btn-primary{background:var(--accent);color:#fff;}
.btn-outline{background:#fff;border:1px solid #ccc;}
.cart-modal{position:fixed;right:10px;top:70px;background:#fff;width:320px;border-radius:12px;box-shadow:0 20px 40px rgba(0,0,0,.25);padding:12px;display:none;z-index:100;}
.cart-item{display:flex;justify-content:space-between;font-size:14px;}
footer{background:#111;color:#fff;text-align:center;padding:14px;margin-top:20px;}
input,select{padding:8px;width:100%;margin-bottom:6px;}
</style>
</head>
<body>

<header>Anna-Lynn Collections</header>
<div class="container">

<nav>
<button class="nav-btn" onclick="filterCat('all')">All</button>
<button class="nav-btn" onclick="filterCat('perfumes')">Perfumes</button>
<button class="nav-btn" onclick="filterCat('wigs')">Wigs</button>
<button class="nav-btn" onclick="filterCat('bags')">Bags</button>
<button class="nav-btn" onclick="filterCat('clothes')">Clothes</button>
<button class="nav-btn" onclick="openAdmin()">Admin</button>
<button class="nav-btn" onclick="toggleCart()">Cart (<span id="cart-count">0</span>)</button>
</nav>

<!-- ADMIN PANEL -->
<section id="admin-panel" style="display:none;background:#fff;padding:16px;border-radius:12px;box-shadow:0 6px 20px rgba(0,0,0,.08);margin-bottom:20px">
<h2>Admin Dashboard</h2>

<h3 id="admin-form-title">Add New Product</h3>
<input type="hidden" id="edit-id">
<input id="p-name" placeholder="Product Name">
<input id="p-price" type="number" placeholder="Price">
<input id="p-stock" type="number" placeholder="Stock">
<select id="p-cat">
<option value="perfumes">Perfumes</option>
<option value="wigs">Wigs</option>
<option value="bags">Bags</option>
<option value="clothes">Clothes</option>
</select>
<input id="p-img" placeholder="Image URL">
<a href="https://imgbb.com/" target="_blank" class="btn btn-outline">Upload Image</a>
<button class="btn btn-primary" onclick="addProduct()">Add Product</button>

<hr style="margin:20px 0">
<h3>Manage Products</h3>
<div id="admin-products"></div>

<hr style="margin:20px 0">
<h3>Order History</h3>
<div id="order-history" style="max-height:300px;overflow:auto;"></div>
<button class="btn" onclick="renderOrders()">Refresh Orders</button>

<hr style="margin:20px 0">
<h3>Sales Report</h3>
<div id="sales-report"></div>
<button class="btn" onclick="calculateSales()">Refresh Sales</button>
</section>

<div class="grid" id="catalog"></div>
</div>

<!-- CART -->
<div class="cart-modal" id="cart">
<h3>Your Cart</h3>
<div id="cart-items"></div>
<hr>
<input id="cust-name" placeholder="Customer Name">
<input id="cust-phone" placeholder="Phone Number">
<input id="cust-address" placeholder="Delivery Address">
<h4>Total: ₦<span id="cart-total">0</span></h4>
<button class="btn btn-primary" onclick="checkout()">Checkout</button>
</div>

<!-- ADMIN LOGIN -->
<div id="login" style="display:none;position:fixed;inset:0;background:rgba(0,0,0,.6);z-index:200;align-items:center;justify-content:center;display:flex">
<div style="background:#fff;padding:20px;border-radius:10px;width:280px">
<h3>Admin Login</h3>
<input id="admin-pass" type="password" placeholder="Password">
<button class="btn btn-primary" onclick="login()">Login</button>
</div>
</div>

<script src="https://js.paystack.co/v1/inline.js"></script>
<script>
const ADMIN_PASS="Annalynn890";
let isAdmin=false;
let products=JSON.parse(localStorage.getItem("products"))||[
{id:"p1",name:"Perfume 1",price:10000,stock:5,category:"perfumes",img:"https://via.placeholder.com/300"}
];
let cart={};
let orders=JSON.parse(localStorage.getItem("orders")||"[]");

function save(){localStorage.setItem("products",JSON.stringify(products))}
function saveOrders(){localStorage.setItem("orders",JSON.stringify(orders))}

function render(cat="all"){
catalog.innerHTML="";
products.filter(p=>cat==="all"||p.category===cat).forEach(p=>{
catalog.innerHTML+=`
<div class="card">
<img src="${p.img}">
<b>${p.name}</b>
<div class="price">₦${p.price.toLocaleString()}</div>
<div>Stock: ${p.stock}</div>
<button class="btn" onclick="addToCart('${p.id}')">Add</button>
${isAdmin?`<button class="btn" onclick="del('${p.id}')">Delete</button><button class="btn" onclick="editProduct('${p.id}')">Edit</button>`:""}
</div>`;
});
updateAdminPanel();
updateCart();
}

function filterCat(c){render(c)}

function addToCart(id){
if(!cart[id]) cart[id]=0;
cart[id]++;
updateCart();
}

function updateCart(){
cart-items.innerHTML="";
let total=0,count=0;
for(let id in cart){
let p=products.find(x=>x.id===id);
total+=p.price*cart[id];
count+=cart[id];
cart-items.innerHTML+=`<div class="cart-item">${p.name} x${cart[id]}</div>`;
}
cart-total.textContent=total.toLocaleString();
cart-count.textContent=count;
}

function toggleCart(){cart.style.display=cart.style.display?"":"block"}

function checkout(){
if(!cust-name.value||!cust-phone.value||!cust-address.value){alert("Fill customer details");return;}
let customer={name:cust-name.value,phone:cust-phone.value,address:cust-address.value,email:"customer@example.com"};
let cartItems=Object.keys(cart).map(id=>{let p=products.find(x=>x.id===id);return {name:p.name,qty:cart[id],total:p.price*cart[id]};});
let total=cartItems.reduce((s,i)=>s+i.total,0);

payWithPaystack(total,customer,cartItems);
}

function payWithPaystack(total,customer,cartItems){
var handler = PaystackPop.setup({
key: 'YOUR_PUBLIC_KEY',
email: customer.email,
amount: total*100,
currency: "NGN",
ref: ''+Math.floor(Math.random()*1000000000+1),
metadata: {custom_fields:[{display_name:"Name",value:customer.name},{display_name:"Phone",value:customer.phone},{display_name:"Address",value:customer.address}]},
callback:function(response){
alert('Payment successful! Ref: '+response.reference);
addOrder(customer,cartItems,total);
renderOrders();
calculateSales();
cart={};
updateCart();
},
onClose:function(){alert('Payment window closed.')}
});
handler.openIframe();
}

function addOrder(customer, cartItems, total){
orders.push({id:Date.now(),customer,items:cartItems,total,date:new Date().toLocaleString()});
saveOrders();
}

function renderOrders(){
const container=document.getElementById("order-history");
container.innerHTML="";
if(orders.length===0){container.innerHTML="<div>No orders yet</div>";return;}
orders.forEach(o=>{
const div=document.createElement("div");
div.style.border="1px solid #ddd";div.style.padding="8px";div.style.marginBottom="8px";
div.innerHTML=`<b>${o.customer.name}</b> | ${o.date} | ₦${o.total.toLocaleString()}<div>${o.items.map(i=>i.name+" x"+i.qty+" – ₦"+i.total.toLocaleString()).join("<br>")}</div>`;
container.appendChild(div);
});
}

function calculateSales(){
let totalSales = orders.reduce((s,o)=>s+o.total,0);
let monthly = {};
orders.forEach(o=>{
let month = new Date(o.date).toLocaleString("en-US",{month:"short",year:"numeric"});
monthly[month] = (monthly[month]||0)+o.total;
});
let reportDiv = document.getElementById("sales-report");
reportDiv.innerHTML=`<b>Total Sales:</b> ₦${totalSales.toLocaleString()}<br>`;
for(let m in monthly) reportDiv.innerHTML+=`${m}: ₦${monthly[m].toLocaleString()}<br>`;
}

function openAdmin(){isAdmin?admin-panel.style.display="block":login.style.display="flex"}

function login(){
const pass=document.getElementById("admin-pass").value;
if(pass===ADMIN_PASS){isAdmin=true;login.style.display="none";admin-panel.style.display="block";render();}
else alert("Wrong password");
}

function addProduct(){
const id=document.getElementById("edit-id").value;
if(!p-name.value||!p-price.value||!p-stock.value){alert("Fill all fields");return;}
if(id){
const p=products.find(x=>x.id===id);p.name=p-name.value;p.price=+p-price.value;p.stock=+p-stock.value;p.category=p-cat.value;p.img=p-img.value||p.img;alert("Product updated");
}else{
products.push({id:Date.now()+"",name:p-name.value,price:+p-price.value,stock:+p-stock.value,category:p-cat.value,img:p-img.value||"https://via.placeholder.com/300"});alert("Product added");
}
save();render();
document.getElementById("edit-id").value="";p-name.value=p-price.value=p-stock.value=p-img.value="";document.getElementById("admin-form-title").textContent="Add New Product";
}

function editProduct(id){
const p=products.find(x=>x.id===id);if(!p)return;
document.getElementById("edit-id").value=p.id;document.getElementById("p-name").value=p.name;document.getElementById("p-price").value=p.price;
document.getElementById("p-stock").value=p.stock;document.getElementById("p-cat").value=p.category;document.getElementById("p-img").value=p.img;
document.getElementById("admin-form-title").textContent="Edit Product";
}

function del(id){if(!confirm("Delete this product?"))return;products=products.filter(p=>p.id!==id);save();render();}

function updateAdminPanel(){if(!isAdmin)return;
let container=document.getElementById("admin-products");container.innerHTML="";
products.forEach(p=>{container.innerHTML+=`<div class="card" style="display:flex;justify-content:space-between;align-items:center"><div><b>${p.name}</b><br>₦${p.price.toLocaleString()} | Stock: ${p.stock}</div><div><button class="btn" onclick="editProduct('${p.id}')">Edit</button><button class="btn" onclick="del('${p.id}')">Delete</button></div></div>`;});}

render();
</script>

<footer>📍 No 16, Dallimore Street | 📞 08167600577</footer>
</body>
</html>

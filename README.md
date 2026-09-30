from flask import Flask, request, render_template_string
from urllib.parse import quote
import json

app = Flask(__name__)

SHOP_NAME = "Dharmender Distributor"
WHATSAPP_NUMBER = "916204467157"

PRODUCTS = [
    {"id": 1, "name": "Notebook", "price": 30, "emoji": "📓"},
    {"id": 2, "name": "Premium Notebook", "price": 40, "emoji": "📔"},
    {"id": 3, "name": "Sharpener", "price": 10, "emoji": "✏️"},
    {"id": 4, "name": "Ball Pen", "price": 10, "emoji": "🖊️"},
    {"id": 5, "name": "Pencil", "price": 5, "emoji": "✏️"},
    {"id": 6, "name": "Eraser", "price": 5, "emoji": "⬜"},
    {"id": 7, "name": "Geometry Box", "price": 80, "emoji": "📐"},
    {"id": 8, "name": "Color Pencil Set", "price": 60, "emoji": "🌈"}
]

HTML = """
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Dharmender Distributor</title>

<style>
* {
    box-sizing: border-box;
}

body {
    margin: 0;
    font-family: Arial, sans-serif;
    background: #f3f4f6;
    color: #111827;
}

.header {
    background: linear-gradient(135deg, #111827, #2563eb);
    color: white;
    text-align: center;
    padding: 35px 15px;
    border-radius: 0 0 28px 28px;
}

.header h1 {
    margin: 0;
    font-size: 30px;
}

.header p {
    margin-bottom: 0;
}

.container {
    max-width: 1000px;
    margin: auto;
    padding: 20px;
}

.products {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
    gap: 16px;
}

.product {
    background: white;
    padding: 18px;
    border-radius: 18px;
    text-align: center;
    box-shadow: 0 4px 15px rgba(0,0,0,.08);
}

.icon {
    font-size: 55px;
    background: #eef2ff;
    padding: 12px;
    border-radius: 15px;
}

.product h3 {
    margin-bottom: 5px;
}

.price {
    color: #2563eb;
    font-size: 22px;
    font-weight: bold;
    margin: 10px;
}

.qty {
    width: 70px;
    padding: 9px;
    text-align: center;
    border: 1px solid #ddd;
    border-radius: 8px;
}

.add {
    width: 100%;
    margin-top: 10px;
    padding: 12px;
    border: 0;
    border-radius: 10px;
    background: #2563eb;
    color: white;
    font-weight: bold;
}

.cart {
    background: white;
    margin-top: 25px;
    padding: 22px;
    border-radius: 20px;
    box-shadow: 0 4px 15px rgba(0,0,0,.08);
}

.cart-item {
    padding: 10px 0;
    border-bottom: 1px solid #eee;
}

.total {
    font-size: 24px;
    font-weight: bold;
    margin: 20px 0;
}

input, textarea, select {
    width: 100%;
    padding: 13px;
    margin: 6px 0 13px;
    border: 1px solid #d1d5db;
    border-radius: 10px;
    font-size: 15px;
}

textarea {
    height: 90px;
    resize: vertical;
}

.cod {
    background: #ecfdf5;
    border: 1px solid #86efac;
    padding: 14px;
    border-radius: 10px;
    margin-bottom: 15px;
}

.order-button {
    width: 100%;
    padding: 16px;
    border: 0;
    border-radius: 12px;
    background: #16a34a;
    color: white;
    font-size: 17px;
    font-weight: bold;
}

.remove {
    border: 0;
    background: #ef4444;
    color: white;
    border-radius: 6px;
    padding: 5px 8px;
    float: right;
}
</style>
</head>

<body>

<div class="header">
    <h1>🛍️ Dharmender Distributor</h1>
    <p>Stationery Products • Easy Ordering • Cash on Delivery</p>
</div>

<div class="container">

<h2>📦 Products</h2>

<div class="products">

{% for p in products %}
<div class="product">

    <div class="icon">{{ p.emoji }}</div>

    <h3>{{ p.name }}</h3>

    <div class="price">₹{{ p.price }}</div>

    <input
        class="qty"
        id="qty{{ p.id }}"
        type="number"
        min="1"
        value="1"
    >

    <button
        class="add"
        onclick="addProduct({{ p.id }}, '{{ p.name }}', {{ p.price }})"
    >
        Add to Cart
    </button>

</div>
{% endfor %}

</div>


<div class="cart">

<h2>🛒 Your Cart</h2>

<div id="cart">
    <p>Your cart is empty.</p>
</div>

<div class="total">
    Total: ₹<span id="total">0</span>
</div>


<h2>📍 Delivery Details</h2>

<form method="POST" action="/order" onsubmit="return sendOrder();">

<label>Customer Name</label>

<input
    type="text"
    name="name"
    placeholder="Full name"
    required
>


<label>Mobile Number</label>

<input
    type="tel"
    name="phone"
    placeholder="10 digit mobile number"
    pattern="[0-9]{10}"
    maxlength="10"
    required
>


<label>Complete Address</label>

<textarea
    name="address"
    placeholder="House number, street, area, landmark"
    required
></textarea>


<label>City / Village</label>

<input
    type="text"
    name="city"
    placeholder="City / Village"
    required
>


<label>PIN Code</label>

<input
    type="text"
    name="pincode"
    placeholder="6 digit PIN code"
    pattern="[0-9]{6}"
    maxlength="6"
    required
>


<label>When do you need the order?</label>

<select name="delivery_time" required>

    <option value="">
        Select delivery time
    </option>

    <option>
        As soon as possible
    </option>

    <option>
        Today
    </option>

    <option>
        Tomorrow
    </option>

    <option>
        Within 2-3 days
    </option>

    <option>
        Within 1 week
    </option>

</select>


<div class="cod">

    💵 <b>Payment: Cash on Delivery</b>

    <br>

    Pay when your order is delivered.

</div>


<input
    type="hidden"
    name="order_data"
    id="order_data"
>


<button class="order-button" type="submit">

    📱 Place Order & Send to WhatsApp

</button>

</form>

</div>

</div>


<script>

let cart = [];


function addProduct(id, name, price) {

    let input = document.getElementById("qty" + id);

    let quantity = parseInt(input.value);

    if (!quantity || quantity < 1) {
        alert("Quantity must be at least 1.");
        return;
    }

    let existing = cart.find(function(item) {
        return item.id === id;
    });

    if (existing) {
        existing.quantity += quantity;
    } else {
        cart.push({
            id: id,
            name: name,
            price: price,
            quantity: quantity
        });
    }

    showCart();
}


function removeProduct(id) {

    cart = cart.filter(function(item) {
        return item.id !== id;
    });

    showCart();
}


function showCart() {

    let box = document.getElementById("cart");

    let total = 0;

    if (cart.length === 0) {

        box.innerHTML = "<p>Your cart is empty.</p>";

        document.getElementById("total").innerText = "0";

        return;
    }

    let html = "";

    cart.forEach(function(item) {

        let subtotal = item.price * item.quantity;

        total += subtotal;

        html += `
        <div class="cart-item">

            <button
                class="remove"
                onclick="removeProduct(${item.id})"
                type="button"
            >
                X
            </button>

            <b>${item.name}</b>

            <br>

            Quantity: ${item.quantity}

            <br>

            ₹${item.price} × ${item.quantity}
            = <b>₹${subtotal}</b>

        </div>
        `;
    });

    box.innerHTML = html;

    document.getElementById("total").innerText = total;
}


function sendOrder() {

    if (cart.length === 0) {

        alert("Please add at least one product.");

        return false;
    }

    document.getElementById("order_data").value =
        JSON.stringify(cart);

    return true;
}

</script>

</body>
</html>
"""


SUCCESS = """
<!DOCTYPE html>
<html>

<head>

<meta charset="UTF-8">

<meta name="viewport"
content="width=device-width, initial-scale=1.0">

<title>Order Received</title>

<style>

body {
    margin: 0;
    padding: 25px;
    background: #f3f4f6;
    font-family: Arial, sans-serif;
}

.box {
    max-width: 550px;
    margin: auto;
    background: white;
    padding: 25px;
    border-radius: 20px;
    box-shadow: 0 5px 20px rgba(0,0,0,.1);
}

.success {
    text-align: center;
    font-size: 55px;
}

h1 {
    text-align: center;
    color: #16a34a;
}

.details {
    background: #f9fafb;
    padding: 18px;
    border-radius: 12px;
    line-height: 1.7;
}

.whatsapp {
    display: block;
    text-align: center;
    text-decoration: none;
    background: #25D366;
    color: white;
    padding: 16px;
    border-radius: 12px;
    margin-top: 20px;
    font-weight: bold;
    font-size: 17px;
}

.home {
    display: block;
    text-align: center;
    text-decoration: none;
    background: #2563eb;
    color: white;
    padding: 14px;
    border-radius: 12px;
    margin-top: 12px;
}

</style>

</head>

<body>

<div class="box">

<div class="success">✅</div>

<h1>Order Ready!</h1>

<p>
Your order details have been prepared.
</p>

<div class="details">

<b>Customer:</b>
{{ name }}

<br>

<b>Mobile:</b>
{{ phone }}

<br>

<b>Address:</b>
{{ address }}

<br>

<b>City:</b>
{{ city }}

<br>

<b>PIN:</b>
{{ pincode }}

<br>

<b>Delivery:</b>
{{ delivery }}

<br>

<b>Payment:</b>
Cash on Delivery

<br>

<b>Total:</b>
₹{{ total }}

<hr>

<b>Products:</b>

<pre style="white-space:pre-wrap;">{{ products }}</pre>

</div>


<a
class="whatsapp"
href="{{ whatsapp }}"
target="_blank"
>

📱 SEND ORDER TO OWNER
<br>
6204467157

</a>


<a
class="home"
href="/"
>

🛍️ Back to Shop

</a>

</div>

</body>

</html>
"""


@app.route("/")
def home():

    return render_template_string(
        HTML,
        products=PRODUCTS
    )


@app.route("/order", methods=["POST"])
def order():

    name = request.form.get("name", "").strip()
    phone = request.form.get("phone", "").strip()
    address = request.form.get("address", "").strip()
    city = request.form.get("city", "").strip()
    pincode = request.form.get("pincode", "").strip()
    delivery = request.form.get("delivery_time", "").strip()

    try:
        cart = json.loads(
            request.form.get("order_data", "[]")
        )
    except Exception:
        cart = []

    if not cart:
        return "No products selected."

    total = 0
    product_lines = []

    for item in cart:

        item_name = str(item.get("name", ""))
        price = float(item.get("price", 0))
        quantity = int(item.get("quantity", 0))

        subtotal = price * quantity
        total += subtotal

        product_lines.append(
            f"• {item_name} × {quantity} = ₹{subtotal:g}"
        )

    products_text = "\n".join(product_lines)

    message = f"""🛍️ NEW ORDER
━━━━━━━━━━━━━━━━━━

🏪 SHOP:
{SHOP_NAME}

👤 CUSTOMER:
{name}

📱 MOBILE:
{phone}

📍 COMPLETE ADDRESS:
{address}

🏙️ CITY / VILLAGE:
{city}

📮 PIN CODE:
{pincode}

📦 PRODUCTS:
{products_text}

━━━━━━━━━━━━━━━━━━

💰 TOTAL:
₹{total:g}

💵 PAYMENT:
CASH ON DELIVERY

⏰ REQUIRED DELIVERY:
{delivery}

━━━━━━━━━━━━━━━━━━

Please confirm the order.
"""

    whatsapp_url = (
        "https://wa.me/"
        + WHATSAPP_NUMBER
        + "?text="
        + quote(message)
    )

    return render_template_string(
        SUCCESS,
        name=name,
        phone=phone,
        address=address,
        city=city,
        pincode=pincode,
        delivery=delivery,
        total=f"{total:g}",
        products=products_text,
        whatsapp=whatsapp_url
    )


if __name__ == "__main__":
    app.run(
        host="0.0.0.0",
        port=5000,
        debug=False
    )from flask import Flask, request, render_template_string, redirect, url_for
import json

app = Flask(__name__)

SHOP_NAME = "Dharmender Distributor"
WHATSAPP_NUMBER = "916204467157"

PRODUCTS = [
    {"id": 1, "name": "Notebook", "price": 30, "emoji": "📓", "desc": "Best quality notebook for school", "stock": 100},
    {"id": 2, "name": "Premium Notebook", "price": 40, "emoji": "📒", "desc": "Premium pages", "stock": 50},
    {"id": 3, "name": "Sharpener", "price": 10, "emoji": "✏️", "desc": "Sharp sharpener", "stock": 200},
    {"id": 4, "name": "Ball Pen", "price": 10, "emoji": "🖊️", "desc": "Smooth writing", "stock": 200},
    {"id": 5, "name": "Pencil", "price": 5, "emoji": "✏️", "desc": "HB Pencil", "stock": 500},
    {"id": 6, "name": "Eraser", "price": 5, "emoji": "🧼", "desc": "Clean eraser", "stock": 300},
    {"id": 7, "name": "Geometry Box", "price": 80, "emoji": "📐", "desc": "Full geometry kit", "stock": 20},
    {"id": 8, "name": "Scale", "price": 15, "emoji": "📏", "desc": "30cm scale", "stock": 100},
]    {"id": 8, "name": "Scale", "price": 15, "emoji": "📏", "desc": "30cm scale", "stock": 100},
    {"id": 9, "name": "Condom", "price": 20, "emoji": "🛡️", "desc": "Safe condom - 1 pc", "stock": 100},

HOME_HTML = """
<h1 style="text-align:center">{{shop}}</h1>
<div style="display:flex; flex-wrap:wrap; gap:12px; justify-content:center; padding:10px">
{% for p in products %}
<a href="/product/{{p.id}}" style="border:1px solid #ddd; width:150px; padding:10px; border-radius:12px; text-decoration:none; color:black; text-align:center">
<div style="font-size:40px">{{p.emoji}}</div>
<h3>{{p.name}}</h3>
<p>Rs. {{p.price}}</p>
</a>
{% endfor %}
</div>
"""

DETAIL_HTML = """
<div style="max-width:500px; margin:auto; padding:20px; font-family:sans-serif">
<a href="/">← Back</a>
<div style="text-align:center; font-size:80px">{{p.emoji}}</div>
<h2>{{p.name}}</h2>
<p>Price: <b>Rs. {{p.price}}</b></p>
<p>Stock: {{p.stock}}</p>
<p>{{p.desc}}</p>
<a href="https://wa.me/{{whatsapp}}?text=Order {{p.name}} Rs.{{p.price}}" 
style="display:block; background:green; color:white; text-align:center; padding:12px; border-radius:8px; text-decoration:none">WhatsApp pe Order Karo</a>
<hr style="margin:20px 0">
<h3>🔧 Yahi se Update Karo (Admin)</h3>
<form method="POST" style="display:flex; flex-direction:column; gap:8px">
<input name="name" value="{{p.name}}">
<input name="price" value="{{p.price}}" type="number">
<input name="stock" value="{{p.stock}}" type="number">
<input name="desc" value="{{p.desc}}">
<input name="emoji" value="{{p.emoji}}">
<button style="background:black; color:white; padding:10px; border-radius:6px">Update Save Karo</button>
</form>
</div>
"""

@app.route('/')
def home():
    return render_template_string(HOME_HTML, products=PRODUCTS, shop=SHOP_NAME)

@app.route('/product/<int:pid>', methods=['GET','POST'])
def detail(pid):
    p = next((x for x in PRODUCTS if x['id']==pid), None)
    if not p: return "Not found"
    if request.method == 'POST':
        p['name'] = request.form['name']
        p['price'] = int(request.form['price'])
        p['stock'] = int(request.form['stock'])
        p['desc'] = request.form['desc']
        p['emoji'] = request.form['emoji']
        return redirect(url_for('detail', pid=pid))
    return render_template_string(DETAIL_HTML, p=p, whatsapp=WHATSAPP_NUMBER)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=10000)

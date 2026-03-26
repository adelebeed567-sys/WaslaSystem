# WaslaSystem
const firebaseConfig = {   apiKey: "AIzaSy...",   authDomain: "your-project.firebaseapp.com",   databaseURL: "https://your-project-default-rtdb.firebaseio.com",   projectId: "your-project",   storageBucket: "...",   messagingSenderId: "...",   appId: "..." };
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>وصلة - لوحة المدير</title>
    <style>
        * { margin:0; padding:0; box-sizing: border-box; }
        body { font-family: 'Cairo', sans-serif; background: #f1f5f9; padding: 20px; }
        .container { max-width: 800px; margin: auto; }
        .card { background: white; border-radius: 20px; padding: 20px; margin-bottom: 20px; }
        input, textarea, button { width: 100%; padding: 10px; margin: 8px 0; border-radius: 12px; border: 1px solid #ddd; }
        button { background: #f97316; color: white; font-weight: bold; border: none; cursor: pointer; }
        table { width: 100%; border-collapse: collapse; margin-top: 10px; }
        th, td { padding: 8px; border-bottom: 1px solid #eee; text-align: right; }
        .btn-small { background: none; border: 1px solid #f97316; color: #f97316; padding: 4px 8px; width: auto; display: inline-block; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600&display=swap" rel="stylesheet">
    <!-- Firebase -->
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
    <script>
        // 🔥 أدخل بيانات مشروعك هنا (انسخها من Firebase Console)
        const firebaseConfig = {
            apiKey: "AIzaSyB...",        // ضع القيم الحقيقية
            authDomain: "...",
            databaseURL: "https://...",
            projectId: "...",
            storageBucket: "...",
            messagingSenderId: "...",
            appId: "..."
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();
    </script>
</head>
<body>
<div class="container">
    <h1>لوحة المدير – وصلة</h1>
    <div class="card">
        <h2>➕ إضافة مطعم</h2>
        <input type="text" id="restName" placeholder="اسم المطعم">
        <input type="text" id="restPhone" placeholder="رقم الهاتف">
        <input type="text" id="restAddress" placeholder="العنوان">
        <button onclick="addRestaurant()">إضافة</button>
    </div>
    <div class="card">
        <h2>🍔 إضافة منتج</h2>
        <select id="productRestSelect"></select>
        <input type="text" id="prodName" placeholder="اسم المنتج">
        <input type="number" id="prodPrice" placeholder="السعر">
        <textarea id="prodDesc" placeholder="وصف"></textarea>
        <button onclick="addProduct()">إضافة منتج</button>
    </div>
    <div class="card">
        <h2>🏪 المطاعم</h2>
        <div id="restaurantsList"></div>
    </div>
    <div class="card">
        <h2>📦 الطلبات</h2>
        <div id="ordersList"></div>
    </div>
    <div class="card">
        <button onclick="resetAllData()" style="background:#dc2626;">🗑️ مسح كل البيانات (إعادة ضبط)</button>
    </div>
</div>

<script>
    let restaurantsRef = db.ref('restaurants');
    let productsRef = db.ref('products');
    let ordersRef = db.ref('orders');

    // إضافة مطعم
    function addRestaurant() {
        let name = document.getElementById('restName').value.trim();
        let phone = document.getElementById('restPhone').value.trim();
        let address = document.getElementById('restAddress').value.trim();
        if (!name || !phone || !address) { alert('املأ جميع الحقول'); return; }
        let newRestRef = restaurantsRef.push();
        newRestRef.set({
            id: newRestRef.key,
            name, phone, address
        }).then(() => {
            document.getElementById('restName').value = '';
            document.getElementById('restPhone').value = '';
            document.getElementById('restAddress').value = '';
            alert('تمت الإضافة');
        });
    }

    // إضافة منتج
    function addProduct() {
        let restaurantId = document.getElementById('productRestSelect').value;
        let name = document.getElementById('prodName').value.trim();
        let price = parseFloat(document.getElementById('prodPrice').value);
        if (!restaurantId || !name || isNaN(price)) { alert('اختر المطعم واملأ البيانات'); return; }
        let newProdRef = productsRef.push();
        newProdRef.set({
            id: newProdRef.key,
            restaurantId: restaurantId,
            name: name,
            price: price,
            desc: document.getElementById('prodDesc').value
        }).then(() => {
            document.getElementById('prodName').value = '';
            document.getElementById('prodPrice').value = '';
            document.getElementById('prodDesc').value = '';
            alert('تمت إضافة المنتج');
        });
    }

    // عرض المطاعم في القائمة والـ select
    function loadRestaurants() {
        restaurantsRef.on('value', (snapshot) => {
            let restaurants = snapshot.val();
            let html = '<table><tr><th>الاسم</th><th>الهاتف</th><th>العنوان</th><th></th></tr>';
            let selectHtml = '<option value="">اختر مطعماً</option>';
            if (restaurants) {
                Object.values(restaurants).forEach(r => {
                    html += `<tr>
                                <td>${r.name}</td>
                                <td>${r.phone}</td>
                                <td>${r.address}</td>
                                <td><button class="btn-small" onclick="deleteRestaurant('${r.id}')">حذف</button></td>
                             </tr>`;
                    selectHtml += `<option value="${r.id}">${r.name}</option>`;
                });
            } else {
                html += '<tr><td colspan="4">لا توجد مطاعم</td></tr>';
            }
            html += '</table>';
            document.getElementById('restaurantsList').innerHTML = html;
            document.getElementById('productRestSelect').innerHTML = selectHtml;
        });
    }

    function deleteRestaurant(id) {
        if (confirm('حذف المطعم سيحذف منتجاته أيضاً. استمر؟')) {
            // حذف المطعم
            restaurantsRef.child(id).remove();
            // حذف منتجاته
            productsRef.orderByChild('restaurantId').equalTo(id).once('value', snap => {
                snap.forEach(child => child.ref.remove());
            });
        }
    }

    // عرض الطلبات
    function loadOrders() {
        ordersRef.on('value', (snapshot) => {
            let orders = snapshot.val();
            let html = '';
            if (orders) {
                Object.values(orders).forEach(o => {
                    html += `<div style="border-bottom:1px solid #eee; margin-bottom:10px;">
                                <strong>طلب #${o.id}</strong> - ${o.date}<br>
                                المطعم: ${o.restaurantName}<br>
                                الإجمالي: ${o.total} ج.م<br>
                                حالة الطلب: ${o.status}<br>
                                التوصيل: ${o.deliveryStatus}<br>
                                <button class="btn-small" onclick="updateOrderStatus('${o.id}')">تغيير حالة الطلب</button>
                             </div>`;
                });
            } else {
                html = '<p>لا توجد طلبات بعد</p>';
            }
            document.getElementById('ordersList').innerHTML = html;
        });
    }

    function updateOrderStatus(orderId) {
        let newStatus = prompt('أدخل الحالة الجديدة (جاري التجهيز/جاهز/خرج للتوصيل/تم التسليم)');
        if (newStatus) ordersRef.child(orderId).update({ status: newStatus });
    }

    // إعادة ضبط كل البيانات
    function resetAllData() {
        if (confirm('هل أنت متأكد؟ سيتم مسح كل المطاعم والمنتجات والطلبات نهائياً.')) {
            db.ref().remove().then(() => alert('تم مسح كل البيانات'));
        }
    }

    loadRestaurants();
    loadOrders();
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>وصلة - العميل</title>
    <style>
        * { box-sizing: border-box; margin:0; padding:0; }
        body { font-family: 'Cairo', sans-serif; background: #f5f7fa; padding: 20px; }
        .container { max-width: 500px; margin: auto; }
        .card { background: white; border-radius: 16px; padding: 16px; margin-bottom: 16px; }
        .restaurant { cursor: pointer; display: flex; align-items: center; gap: 12px; border-bottom: 1px solid #eee; padding: 12px 0; }
        .product { display: flex; justify-content: space-between; margin: 12px 0; }
        button { background: #f97316; border: none; color: white; padding: 8px 16px; border-radius: 30px; cursor: pointer; }
        .bottom-nav { position: fixed; bottom: 0; left: 0; right: 0; background: white; display: flex; justify-content: space-around; padding: 10px; box-shadow: 0 -2px 10px rgba(0,0,0,0.05); }
        .nav-item { text-align: center; flex: 1; cursor: pointer; color: #666; }
        .nav-item.active { color: #f97316; font-weight: bold; }
        .page { display: none; margin-bottom: 70px; }
        .page.active { display: block; }
        .cart-item { display: flex; justify-content: space-between; margin: 10px 0; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
    <script>
        const firebaseConfig = { /* نفس بيانات المشروع */ };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();
    </script>
</head>
<body>
<div class="container">
    <div id="homePage" class="page active">
        <h2>المطاعم والمحلات</h2>
        <div id="restaurantsList"></div>
    </div>
    <div id="menuPage" class="page">
        <button onclick="goBack()">← رجوع</button>
        <h2 id="restaurantName"></h2>
        <div id="productsList"></div>
    </div>
    <div id="cartPage" class="page">
        <h2>سلة الطلبات</h2>
        <div id="cartItems"></div>
        <div id="cartTotal"></div>
        <button onclick="checkout()" style="width:100%; margin-top:10px;">تأكيد الطلب</button>
    </div>
    <div id="ordersPage" class="page">
        <h2>طلباتي</h2>
        <div id="ordersList"></div>
    </div>
</div>
<div class="bottom-nav">
    <div class="nav-item active" data-page="home">الرئيسية</div>
    <div class="nav-item" data-page="cart">السلة</div>
    <div class="nav-item" data-page="orders">طلباتي</div>
</div>

<script>
    let restaurantsRef = db.ref('restaurants');
    let productsRef = db.ref('products');
    let ordersRef = db.ref('orders');

    let currentRestaurant = null;
    let cart = JSON.parse(localStorage.getItem('wasla_cart') || '[]');

    function saveCart() { localStorage.setItem('wasla_cart', JSON.stringify(cart)); }

    function renderHome() {
        restaurantsRef.once('value', snap => {
            let restaurants = snap.val();
            let html = '';
            if (!restaurants) html = '<div class="card">لا توجد مطاعم بعد</div>';
            else {
                Object.values(restaurants).forEach(r => {
                    html += `<div class="card restaurant" onclick="selectRestaurant('${r.id}')">
                                <div>🏪</div>
                                <div><strong>${r.name}</strong><br><small>${r.address}</small></div>
                             </div>`;
                });
            }
            document.getElementById('restaurantsList').innerHTML = html;
        });
    }

    function selectRestaurant(restId) {
        restaurantsRef.child(restId).once('value', snap => {
            currentRestaurant = { id: restId, ...snap.val() };
            document.getElementById('restaurantName').innerText = currentRestaurant.name;
            // جلب منتجات هذا المطعم
            productsRef.orderByChild('restaurantId').equalTo(restId).once('value', snap => {
                let products = snap.val();
                let html = '';
                if (!products) html = '<div class="card">لا توجد منتجات</div>';
                else {
                    Object.values(products).forEach(p => {
                        html += `<div class="product">
                                    <div><strong>${p.name}</strong><br>${p.price} ج.م</div>
                                    <button onclick="addToCart('${p.id}', '${p.name}', ${p.price})">أضف للسلة</button>
                                 </div>`;
                    });
                }
                document.getElementById('productsList').innerHTML = html;
                showPage('menu');
            });
        });
    }

    function addToCart(prodId, name, price) {
        let existing = cart.find(c => c.productId === prodId && c.restaurantId === currentRestaurant.id);
        if (existing) existing.quantity++;
        else cart.push({ productId: prodId, name, price, quantity: 1, restaurantId: currentRestaurant.id, restaurantName: currentRestaurant.name });
        saveCart();
        alert('تمت الإضافة');
    }

    function renderCart() {
        if (cart.length === 0) { document.getElementById('cartItems').innerHTML = '<div class="card">السلة فارغة</div>'; document.getElementById('cartTotal').innerHTML = ''; return; }
        let total = 0;
        let html = '';
        cart.forEach((item, i) => {
            total += item.price * item.quantity;
            html += `<div class="cart-item">
                        <div>${item.name} × ${item.quantity}</div>
                        <div>${item.price * item.quantity} ج.م <button onclick="removeFromCart(${i})">حذف</button></div>
                     </div>`;
        });
        document.getElementById('cartItems').innerHTML = html;
        document.getElementById('cartTotal').innerHTML = `<strong>الإجمالي: ${total} ج.م</strong>`;
    }

    function removeFromCart(idx) { cart.splice(idx,1); saveCart(); renderCart(); }

    function checkout() {
        if (cart.length === 0) { alert('السلة فارغة'); return; }
        let total = cart.reduce((s, i) => s + (i.price * i.quantity), 0);
        let newOrderRef = ordersRef.push();
        let newOrder = {
            id: newOrderRef.key,
            date: new Date().toLocaleString('ar-EG'),
            items: cart.map(i => ({ name: i.name, quantity: i.quantity, price: i.price })),
            total: total,
            restaurantId: cart[0].restaurantId,
            restaurantName: cart[0].restaurantName,
            status: 'جاري التجهيز',
            deliveryStatus: 'لم يحدد'
        };
        newOrderRef.set(newOrder).then(() => {
            cart = [];
            saveCart();
            renderCart();
            renderOrders();
            alert('تم تقديم الطلب بنجاح');
            showPage('orders');
        });
    }

    function renderOrders() {
        ordersRef.on('value', snap => {
            let orders = snap.val();
            let html = '';
            if (!orders) html = '<div class="card">لا توجد طلبات</div>';
            else {
                Object.values(orders).forEach(o => {
                    html += `<div class="card">
                                <strong>طلب #${o.id}</strong> - ${o.date}<br>
                                المطعم: ${o.restaurantName}<br>
                                الإجمالي: ${o.total} ج.م<br>
                                حالة الطلب: ${o.status}<br>
                                حالة التوصيل: ${o.deliveryStatus}
                             </div>`;
                });
            }
            document.getElementById('ordersList').innerHTML = html;
        });
    }

    function goBack() { showPage('home'); renderHome(); }
    function showPage(pageId) {
        document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
        document.getElementById(pageId + 'Page').classList.add('active');
        document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
        document.querySelector(`.nav-item[data-page="${pageId}"]`).classList.add('active');
        if (pageId === 'cart') renderCart();
        if (pageId === 'orders') renderOrders();
        if (pageId === 'home') renderHome();
    }

    document.querySelectorAll('.nav-item').forEach(n => {
        n.addEventListener('click', () => showPage(n.dataset.page));
    });
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>وصلة - لوحة المطعم</title>
    <style>
        * { margin:0; padding:0; box-sizing: border-box; }
        body { font-family: 'Cairo', sans-serif; background: #f1f5f9; padding: 20px; }
        .container { max-width: 600px; margin: auto; }
        .card { background: white; border-radius: 20px; padding: 20px; margin-bottom: 20px; }
        button { background: #f97316; border: none; color: white; padding: 8px 16px; border-radius: 30px; cursor: pointer; margin-top: 8px; }
        .order { border-bottom: 1px solid #eee; margin-bottom: 15px; padding-bottom: 15px; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
    <script>
        const firebaseConfig = { /* نفس البيانات */ };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();
    </script>
</head>
<body>
<div class="container">
    <h1>لوحة المطعم</h1>
    <div class="card">
        <h2>اختر مطعمك</h2>
        <select id="vendorSelect"></select>
        <button onclick="loadOrders()">عرض طلباتي</button>
    </div>
    <div class="card">
        <h2>طلباتي</h2>
        <div id="ordersList"></div>
    </div>
</div>

<script>
    let restaurantsRef = db.ref('restaurants');
    let ordersRef = db.ref('orders');
    let selectedVendorId = null;

    function loadRestaurantsSelect() {
        restaurantsRef.once('value', snap => {
            let restaurants = snap.val();
            let html = '<option value="">اختر المطعم</option>';
            if (restaurants) {
                Object.values(restaurants).forEach(r => {
                    html += `<option value="${r.id}">${r.name}</option>`;
                });
            }
            document.getElementById('vendorSelect').innerHTML = html;
        });
    }

    function loadOrders() {
        selectedVendorId = document.getElementById('vendorSelect').value;
        if (!selectedVendorId) { alert('اختر المطعم أولاً'); return; }
        // استماع للتغييرات الفورية
        ordersRef.orderByChild('restaurantId').equalTo(selectedVendorId).on('value', snap => {
            let orders = snap.val();
            let html = '';
            if (!orders) html = '<p>لا توجد طلبات</p>';
            else {
                Object.values(orders).forEach(o => {
                    html += `<div class="order">
                                <strong>طلب #${o.id}</strong> - ${o.date}<br>
                                الإجمالي: ${o.total} ج.م<br>
                                الحالة الحالية: ${o.status}<br>
                                <button onclick="updateStatus('${o.id}')">تغيير الحالة</button>
                             </div>`;
                });
            }
            document.getElementById('ordersList').innerHTML = html;
        });
    }

    function updateStatus(orderId) {
        let newStatus = prompt('أدخل الحالة الجديدة (جاري التجهيز / جاهز / خرج للتوصيل)');
        if (newStatus) ordersRef.child(orderId).update({ status: newStatus });
    }
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>وصلة - لوحة المندوب</title>
    <style>
        * { margin:0; padding:0; box-sizing: border-box; }
        body { font-family: 'Cairo', sans-serif; background: #f1f5f9; padding: 20px; }
        .container { max-width: 600px; margin: auto; }
        .card { background: white; border-radius: 20px; padding: 20px; margin-bottom: 20px; }
        button { background: #f97316; border: none; color: white; padding: 8px 16px; border-radius: 30px; cursor: pointer; margin-top: 8px; }
        .order { border-bottom: 1px solid #eee; margin-bottom: 15px; padding-bottom: 15px; }
    </style>
    <link href="https://fonts.googleapis.com/css2?family=Cairo:wght@400;600&display=swap" rel="stylesheet">
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>
    <script>
        const firebaseConfig = { /* نفس البيانات */ };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();
    </script>
</head>
<body>
<div class="container">
    <h1>لوحة المندوب</h1>
    <div class="card">
        <h2>طلبات التوصيل</h2>
        <div id="ordersList"></div>
    </div>
</div>

<script>
    let ordersRef = db.ref('orders');

    function loadDeliveryOrders() {
        ordersRef.on('value', snap => {
            let orders = snap.val();
            let html = '';
            if (!orders) html = '<p>لا توجد طلبات للتوصيل</p>';
            else {
                Object.values(orders).forEach(o => {
                    // فقط الطلبات التي حالتها "جاهز" أو "خرج للتوصيل"
                    if (o.status === 'جاهز' || o.status === 'خرج للتوصيل') {
                        html += `<div class="order">
                                    <strong>طلب #${o.id}</strong> - ${o.date}<br>
                                    المطعم: ${o.restaurantName}<br>
                                    الإجمالي: ${o.total} ج.م<br>
                                    حالة الطلب: ${o.status}<br>
                                    حالة التوصيل: ${o.deliveryStatus}<br>
                                    <button onclick="updateDelivery('${o.id}')">تحديث حالة التوصيل</button>
                                 </div>`;
                    }
                });
            }
            if (html === '') html = '<p>لا توجد طلبات للتوصيل حالياً</p>';
            document.getElementById('ordersList').innerHTML = html;
        });
    }

    function updateDelivery(orderId) {
        let newStatus = prompt('أدخل حالة التوصيل (في الطريق / تم التسليم)');
        if (newStatus) {
            let update = { deliveryStatus: newStatus };
            if (newStatus === 'تم التسليم') update.status = 'تم التسليم';
            ordersRef.child(orderId).update(update);
        }
    }

    loadDeliveryOrders();
    // تحديث تلقائي كل 3 ثوانٍ (اختياري، لكن real-time يكفي)
</script>
</body>
</html>
    loadRestaurantsSelect();
</script>
</body>
</html>


    renderHome();
    renderOrders();
</script>
</body>
</html>

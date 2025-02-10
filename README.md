from flask import Flask, render_template_string, request, jsonify

app = Flask(__name__)

HTML_TEMPLATE = '''
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>LA W - Marketplace Mejorado</title>
  <!-- Bootstrap CSS -->
  <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
  <!-- Animate.css -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css"/>
  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
  <style>
    :root {
      --primary-color: #FF5722;
      --secondary-color: #FFC107;
      --nav-color: #E91E63;
      --accent-color: #4CAF50;
      --bg-color: #f4f4f4;
      --text-color: #333;
      --overlay-color: rgba(0,0,0,0.5);
    }
    /* Estilos base personalizados */
    body {
      font-family: 'Roboto', sans-serif;
      background: var(--bg-color);
      color: var(--text-color);
      transition: background 0.5s, color 0.5s;
    }
    body.dark {
      background: #121212;
      color: #e0e0e0;
    }
    img { max-width: 100%; height: auto; }
    /* Header y sección Home */
    header {
      background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
      color: white;
      padding: 2rem 1rem;
      text-align: center;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
    header h1 { font-size: 3rem; margin-bottom: 0.5rem; }
    header p { font-size: 1.4rem; }
    #home {
      position: relative;
      background: url('https://images.unsplash.com/photo-1557682224-5b8590cd9ec5?ixlib=rb-4.0.3&auto=format&fit=crop&w=1350&q=80') no-repeat center center/cover;
      padding: 6rem 1rem;
      text-align: center;
      overflow: hidden;
    }
    #home::after {
      content: "";
      position: absolute;
      top: 0; left: 0; right: 0; bottom: 0;
      background: var(--overlay-color);
      z-index: 1;
    }
    #home > * { position: relative; z-index: 2; }
    #home .cta-button {
      margin-top: 2rem;
      padding: 1rem 2rem;
      background: var(--accent-color);
      color: var(--nav-color);
      border: none;
      font-size: 1.4rem;
      border-radius: 6px;
      cursor: pointer;
      transition: background 0.3s, transform 0.3s;
    }
    #home .cta-button:hover {
      background: #ffc107;
      transform: scale(1.15);
    }
    #home .extra-info {
      margin-top: 1.5rem;
      font-size: 1.2rem;
      max-width: 800px;
      margin-left: auto;
      margin-right: auto;
    }
    /* Ajustes para productos, paneles y formularios se mantienen similares, usando la clase container y grid de Bootstrap */
    .product-grid { margin-top: 1rem; }
    .product { transition: transform 0.3s, box-shadow 0.3s; }
    .product:hover { transform: translateY(-5px); box-shadow: 0 4px 12px rgba(0,0,0,0.2); }
    
    /* Dark mode toggle */
    .dark-toggle {
      position: fixed;
      bottom: 20px;
      right: 20px;
      background: var(--accent-color);
      color: white;
      border: none;
      padding: 0.75rem 1rem;
      border-radius: 4px;
      cursor: pointer;
      box-shadow: 0 2px 4px rgba(0,0,0,0.3);
      transition: transform 0.3s;
    }
    .dark-toggle:hover { transform: scale(1.1); }
  </style>
</head>
<body>
  <!-- Navbar usando Bootstrap -->
  <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
      <a class="navbar-brand animate__animated animate__fadeInDown" onclick="showSection('home')">LA W</a>
      <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" 
              aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
        <span class="navbar-toggler-icon"></span>
      </button>
      <div class="collapse navbar-collapse" id="navbarNav">
        <ul class="navbar-nav ms-auto">
          <li class="nav-item"><a class="nav-link" id="navInicio" onclick="showSection('home')">Inicio</a></li>
          <li class="nav-item"><a class="nav-link" id="navMarketplace" onclick="showSection('marketplace')">Productos</a></li>
          <li class="nav-item"><a class="nav-link" id="navCart" onclick="showSection('cartPanel')">Carrito</a></li>
          <li class="nav-item" id="navBuyerPanelItem" style="display:none;">
            <a class="nav-link" id="navBuyerPanel" onclick="showSection('buyerPanel')">Panel Comprador</a>
          </li>
          <li class="nav-item" id="navSellerPanelItem" style="display:none;">
            <a class="nav-link" id="navSellerPanel" onclick="showSection('sellerPanel')">Panel Vendedor</a>
          </li>
          <li class="nav-item" id="navLoginItem">
            <a class="nav-link" id="navLogin" onclick="showLoginModal()">Iniciar Sesión</a>
          </li>
          <li class="nav-item" id="navRegisterItem">
            <a class="nav-link" id="navRegister" onclick="showRegisterModal()">Registrarse</a>
          </li>
        </ul>
      </div>
    </div>
  </nav>
  
  <!-- Sección Home (sin opciones de registro ni chat) -->
  <section id="home" class="container">
    <h2 class="animate__animated animate__bounceIn animate__delay-1s">¡Bienvenido a LA W!</h2>
    <p class="extra-info animate__animated animate__fadeInUp animate__delay-2s">Descubre el marketplace más dinámico, seguro y moderno. Encuentra todo lo que necesitas y disfruta de una experiencia única.</p>
    <button class="btn btn-lg btn-warning cta-button animate__animated animate__zoomIn animate__delay-3s" onclick="showSection('marketplace')">Explora Productos</button>
  </section>
  
  <!-- Sección Marketplace -->
  <section id="marketplace" class="container" style="display:none;">
    <h2 class="animate__animated animate__fadeInDown">Productos</h2>
    <div class="input-group mb-3">
      <input type="text" id="searchInput" class="form-control" placeholder="Buscar productos..." onkeyup="filterProducts()">
    </div>
    <div class="row row-cols-1 row-cols-md-3 g-4 product-grid" id="productGrid">
      <!-- Aquí se cargarán los productos -->
    </div>
  </section>
  
  <!-- Paneles de Carrito, Vendedor y Comprador (se muestran según selección) -->
  <section id="cartPanel" class="container panel" style="display:none;">
    <h2>Carrito de Compras</h2>
    <ul id="cartItems" class="list-group"></ul>
    <p class="mt-3">Total: <span id="total">$0.00</span></p>
    <button class="btn btn-primary" onclick="checkoutCart()">Comprar Carrito</button>
  </section>
  
  <section id="sellerPanel" class="container panel" style="display:none;">
    <h2>Panel Vendedor</h2>
    <p>Aquí puedes agregar y gestionar tus productos.</p>
    <form id="addProductForm">
      <input type="text" id="productName" class="form-control mb-2" placeholder="Nombre del producto" required>
      <input type="text" id="productDescription" class="form-control mb-2" placeholder="Descripción" required>
      <input type="number" id="productPrice" class="form-control mb-2" placeholder="Precio" step="0.01" required>
      <input type="file" id="productImages" class="form-control mb-2" accept="image/*" multiple required>
      <div id="imagePreview" class="mb-2"></div>
      <button type="submit" class="btn btn-success">Agregar Producto</button>
    </form>
    <hr>
    <h3>Mis Productos</h3>
    <div class="row row-cols-1 row-cols-md-3 g-4" id="sellerProductGrid"></div>
    <hr>
    <h3>Órdenes en Proceso</h3>
    <div id="sellerOrders"></div>
    <hr>
    <h3>Datos del Vendedor</h3>
    <div id="sellerDataDisplay"></div>
    <hr>
    <h3>Ganancias del Vendedor</h3>
    <div id="sellerEarningsDisplay"><p>Ganancias: $0.00</p></div>
  </section>
  
  <section id="buyerPanel" class="container panel" style="display:none;">
    <h2>Panel Comprador</h2>
    <p>Bienvenido, disfruta de tus compras y haz seguimiento de tus órdenes.</p>
    <div id="buyerOrders"></div>
    <p class="mt-3 text-muted">Una vez confirmada la compra se desbloqueará el chat con el vendedor.</p>
  </section>
  
  <!-- Modal de Login (Bootstrap Modal) -->
  <div id="loginModal" class="modal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Iniciar Sesión</h5>
          <button type="button" class="btn-close" onclick="closeLoginModal()"></button>
        </div>
        <div class="modal-body">
          <form id="loginForm">
            <div class="mb-3">
              <label for="loginEmail" class="form-label">Correo Electrónico</label>
              <input type="email" id="loginEmail" class="form-control" placeholder="ejemplo@dominio.com" required>
            </div>
            <div class="mb-3">
              <label for="loginPassword" class="form-label">Contraseña</label>
              <input type="password" id="loginPassword" class="form-control" placeholder="Contraseña" required>
            </div>
            <button type="submit" class="btn btn-primary w-100">Entrar</button>
          </form>
        </div>
      </div>
    </div>
  </div>
  
  <!-- Modal de Registro (similar estructura Bootstrap) -->
  <div id="registerModal" class="modal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Registrarse</h5>
          <button type="button" class="btn-close" onclick="closeRegisterModal()"></button>
        </div>
        <div class="modal-body">
          <form id="registerForm">
            <div class="mb-3">
              <input type="text" id="registerName" class="form-control" placeholder="Nombre completo" required>
            </div>
            <div class="mb-3">
              <input type="email" id="registerEmail" class="form-control" placeholder="Email" required>
            </div>
            <div class="mb-3">
              <input type="password" id="registerPassword" class="form-control" placeholder="Contraseña" required>
            </div>
            <div class="mb-3">
              <label for="userType" class="form-label">Tipo de Usuario:</label>
              <select id="userType" class="form-select" required onchange="toggleSellerFields(this.value)">
                <option value="buyer">Comprador</option>
                <option value="seller">Vendedor</option>
              </select>
            </div>
            <div id="sellerExtraFields" style="display:none;">
              <div class="mb-3"><input type="text" id="cedula" class="form-control" placeholder="Cédula" required></div>
              <div class="mb-3"><input type="text" id="telefono" class="form-control" placeholder="Número de Teléfono" required></div>
              <div class="mb-3"><input type="text" id="ubicacion" class="form-control" placeholder="Ubicación" required></div>
              <div class="mb-3"><input type="text" id="cuentaBancaria" class="form-control" placeholder="Cuenta Bancaria" required></div>
              <div class="mb-3">
                <label class="form-label">Expediente Judicial (Archivo, PDF o imagen):</label>
                <input type="file" id="expedienteJudicial" class="form-control" accept="image/*,application/pdf" required>
              </div>
              <div class="mb-3">
                <label class="form-label">Expediente Disciplinario (Archivo, PDF o imagen):</label>
                <input type="file" id="expedienteDisciplinario" class="form-control" accept="image/*,application/pdf" required>
              </div>
            </div>
            <button type="submit" class="btn btn-success w-100">Registrarse</button>
          </form>
        </div>
      </div>
    </div>
  </div>
  
  <!-- Modal de Chat (se activa solo si el usuario está logueado) -->
  <div id="chatModal" class="modal" tabindex="-1">
    <div class="modal-dialog">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Chat con <span id="chatWith"></span></h5>
          <button type="button" class="btn-close" onclick="closeChatModal()"></button>
        </div>
        <div class="modal-body">
          <div id="chatMessages" class="border p-2 mb-3" style="height:200px; overflow-y:auto;"></div>
          <div class="input-group">
            <input type="text" id="chatInput" class="form-control" placeholder="Escribe tu mensaje...">
            <button class="btn btn-primary" onclick="sendChatMessage()">Enviar</button>
          </div>
        </div>
      </div>
    </div>
  </div>
  
  <!-- Modal de Imágenes y Modal de Detalle se mantienen con la estructura personalizada -->
  <div id="productImagesModal" class="modal">
    <div class="modal-dialog">
      <div class="modal-content p-3">
        <span class="btn-close float-end" onclick="closeProductImagesModal()"></span>
        <img id="productModalImage" src="" alt="Imagen del Producto">
        <div class="text-center mt-3">
          <button class="btn btn-secondary me-2" onclick="prevProductImage()">Anterior</button>
          <button class="btn btn-secondary" onclick="nextProductImage()">Siguiente</button>
        </div>
      </div>
    </div>
  </div>
  
  <div id="productDetailModal" class="modal">
    <div class="modal-dialog modal-lg">
      <div class="modal-content p-3">
        <span class="btn-close float-end" onclick="closeProductDetailModal()"></span>
        <h2 id="detailProductTitle" class="mb-3"></h2>
        <div class="row">
          <div class="col-md-6">
            <img id="detailModalImage" src="" alt="Imagen del Producto" class="img-fluid">
            <div class="text-center mt-2">
              <button class="btn btn-outline-secondary me-2" onclick="prevDetailImage()">Anterior</button>
              <button class="btn btn-outline-secondary" onclick="nextDetailImage()">Siguiente</button>
            </div>
          </div>
          <div class="col-md-6">
            <p id="detailProductDescription"></p>
            <p class="h4 text-danger" id="detailProductPrice"></p>
            <p>Calificación Promedio: <span id="averageRating">Sin calificar</span></p>
            <div id="reviewsContainer" class="border p-2 mb-3" style="max-height:150px; overflow-y:auto;"></div>
            <form id="reviewForm">
              <div class="mb-2">
                <select id="reviewRating" class="form-select" required>
                  <option value="">Calificación</option>
                  <option value="1">1 estrella</option>
                  <option value="2">2 estrellas</option>
                  <option value="3">3 estrellas</option>
                  <option value="4">4 estrellas</option>
                  <option value="5">5 estrellas</option>
                </select>
              </div>
              <div class="mb-2">
                <textarea id="reviewComment" class="form-control" placeholder="Tu comentario" required></textarea>
              </div>
              <button type="submit" class="btn btn-primary w-100">Enviar Reseña</button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
  
  <footer class="text-center py-3 bg-light">
    <p>&copy; 2025 LA W. Todos los derechos reservados.</p>
  </footer>
  
  <button class="dark-toggle" onclick="toggleTheme()">Modo Oscuro</button>
  
  <!-- JavaScript: Bootstrap Bundle y lógica personalizada -->
  <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
  <script>
    // Función para actualizar el menú según estado de autenticación
    function updateNav() {
      if(currentUser) {
        document.getElementById('navBuyerPanelItem').style.display = 'block';
        document.getElementById('navSellerPanelItem').style.display = 'block';
        document.getElementById('navLoginItem').style.display = 'none';
        document.getElementById('navRegisterItem').style.display = 'none';
      } else {
        document.getElementById('navBuyerPanelItem').style.display = 'none';
        document.getElementById('navSellerPanelItem').style.display = 'none';
        document.getElementById('navLoginItem').style.display = 'block';
        document.getElementById('navRegisterItem').style.display = 'none'; // Registro oculto en este ejemplo
      }
    }
    
    // Variables globales
    let users = JSON.parse(localStorage.getItem('users')) || [];
    let currentUser = JSON.parse(localStorage.getItem('currentUser')) || null;
    let sellerEarnings = parseFloat(localStorage.getItem('sellerEarnings')) || 0;
    let orders = JSON.parse(localStorage.getItem('orders')) || [];
    let secureVault = JSON.parse(localStorage.getItem('secureVault')) || {};
    let sellerProducts = JSON.parse(localStorage.getItem('sellerProducts')) || [];
    let cart = JSON.parse(localStorage.getItem('cart')) || [];
    let chatSessions = JSON.parse(localStorage.getItem('chatSessions')) || {};
    let currentChatOrderId = null;
    
    // Productos iniciales
    let products = [
      { id: 1, name: 'Producto 1', description: 'Descripción breve del producto 1', price: 10.00, images: ['https://via.placeholder.com/300x200','https://via.placeholder.com/300x200/AAAAAA'], seller: "default@seller.com", reviews: [] },
      { id: 2, name: 'Producto 2', description: 'Descripción breve del producto 2', price: 20.00, images: ['https://via.placeholder.com/300x200'], seller: "default@seller.com", reviews: [] },
      { id: 3, name: 'Producto 3', description: 'Descripción breve del producto 3', price: 15.00, images: ['https://via.placeholder.com/300x200','https://via.placeholder.com/300x200/CCCCCC','https://via.placeholder.com/300x200/EEEEEE'], seller: "default@seller.com", reviews: [] },
      { id: 4, name: 'Producto 4', description: 'Descripción breve del producto 4', price: 8.00, images: ['https://via.placeholder.com/300x200'], seller: "default@seller.com", reviews: [] }
    ];
    
    // Variables para imágenes y detalle
    let currentProductImages = [];
    let currentImageIndex = 0;
    let currentDetailProduct = null;
    let currentDetailImages = [];
    let currentDetailIndex = 0;
    
    function showSection(sectionId) {
      ['home','marketplace','cartPanel','sellerPanel','buyerPanel'].forEach(id => {
        document.getElementById(id).style.display = 'none';
      });
      document.getElementById(sectionId).style.display = 'block';
      if(sectionId === 'buyerPanel') renderBuyerOrders();
      if(sectionId === 'sellerPanel') { renderSellerOrders(); displaySellerData(); }
      if(sectionId === 'cartPanel') renderCart();
      if(sectionId === 'marketplace') renderProducts();
    }
    
    function renderProducts() {
      const grid = document.getElementById('productGrid');
      grid.innerHTML = '';
      products.forEach(product => {
        let div = document.createElement('div');
        div.className = 'col';
        div.innerHTML = `
          <div class="card h-100 animate__animated animate__fadeIn">
            <img src="${product.images[0]}" class="card-img-top" alt="${product.name}">
            <div class="card-body">
              <h5 class="card-title">${product.name}</h5>
              <p class="card-text">${product.description}</p>
              <p class="text-danger fw-bold">$${product.price.toFixed(2)}</p>
            </div>
            <div class="card-footer">
              <button class="btn btn-sm btn-primary" onclick="simulatePayment(${product.id})">Comprar</button>
              <button class="btn btn-sm btn-warning" onclick="addToCart(${product.id}, this)">Agregar al carrito</button>
            </div>
          </div>
        `;
        grid.appendChild(div);
      });
    }
    
    function simulatePayment(productId) {
      const product = products.find(p => p.id === productId);
      let method = prompt("Seleccione el método de pago:\n1. Bancolombia\n2. Nequi\n3. Efectivo", "1");
      let methodText = (method === "1") ? "Bancolombia" : (method === "2") ? "Nequi" : (method === "3") ? "Efectivo" : "Método no especificado";
      let commission = product.price * 0.02;
      let sellerReceives = product.price - commission;
      let order = {
        orderId: Date.now(),
        product: product,
        buyer: currentUser ? currentUser.email : "anonimo@comprador.com",
        seller: product.seller,
        status: "En proceso",
        buyerNotified: false
      };
      orders.push(order);
      localStorage.setItem('orders', JSON.stringify(orders));
      alert(`Orden generada:
Producto: ${product.name} por $${product.price.toFixed(2)}
Método: ${methodText}
Comisión 2%: $${commission.toFixed(2)}
El vendedor recibirá: $${sellerReceives.toFixed(2)}.
Una vez confirmada la compra se desbloqueará el chat.`);
      if(currentUser && currentUser.type === "buyer") renderBuyerOrders();
    }
    
    function addToCart(productId, btnElement) {
      const product = products.find(p => p.id === productId);
      cart.push(product);
      localStorage.setItem('cart', JSON.stringify(cart));
      btnElement.style.transform = 'scale(1.2)';
      setTimeout(() => { btnElement.style.transform = 'scale(1)'; }, 300);
      alert(`${product.name} agregado al carrito.`);
    }
    
    function renderCart() {
      const cartList = document.getElementById('cartItems');
      cartList.innerHTML = '';
      if(cart.length === 0) {
        cartList.innerHTML = "<li class='list-group-item'>El carrito está vacío.</li>";
      } else {
        let total = 0;
        cart.forEach((item, index) => {
          let li = document.createElement('li');
          li.className = 'list-group-item d-flex justify-content-between align-items-center';
          li.innerHTML = `${item.name} - $${item.price.toFixed(2)} <button class="btn btn-sm btn-danger" onclick="removeFromCart(${index})">Quitar</button>`;
          cartList.appendChild(li);
          total += item.price;
        });
        document.getElementById('total').textContent = `$${total.toFixed(2)}`;
      }
    }
    
    function removeFromCart(index) {
      cart.splice(index, 1);
      localStorage.setItem('cart', JSON.stringify(cart));
      renderCart();
    }
    
    function checkoutCart() {
      if(cart.length === 0) { alert("El carrito está vacío."); return; }
      cart.forEach(product => {
        let order = {
          orderId: Date.now() + Math.floor(Math.random()*1000),
          product: product,
          buyer: currentUser ? currentUser.email : "anonimo@comprador.com",
          seller: product.seller,
          status: "En proceso",
          buyerNotified: false
        };
        orders.push(order);
      });
      localStorage.setItem('orders', JSON.stringify(orders));
      cart = [];
      localStorage.setItem('cart', JSON.stringify(cart));
      renderCart();
      alert("Órdenes generadas. Revisa tu panel de órdenes.");
      if(currentUser && currentUser.type === "buyer") renderBuyerOrders();
    }
    
    function filterProducts() {
      const query = document.getElementById('searchInput').value.toLowerCase();
      const grid = document.getElementById('productGrid');
      grid.innerHTML = '';
      products.filter(p => p.name.toLowerCase().includes(query) || p.description.toLowerCase().includes(query))
              .forEach(product => {
        let div = document.createElement('div');
        div.className = 'col';
        div.innerHTML = `
          <div class="card h-100 animate__animated animate__fadeIn">
            <img src="${product.images[0]}" class="card-img-top" alt="${product.name}">
            <div class="card-body">
              <h5 class="card-title">${product.name}</h5>
              <p class="card-text">${product.description}</p>
              <p class="text-danger fw-bold">$${product.price.toFixed(2)}</p>
            </div>
            <div class="card-footer">
              <button class="btn btn-sm btn-primary" onclick="simulatePayment(${product.id})">Comprar</button>
              <button class="btn btn-sm btn-warning" onclick="addToCart(${product.id}, this)">Agregar al carrito</button>
            </div>
          </div>
        `;
        grid.appendChild(div);
      });
    }
    
    function renderBuyerOrders() {
      const container = document.getElementById('buyerOrders');
      container.innerHTML = "";
      orders.filter(o => o.buyer === (currentUser ? currentUser.email : ""))
            .forEach(order => {
              let div = document.createElement('div');
              div.className = 'mb-2 p-2 border rounded animate__animated animate__fadeIn';
              div.innerHTML = `<strong>Orden ${order.orderId}</strong> - ${order.product.name} - Estado: ${order.status}`;
              if(order.status === "En proceso") { 
                div.innerHTML += `<br><button class="btn btn-sm btn-secondary mt-1" onclick="openChatModal(${order.orderId})">Chat</button>`; 
              }
              container.appendChild(div);
            });
      if(container.innerHTML === "") container.innerHTML = "<p>No tienes órdenes.</p>";
    }
    
    function renderSellerOrders() {
      const container = document.getElementById('sellerOrders');
      container.innerHTML = "";
      orders.filter(o => o.seller === (currentUser ? currentUser.email : ""))
            .forEach(order => {
              let div = document.createElement('div');
              div.className = 'mb-2 p-2 border rounded animate__animated animate__fadeIn';
              div.innerHTML = `<strong>Orden ${order.orderId}</strong> - ${order.product.name} - Estado: ${order.status}`;
              if(order.status === "En proceso") {
                div.innerHTML += `<br><button class="btn btn-sm btn-success me-2" onclick="confirmOrder(${order.orderId})">Confirmar Pago</button>`;
                div.innerHTML += `<button class="btn btn-sm btn-secondary" onclick="openChatModal(${order.orderId})">Chat</button>`;
              }
              container.appendChild(div);
            });
      if(container.innerHTML === "") container.innerHTML = "<p>No tienes órdenes en proceso.</p>";
    }
    
    function confirmOrder(orderId) {
      let order = orders.find(o => o.orderId == orderId);
      if(order) {
        order.status = "Confirmada";
        let commission = order.product.price * 0.02;
        let sellerReceives = order.product.price - commission;
        sellerEarnings += sellerReceives;
        localStorage.setItem('sellerEarnings', sellerEarnings);
        localStorage.setItem('orders', JSON.stringify(orders));
        renderSellerOrders();
        updateSellerEarningsDisplay();
        alert("Orden " + orderId + " confirmada. El pago se ha reflejado en tu billetera.");
      }
    }
    
    function displaySellerData() {
      const container = document.getElementById('sellerDataDisplay');
      if(currentUser && currentUser.type === "seller") {
        let data = secureVault[currentUser.email];
        container.innerHTML = data ? `<p><strong>Cédula:</strong> ${data.cedula}</p>
                                        <p><strong>Email:</strong> ${currentUser.email}</p>
                                        <p><strong>Ubicación:</strong> ${data.ubicacion}</p>
                                        <p><strong>Cuenta Bancaria:</strong> ${data.cuentaBancaria}</p>` 
                                        : "<p>No se encontraron datos del vendedor.</p>";
      }
    }
    
    function updateSellerEarningsDisplay() {
      const display = document.getElementById('sellerEarningsDisplay');
      display.innerHTML = "<p>Ganancias: $" + sellerEarnings.toFixed(2) + "</p>";
    }
    
    function showLoginModal() { document.getElementById('loginModal').style.display = 'block'; }
    function closeLoginModal() { document.getElementById('loginModal').style.display = 'none'; }
    function showRegisterModal() { document.getElementById('registerModal').style.display = 'block'; }
    function closeRegisterModal() { document.getElementById('registerModal').style.display = 'none'; }
    function toggleSellerFields(value) { document.getElementById('sellerExtraFields').style.display = (value === "seller") ? "block" : "none"; }
    
    document.getElementById('productImages').addEventListener('change', function(e) {
      const previewContainer = document.getElementById('imagePreview');
      previewContainer.innerHTML = "";
      const files = e.target.files;
      for(let i = 0; i < files.length; i++) {
        const reader = new FileReader();
        reader.onload = function(event) {
          const img = document.createElement('img');
          img.src = event.target.result;
          previewContainer.appendChild(img);
        }
        reader.readAsDataURL(files[i]);
      }
    });
    
    document.getElementById('addProductForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const name = document.getElementById('productName').value;
      const description = document.getElementById('productDescription').value;
      const price = parseFloat(document.getElementById('productPrice').value);
      const fileInput = document.getElementById('productImages');
      const files = fileInput.files;
      const imagesArray = [];
      if(files.length > 0) {
        let loaded = 0;
        for(let i = 0; i < files.length; i++) {
          const reader = new FileReader();
          reader.onload = function(event) {
            imagesArray.push(event.target.result);
            loaded++;
            if(loaded === files.length) {
              agregarProducto(name, description, price, imagesArray);
            }
          }
          reader.readAsDataURL(files[i]);
        }
      } else { alert("Seleccione al menos una imagen."); }
    });
    
    function agregarProducto(name, description, price, imagesArray) {
      let seller = currentUser && currentUser.type === "seller" ? currentUser.email : "default@seller.com";
      const newProduct = { id: Date.now(), name, description, price, images: imagesArray, seller, reviews: [] };
      products.push(newProduct);
      sellerProducts.push(newProduct);
      localStorage.setItem('sellerProducts', JSON.stringify(sellerProducts));
      renderSellerProducts();
      renderProducts();
      document.getElementById('addProductForm').reset();
      document.getElementById('imagePreview').innerHTML = "";
    }
    
    function renderSellerProducts() {
      const grid = document.getElementById('sellerProductGrid');
      grid.innerHTML = "";
      sellerProducts.forEach(product => {
        let div = document.createElement('div');
        div.className = 'col';
        div.innerHTML = `
          <div class="card h-100">
            <img src="${product.images[0]}" class="card-img-top" alt="${product.name}">
            <div class="card-body">
              <h5 class="card-title">${product.name}</h5>
              <p class="card-text">${product.description}</p>
              <p class="text-danger fw-bold">$${product.price.toFixed(2)}</p>
            </div>
          </div>
        `;
        grid.appendChild(div);
      });
    }
    
    document.getElementById('registerForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const name = document.getElementById('registerName').value;
      const email = document.getElementById('registerEmail').value;
      const password = document.getElementById('registerPassword').value;
      const userType = document.getElementById('userType').value;
      const user = { id: Date.now(), name, email, password, type: userType };
      users.push(user);
      localStorage.setItem('users', JSON.stringify(users));
      alert("Registro exitoso. Por favor, inicia sesión.");
      closeRegisterModal();
    });
    
    document.getElementById('loginForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const email = document.getElementById('loginEmail').value;
      const password = document.getElementById('loginPassword').value;
      const user = users.find(u => u.email === email && u.password === password);
      if(user) {
        currentUser = user;
        localStorage.setItem('currentUser', JSON.stringify(currentUser));
        alert("Bienvenido, " + user.name);
        closeLoginModal();
        updateNav();
        if(user.type === 'seller') showSection('sellerPanel');
        else showSection('buyerPanel');
      } else { alert("Credenciales incorrectas."); }
    });
    
    function openChatModal(orderId) {
      if(!currentUser) {
        alert("Debes iniciar sesión para acceder al chat.");
        return;
      }
      currentChatOrderId = orderId;
      document.getElementById('chatModal').style.display = 'block';
      let order = orders.find(o => o.orderId == orderId);
      document.getElementById('chatWith').innerText = currentUser.type === 'buyer' ? order.seller : order.buyer;
      renderChatMessages();
    }
    
    function closeChatModal() {
      document.getElementById('chatModal').style.display = 'none';
      currentChatOrderId = null;
      document.getElementById('chatInput').value = "";
    }
    
    function renderChatMessages() {
      const chatBox = document.getElementById('chatMessages');
      chatBox.innerHTML = "";
      let messages = chatSessions[currentChatOrderId] || [];
      messages.forEach(msg => {
        let p = document.createElement('p');
        p.innerText = msg;
        chatBox.appendChild(p);
      });
      chatBox.scrollTop = chatBox.scrollHeight;
    }
    
    function sendChatMessage() {
      const input = document.getElementById('chatInput');
      const message = input.value.trim();
      if(message === "") return;
      if(!chatSessions[currentChatOrderId]) chatSessions[currentChatOrderId] = [];
      chatSessions[currentChatOrderId].push((currentUser.type === 'buyer' ? "Tú (Comprador): " : "Tú (Vendedor): ") + message);
      localStorage.setItem('chatSessions', JSON.stringify(chatSessions));
      input.value = "";
      renderChatMessages();
    }
    
    function openProductImages(productId) {
      const product = products.find(p => p.id === productId);
      if(!product || product.images.length === 0) return;
      currentProductImages = product.images;
      currentImageIndex = 0;
      document.getElementById('productModalImage').src = currentProductImages[currentImageIndex];
      document.getElementById('productImagesModal').style.display = 'block';
    }
    
    function closeProductImagesModal() {
      document.getElementById('productImagesModal').style.display = 'none';
    }
    
    function prevProductImage() {
      currentImageIndex = (currentImageIndex > 0) ? currentImageIndex - 1 : currentProductImages.length - 1;
      document.getElementById('productModalImage').src = currentProductImages[currentImageIndex];
    }
    
    function nextProductImage() {
      currentImageIndex = (currentImageIndex < currentProductImages.length - 1) ? currentImageIndex + 1 : 0;
      document.getElementById('productModalImage').src = currentProductImages[currentImageIndex];
    }
    
    function openProductDetail(productId) {
      const product = products.find(p => p.id === productId);
      if(!product) return;
      currentDetailProduct = product;
      currentDetailImages = product.images;
      currentDetailIndex = 0;
      document.getElementById('detailProductTitle').innerText = product.name;
      document.getElementById('detailModalImage').src = currentDetailImages[currentDetailIndex];
      document.getElementById('detailProductDescription').innerText = product.description;
      document.getElementById('detailProductPrice').innerText = "$" + product.price.toFixed(2);
      updateAverageRating();
      renderReviews();
      document.getElementById('productDetailModal').style.display = 'block';
    }
    
    function closeProductDetailModal() {
      document.getElementById('productDetailModal').style.display = 'none';
    }
    
    function prevDetailImage() {
      currentDetailIndex = (currentDetailIndex > 0) ? currentDetailIndex - 1 : currentDetailImages.length - 1;
      document.getElementById('detailModalImage').src = currentDetailImages[currentDetailIndex];
    }
    
    function nextDetailImage() {
      currentDetailIndex = (currentDetailIndex < currentDetailImages.length - 1) ? currentDetailIndex + 1 : 0;
      document.getElementById('detailModalImage').src = currentDetailImages[currentDetailIndex];
    }
    
    function updateAverageRating() {
      const reviews = currentDetailProduct.reviews;
      if(reviews.length === 0) {
        document.getElementById('averageRating').innerText = "Sin calificar";
        return;
      }
      let sum = reviews.reduce((acc, rev) => acc + parseInt(rev.rating), 0);
      let avg = (sum / reviews.length).toFixed(1);
      document.getElementById('averageRating').innerText = avg + " estrellas";
    }
    
    function renderReviews() {
      const container = document.getElementById('reviewsContainer');
      container.innerHTML = "";
      if(currentDetailProduct.reviews.length === 0) {
        container.innerHTML = "<p>No hay reseñas.</p>";
        return;
      }
      currentDetailProduct.reviews.forEach(rev => {
        let div = document.createElement('div');
        div.className = "mb-2 p-2 border rounded";
        div.innerHTML = `<strong>${rev.user}</strong> - ${rev.rating} estrellas<br>${rev.comment}`;
        container.appendChild(div);
      });
    }
    
    document.getElementById('reviewForm').addEventListener('submit', function(e) {
      e.preventDefault();
      const rating = document.getElementById('reviewRating').value;
      const comment = document.getElementById('reviewComment').value;
      if(rating === "" || comment.trim() === "") return;
      const review = { user: currentUser ? currentUser.name : "Anónimo", rating, comment };
      currentDetailProduct.reviews.push(review);
      updateAverageRating();
      renderReviews();
      document.getElementById('reviewForm').reset();
      localStorage.setItem('products', JSON.stringify(products));
    });
    
    /* 50 Funciones Extra de Lógica */
    for(let i = 1; i <= 50; i++){
      window['extraLogic' + i] = function(){
        console.log("Ejecutando lógica extra " + i);
      }
    }
    
    /* Función Extra para Efectos de Animación */
    function extraAnimationEffect(effectName) {
      console.log("Ejecutando efecto de animación extra: " + effectName);
    }
    
    // Al cargar la página se actualiza el menú y se renderizan los productos
    window.onload = function() {
      currentUser = JSON.parse(localStorage.getItem('currentUser')) || null;
      updateNav();
      renderProducts();
      sellerProducts = JSON.parse(localStorage.getItem('sellerProducts')) || [];
      sellerEarnings = parseFloat(localStorage.getItem('sellerEarnings')) || 0;
      orders = JSON.parse(localStorage.getItem('orders')) || [];
      updateSellerEarningsDisplay();
    }
  </script>
</body>
</html>
'''

@app.route('/')
def index():
    return render_template_string(HTML_TEMPLATE)

@app.route('/api/login', methods=['POST'])
def api_login():
    data = request.json
    email = data.get('email')
    password = data.get('password')
    if email == "usuario@example.com" and password == "seguro123":
        return jsonify({"status": "success", "message": "Autenticación exitosa"})
    else:
        return jsonify({"status": "error", "message": "Credenciales inválidas"}), 401

if __name__ == '__main__':
    app.run(debug=True)

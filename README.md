from flask import Flask, render_template_string, request, jsonify

app = Flask(__name__)

HTML_TEMPLATE = '''
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>LA W - Marketplace Mejorado</title>
  <!-- Google Fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
  <style>
    /* Variables de color actualizadas a tonos vibrantes */
    :root {
      --primary-color: #FF5722;      /* Naranja vibrante */
      --secondary-color: #FFC107;    /* Ámbar */
      --nav-color: #E91E63;          /* Rosa fuerte */
      --accent-color: #4CAF50;       /* Verde */
      --bg-color: #f4f4f4;
      --text-color: #333;
    }
    /* Estilos base, animaciones y transición para modo oscuro */
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body {
      font-family: 'Roboto', sans-serif;
      background: var(--bg-color);
      color: var(--text-color);
      line-height: 1.6;
      animation: fadeIn 1s ease-in;
      transition: background 0.5s, color 0.5s;
    }
    /* Modo oscuro (se activa añadiendo la clase "dark" al body) */
    body.dark {
      background: #121212;
      color: #e0e0e0;
    }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    
    /* Encabezado y Home */
    header {
      background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
      color: white;
      padding: 1.5rem;
      text-align: center;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
      animation: slideDown 0.8s ease-out;
    }
    @keyframes slideDown { from { transform: translateY(-100%); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
    header h1 { font-size: 2.8rem; margin-bottom: 0.5rem; }
    
    #home {
      position: relative;
      background: url('https://images.unsplash.com/photo-1557682224-5b8590cd9ec5?ixlib=rb-4.0.3&auto=format&fit=crop&w=1350&q=80') no-repeat center center/cover;
      animation: backgroundMove 30s linear infinite;
      color: white;
      padding: 4rem 1rem;
      text-align: center;
      overflow: hidden;
    }
    #home::after {
      content: "";
      position: absolute;
      top: 0; left: 0; right: 0; bottom: 0;
      background: rgba(0,0,0,0.4);
      z-index: 1;
    }
    #home > * { position: relative; z-index: 2; }
    @keyframes backgroundMove {
      0% { background-position: 0% 50%; }
      50% { background-position: 100% 50%; }
      100% { background-position: 0% 50%; }
    }
    #home .cta-button {
      margin-top: 1.5rem;
      padding: 0.75rem 1.5rem;
      background: var(--accent-color);
      color: var(--nav-color);
      border: none;
      font-size: 1.2rem;
      border-radius: 4px;
      cursor: pointer;
      transition: background 0.3s, transform 0.3s;
      animation: pulse 2s infinite;
    }
    @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.05); } 100% { transform: scale(1); } }
    #home .cta-button:hover { background: #ffc107; }
    #home .extra-info {
      margin-top: 1.5rem;
      font-size: 1.1rem;
      max-width: 800px;
      margin-left: auto;
      margin-right: auto;
    }
    
    /* Navegación */
    nav {
      background: var(--nav-color);
      padding: 0.75rem 1rem;
      text-align: center;
      position: sticky;
      top: 0;
      z-index: 100;
      box-shadow: 0 2px 3px rgba(0,0,0,0.15);
      animation: fadeIn 1s ease-in;
    }
    nav a {
      color: white;
      margin: 0 1rem;
      text-decoration: none;
      font-weight: 500;
      transition: color 0.3s, transform 0.3s;
      cursor: pointer;
    }
    nav a:hover { color: var(--accent-color); transform: scale(1.05); }
    
    /* Contenedores */
    .container { width: 90%; max-width: 1200px; margin: 2rem auto; padding: 1rem; animation: fadeIn 1s ease-in; }
    
    /* Grid de Productos */
    .product-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
      gap: 1.5rem;
    }
    .product {
      background: white;
      border-radius: 8px;
      overflow: hidden;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      transition: transform 0.3s, box-shadow 0.3s;
    }
    .product:hover { transform: translateY(-5px); box-shadow: 0 4px 12px rgba(0,0,0,0.2); }
    .product img { width: 100%; display: block; }
    .product-content { padding: 1rem; }
    .product-content h3 { margin-bottom: 0.5rem; font-size: 1.2rem; }
    .product-content p { font-size: 0.95rem; line-height: 1.4; }
    .product-content .price { font-weight: bold; margin-top: 0.5rem; font-size: 1rem; color: var(--primary-color); }
    .product-content button {
      margin-top: 0.75rem; width: 100%; padding: 0.5rem;
      background: var(--primary-color); border: none; color: white;
      border-radius: 4px; cursor: pointer;
      transition: background 0.3s, transform 0.3s;
    }
    .product-content button:hover { background: var(--nav-color); transform: scale(1.05); }
    
    /* Paneles: Carrito, Vendedor, Comprador */
    .panel {
      display: none;
      background: white;
      padding: 1rem;
      border-radius: 8px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      margin-bottom: 2rem;
      animation: slideUp 0.8s ease-out;
    }
    @keyframes slideUp { from { transform: translateY(50px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
    .panel.active { display: block; }
    
    /* Carrito */
    #cartPanel ul { list-style: none; margin: 1rem 0; padding: 0; }
    #cartPanel li { padding: 0.75rem; border-bottom: 1px solid #ccc; }
    
    /* Formularios */
    form input, form select, form textarea, form button {
      width: 100%; padding: 0.75rem; margin-bottom: 0.75rem;
      font-size: 1rem; border-radius: 4px; border: 1px solid #ccc;
      transition: border 0.3s;
    }
    form input:focus, form select:focus, form textarea:focus { border-color: var(--primary-color); outline: none; }
    form button { background: var(--primary-color); border: none; color: white; cursor: pointer; transition: background 0.3s, transform 0.3s; }
    form button:hover { background: var(--nav-color); transform: scale(1.05); }
    
    /* Previsualización de imágenes */
    .image-preview { display: flex; gap: 0.5rem; flex-wrap: wrap; margin-bottom: 0.75rem; }
    .image-preview img {
      width: 80px; height: 80px; object-fit: cover; border: 1px solid #ccc;
      border-radius: 4px; transition: transform 0.3s;
    }
    .image-preview img:hover { transform: scale(1.1); }
    
    /* Modales */
    .modal {
      display: none;
      position: fixed; z-index: 2000;
      left: 0; top: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.6); animation: fadeIn 0.5s;
    }
    .modal-content {
      background: #fff;
      margin: 10% auto; padding: 1.5rem;
      width: 90%; max-width: 500px;
      border-radius: 8px; position: relative;
      animation: slideIn 0.5s;
    }
    @keyframes slideIn { from { transform: translateY(-50px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
    .close { position: absolute; top: 10px; right: 15px; font-size: 1.5rem; color: #333; cursor: pointer; }
    
    /* Modal de Chat */
    #chatModal #chatMessages { height: 200px; overflow-y: auto; border: 1px solid #ccc; padding: 0.5rem; margin-bottom: 1rem; }
    /* Modal de Imágenes (Lightbox) */
    #productImagesModal img { width: 100%; height: auto; }
    /* Modal de Detalle del Producto */
    #productDetailModal .reviews { max-height: 150px; overflow-y: auto; border-top: 1px solid #ccc; padding-top: 0.5rem; margin-top: 0.5rem; }
    #productDetailModal .review { background: #f0f0f0; padding: 0.5rem; margin-bottom: 0.5rem; border-radius: 4px; }
    
    /* Responsive */
    @media(max-width: 768px) { .product { grid-column: span 2; } }
    @media(max-width: 480px) { .product { grid-column: span 1; } }
    
    /* Botón fijo para Modo Oscuro */
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
  <!-- Efectos de Sonido -->
  <audio id="soundPurchase" src="https://example.com/sound_purchase.mp3"></audio>
  <audio id="soundCart" src="https://example.com/sound_cart.mp3"></audio>
  <audio id="soundRegister" src="https://example.com/sound_register.mp3"></audio>
  <audio id="soundConfirm" src="https://example.com/sound_confirm.mp3"></audio>
  <audio id="soundChat" src="https://example.com/sound_chat.mp3"></audio>
  
  <!-- Encabezado y Navegación -->
  <header>
    <h1>LA W - Marketplace</h1>
    <p>Compra y vende con confianza</p>
  </header>
  <nav>
    <a onclick="showSection('home')">Inicio</a>
    <a onclick="showSection('marketplace')">Productos</a>
    <a onclick="showSection('cartPanel')">Carrito</a>
    <a onclick="showSection('sellerPanel')">Panel Vendedor</a>
    <a onclick="showSection('buyerPanel')">Panel Comprador</a>
    <a onclick="showLoginModal()">Iniciar Sesión</a>
    <a onclick="showRegisterModal()">Registrarse</a>
  </nav>
  
  <!-- Sección Home -->
  <div class="container" id="home">
    <h2>Descubre, compra y vende con estilo</h2>
    <p class="extra-info">Únete a LA W y forma parte del marketplace más dinámico y seguro. ¡Tu satisfacción es nuestra prioridad!</p>
    <button class="cta-button" onclick="showSection('marketplace')">Explora Productos</button>
  </div>
  
  <!-- Sección Marketplace -->
  <div class="container" id="marketplace" style="display:none;">
    <h2>Productos</h2>
    <div class="search-bar">
      <input type="text" id="searchInput" placeholder="Buscar productos..." onkeyup="filterProducts()">
    </div>
    <div class="product-grid" id="productGrid">
      <!-- Productos se cargarán dinámicamente -->
    </div>
  </div>
  
  <!-- Panel Carrito -->
  <div class="container panel" id="cartPanel">
    <h2>Carrito de Compras</h2>
    <ul id="cartItems"></ul>
    <p>Total: <span id="total">$0.00</span></p>
    <button onclick="checkoutCart()">Comprar Carrito</button>
  </div>
  
  <!-- Panel Vendedor -->
  <div class="container panel" id="sellerPanel">
    <h2>Panel Vendedor</h2>
    <p>Aquí puedes agregar y gestionar tus productos.</p>
    <form id="addProductForm">
      <input type="text" id="productName" placeholder="Nombre del producto" required>
      <input type="text" id="productDescription" placeholder="Descripción" required>
      <input type="number" id="productPrice" placeholder="Precio" step="0.01" required>
      <input type="file" id="productImages" accept="image/*" multiple required>
      <div class="image-preview" id="imagePreview"></div>
      <button type="submit">Agregar Producto</button>
    </form>
    <hr>
    <h3>Mis Productos</h3>
    <div class="product-grid" id="sellerProductGrid"></div>
    <hr>
    <h3>Órdenes en Proceso</h3>
    <div id="sellerOrders"></div>
    <hr>
    <h3>Datos del Vendedor</h3>
    <div id="sellerDataDisplay"></div>
    <hr>
    <h3>Ganancias del Vendedor</h3>
    <div id="sellerEarningsDisplay"><p>Ganancias: $0.00</p></div>
  </div>
  
  <!-- Panel Comprador -->
  <div class="container panel" id="buyerPanel">
    <h2>Panel Comprador</h2>
    <p>Bienvenido, disfruta de tus compras y haz seguimiento de tus órdenes.</p>
    <div id="buyerOrders"></div>
    <div class="info" style="margin-top:1rem; font-size:0.95rem; color:var(--nav-color);">
      Una vez confirmada la compra se desbloqueará el chat con el vendedor.
    </div>
  </div>
  
  <!-- Modal de Login -->
  <div id="loginModal" class="modal">
    <div class="modal-content">
      <span class="close" onclick="closeLoginModal()">&times;</span>
      <h2>Iniciar Sesión</h2>
      <form id="loginForm">
        <input type="email" id="loginEmail" placeholder="Email" required>
        <input type="password" id="loginPassword" placeholder="Contraseña" required>
        <button type="submit">Entrar</button>
      </form>
    </div>
  </div>
  
  <!-- Modal de Registro -->
  <div id="registerModal" class="modal">
    <div class="modal-content">
      <span class="close" onclick="closeRegisterModal()">&times;</span>
      <h2>Registrarse</h2>
      <form id="registerForm">
        <input type="text" id="registerName" placeholder="Nombre completo" required>
        <input type="email" id="registerEmail" placeholder="Email" required>
        <input type="password" id="registerPassword" placeholder="Contraseña" required>
        <label for="userType">Tipo de Usuario:</label>
        <select id="userType" required onchange="toggleSellerFields(this.value)">
          <option value="buyer">Comprador</option>
          <option value="seller">Vendedor</option>
        </select>
        <!-- Campos adicionales para vendedores -->
        <div id="sellerExtraFields">
          <input type="text" id="cedula" placeholder="Cédula" required>
          <input type="text" id="telefono" placeholder="Número de Teléfono" required>
          <input type="text" id="ubicacion" placeholder="Ubicación" required>
          <input type="text" id="cuentaBancaria" placeholder="Cuenta Bancaria" required>
          <label>Expediente Judicial (Archivo, PDF o imagen):</label>
          <input type="file" id="expedienteJudicial" accept="image/*,application/pdf" required>
          <label>Expediente Disciplinario (Archivo, PDF o imagen):</label>
          <input type="file" id="expedienteDisciplinario" accept="image/*,application/pdf" required>
        </div>
        <button type="submit">Registrarse</button>
      </form>
    </div>
  </div>
  
  <!-- Modal de Chat -->
  <div id="chatModal" class="modal">
    <div class="modal-content">
      <span class="close" onclick="closeChatModal()">&times;</span>
      <h2>Chat con <span id="chatWith"></span></h2>
      <div id="chatMessages" style="height:200px; overflow-y:auto; border:1px solid #ccc; padding:0.5rem; margin-bottom:1rem;"></div>
      <input type="text" id="chatInput" placeholder="Escribe tu mensaje..." style="width:80%;">
      <button onclick="sendChatMessage()">Enviar</button>
    </div>
  </div>
  
  <!-- Modal de Imágenes del Producto (Lightbox) -->
  <div id="productImagesModal" class="modal">
    <div class="modal-content">
      <span class="close" onclick="closeProductImagesModal()">&times;</span>
      <img id="productModalImage" src="" alt="Imagen del Producto">
      <div style="text-align: center; margin-top: 1rem;">
        <button onclick="prevProductImage()">Anterior</button>
        <button onclick="nextProductImage()">Siguiente</button>
      </div>
    </div>
  </div>
  
  <!-- Modal de Detalle del Producto (con reseñas y rating) -->
  <div id="productDetailModal" class="modal">
    <div class="modal-content">
      <span class="close" onclick="closeProductDetailModal()">&times;</span>
      <h2 id="detailProductTitle"></h2>
      <div id="detailProductImages" style="position: relative;">
        <img id="detailModalImage" src="" alt="Imagen del Producto" style="width:100%;">
        <div style="text-align: center; margin-top: 0.5rem;">
          <button onclick="prevDetailImage()">Anterior</button>
          <button onclick="nextDetailImage()">Siguiente</button>
        </div>
      </div>
      <p id="detailProductDescription"></p>
      <p class="price" id="detailProductPrice"></p>
      <p>Calificación Promedio: <span id="averageRating">Sin calificar</span></p>
      <div class="reviews" id="reviewsContainer"></div>
      <form id="reviewForm">
        <h3>Agrega tu reseña</h3>
        <select id="reviewRating" required>
          <option value="">Calificación</option>
          <option value="1">1 estrella</option>
          <option value="2">2 estrellas</option>
          <option value="3">3 estrellas</option>
          <option value="4">4 estrellas</option>
          <option value="5">5 estrellas</option>
        </select>
        <textarea id="reviewComment" placeholder="Tu comentario" required></textarea>
        <button type="submit">Enviar Reseña</button>
      </form>
    </div>
  </div>
  
  <!-- Pie de Página -->
  <footer>
    <p>&copy; 2025 LA W. Todos los derechos reservados.</p>
  </footer>
  
  <!-- Botón fijo para alternar Modo Oscuro -->
  <button class="dark-toggle" onclick="toggleTheme()">Modo Oscuro</button>
  
  <!-- JavaScript: Funciones existentes + Nuevas funciones -->
  <script>
    /* Variables globales y simulación de almacenamiento */
    let users = JSON.parse(localStorage.getItem('users')) || [];
    let currentUser = JSON.parse(localStorage.getItem('currentUser')) || null;
    let sellerEarnings = parseFloat(localStorage.getItem('sellerEarnings')) || 0;
    let orders = JSON.parse(localStorage.getItem('orders')) || [];
    let secureVault = JSON.parse(localStorage.getItem('secureVault')) || {};
    let sellerProducts = JSON.parse(localStorage.getItem('sellerProducts')) || [];
    let cart = JSON.parse(localStorage.getItem('cart')) || [];
    let chatSessions = JSON.parse(localStorage.getItem('chatSessions')) || {};
    let currentChatOrderId = null;
    
    /* Productos iniciales (cada producto incluye un array de reseñas) */
    let products = [
      { id: 1, name: 'Producto 1', description: 'Descripción breve del producto 1', price: 10.00, images: ['https://via.placeholder.com/300x200', 'https://via.placeholder.com/300x200/AAAAAA'], seller: "default@seller.com", reviews: [] },
      { id: 2, name: 'Producto 2', description: 'Descripción breve del producto 2', price: 20.00, images: ['https://via.placeholder.com/300x200'], seller: "default@seller.com", reviews: [] },
      { id: 3, name: 'Producto 3', description: 'Descripción breve del producto 3', price: 15.00, images: ['https://via.placeholder.com/300x200', 'https://via.placeholder.com/300x200/CCCCCC', 'https://via.placeholder.com/300x200/EEEEEE'], seller: "default@seller.com", reviews: [] },
      { id: 4, name: 'Producto 4', description: 'Descripción breve del producto 4', price: 8.00, images: ['https://via.placeholder.com/300x200'], seller: "default@seller.com", reviews: [] }
    ];
    
    /* Variables para modales de imágenes y detalle */
    let currentProductImages = [];
    let currentImageIndex = 0;
    let currentDetailProduct = null;
    let currentDetailImages = [];
    let currentDetailIndex = 0;
    
    /* Función para mostrar secciones */
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
    
    /* Renderizar productos en Marketplace (incluye botones de acciones) */
    function renderProducts() {
      const grid = document.getElementById('productGrid');
      grid.innerHTML = '';
      products.forEach(product => {
        let div = document.createElement('div');
        div.className = 'product';
        div.innerHTML = `
          <img src="${product.images[0]}" alt="${product.name}">
          <div class="product-content">
            <h3>${product.name}</h3>
            <p>${product.description}</p>
            <p class="price">$${product.price.toFixed(2)}</p>
            <button onclick="simulatePayment(${product.id})">Comprar</button>
            <button onclick="addToCart(${product.id}, this)">Agregar al carrito</button>
            ${ product.images.length > 1 ? `<button onclick="openProductImages(${product.id})">Ver imágenes</button>` : '' }
            <button onclick="openProductDetail(${product.id})">Ver detalles</button>
          </div>
        `;
        grid.appendChild(div);
      });
    }
    
    /* Funciones de pago, carrito, órdenes y chat */
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
      document.getElementById('soundPurchase').play();
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
      document.getElementById('soundCart').play();
      alert(`${product.name} agregado al carrito.`);
    }
    
    function renderCart() {
      const cartList = document.getElementById('cartItems');
      cartList.innerHTML = '';
      if(cart.length === 0) {
        cartList.innerHTML = "<li>El carrito está vacío.</li>";
      } else {
        let total = 0;
        cart.forEach((item, index) => {
          let li = document.createElement('li');
          li.innerHTML = `${item.name} - $${item.price.toFixed(2)} <button onclick="removeFromCart(${index})">Quitar</button>`;
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
      document.getElementById('soundPurchase').play();
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
        div.className = 'product';
        div.innerHTML = `
          <img src="${product.images[0]}" alt="${product.name}">
          <div class="product-content">
            <h3>${product.name}</h3>
            <p>${product.description}</p>
            <p class="price">$${product.price.toFixed(2)}</p>
            <button onclick="simulatePayment(${product.id})">Comprar</button>
            <button onclick="addToCart(${product.id}, this)">Agregar al carrito</button>
            ${ product.images.length > 1 ? `<button onclick="openProductImages(${product.id})">Ver imágenes</button>` : '' }
            <button onclick="openProductDetail(${product.id})">Ver detalles</button>
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
              div.className = 'order';
              div.innerHTML = `<strong>Orden ${order.orderId}</strong> - ${order.product.name} - Estado: ${order.status}`;
              if(order.status === "En proceso") { div.innerHTML += `<br><button onclick="openChatModal(${order.orderId})">Chat</button>`; }
              else if(order.status === "Confirmada" && !order.buyerNotified) { 
                div.innerHTML += `<div class="info">¡Felicitaciones! Has hecho tu primera compra, gracias por confiar en nosotros.</div>`;
                order.buyerNotified = true;
                localStorage.setItem('orders', JSON.stringify(orders));
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
              div.className = 'order';
              div.innerHTML = `<strong>Orden ${order.orderId}</strong> - ${order.product.name} - Estado: ${order.status}`;
              if(order.status === "En proceso") { 
                div.innerHTML += `<br><button onclick="confirmOrder(${order.orderId})">Confirmar Pago</button>`;
                div.innerHTML += `<button onclick="openChatModal(${order.orderId})">Chat</button>`;
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
        document.getElementById('soundConfirm').play();
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
        div.className = 'product';
        div.innerHTML = `
          <img src="${product.images[0]}" alt="${product.name}">
          <div class="product-content">
            <h3>${product.name}</h3>
            <p>${product.description}</p>
            <p class="price">$${product.price.toFixed(2)}</p>
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
      document.getElementById('soundRegister').play();
      if(userType === "seller") {
        const cedula = document.getElementById('cedula').value;
        const telefono = document.getElementById('telefono').value;
        const ubicacion = document.getElementById('ubicacion').value;
        const cuentaBancaria = document.getElementById('cuentaBancaria').value;
        const expedienteJudicialInput = document.getElementById('expedienteJudicial');
        const expedienteDisciplinarioInput = document.getElementById('expedienteDisciplinario');
        function readFile(file) {
          return new Promise((resolve, reject) => {
            const reader = new FileReader();
            reader.onload = () => resolve(reader.result);
            reader.onerror = reject;
            reader.readAsDataURL(file);
          });
        }
        Promise.all([
          readFile(expedienteJudicialInput.files[0]),
          readFile(expedienteDisciplinarioInput.files[0])
        ]).then(results => {
          secureVault[email] = {
            cedula, telefono, ubicacion, cuentaBancaria,
            expedienteJudicial: results[0],
            expedienteDisciplinario: results[1]
          };
          localStorage.setItem('secureVault', JSON.stringify(secureVault));
          alert("Registro exitoso como vendedor. Por favor, inicia sesión.");
          closeRegisterModal();
        }).catch(err => { alert("Error al leer los archivos. Inténtalo de nuevo."); });
      } else {
        alert("Registro exitoso. Por favor, inicia sesión.");
        closeRegisterModal();
      }
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
        if(user.type === 'seller') showSection('sellerPanel');
        else showSection('buyerPanel');
      } else { alert("Credenciales incorrectas."); }
    });
    
    /* Funciones para Chat */
    function openChatModal(orderId) {
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
      messages.forEach(msg => { let p = document.createElement('p'); p.innerText = msg; chatBox.appendChild(p); });
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
      document.getElementById('soundChat').play();
    }
    
    /* Modal de Imágenes (Lightbox) */
    function openProductImages(productId) {
      const product = products.find(p => p.id === productId);
      if(!product || product.images.length === 0) return;
      currentProductImages = product.images;
      currentImageIndex = 0;
      document.getElementById('productModalImage').src = currentProductImages[currentImageIndex];
      document.getElementById('productImagesModal').style.display = 'block';
    }
    function closeProductImagesModal() { document.getElementById('productImagesModal').style.display = 'none'; }
    function prevProductImage() {
      currentImageIndex = (currentImageIndex > 0) ? currentImageIndex - 1 : currentProductImages.length - 1;
      document.getElementById('productModalImage').src = currentProductImages[currentImageIndex];
    }
    function nextProductImage() {
      currentImageIndex = (currentImageIndex < currentProductImages.length - 1) ? currentImageIndex + 1 : 0;
      document.getElementById('productModalImage').src = currentProductImages[currentImageIndex];
    }
    
    /* Modal de Detalle del Producto (con reseñas) */
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
    function closeProductDetailModal() { document.getElementById('productDetailModal').style.display = 'none'; }
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
      if(reviews.length === 0) { document.getElementById('averageRating').innerText = "Sin calificar"; return; }
      let sum = reviews.reduce((acc, rev) => acc + parseInt(rev.rating), 0);
      let avg = (sum / reviews.length).toFixed(1);
      document.getElementById('averageRating').innerText = avg + " estrellas";
    }
    function renderReviews() {
      const container = document.getElementById('reviewsContainer');
      container.innerHTML = "";
      if(currentDetailProduct.reviews.length === 0) { container.innerHTML = "<p>No hay reseñas.</p>"; return; }
      currentDetailProduct.reviews.forEach(rev => {
        let div = document.createElement('div');
        div.className = "review";
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
    
    /* Nuevas Funciones (más de 50) - Estas funciones de ejemplo muestran alertas y se pueden ampliar */
    function sortProductsByPrice() { alert("Función: ordenar productos por precio."); }
    function sortProductsByRating() { alert("Función: ordenar productos por calificación."); }
    function paginateProducts(pageNumber) { alert("Función: paginar productos en la página " + pageNumber); }
    function addToWishlist(productId) { alert("Función: agregar producto " + productId + " a la lista de deseos."); }
    function removeFromWishlist(productId) { alert("Función: eliminar producto " + productId + " de la lista de deseos."); }
    function viewWishlist() { alert("Función: ver lista de deseos."); }
    function compareProducts(productId) { alert("Función: agregar producto " + productId + " a la comparación."); }
    function removeFromCompare(productId) { alert("Función: eliminar producto " + productId + " de la comparación."); }
    function shareProduct(productId) { alert("Función: compartir producto " + productId + " en redes sociales."); }
    function zoomProductImage(productId) { alert("Función: hacer zoom en imagen del producto " + productId); }
    function toggleTheme() { 
      // Alterna el modo oscuro real: agrega/quita la clase 'dark' en el body
      document.body.classList.toggle('dark');
    }
    function switchLanguage(lang) { alert("Función: cambiar idioma a " + lang); }
    function convertCurrency(amount, currency) { alert("Función: convertir " + amount + " a " + currency); }
    function showUserProfile() { alert("Función: mostrar perfil del usuario."); }
    function updateUserProfile() { alert("Función: actualizar perfil del usuario."); }
    function viewOrderHistory() { alert("Función: ver historial de órdenes."); }
    function submitReview(productId) { alert("Función: enviar reseña para producto " + productId); }
    function upvoteReview(reviewId) { alert("Función: votar útil la reseña " + reviewId); }
    function downvoteReview(reviewId) { alert("Función: votar no útil la reseña " + reviewId); }
    function attachFileToChat(orderId) { alert("Función: adjuntar archivo al chat para la orden " + orderId); }
    function notifyNewOrder() { alert("Función: notificar nueva orden."); }
    function simulatePushNotification() { alert("Función: simular notificación push."); }
    function multiStepCheckout() { alert("Función: proceso de compra en múltiples pasos."); }
    function simulatePaymentGateway() { alert("Función: simular integración de pasarela de pago."); }
    function trackOrder(orderId) { alert("Función: rastrear orden " + orderId); }
    function addShippingDetails(orderId) { alert("Función: agregar detalles de envío para orden " + orderId); }
    function calculateShipping(orderId) { alert("Función: calcular costo de envío para orden " + orderId); }
    function applyCouponCode(code) { alert("Función: aplicar cupón " + code); }
    function updateLoyaltyPoints(userId) { alert("Función: actualizar puntos de fidelidad para usuario " + userId); }
    function showRecentlyViewed() { alert("Función: mostrar productos recientemente vistos."); }
    function recommendProducts() { alert("Función: recomendar productos basados en historial."); }
    function showSellerAnalytics() { alert("Función: mostrar analíticas del vendedor."); }
    function manageInventory() { alert("Función: gestionar inventario del vendedor."); }
    function alertLowInventory(productId) { alert("Función: alerta de inventario bajo para producto " + productId); }
    function editProduct(productId) { alert("Función: editar producto " + productId); }
    function deleteProduct(productId) { alert("Función: eliminar producto " + productId); }
    function cancelOrder(orderId) { alert("Función: cancelar orden " + orderId); }
    function requestRefund(orderId) { alert("Función: solicitar reembolso para orden " + orderId); }
    function createSupportTicket() { alert("Función: crear ticket de soporte."); }
    function viewFAQ() { alert("Función: ver preguntas frecuentes."); }
    function viewBlog() { alert("Función: ver blog o noticias."); }
    function simulateEmailVerification() { alert("Función: simular verificación de email."); }
    function simulateCreditCardPayment() { alert("Función: simular pago con tarjeta de crédito."); }
    function downloadInvoice(orderId) { alert("Función: descargar factura para orden " + orderId); }
    function sortOrdersByDate() { alert("Función: ordenar órdenes por fecha."); }
    function filterOrdersByStatus(status) { alert("Función: filtrar órdenes por estado: " + status); }
    function showAdminPanel() { alert("Función: mostrar panel de administración."); }
    function manageUsers() { alert("Función: gestionar usuarios."); }
    function manageProducts() { alert("Función: gestionar productos."); }
    function liveChatSearch(query) { alert("Función: búsqueda en chat: " + query); }
    function showTypingIndicator() { alert("Función: mostrar indicador de escritura."); }
    function animatePanelTransition(panelId) { alert("Función: animar transición para panel " + panelId); }
    function customizeTheme(options) { alert("Función: personalizar tema con opciones " + JSON.stringify(options)); }
    function viewProductRecommendations() { alert("Función: ver recomendaciones de productos."); }
    function addToFavorites(productId) { alert("Función: agregar producto " + productId + " a favoritos."); }
    function removeFromFavorites(productId) { alert("Función: eliminar producto " + productId + " de favoritos."); }
    function productQandA(productId) { alert("Función: preguntas y respuestas para producto " + productId); }
    function reportProduct(productId) { alert("Función: reportar producto " + productId + " por contenido inapropiado."); }
    function updateSEOMetaTags() { alert("Función: actualizar etiquetas SEO."); }
    function generateSitemap() { alert("Función: generar sitemap."); }
    function integrateAnalytics() { alert("Función: integrar Google Analytics."); }
    function simulateSocialLogin() { alert("Función: simular inicio de sesión con redes sociales."); }
    function bulkUploadProducts(csvData) { alert("Función: subir productos en masa desde CSV."); }
    function subscribeNewsletter(email) { alert("Función: suscribir " + email + " al newsletter."); }
    function showBreadcrumbNavigation() { alert("Función: mostrar navegación por migas de pan."); }
    function paginateReviews(productId, page) { alert("Función: paginar reseñas para producto " + productId + " en página " + page); }
    function sortReviewsBy(criteria) { alert("Función: ordenar reseñas por " + criteria); }
    function addProductTags(productId, tags) { alert("Función: agregar etiquetas " + tags + " al producto " + productId); }
    function filterProductsByTags(tag) { alert("Función: filtrar productos por etiqueta " + tag); }
    function showComparisonTable() { alert("Función: mostrar tabla comparativa de productos."); }
    function showSkeletonLoading() { alert("Función: mostrar pantalla de carga (skeleton)."); }
    function liveChatBot() { alert("Función: iniciar bot de chat."); }
    function leaveProductQuestion(productId) { alert("Función: dejar pregunta para producto " + productId); }
    function rateSeller(sellerId, rating) { alert("Función: calificar al vendedor " + sellerId + " con " + rating + " estrellas."); }
    function followSeller(sellerId) { alert("Función: seguir al vendedor " + sellerId); }
    function showNotificationsCenter() { alert("Función: mostrar centro de notificaciones."); }
    function updateProfilePicture() { alert("Función: actualizar foto de perfil."); }
    function changePassword() { alert("Función: cambiar contraseña."); }
    function showSecuritySettings() { alert("Función: mostrar configuración de seguridad."); }
    function simulateTwoFactorAuth() { alert("Función: simular autenticación de dos factores."); }
    function logoutUser() { alert("Función: cerrar sesión."); }
    function rememberMeOption() { alert("Función: opción de recordar usuario."); }
    function showCookieConsent() { alert("Función: mostrar banner de consentimiento de cookies."); }
    function showGDPRCompliance() { alert("Función: mostrar página de cumplimiento GDPR."); }
    function viewTermsAndConditions() { alert("Función: ver términos y condiciones."); }
    function viewPrivacyPolicy() { alert("Función: ver política de privacidad."); }
    function searchFAQ(query) { alert("Función: buscar en FAQ: " + query); }
    function switchLanguageMultilingual(lang) { alert("Función: cambiar idioma a " + lang + " (multilingüe)."); }
    function updateOrderStatusRealTime(orderId) { alert("Función: actualizar estado de orden " + orderId + " en tiempo real."); }
    function autoLogoutAfterInactivity() { alert("Función: auto logout tras inactividad."); }
    function exportOrderHistoryCSV(userId) { alert("Función: exportar historial de órdenes a CSV para usuario " + userId); }
    function leaveSellerTip(orderId, amount) { alert("Función: dejar propina de $" + amount + " para orden " + orderId); }
    function rewardsSystem() { alert("Función: sistema de recompensas y badges."); }
    function giftProduct(productId, recipientEmail) { alert("Función: regalar producto " + productId + " a " + recipientEmail); }
    function productSubscription(productId) { alert("Función: suscripción para producto " + productId); }
    function simulateEmailNotification() { alert("Función: simular notificación por email."); }
    function simulateSMSNotification() { alert("Función: simular notificación por SMS."); }
    function customizeEmailTemplate(templateOptions) { alert("Función: personalizar plantilla de email con opciones " + JSON.stringify(templateOptions)); }
    function viewSimilarProducts(category) { alert("Función: ver productos similares de la categoría " + category); }
    function filterProductsByPriceRange(min, max) { alert("Función: filtrar productos entre $" + min + " y $" + max); }
    function showMapForSellerLocation(sellerId) { alert("Función: mostrar mapa para la ubicación del vendedor " + sellerId); }
    
    /* Fin de las funciones nuevas */
    
    /* Inicialización */
    window.onload = function() {
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

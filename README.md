from flask import Flask, render_template_string, request, jsonify, url_for
import random
from itsdangerous import URLSafeTimedSerializer, SignatureExpired, BadSignature

app = Flask(__name__)
app.config['SECRET_KEY'] = 'supersecretkey'
serializer = URLSafeTimedSerializer(app.config['SECRET_KEY'])

HTML_TEMPLATE = '''
<!DOCTYPE html>
<html lang="es">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="LA W - Marketplace: Tu plataforma segura para comprar y vender en línea.">
    <meta name="keywords" content="marketplace, comprar, vender, online, seguro">
    <meta property="og:title" content="LA W - Marketplace">
    <meta property="og:description" content="Descubre un marketplace seguro y profesional para comprar y vender en línea.">
    <meta property="og:image" content="https://via.placeholder.com/1200x630.png?text=LA+W+Marketplace">
    <title>LA W - Marketplace</title>
    <!-- Bootstrap CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <!-- Animate.css -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css"/>
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap" rel="stylesheet">
    <style>
      :root {
        --bg-color: #E0F7FA;          /* Fondo azul claro */
        --container-bg: #FFFFFF;       /* Superficies blancas */
        --primary-color: #0277BD;      /* Azul oscuro */
        --secondary-color: #66B2FF;    /* Azul claro */
        --nav-color: #0277BD;
        --accent-color: #01579B;
        --text-color: #343a40;
        --danger-color: #D32F2F;
        --overlay-color: rgba(0,0,0,0.5);
      }
      /* Textos destacados en blanco con sombra azul */
      .color-cycle {
        color: #FFFFFF !important;
        text-shadow: 2px 2px 4px #0277BD;
      }
      @keyframes swing { 20% { transform: rotate(15deg); } 40% { transform: rotate(-10deg); } 60% { transform: rotate(5deg); } 80% { transform: rotate(-5deg); } 100% { transform: rotate(0deg); } }
      .swing { animation: swing 1s ease-in-out; }
      @keyframes heartBeat { 0%, 100% { transform: scale(1); } 25% { transform: scale(1.1); } 50% { transform: scale(0.95); } 75% { transform: scale(1.05); } }
      .heartBeat { animation: heartBeat 1.5s infinite; }
      * { box-sizing: border-box; margin: 0; padding: 0; }
      body { font-family: 'Roboto', sans-serif; background: var(--bg-color); color: var(--text-color); transition: background 0.5s, color 0.5s; }
      body.dark { background: #121212; color: #e0e0e0; }
      img { max-width: 100%; height: auto; }
      header { background: linear-gradient(135deg, var(--primary-color), var(--secondary-color)); color: white; padding: 2rem 1rem; text-align: center; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
      header h1 { font-size: 3rem; margin-bottom: 0.5rem; }
      header p { font-size: 1.4rem; }
      #home { position: relative; background: url('https://images.unsplash.com/photo-1557682224-5b8590cd9ec5?ixlib=rb-4.0.3&auto=format&fit=crop&w=1350&q=80') no-repeat center center/cover; padding: 6rem 1rem; text-align: center; overflow: hidden; }
      #home::after { content: ""; position: absolute; top: 0; left: 0; right: 0; bottom: 0; background: var(--overlay-color); z-index: 1; }
      #home > * { position: relative; z-index: 2; }
      #home .cta-button { margin-top: 2rem; padding: 1rem 2rem; background: var(--accent-color); color: var(--nav-color); border: none; font-size: 1.4rem; border-radius: 6px; cursor: pointer; transition: background 0.3s, transform 0.3s; }
      #home .cta-button:hover { background: var(--secondary-color); transform: scale(1.15); }
      #home .extra-info { margin-top: 1.5rem; font-size: 1.2rem; max-width: 800px; margin-left: auto; margin-right: auto; }
      .home-invite { background-color: var(--container-bg); padding: 1.5rem; border-radius: 8px; box-shadow: 0 2px 6px rgba(0,0,0,0.1); margin-bottom: 2rem; }
      #info { margin-top: 3rem; }
      #info .info-row { margin-bottom: 2rem; }
      #info .info-row img { border-radius: 8px; border: 3px solid var(--container-bg); box-shadow: 0 2px 6px rgba(0,0,0,0.3); }
      .dark-toggle { position: fixed; bottom: 20px; right: 20px; background: var(--accent-color); color: var(--container-bg); border: none; padding: 0.75rem 1rem; border-radius: 4px; cursor: pointer; box-shadow: 0 2px 4px rgba(0,0,0,0.3); transition: transform 0.3s; }
      .dark-toggle:hover { transform: scale(1.1); }
      .bot-button { position: fixed; bottom: 20px; left: 20px; z-index: 1050; }
      .btn-ver { font-size: 0.9rem; margin-right: 0.5rem; }
      @media(max-width: 768px) { header h1 { font-size: 2.5rem; } header p { font-size: 1.2rem; } .cta-button { font-size: 1.1rem; padding: 0.8rem 1.5rem; } }
      @media(max-width: 480px) { header h1 { font-size: 2rem; } .cta-button { font-size: 1rem; padding: 0.5rem 1rem; } }
    </style>
  </head>
  <body>
    <!-- Navbar -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark">
      <div class="container">
        <a class="navbar-brand animate__animated animate__slideInLeft" href="#" onclick="showSection('home')">LA W</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav" 
                aria-controls="navbarNav" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav ms-auto" id="navLinks">
            <li class="nav-item"><a class="nav-link" href="#" onclick="showSection('home')">Inicio</a></li>
            <li class="nav-item"><a class="nav-link" href="#" onclick="showSection('marketplace')">Productos</a></li>
            <li class="nav-item"><a class="nav-link" href="#" onclick="showSection('cartPanel'); checkAccessCart()">Carrito</a></li>
            <li class="nav-item" id="profileLink" style="display:none;"><a class="nav-link" href="#" onclick="openProfile()">Mi Perfil</a></li>
            <li class="nav-item"><a class="nav-link" href="#" onclick="showLoginModal()">Iniciar Sesión</a></li>
            <li class="nav-item"><a class="nav-link" href="#" onclick="showRegisterChoiceModal()">Registrarse</a></li>
          </ul>
        </div>
      </div>
    </nav>
    
    <!-- Sección Home -->
    <section id="home" class="container">
      <div class="home-invite animate__animated animate__fadeInUp">
        <h2 class="color-cycle">¡Bienvenido a LA W - Marketplace!</h2>
        <p class="color-cycle">Somos el espacio ideal donde compradores y vendedores se unen en un entorno seguro y moderno.</p>
        <p class="color-cycle">Nuestro compromiso es ofrecerte tecnología de punta y seguridad en cada transacción.</p>
        <p class="color-cycle">¡Explora nuestros productos y descubre lo que tenemos para ti!</p>
      </div>
      <h2 class="animate__animated animate__bounceIn animate__delay-1s color-cycle">¡Bienvenido a LA W!</h2>
      <p class="extra-info animate__animated animate__fadeInUp animate__delay-2s swing color-cycle">Un marketplace dinámico, seguro y profesional.</p>
      <button class="btn btn-lg btn-warning cta-button animate__animated animate__zoomIn animate__delay-3s heartBeat" onclick="showSection('marketplace')">Explora Productos</button>
      
      <!-- Sección informativa -->
      <section id="info" class="mt-5">
        <div class="row align-items-center info-row">
          <div class="col-md-6">
            <h3 class="animate__animated animate__fadeInLeft color-cycle">Nuestro Objetivo</h3>
            <p class="animate__animated animate__fadeInUp color-cycle">Conectar a compradores y vendedores en un ambiente 100% seguro y transparente.</p>
          </div>
          <div class="col-md-6">
            <img src="https://source.unsplash.com/600x400/?commerce,online" alt="Comercio Online" class="img-fluid animate__animated animate__zoomIn">
          </div>
        </div>
        <div class="row align-items-center info-row">
          <div class="col-md-6 order-md-2">
            <h3 class="animate__animated animate__fadeInRight color-cycle">Seguro para Comprar</h3>
            <p class="animate__animated animate__fadeInUp color-cycle">Comprar en LA W es seguro y confiable. Protegemos tus datos y garantizamos transacciones transparentes.</p>
          </div>
          <div class="col-md-6 order-md-1">
            <img src="https://source.unsplash.com/600x400/?shopping,secure" alt="Compra Segura" class="img-fluid animate__animated animate__zoomIn">
          </div>
        </div>
        <div class="row align-items-center info-row">
          <div class="col-md-6">
            <h3 class="animate__animated animate__fadeInLeft color-cycle">Seguro para Vender</h3>
            <p class="animate__animated animate__fadeInUp color-cycle">Vender en LA W es tan seguro como comprar. Disfruta de un entorno confiable con soporte continuo para potenciar tus ventas.</p>
          </div>
          <div class="col-md-6">
            <img src="https://source.unsplash.com/600x400/?seller,online" alt="Venta Segura" class="img-fluid animate__animated animate__zoomIn">
          </div>
        </div>
      </section>
    </section>
    
    <!-- Sección Marketplace -->
    <section id="marketplace" class="container" style="display:none;">
      <h2 class="animate__animated animate__fadeInDown">Productos</h2>
      <div class="input-group mb-3">
        <input type="text" id="searchInput" class="form-control" placeholder="Buscar productos..." onkeyup="filterProducts()">
      </div>
      <div class="row row-cols-1 row-cols-md-3 g-4" id="productGrid"></div>
    </section>
    
    <!-- Panel de Carrito -->
    <section id="cartPanel" class="container panel" style="display:none;">
      <h2>Carrito de Compras</h2>
      <ul id="cartItems" class="list-group"></ul>
      <p class="mt-3">Total: <span id="total">$0.00</span></p>
      <button class="btn btn-primary" onclick="checkAccessCart()">Ver Carrito y Comprar</button>
    </section>
    
    <!-- Perfil de Comprador -->
    <section id="buyerProfile" class="container panel" style="display:none;">
      <h2>Mi Perfil - Comprador</h2>
      <div class="card">
        <div class="card-body">
          <p><strong>Nombre:</strong> <span id="buyerName"></span></p>
          <p><strong>Email:</strong> <span id="buyerEmail"></span></p>
          <p><strong>Verificación:</strong> <span id="buyerVerified" class="badge bg-success">Verificado</span></p>
        </div>
      </div>
    </section>
    
    <!-- Perfil de Vendedor -->
    <section id="sellerProfile" class="container panel" style="display:none;">
      <h2>Mi Perfil - Vendedor</h2>
      <div class="card">
        <div class="card-body">
          <p><strong>Nombre:</strong> <span id="sellerName"></span></p>
          <p><strong>Email:</strong> <span id="sellerEmail"></span></p>
          <p><strong>Cédula:</strong> <span id="sellerCedula"></span></p>
          <p><strong>Teléfono:</strong> <span id="sellerTelefono"></span></p>
          <p><strong>Ubicación:</strong> <span id="sellerUbicacion"></span></p>
          <p><strong>Método de Pago:</strong> <span id="sellerMetodo" class="badge bg-warning text-dark">Verificado</span></p>
        </div>
      </div>
    </section>
    
    <!-- Modal de Elección de Registro -->
    <div id="registerChoiceModal" class="modal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Elige tu tipo de registro</h5>
            <button type="button" class="btn-close" onclick="closeRegisterChoiceModal()"></button>
          </div>
          <div class="modal-body">
            <p>¿Cómo deseas registrarte? Esto te permitirá acceder al panel adecuado.</p>
            <div class="d-flex justify-content-around">
              <button class="btn btn-primary" onclick="selectRegisterType('buyer')">Como Comprador</button>
              <button class="btn btn-success" onclick="selectRegisterType('seller')">Como Vendedor</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Modal de Registro -->
    <div id="registerModal" class="modal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Registro</h5>
            <button type="button" class="btn-close" onclick="closeRegisterModal()"></button>
          </div>
          <div class="modal-body">
            <div id="vendorInviteMessage" class="alert alert-info animate__animated animate__fadeInDown" style="display:none;">
              <h4 class="alert-heading">¡Bienvenido, Vendedor!</h4>
              <p>Ingresa tu cédula, número de teléfono, nombre completo, ubicación y selecciona tu método de pago (Bancolombia, Nequi o Efectivo). Este será el medio por el cual recibirás tus pagos.</p>
            </div>
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
              <input type="hidden" id="userType" value="buyer">
              <div id="sellerExtraFields" style="display:none;">
                <div class="mb-3"><input type="text" id="cedula" class="form-control" placeholder="Cédula" required></div>
                <div class="mb-3"><input type="text" id="telefono" class="form-control" placeholder="Número de Teléfono" required></div>
                <div class="mb-3"><input type="text" id="ubicacion" class="form-control" placeholder="Ubicación" required></div>
                <div class="mb-3">
                  <select id="cuentaBancaria" class="form-select" required>
                    <option value="">Seleccione su método de pago</option>
                    <option value="Bancolombia">Bancolombia</option>
                    <option value="Nequi">Nequi</option>
                    <option value="Efectivo">Efectivo</option>
                  </select>
                  <small class="form-text text-muted">Este será el medio por el cual recibirás tus pagos.</small>
                </div>
              </div>
              <button type="submit" class="btn btn-success w-100">Enviar Registro</button>
            </form>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Modal de Verificación de Teléfono -->
    <div id="phoneVerificationModal" class="modal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Verificación de Teléfono</h5>
            <button type="button" class="btn-close" onclick="closePhoneVerificationModal()"></button>
          </div>
          <div class="modal-body">
            <p>Se ha enviado un código a tu número de teléfono. Ingresa el código recibido.</p>
            <div class="mb-3">
              <input type="text" id="phoneVerificationInput" class="form-control" placeholder="Código de verificación">
            </div>
            <button class="btn btn-primary w-100" onclick="verifyPhoneCode()">Confirmar Código</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Modal de Verificación de Correo -->
    <div id="emailVerificationModal" class="modal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Verificación de Correo</h5>
            <button type="button" class="btn-close" onclick="closeEmailVerificationModal()"></button>
          </div>
          <div class="modal-body">
            <p>Se ha enviado un código a tu correo. Ingresa el código recibido.</p>
            <div class="mb-3">
              <input type="text" id="emailVerificationInput" class="form-control" placeholder="Código de verificación">
            </div>
            <button class="btn btn-primary w-100" onclick="verifyEmailCode()">Confirmar Código</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Modal de Login -->
    <div id="loginModal" class="modal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content p-3">
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
    
    <!-- Modal de Chat -->
    <div id="chatModal" class="modal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content p-3">
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
    
    <!-- Modal de Imágenes -->
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
    
    <!-- Modal de Detalle del Producto -->
    <div id="productDetailModal" class="modal">
      <div class="modal-dialog modal-lg">
        <div class="modal-content p-3">
          <span class="btn-close float-end" onclick="closeProductDetailModal()"></span>
          <h2 id="detailProductTitle" class="mb-3 animate__animated animate__flipInX"></h2>
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
    
    <!-- Botón flotante para Asistente Virtual -->
    <button class="btn btn-info bot-button animate__animated animate__bounceIn" onclick="openBotModal()">Asistente Virtual</button>
    
    <!-- Modal de Asistente Virtual -->
    <div id="botModal" class="modal" tabindex="-1">
      <div class="modal-dialog modal-sm">
        <div class="modal-content p-3">
          <div class="modal-header">
            <h5 class="modal-title">Asistente Virtual</h5>
            <button type="button" class="btn-close" onclick="closeBotModal()"></button>
          </div>
          <div class="modal-body">
            <div class="mb-2">
              <label for="botTypeSelect" class="form-label">Elige el tipo de asistencia:</label>
              <select id="botTypeSelect" class="form-select">
                <option value="atencion">Atención al Cliente</option>
                <option value="recomendacion">Recomendación de Productos</option>
                <option value="soporte">Soporte en Tiempo Real</option>
                <option value="tutorial">Asistente Virtual (Tutoriales)</option>
              </select>
            </div>
            <div class="mb-2">
              <input type="text" id="botInput" class="form-control" placeholder="Escribe tu pregunta...">
            </div>
            <button class="btn btn-primary w-100" onclick="sendBotMessage()">Enviar</button>
            <div id="botResponse" class="mt-3"></div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Bootstrap Bundle y JavaScript -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>
    <script>
      // Variables globales para verificación
      let pendingUser = null;
      let currentPhoneCode = null;
      let currentEmailCode = null;
      
      function updateNav() {
        if (currentUser && currentUser.verified) {
          document.getElementById('profileLink').style.display = 'block';
        } else {
          document.getElementById('profileLink').style.display = 'none';
        }
      }
      
      function openProfile() {
        if (currentUser.type === 'seller') {
          document.getElementById('sellerName').innerText = currentUser.name;
          document.getElementById('sellerEmail').innerText = currentUser.email;
          let vault = secureVault[currentUser.email] || {};
          document.getElementById('sellerCedula').innerText = vault.cedula || '';
          document.getElementById('sellerTelefono').innerText = vault.telefono || '';
          document.getElementById('sellerUbicacion').innerText = vault.ubicacion || '';
          document.getElementById('sellerMetodo').innerText = vault.cuentaBancaria || 'No especificado';
          showSection('sellerProfile');
        } else {
          document.getElementById('buyerName').innerText = currentUser.name;
          document.getElementById('buyerEmail').innerText = currentUser.email;
          showSection('buyerProfile');
        }
      }
      
      // Funciones para verificación de teléfono y correo en dos pasos
      function openPhoneVerificationModal() {
        var modal = new bootstrap.Modal(document.getElementById('phoneVerificationModal'));
        modal.show();
      }
      function closePhoneVerificationModal() {
        var modal = bootstrap.Modal.getInstance(document.getElementById('phoneVerificationModal'));
        if(modal) modal.hide();
      }
      function verifyPhoneCode() {
        const inputCode = document.getElementById('phoneVerificationInput').value;
        if(inputCode == currentPhoneCode) {
          pendingUser.phone_verified = true;
          alert("Teléfono verificado. Ahora se enviará el código de verificación al correo.");
          closePhoneVerificationModal();
          currentEmailCode = Math.floor(Math.random() * 900000) + 100000;
          alert("Código de verificación de correo (para pruebas): " + currentEmailCode);
          openEmailVerificationModal();
        } else {
          alert("Código de teléfono incorrecto.");
        }
      }
      function openEmailVerificationModal() {
        var modal = new bootstrap.Modal(document.getElementById('emailVerificationModal'));
        modal.show();
      }
      function closeEmailVerificationModal() {
        var modal = bootstrap.Modal.getInstance(document.getElementById('emailVerificationModal'));
        if(modal) modal.hide();
      }
      function verifyEmailCode() {
        const inputCode = document.getElementById('emailVerificationInput').value;
        if(inputCode == currentEmailCode) {
          pendingUser.email_verified = true;
          pendingUser.verified = true;
          users.push(pendingUser);
          localStorage.setItem('users', JSON.stringify(users));
          currentUser = pendingUser;
          localStorage.setItem('currentUser', JSON.stringify(currentUser));
          alert("¡Cuenta verificada exitosamente!");
          closeEmailVerificationModal();
          updateNav();
          if(currentUser.type === 'seller') showSection('sellerPanel');
          else showSection('buyerPanel');
        } else {
          alert("Código de correo incorrecto.");
        }
      }
      
      // Eventos para registro y login
      document.getElementById('registerForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const name = document.getElementById('registerName').value;
        const email = document.getElementById('registerEmail').value;
        const password = document.getElementById('registerPassword').value;
        const userType = document.getElementById('userType').value;
        pendingUser = { id: Date.now(), name, email, password, type: userType, phone_verified: false, email_verified: false, verified: false };
        if(userType === "seller") {
          const cedula = document.getElementById('cedula').value;
          const telefono = document.getElementById('telefono').value;
          const ubicacion = document.getElementById('ubicacion').value;
          const cuentaBancaria = document.getElementById('cuentaBancaria').value;
          secureVault[email] = { cedula, telefono, ubicacion, cuentaBancaria };
          localStorage.setItem('secureVault', JSON.stringify(secureVault));
        }
        currentPhoneCode = Math.floor(Math.random() * 900000) + 100000;
        alert("Se ha enviado un código a tu teléfono (para pruebas): " + currentPhoneCode);
        closeRegisterModal();
        openPhoneVerificationModal();
      });
      
      document.getElementById('loginForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const email = document.getElementById('loginEmail').value;
        const password = document.getElementById('loginPassword').value;
        const user = users.find(u => u.email === email && u.password === password);
        if(user) {
          if(!user.verified) {
            alert("Tu cuenta no ha sido verificada. Por favor, verifica tu número y correo.");
            return;
          }
          currentUser = user;
          localStorage.setItem('currentUser', JSON.stringify(currentUser));
          alert("Bienvenido, " + user.name);
          closeLoginModal();
          updateNav();
          if(user.type === 'seller') showSection('sellerPanel');
          else showSection('buyerPanel');
        } else {
          alert("Credenciales incorrectas.");
        }
      });
      
      // Funciones para el resto de la lógica (productos, carrito, chat, etc.)
      let cart = JSON.parse(localStorage.getItem('cart')) || [];
      let orders = JSON.parse(localStorage.getItem('orders')) || [];
      let sellerProducts = JSON.parse(localStorage.getItem('sellerProducts')) || [];
      let chatSessions = JSON.parse(localStorage.getItem('chatSessions')) || {};
      let currentChatOrderId = null;
      let products = [
        { id: 1, name: 'Producto 1', description: 'Descripción breve del producto 1', price: 10.00, images: ['https://via.placeholder.com/300x200','https://via.placeholder.com/300x200/AAAAAA'], seller: "default@seller.com", reviews: [] },
        { id: 2, name: 'Producto 2', description: 'Descripción breve del producto 2', price: 20.00, images: ['https://via.placeholder.com/300x200'], seller: "default@seller.com", reviews: [] },
        { id: 3, name: 'Producto 3', description: 'Descripción breve del producto 3', price: 15.00, images: ['https://via.placeholder.com/300x200','https://via.placeholder.com/300x200/CCCCCC','https://via.placeholder.com/300x200/EEEEEE'], seller: "default@seller.com", reviews: [] },
        { id: 4, name: 'Producto 4', description: 'Descripción breve del producto 4', price: 8.00, images: ['https://via.placeholder.com/300x200'], seller: "default@seller.com", reviews: [] }
      ];
      let currentProductImages = [];
      let currentImageIndex = 0;
      let currentDetailProduct = null;
      let currentDetailImages = [];
      let currentDetailIndex = 0;
      
      function showSection(sectionId) {
        ['home','marketplace','cartPanel','sellerPanel','buyerPanel','buyerProfile','sellerProfile'].forEach(id => {
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
            <div class="card h-100 animate__animated animate__zoomIn">
              <img src="${product.images[0]}" class="card-img-top" alt="${product.name}">
              <div class="card-body">
                <h5 class="card-title">${product.name}</h5>
                <p class="card-text">${product.description}</p>
                <p class="text-danger fw-bold">$${product.price.toFixed(2)}</p>
              </div>
              <div class="card-footer">
                <button class="btn btn-sm btn-primary" onclick="if(!currentUser){ alert('Debes registrarte para comprar.'); selectRegisterType('buyer'); } else { simulatePayment(${product.id}); }">Comprar</button>
                <button class="btn btn-sm btn-warning" onclick="if(!currentUser){ alert('Debes registrarte para agregar al carrito.'); selectRegisterType('buyer'); } else { addToCart(${product.id}, this); }">Agregar al carrito</button>
                <button class="btn btn-sm btn-info btn-ver" onclick="openProductDetail(${product.id})">Ver Producto</button>
              </div>
            </div>
          `;
          grid.appendChild(div);
        });
      }
      
      function simulatePayment(productId) {
        if(!currentUser){
          alert("Debes registrarte para comprar.");
          selectRegisterType('buyer');
          return;
        }
        const product = products.find(p => p.id === productId);
        let sellerMethod = "No especificado";
        if(secureVault[product.seller] && secureVault[product.seller].cuentaBancaria) {
          sellerMethod = secureVault[product.seller].cuentaBancaria;
        }
        let commission = product.price * 0.02;
        let sellerReceives = product.price - commission;
        let order = {
          orderId: Date.now(),
          product: product,
          buyer: currentUser.email,
          seller: product.seller,
          status: "En proceso",
          buyerNotified: false
        };
        orders.push(order);
        localStorage.setItem('orders', JSON.stringify(orders));
        alert(`Orden generada:
Producto: ${product.name} por $${product.price.toFixed(2)}
Método de pago: ${sellerMethod}
Comisión 2%: $${commission.toFixed(2)}
El vendedor recibirá: $${sellerReceives.toFixed(2)}.
Una vez confirmada la compra se desbloqueará el chat.`);
        renderBuyerOrders();
      }
      
      function addToCart(productId, btnElement) {
        if(!currentUser){
          alert("Debes registrarte para agregar al carrito.");
          selectRegisterType('buyer');
          return;
        }
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
        if(!currentUser){
          alert("Debes registrarte para ver el carrito.");
          selectRegisterType('buyer');
          return;
        }
        if(cart.length === 0) { alert("El carrito está vacío."); return; }
        cart.forEach(product => {
          let order = {
            orderId: Date.now() + Math.floor(Math.random()*1000),
            product: product,
            buyer: currentUser.email,
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
        renderBuyerOrders();
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
                <button class="btn btn-sm btn-info btn-ver" onclick="openProductDetail(${product.id})">Ver Producto</button>
              </div>
            </div>
          `;
          grid.appendChild(div);
        });
      }
      
      function renderBuyerOrders() {
        const container = document.getElementById('buyerOrders');
        container.innerHTML = "";
        orders.filter(o => o.buyer === currentUser.email)
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
        orders.filter(o => o.seller === currentUser.email)
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
                                          <p><strong>Método de Pago:</strong> ${data.cuentaBancaria}</p>` 
                                          : "<p>No se encontraron datos del vendedor.</p>";
        }
      }
      
      function updateSellerEarningsDisplay() {
        const display = document.getElementById('sellerEarningsDisplay');
        display.innerHTML = "<p>Ganancias: $" + sellerEarnings.toFixed(2) + "</p>";
      }
      
      // Funciones para modales de Bootstrap
      function showLoginModal() { 
        var modal = new bootstrap.Modal(document.getElementById('loginModal'));
        modal.show();
      }
      function closeLoginModal() { 
        var modal = bootstrap.Modal.getInstance(document.getElementById('loginModal'));
        if(modal) modal.hide();
      }
      function showRegisterChoiceModal() { 
        var modal = new bootstrap.Modal(document.getElementById('registerChoiceModal'));
        modal.show();
      }
      function closeRegisterChoiceModal() {
        var modal = bootstrap.Modal.getInstance(document.getElementById('registerChoiceModal'));
        if(modal) modal.hide();
      }
      function showRegisterModal() { 
        var modal = new bootstrap.Modal(document.getElementById('registerModal'));
        modal.show();
      }
      function closeRegisterModal() { 
        var modal = bootstrap.Modal.getInstance(document.getElementById('registerModal'));
        if(modal) modal.hide();
      }
      
      function toggleSellerFields(value) { 
        document.getElementById('sellerExtraFields').style.display = (value === "seller") ? "block" : "none"; 
        document.getElementById('vendorInviteMessage').style.display = (value === "seller") ? "block" : "none";
      }
      
      // Eventos para imágenes y productos
      document.getElementById('productImages').addEventListener('change', function(e) {
        const previewContainer = document.getElementById('imagePreview');
        previewContainer.innerHTML = "";
        const files = e.target.files;
        for(let i = 0; i < files.length; i++){
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
        if(files.length > 0){
          let loaded = 0;
          for(let i = 0; i < files.length; i++){
            const reader = new FileReader();
            reader.onload = function(event) {
              imagesArray.push(event.target.result);
              loaded++;
              if(loaded === files.length){
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
        pendingUser = { id: Date.now(), name, email, password, type: userType, phone_verified: false, email_verified: false, verified: false };
        if(userType === "seller") {
          const cedula = document.getElementById('cedula').value;
          const telefono = document.getElementById('telefono').value;
          const ubicacion = document.getElementById('ubicacion').value;
          const cuentaBancaria = document.getElementById('cuentaBancaria').value;
          secureVault[email] = { cedula, telefono, ubicacion, cuentaBancaria };
          localStorage.setItem('secureVault', JSON.stringify(secureVault));
        }
        currentPhoneCode = Math.floor(Math.random() * 900000) + 100000;
        alert("Se ha enviado un código a tu teléfono (para pruebas): " + currentPhoneCode);
        closeRegisterModal();
        openPhoneVerificationModal();
      });
      
      document.getElementById('loginForm').addEventListener('submit', function(e) {
        e.preventDefault();
        const email = document.getElementById('loginEmail').value;
        const password = document.getElementById('loginPassword').value;
        const user = users.find(u => u.email === email && u.password === password);
        if(user) {
          if(!user.verified) {
            alert("Tu cuenta no ha sido verificada. Por favor, verifica tu número y correo.");
            return;
          }
          currentUser = user;
          localStorage.setItem('currentUser', JSON.stringify(currentUser));
          alert("Bienvenido, " + user.name);
          closeLoginModal();
          updateNav();
          if(user.type === 'seller') showSection('sellerPanel');
          else showSection('buyerPanel');
        } else {
          alert("Credenciales incorrectas.");
        }
      });
      
      // Funciones para chat y detalle de producto
      let currentChatOrderId = null;
      
      function openChatModal(orderId) {
        if(!currentUser) {
          alert("Debes iniciar sesión para acceder al chat.");
          selectRegisterType('buyer');
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
      
      let currentDetailProduct = null;
      let currentDetailImages = [];
      let currentDetailIndex = 0;
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
          div.className = "mb-2 p-2 border rounded";
          div.innerHTML = `<strong>${rev.user}</strong> - ${rev.rating} estrellas<br>${rev.comment}`;
          container.appendChild(div);
        });
      }
      
      for(let i = 1; i <= 50; i++){
        window['extraLogic' + i] = function(){ console.log("Ejecutando lógica extra " + i); }
      }
      function extraAnimationEffect(effectName) { console.log("Ejecutando efecto de animación extra: " + effectName); }
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

# Ruta para simular envío de código de verificación (para pruebas)
@app.route('/send-verification')
def send_verification():
    email = request.args.get('email')
    if not email:
        return "Falta el parámetro email.", 400
    code = random.randint(100000, 999999)
    return str(code)

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

<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Style The W - Demo Multicatalogos</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <!-- Fuente Montserrat -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&display=swap" rel="stylesheet">
  <style>
    /* Estilos Globales */
    * { box-sizing: border-box; }
    body, html { margin: 0; padding: 0; font-family: 'Montserrat', sans-serif; background: #fff; color: #333; transition: background 0.3s, color 0.3s; }
    a { color: #333; text-decoration: none; }
    
    /* Modo Oscuro */
    .dark-mode { background: #121212; color: #eee; }
    .dark-mode header { background: #333; }
    .dark-mode nav a { color: #eee; }
    .dark-mode .banner { filter: brightness(0.8); }
    .dark-mode .info-text { background: linear-gradient(135deg, #2e2e2e, #555); border-left-color: #FFA500; box-shadow: 0 4px 8px rgba(255,255,255,0.1); }
    
    /* Botón Modo Oscuro */
    #modeToggle {
      position: fixed;
      top: 15px;
      right: 15px;
      background: #333;
      color: #fff;
      border: none;
      padding: 8px 12px;
      border-radius: 5px;
      cursor: pointer;
      z-index: 1001;
      transition: background 0.3s;
    }
    #modeToggle:hover { background: #555; }
    
    /* Header y Banner */
    header { background: #f9f9f9; padding: 15px; text-align: center; }
    nav a { margin: 0 10px; font-weight: 700; }
    nav a:hover { color: #e63946; }
    .banner { height: 70vh; background: url('https://via.placeholder.com/1600x900') center/cover no-repeat; display: flex; align-items: center; justify-content: center; color: #fff; position: relative; }
    .banner::after { content: ''; position: absolute; top:0; left:0; right:0; bottom:0; background: rgba(0,0,0,0.4); }
    .banner-content { position: relative; text-align: center; z-index: 1; }
    .banner-content h1 { font-size: 3em; margin: 0; text-shadow: 2px 2px 10px #FF4500; }
    .banner-content p { font-size: 1.5em; margin-top: 10px; }
    
    /* Secciones */
    section { padding: 40px 20px; }
    .section-title { text-align: center; font-size: 2em; margin-bottom: 20px; }
    
    /* Dropdown de Catálogos */
    .catalog-select { text-align: center; margin-bottom: 20px; }
    .catalog-select select { padding: 8px; font-size: 1em; }
    
    /* Catálogo de Productos */
    .catalogo { display: grid; grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); gap: 20px; padding: 0 20px; }
    .producto { border: 1px solid #ddd; padding: 15px; position: relative; }
    .producto h3 { margin: 10px 0; font-size: 1.2em; }
    .producto p { margin: 5px 0; font-weight: bold; }
    .status-label { position: absolute; top: 10px; left: 10px; background: rgba(255,69,0,0.9); color: #fff; padding: 5px 10px; border-radius: 3px; font-size: 0.9em; }
    .status-label.sold { background: rgba(128,128,128,0.9); }
    .producto img { width: 100%; max-width: 150px; margin: 5px; }
    
    /* Slider de Imágenes */
    .slider { position: relative; overflow: hidden; height: 300px; margin-bottom: 10px; }
    .slides { display: flex; transition: transform 0.5s ease-in-out; }
    .slider img { width: 100%; flex-shrink: 0; object-fit: cover; }
    .prev, .next { position: absolute; top: 50%; transform: translateY(-50%); background: rgba(0,0,0,0.5); color: #fff; border: none; padding: 10px; cursor: pointer; border-radius: 50%; }
    .prev { left: 10px; } .next { right: 10px; }
    
    /* Sección de Contacto e Información */
    .contact-info { text-align: center; font-size: 1.1em; margin-bottom: 20px; }
    .info-text { max-width: 800px; margin: 40px auto; padding: 30px; background: linear-gradient(135deg, #f5f7fa, #c3cfe2); border-left: 5px solid #FF4500; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); font-style: italic; }
    .info-text p { margin-bottom: 20px; }
    
    /* Modal para Admin Login */
    .modal { display: none; position: fixed; top:0; left:0; right:0; bottom:0; background: rgba(0,0,0,0.5); align-items: center; justify-content: center; z-index: 2000; }
    .modal-content { background: #fff; padding: 20px; border-radius: 5px; max-width: 400px; width: 90%; }
    
    /* Panel de Administración */
    #admin-panel { max-width: 600px; margin: 40px auto; padding: 20px; border: 1px solid #ddd; border-radius: 8px; box-shadow: 0 4px 8px rgba(0,0,0,0.1); display: none; }
    #admin-panel form { display: block; }
    #admin-panel label { display: block; margin-bottom: 10px; }
    #admin-panel input, #admin-panel select { width: 100%; padding: 8px; margin-bottom: 20px; }
    #admin-panel button { padding: 10px 20px; }
  </style>
</head>
<body>
  <!-- Botón de Modo Oscuro -->
  <button id="modeToggle">Modo Oscuro</button>
  
  <!-- Limpiar flag de admin al cargar -->
  <script>localStorage.removeItem("isAdmin");</script>
  
  <!-- Header -->
  <header>
    <nav>
      <a href="#">Inicio</a>
      <a href="#catalogo">Catálogo</a>
      <a href="#contacto">Contacto</a>
      <a href="#" class="admin-link" onclick="showAdminLogin()">Admin</a>
    </nav>
  </header>
  
  <!-- Banner -->
  <div class="banner">
    <div class="banner-content">
      <h1>Style The W</h1>
      <p>Viste con estilo, vive con pasión</p>
    </div>
  </div>
  
  <!-- Dropdown para seleccionar catálogo -->
  <div class="catalog-select">
    <select id="catalogSelect" onchange="renderCatalog()"></select>
  </div>
  
  <!-- Catálogo de Productos -->
  <section id="catalogo">
    <h2 class="section-title">Catálogo de Productos</h2>
    <div class="catalogo" id="catalogo-container">
      <!-- Se cargarán los productos del catálogo seleccionado -->
    </div>
  </section>
  
  <!-- Sección de Contacto -->
  <section id="contacto">
    <h2 class="section-title">Contacto</h2>
    <div class="contact-info">
      <p>Para más información, contáctanos vía WhatsApp o llámanos: <strong>32251373388</strong> y <strong>321243232</strong>.</p>
    </div>
  </section>
  
  <!-- Sección de Información -->
  <section class="info-text">
    <p>
      En 🔥Style the W🔥 encontrarás ropa👗🩳 para todos los estilos a precios💸 increíbles. Visítanos en Tadó, Chocó, y disfruta de 📦 contra entrega🚚 para mayor seguridad. ¡Vive la moda con nosotros!✨
    </p>
    <p>
      Nuestra tienda es el punto de encuentro para quienes buscan tendencias, calidad y autenticidad. Cada prenda ha sido seleccionada con esmero para ofrecerte una experiencia única en moda. Te invitamos a explorar nuestras colecciones, descubrir nuevos looks y ser parte de una comunidad que celebra la pasión por el estilo.
    </p>
    <p>
      Además, en cada producto podrás ver en la imagen si el artículo está "EN VENTA" o si ya está "VENDIDO", brindándote la seguridad de que compras con total información. ¡Únete y transforma tu manera de vestir!
    </p>
  </section>
  
  <!-- Modal de Admin Login -->
  <div id="adminLoginModal" class="modal">
    <div class="modal-content">
      <h2>Admin Login</h2>
      <form id="adminLoginForm">
        <label for="username">Usuario:</label>
        <input type="text" id="username" required>
        <label for="password">Contraseña:</label>
        <input type="password" id="password" required>
        <button type="submit">Ingresar</button>
        <button type="button" onclick="hideAdminLogin()">Cancelar</button>
      </form>
    </div>
  </div>
  
  <!-- Panel de Administración (oculto por defecto) -->
  <section id="admin-panel">
    <h2>Panel de Administración</h2>
    <!-- Dropdown para seleccionar catálogo en el panel admin -->
    <label>Seleccionar Catálogo:
      <select id="admin-catalog-select"></select>
    </label>
    <button type="button" onclick="createNewCatalog()">Crear Nuevo Catálogo</button>
    <br><br>
    <form id="adminProductForm">
      <label>Nombre del Producto:
        <input type="text" id="admin-product-name" required>
      </label>
      <label>Precio:
        <input type="number" id="admin-product-price" step="0.01" required>
      </label>
      <label>Status:
        <select id="admin-product-status">
          <option value="EN VENTA">EN VENTA</option>
          <option value="VENDIDO">VENDIDO</option>
        </select>
      </label>
      <label>Subir nuevas imágenes:
        <input type="file" id="admin-product-images" multiple>
      </label>
      <button type="button" onclick="addOrUpdateProduct()">Guardar Producto</button>
    </form>
    <br>
    <button onclick="logoutAdmin()">Cerrar Sesión</button>
  </section>
  
  <script>
    // Modo Oscuro: Alternar al hacer clic en el botón
    document.getElementById("modeToggle").addEventListener("click", function() {
      document.body.classList.toggle("dark-mode");
      this.textContent = document.body.classList.contains("dark-mode") ? "Modo Claro" : "Modo Oscuro";
    });
    
    // Inicializar catálogos en localStorage si no existen
    if (!localStorage.getItem("catalogs")) {
      const defaultCatalog = { id: 1, name: "General", products: [] };
      localStorage.setItem("catalogs", JSON.stringify([defaultCatalog]));
    }
    
    // Función para llenar el dropdown de catálogos en la vista pública
    function populateCatalogSelect() {
      const catalogs = JSON.parse(localStorage.getItem("catalogs"));
      const select = document.getElementById("catalogSelect");
      select.innerHTML = "";
      catalogs.forEach(catalog => {
        const option = document.createElement("option");
        option.value = catalog.id;
        option.textContent = catalog.name;
        select.appendChild(option);
      });
    }
    populateCatalogSelect();
    
    // Renderizar catálogo seleccionado en la vista pública
    function renderCatalog() {
      const catalogs = JSON.parse(localStorage.getItem("catalogs"));
      const select = document.getElementById("catalogSelect");
      const selectedCatalogId = parseInt(select.value) || catalogs[0].id;
      const catalog = catalogs.find(c => c.id === selectedCatalogId);
      const container = document.getElementById("catalogo-container");
      container.innerHTML = "";
      if (catalog && catalog.products.length > 0) {
        catalog.products.forEach((product, index) => {
          const prodDiv = document.createElement("div");
          prodDiv.className = "producto";
          
          const statusSpan = document.createElement("span");
          statusSpan.className = "status-label";
          if (product.status === "VENDIDO") {
            statusSpan.classList.add("sold");
          }
          statusSpan.textContent = product.status;
          prodDiv.appendChild(statusSpan);
          
          // Si existen imágenes, crear slider simple
          if (product.images.length > 0) {
            const sliderDiv = document.createElement("div");
            sliderDiv.className = "slider";
            const slidesDiv = document.createElement("div");
            slidesDiv.className = "slides";
            product.images.forEach(imgData => {
              const img = document.createElement("img");
              img.src = imgData;
              img.alt = "Imagen del producto";
              slidesDiv.appendChild(img);
            });
            sliderDiv.appendChild(slidesDiv);
            const prevBtn = document.createElement("button");
            prevBtn.className = "prev";
            prevBtn.innerHTML = "&#10094;";
            sliderDiv.appendChild(prevBtn);
            const nextBtn = document.createElement("button");
            nextBtn.className = "next";
            nextBtn.innerHTML = "&#10095;";
            sliderDiv.appendChild(nextBtn);
            prodDiv.appendChild(sliderDiv);
            
            let currentIndex = 0;
            const total = product.images.length;
            prevBtn.addEventListener("click", () => {
              currentIndex = (currentIndex === 0) ? total - 1 : currentIndex - 1;
              slidesDiv.style.transform = `translateX(-${currentIndex * 100}%)`;
            });
            nextBtn.addEventListener("click", () => {
              currentIndex = (currentIndex + 1) % total;
              slidesDiv.style.transform = `translateX(-${currentIndex * 100}%)`;
            });
          }
          
          const h3 = document.createElement("h3");
          h3.textContent = product.name;
          prodDiv.appendChild(h3);
          
          const pPrice = document.createElement("p");
          pPrice.textContent = "$ " + product.price;
          prodDiv.appendChild(pPrice);
          
          container.appendChild(prodDiv);
        });
      } else {
        container.innerHTML = "<p style='text-align:center;'>No hay productos en este catálogo.</p>";
      }
    }
    renderCatalog();
    
    // Funciones para Admin Login Modal
    function showAdminLogin() {
      document.getElementById("adminLoginModal").style.display = "flex";
    }
    function hideAdminLogin() {
      document.getElementById("adminLoginModal").style.display = "none";
    }
    document.getElementById("adminLoginForm").addEventListener("submit", function(e) {
      e.preventDefault();
      const username = document.getElementById("username").value;
      const password = document.getElementById("password").value;
      if (username === "admin" && password === "admin123") {
        localStorage.setItem("isAdmin", "true");
        hideAdminLogin();
        showAdminPanel();
      } else {
        alert("Credenciales incorrectas");
      }
    });
    
    function showAdminPanel() {
      document.getElementById("admin-panel").style.display = "block";
      populateAdminCatalogSelect();
      // Limpiar campos del formulario para agregar un nuevo producto
      document.getElementById("admin-product-name").value = "";
      document.getElementById("admin-product-price").value = "";
      document.getElementById("admin-product-status").value = "EN VENTA";
    }
    
    function logoutAdmin() {
      localStorage.removeItem("isAdmin");
      document.getElementById("admin-panel").style.display = "none";
    }
    
    // Llenar el dropdown de catálogos en el panel de administración
    function populateAdminCatalogSelect() {
      const catalogs = JSON.parse(localStorage.getItem("catalogs"));
      const select = document.getElementById("admin-catalog-select");
      select.innerHTML = "";
      catalogs.forEach(catalog => {
        const option = document.createElement("option");
        option.value = catalog.id;
        option.textContent = catalog.name;
        select.appendChild(option);
      });
    }
    
    // Función para crear un nuevo catálogo desde el panel admin
    function createNewCatalog() {
      const catalogName = prompt("Ingrese el nombre del nuevo catálogo:");
      if (catalogName) {
        let catalogs = JSON.parse(localStorage.getItem("catalogs"));
        let newId = catalogs.reduce((max, cat) => Math.max(max, cat.id), 0) + 1;
        catalogs.push({ id: newId, name: catalogName, products: [] });
        localStorage.setItem("catalogs", JSON.stringify(catalogs));
        populateCatalogSelect();
        populateAdminCatalogSelect();
        alert("Nuevo catálogo creado");
      }
    }
    
    // Función para agregar o actualizar un producto en el catálogo seleccionado
    function addOrUpdateProduct() {
      const newName = document.getElementById("admin-product-name").value;
      const newPrice = parseFloat(document.getElementById("admin-product-price").value);
      const newStatus = document.getElementById("admin-product-status").value;
      const catalogId = parseInt(document.getElementById("admin-catalog-select").value);
      const files = document.getElementById("admin-product-images").files;
      
      let catalogs = JSON.parse(localStorage.getItem("catalogs"));
      let catalog = catalogs.find(c => c.id === catalogId);
      if (!catalog) { alert("Catálogo no encontrado"); return; }
      
      let newProduct = { name: newName, price: newPrice, status: newStatus, images: [] };
      
      if (files.length > 0) {
        const promises = [];
        for (let i = 0; i < files.length; i++) {
          promises.push(new Promise((resolve, reject) => {
            const reader = new FileReader();
            reader.onload = function(e) {
              resolve(e.target.result);
            };
            reader.onerror = function() {
              reject("Error leyendo archivo");
            };
            reader.readAsDataURL(files[i]);
          }));
        }
        Promise.all(promises).then(results => {
          newProduct.images = results;
          catalog.products.push(newProduct);
          localStorage.setItem("catalogs", JSON.stringify(catalogs));
          renderCatalog();
          alert("Producto agregado");
        }).catch(err => alert(err));
      } else {
        catalog.products.push(newProduct);
        localStorage.setItem("catalogs", JSON.stringify(catalogs));
        renderCatalog();
        alert("Producto agregado");
      }
    }
    
    // Mostrar el panel de administración si el flag de admin está activo
    if (localStorage.getItem("isAdmin") === "true") {
      showAdminPanel();
    }
  </script>
</body>
</html>

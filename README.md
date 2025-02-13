<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Style The W</title>
  <!-- Fuente moderna -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&display=swap" rel="stylesheet">
  <style>
    /* Reset y estilos base */
    * { box-sizing: border-box; }
    body, html {
      margin: 0;
      padding: 0;
      font-family: 'Montserrat', sans-serif;
      scroll-behavior: smooth;
      background: #fff;
      color: #333;
    }
    a { color: #333; }
    
    /* Header y navegación flotante */
    header {
      position: fixed;
      top: 0;
      width: 100%;
      background: rgba(255,255,255,0.95);
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
      z-index: 1000;
      transition: background 0.3s;
    }
    header nav {
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 15px 0;
    }
    header nav a {
      margin: 0 20px;
      text-decoration: none;
      font-weight: 700;
      transition: color 0.3s;
    }
    header nav a:hover { color: #e63946; }
    
    /* Botón de modo oscuro/claro */
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
    
    /* Banner con efecto parallax y overlay */
    .banner {
      height: 70vh;
      background: url('https://via.placeholder.com/1600x900') center/cover no-repeat;
      display: flex;
      align-items: center;
      justify-content: center;
      position: relative;
      margin-top: 70px;
    }
    .banner::after {
      content: '';
      position: absolute;
      top:0; left:0; right:0; bottom:0;
      background: rgba(0,0,0,0.4);
    }
    .banner-content {
      position: relative;
      text-align: center;
      z-index: 1;
    }
    .banner-content h1 { 
      font-size: 3em; 
      margin: 0;
      text-shadow: 2px 2px 10px #FF4500;
    }
    .banner-content p { font-size: 1.5em; margin: 10px 0 0; }
    
    /* Secciones */
    section {
      padding: 60px 20px;
    }
    .section-title {
      text-align: center;
      margin-bottom: 40px;
      font-size: 2em;
    }
    
    /* Catálogo de Productos */
    .catalogo {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
      gap: 20px;
      padding: 0 20px;
    }
    
    /* Tarjeta de producto con slider */
    .producto {
      background: #fff;
      border: 1px solid #ddd;
      border-radius: 8px;
      overflow: hidden;
      transition: transform 0.3s, box-shadow 0.3s;
      position: relative;
    }
    .producto:hover {
      transform: scale(1.03);
      box-shadow: 0 4px 10px rgba(0,0,0,0.15);
    }
    
    /* Etiqueta de estado (venta/sold) */
    .status-label {
      position: absolute;
      top: 10px;
      left: 10px;
      background: rgba(255, 69, 0, 0.9); /* Naranja neón */
      color: #fff;
      padding: 5px 10px;
      font-size: 0.9em;
      border-radius: 3px;
      z-index: 2;
    }
    .status-label.sold {
      background: rgba(128, 128, 128, 0.9);
    }
    
    /* Slider de imágenes */
    .slider {
      position: relative;
      overflow: hidden;
      height: 500px;
    }
    .slides {
      display: flex;
      transition: transform 0.5s ease-in-out;
    }
    .slider img {
      width: 100%;
      flex-shrink: 0;
      object-fit: cover;
      loading: lazy;
    }
    .prev, .next {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      background: rgba(0,0,0,0.5);
      color: #fff;
      border: none;
      padding: 10px;
      cursor: pointer;
      border-radius: 50%;
    }
    .prev { left: 10px; }
    .next { right: 10px; }
    
    /* Información del producto */
    .producto .info {
      padding: 15px;
      text-align: center;
    }
    .producto .info h3 {
      margin: 10px 0;
      font-size: 1.2em;
    }
    .producto .info p {
      margin: 5px 0;
      font-size: 1em;
      font-weight: bold;
    }
    .whatsapp-btn {
      display: inline-block;
      background: #25d366;
      color: #fff;
      padding: 10px 20px;
      margin: 10px 0;
      border-radius: 5px;
      text-decoration: none;
      transition: background 0.3s;
    }
    .whatsapp-btn:hover { background: #1da851; }
    
    /* Botón Volver Arriba */
    #backToTop {
      position: fixed;
      bottom: 30px;
      right: 30px;
      background: #333;
      color: #fff;
      border: none;
      padding: 10px 15px;
      border-radius: 5px;
      cursor: pointer;
      display: none;
      z-index: 1000;
    }
    
    /* Texto informativo largo con diseño estético */
    .info-text {
      max-width: 800px;
      margin: 40px auto;
      padding: 30px;
      line-height: 1.8;
      text-align: justify;
      font-size: 1.2em;
      background: linear-gradient(135deg, #f5f7fa, #c3cfe2);
      border-left: 5px solid #FF4500;
      border-radius: 8px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
      font-style: italic;
    }
    .info-text p {
      margin-bottom: 20px;
    }
    
    /* Sección de Contacto con dos números */
    .contact-info {
      text-align: center;
      font-size: 1.1em;
      margin-bottom: 20px;
    }
    
    /* Modo Oscuro */
    .dark-mode {
      background: #121212;
      color: #eee;
    }
    .dark-mode header { background: rgba(18, 18, 18, 0.95); }
    .dark-mode header nav a { color: #eee; }
    .dark-mode .producto { background: #1e1e1e; border-color: #333; }
  </style>
</head>
<body>
  <!-- Botón de modo oscuro/claro -->
  <button id="modeToggle">Modo Oscuro</button>
  
  <!-- Header con navegación -->
  <header id="header">
    <nav>
      <a href="#home">Inicio</a>
      <a href="#catalogo">Catálogo</a>
      <a href="#contacto">Contacto</a>
    </nav>
  </header>
  
  <!-- Banner principal -->
  <div class="banner" id="home">
    <div class="banner-content">
      <h1>Style The W</h1>
      <p>Viste con estilo, vive con pasión</p>
    </div>
  </div>
  
  <!-- Catálogo de Productos -->
  <section id="catalogo">
    <h2 class="section-title">Catálogo de Productos</h2>
    <div class="catalogo">
      <!-- Producto 1 -->
      <div class="producto">
        <span class="status-label">EN VENTA</span>
        <div class="slider">
          <div class="slides">
            <img src="https://via.placeholder.com/400x500?text=Imagen+1" alt="Imagen 1">
            <img src="https://via.placeholder.com/400x500?text=Imagen+2" alt="Imagen 2">
            <img src="https://via.placeholder.com/400x500?text=Imagen+3" alt="Imagen 3">
            <img src="https://via.placeholder.com/400x500?text=Imagen+4" alt="Imagen 4">
            <img src="https://via.placeholder.com/400x500?text=Imagen+5" alt="Imagen 5">
            <img src="https://via.placeholder.com/400x500?text=Imagen+6" alt="Imagen 6">
            <img src="https://via.placeholder.com/400x500?text=Imagen+7" alt="Imagen 7">
          </div>
          <button class="prev">&#10094;</button>
          <button class="next">&#10095;</button>
        </div>
        <div class="info">
          <h3>Camiseta Blanca</h3>
          <p>$20.00</p>
          <a href="https://wa.me/1234567890?text=Interesado%20en%20la%20Camiseta%20Blanca" class="whatsapp-btn" target="_blank">WhatsApp</a>
        </div>
      </div>
      
      <!-- Producto 2 -->
      <div class="producto">
        <span class="status-label sold">VENDIDO</span>
        <div class="slider">
          <div class="slides">
            <img src="https://via.placeholder.com/400x500?text=Imagen+1" alt="Imagen 1">
            <img src="https://via.placeholder.com/400x500?text=Imagen+2" alt="Imagen 2">
            <img src="https://via.placeholder.com/400x500?text=Imagen+3" alt="Imagen 3">
            <img src="https://via.placeholder.com/400x500?text=Imagen+4" alt="Imagen 4">
            <img src="https://via.placeholder.com/400x500?text=Imagen+5" alt="Imagen 5">
            <img src="https://via.placeholder.com/400x500?text=Imagen+6" alt="Imagen 6">
          </div>
          <button class="prev">&#10094;</button>
          <button class="next">&#10095;</button>
        </div>
        <div class="info">
          <h3>Jeans Clásicos</h3>
          <p>$35.00</p>
          <a href="https://wa.me/1234567890?text=Interesado%20en%20los%20Jeans%20Cl%C3%A1sicos" class="whatsapp-btn" target="_blank">WhatsApp</a>
        </div>
      </div>
    </div>
  </section>
  
  <!-- Sección de Contacto -->
  <section id="contacto">
    <h2 class="section-title">Contacto</h2>
    <div class="contact-info">
      <p>Para más información, contáctanos vía WhatsApp o llámanos: <strong>32251373388</strong> y <strong>321243232</strong>.</p>
    </div>
    <div style="text-align: center; margin-top: 20px;">
      <a href="https://wa.me/1234567890?text=Quiero%20más%20informaci%C3%B3n" class="whatsapp-btn" target="_blank">Enviar WhatsApp</a>
    </div>
  </section>
  
  <!-- Texto informativo largo con diseño estético -->
  <section id="info" class="info-text">
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
  
  <!-- Botón Volver Arriba -->
  <button id="backToTop">Arriba</button>
  
  <script>
    // Alternar modo oscuro/claro
    const modeToggle = document.getElementById('modeToggle');
    modeToggle.addEventListener('click', () => {
      document.body.classList.toggle('dark-mode');
      modeToggle.textContent = document.body.classList.contains('dark-mode') ? 'Modo Claro' : 'Modo Oscuro';
    });
    
    // Mostrar botón "Volver Arriba"
    const backToTop = document.getElementById('backToTop');
    window.addEventListener('scroll', () => {
      backToTop.style.display = window.pageYOffset > 300 ? 'block' : 'none';
    });
    
    backToTop.addEventListener('click', () => {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    });
    
    // Funcionalidad del slider para cada producto
    document.querySelectorAll('.slider').forEach(slider => {
      const slides = slider.querySelector('.slides');
      const images = slides.querySelectorAll('img');
      let currentIndex = 0;
      const total = images.length;
      
      slider.querySelector('.prev').addEventListener('click', () => {
        currentIndex = (currentIndex === 0) ? total - 1 : currentIndex - 1;
        slides.style.transform = `translateX(-${currentIndex * 100}%)`;
      });
      
      slider.querySelector('.next').addEventListener('click', () => {
        currentIndex = (currentIndex + 1) % total;
        slides.style.transform = `translateX(-${currentIndex * 100}%)`;
      });
    });
  </script>
</body>
</html>

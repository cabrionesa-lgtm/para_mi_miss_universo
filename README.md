<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Para Ti ✨</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }
    body {
      background-color: #05050d;
      color: #ffffff;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      overflow: hidden;
      width: 100vw;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
    }
    #canvas-container {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 1;
    }
    canvas {
      width: 100%;
      height: 100%;
      display: block;
    }
    .ui-overlay {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: 10;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      padding: 20px;
      pointer-events: none;
    }
    .interactive {
      pointer-events: auto;
    }
    
    /* Screen 1: Loading / Start Button */
    #start-screen {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(5, 5, 13, 0.95);
      z-index: 20;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      transition: opacity 1s ease;
    }
    .card {
      background: rgba(255, 255, 255, 0.05);
      border: 1px solid rgba(255, 255, 255, 0.1);
      backdrop-filter: blur(10px);
      padding: 40px;
      border-radius: 20px;
      text-align: center;
      max-width: 400px;
      width: 90%;
      box-shadow: 0 10px 30px rgba(0,0,0,0.5);
    }
    .card h1 {
      font-size: 1.8rem;
      margin-bottom: 20px;
      color: #ff758c;
    }
    .progress-bar-bg {
      width: 100%;
      height: 10px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 5px;
      overflow: hidden;
      margin-bottom: 15px;
    }
    .progress-bar-fill {
      height: 100%;
      width: 0%;
      background: linear-gradient(90deg, #ff758c, #ff7eb3);
      transition: width 0.3s ease;
    }
    .status-text {
      font-size: 0.9rem;
      color: #aaa;
      margin-bottom: 25px;
    }
    .btn-start {
      background: linear-gradient(135deg, #ff758c 0%, #ff7eb3 100%);
      color: white;
      border: none;
      padding: 14px 32px;
      font-size: 1.1rem;
      font-weight: bold;
      border-radius: 30px;
      cursor: pointer;
      box-shadow: 0 0 15px rgba(255, 117, 140, 0.4);
      transition: all 0.3s ease;
      display: none;
    }
    .btn-start:hover {
      transform: scale(1.05);
      box-shadow: 0 0 25px rgba(255, 117, 140, 0.7);
    }

    /* Subtitles / Messages */
    #subtitle-box {
      margin-top: 30px;
      background: rgba(0, 0, 0, 0.6);
      padding: 10px 20px;
      border-radius: 15px;
      border: 1px solid rgba(255, 255, 255, 0.1);
      font-size: 1rem;
      color: #ffc2d1;
      text-align: center;
      opacity: 0;
      transition: opacity 0.5s ease;
    }
    
    /* Control Panel bottom */
    .controls {
      margin-bottom: 20px;
      display: flex;
      gap: 15px;
    }
    .control-btn {
      background: rgba(255, 255, 255, 0.1);
      border: 1px solid rgba(255, 255, 255, 0.2);
      color: white;
      padding: 10px 18px;
      border-radius: 20px;
      backdrop-filter: blur(5px);
      cursor: pointer;
      font-size: 0.85rem;
      transition: all 0.3s ease;
    }
    .control-btn:hover {
      background: rgba(255, 117, 140, 0.3);
      border-color: #ff758c;
    }
  </style>
  <!-- Three.js Library -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

  <!-- Start Screen -->
  <div id="start-screen">
    <div class="card interactive">
      <h1>Preparando todo para ti</h1>
      <div class="progress-bar-bg">
        <div class="progress-bar-fill" id="progress"></div>
      </div>
      <div class="status-text" id="status">Cargando universo...</div>
      <button class="btn-start" id="btn-start">✨ Comenzar ✨</button>
    </div>
  </div>

  <!-- Main Canvas -->
  <div id="canvas-container"></div>

  <!-- UI Overlay -->
  <div class="ui-overlay">
    <div id="subtitle-box">✨ Toca el universo para interactuar</div>
    <div class="controls interactive">
      <button class="control-btn" id="btn-hearts">💖 Enviar Amor</button>
    </div>
  </div>

  <script>
    // --- Configuración e Inicialización ---
    const container = document.getElementById('canvas-container');
    const startScreen = document.getElementById('start-screen');
    const progressBar = document.getElementById('progress');
    const statusText = document.getElementById('status');
    const btnStart = document.getElementById('btn-start');
    const subtitleBox = document.getElementById('subtitle-box');
    const btnHearts = document.getElementById('btn-hearts');
    
    let scene, camera, renderer, particleSystem;
    let particleCount = 12000;

    const textos = ["MI NIÑA", "PRECIOSA", "GRACIAS", "POR EXISTIR"];
    const frasesSubtitulo = [
      "Así como Saturno necesita sus anillos para no perderse... 🪐",
      "Yo solo te necesito a ti para no apagarme ✨",
      "Eres el centro de mi universo infinito 💖",
      "Las estrellas brillan más cuando pienso en ti... 💕"
    ];

    // Cargar simulación
    let loadProgress = 0;
    const interval = setInterval(() => {
      loadProgress += 20;
      progressBar.style.width = loadProgress + '%';
      if (loadProgress === 40) statusText.innerText = "Inicializando efectos...";
      if (loadProgress === 80) statusText.innerText = "Creando galaxias...";
      if (loadProgress >= 100) {
        clearInterval(interval);
        statusText.innerText = "¡Todo listo mi amor!";
        btnStart.style.display = 'inline-block';
      }
    }, 300);

    btnStart.addEventListener('click', () => {
      startScreen.style.opacity = '0';
      setTimeout(() => {
        startScreen.style.display = 'none';
        subtitleBox.style.opacity = '1';
        iniciarSecuencia();
      }, 1000);
    });

    // --- Scene Setup ---
    function initThreeJS() {
      scene = new THREE.Scene();
      camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
      camera.position.z = 150;

      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setPixelRatio(window.devicePixelRatio);
      container.appendChild(renderer.domElement);

      // Partículas
      const geometry = new THREE.BufferGeometry();
      const positions = new Float32Array(particleCount * 3);
      const colors = new Float32Array(particleCount * 3);

      const color1 = new THREE.Color("#ff758c");
      const color2 = new THREE.Color("#ff7eb3");
      const colorWhite = new THREE.Color("#ffffff");

      // Inicialmente posiciones aleatorias dispersas (nube suave)
      for (let i = 0; i < particleCount; i++) {
        positions[i * 3] = (Math.random() - 0.5) * 300;
        positions[i * 3 + 1] = (Math.random() - 0.5) * 300;
        positions[i * 3 + 2] = (Math.random() - 0.5) * 300;

        let mixColor = Math.random() > 0.5 ? color1 : (Math.random() > 0.2 ? color2 : colorWhite);
        colors[i * 3] = mixColor.r;
        colors[i * 3 + 1] = mixColor.g;
        colors[i * 3 + 2] = mixColor.b;
      }

      geometry.setAttribute('position', new THREE.BufferAttribute(positions, 3));
      geometry.setAttribute('color', new THREE.BufferAttribute(colors, 3));

      // Texture para partículas suaves
      const canvasTex = document.createElement('canvas');
      canvasTex.width = 16; canvasTex.height = 16;
      const ctxTex = canvasTex.getContext('2d');
      const grad = ctxTex.createRadialGradient(8,8,0, 8,8,8);
      grad.addColorStop(0, 'rgba(255,255,255,1)');
      grad.addColorStop(1, 'rgba(255,255,255,0)');
      ctxTex.fillStyle = grad;
      ctxTex.fillRect(0,0,16,16);
      const pTexture = new THREE.CanvasTexture(canvasTex);

      const material = new THREE.PointsMaterial({
        size: 1.8,
        vertexColors: true,
        map: pTexture,
        transparent: true,
        blending: THREE.AdditiveBlending,
        depthWrite: false
      });

      particleSystem = new THREE.Points(geometry, material);
      scene.add(particleSystem);

      window.addEventListener('resize', onWindowResize);
    }

    // --- Posiciones de Texto ---
    function getTextPositions(text) {
      const canvas = document.createElement('canvas');
      const ctx = canvas.getContext('2d');
      canvas.width = 600;
      canvas.height = 200;

      ctx.font = 'Bold 70px Arial';
      ctx.fillStyle = 'white';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      ctx.fillText(text, canvas.width / 2, canvas.height / 2);

      const imgData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const points = [];

      for (let y = 0; y < canvas.height; y += 3) {
        for (let x = 0; x < canvas.width; x += 3) {
          const alpha = imgData.data[(y * canvas.width + x) * 4 + 3];
          if (alpha > 128) {
            points.push({
              x: (x - canvas.width / 2) * 0.6,
              y: -(y - canvas.height / 2) * 0.6,
              z: (Math.random() - 0.5) * 10
            });
          }
        }
      }
      return points;
    }

    // --- Posiciones del Sistema de Saturno (Planeta + Anillos) ---
    function getSaturnPositions() {
      const points = [];
      const sphereParticles = Math.floor(particleCount * 0.25); // 25% para la esfera central
      const ringParticles = particleCount - sphereParticles;    // 75% para los anillos

      // 1. Esfera central (Saturno)
      for (let i = 0; i < sphereParticles; i++) {
        let u = Math.random();
        let v = Math.random();
        let theta = u * 2.0 * Math.PI;
        let phi = Math.acos(2.0 * v - 1.0);
        let r = 18 + Math.random() * 2; // Radio de la esfera central

        let x = r * Math.sin(phi) * Math.cos(theta);
        let y = r * Math.sin(phi) * Math.sin(theta);
        let z = r * Math.cos(phi);

        points.push({ x, y, z });
      }

      // 2. Anillos concéntricos alrededor
      for (let i = 0; i < ringParticles; i++) {
        let radius = Math.random() * 55 + 28; // Radio desde el borde de la esfera
        let theta = Math.random() * Math.PI * 2;
        let ringHeight = (Math.random() - 0.5) * 1.5; // Espesor del anillo

        // Inclinación del anillo (rotación suave en 3D)
        let rawX = radius * Math.cos(theta);
        let rawY = ringHeight;
        let rawZ = radius * Math.sin(theta);

        // Aplicar inclinación de 25 grados
        let angle = 0.45;
        let x = rawX;
        let y = rawY * Math.cos(angle) - rawZ * Math.sin(angle);
        let z = rawY * Math.sin(angle) + rawZ * Math.cos(angle);

        points.push({ x, y, z });
      }

      return points;
    }

    let targetPositions = [];

    function morphTo(targetPoints) {
      targetPositions = [];
      for (let i = 0; i < particleCount; i++) {
        let pt = targetPoints[i % targetPoints.length];
        targetPositions.push(pt.x, pt.y, pt.z);
      }
    }

    function iniciarSecuencia() {
      let step = 0;
      
      // Primera palabra inmediatamente
      subtitleBox.innerText = `"${textos[0]}"`;
      morphTo(getTextPositions(textos[0]));
      step++;

      const intervalText = setInterval(() => {
        if (step < textos.length) {
          subtitleBox.innerText = `"${textos[step]}"`;
          const pts = getTextPositions(textos[step]);
          morphTo(pts);
          step++;
        } else {
          clearInterval(intervalText);
          // Transición final a Saturno con los Anillos
          subtitleBox.innerText = "🪐 Así como Saturno necesita sus anillos... Yo solo te necesito a ti 💖";
          morphTo(getSaturnPositions());
        }
      }, 2800);
    }

    // --- Animación Continua ---
    function animate() {
      requestAnimationFrame(animate);

      const positions = particleSystem.geometry.attributes.position.array;

      if (targetPositions.length > 0) {
        for (let i = 0; i < particleCount * 3; i++) {
          positions[i] += (targetPositions[i] - positions[i]) * 0.05;
        }
        particleSystem.geometry.attributes.position.needsUpdate = true;
      }

      // Rotación suave del sistema completo
      particleSystem.rotation.y += 0.003;
      renderer.render(scene, camera);
    }

    function onWindowResize() {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    }

    // Interacción con el botón
    btnHearts.addEventListener('click', () => {
      let randomSubtitle = frasesSubtitulo[Math.floor(Math.random() * frasesSubtitulo.length)];
      subtitleBox.innerText = randomSubtitle;
    });

    initThreeJS();
    animate();
  </script>
</body>
</html>

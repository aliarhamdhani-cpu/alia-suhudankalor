# alia-suhudankalor
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Simulasi WebAR - Suhu dan Kalor</title>
  
  <!-- CSS Styling -->
  <style>
    body {
      margin: 0;
      padding: 0;
      overflow: hidden;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }

    #webgl-container {
      width: 100vw;
      height: 100vh;
      display: block;
    }

    /* Floating UI Overlay */
    #ui-container {
      position: absolute;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      width: 90%;
      max-width: 400px;
      background: rgba(255, 255, 255, 0.85);
      backdrop-filter: blur(10px);
      padding: 15px 20px;
      border-radius: 16px;
      box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
      box-sizing: border-box;
      z-index: 10;
    }

    .control-group {
      margin-bottom: 12px;
    }

    label {
      font-weight: bold;
      font-size: 14px;
      display: block;
      margin-bottom: 5px;
      color: #333;
    }

    select, input[type="range"] {
      width: 100%;
      padding: 6px;
      border-radius: 8px;
      border: 1px solid #ccc;
      box-sizing: border-box;
    }

    .info-display {
      font-size: 13px;
      color: #444;
      background: rgba(0,0,0,0.05);
      padding: 8px;
      border-radius: 8px;
      margin-top: 8px;
    }

    #ar-button {
      position: absolute;
      top: 20px;
      right: 20px;
      padding: 10px 18px;
      background-color: #ff5722;
      color: white;
      border: none;
      border-radius: 20px;
      font-weight: bold;
      cursor: pointer;
      z-index: 10;
      box-shadow: 0 4px 10px rgba(0,0,0,0.3);
    }
  </style>

  <!-- Import Three.js -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

  <!-- AR Launch Button (Untuk device yang mendukung WebXR) -->
  <button id="ar-button">Masuk ke AR</button>

  <!-- Canvas Three.js -->
  <div id="webgl-container"></div>

  <!-- UI Panel Kontrol -->
  <div id="ui-container">
    <div class="control-group">
      <label for="material-select">Pilih Material / Benda:</label>
      <select id="material-select">
        <option value="besi">Besi (c = 450 J/kg°C)</option>
        <option value="air">Air (c = 4200 J/kg°C)</option>
        <option value="es">Es (c = 2100 J/kg°C)</option>
      </select>
    </div>

    <div class="control-group">
      <label for="temp-slider">Suhu ($T$): <span id="temp-value">25</span> °C</label>
      <input type="range" id="temp-slider" min="0" max="100" value="25">
    </div>

    <div class="info-display">
      <strong>Persamaan Kalor:</strong> $Q = m \cdot c \cdot \Delta T$<br>
      <span id="formula-info">Kecepatan kinetik partikel berbanding lurus dengan suhu.</span>
    </div>
  </div>

  <!-- Script Aplikasi -->
  <script>
    // --- 1. SET UP SCENE, CAMERA, & RENDERER ---
    const container = document.getElementById('webgl-container');
    const scene = new THREE.Scene();
    
    const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 1.5, 3);

    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.xr.enabled = true; // Mengaktifkan dukungan AR/WebXR
    container.appendChild(renderer.domElement);

    // Pencahayaan
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
    scene.add(ambientLight);

    const dirLight = new THREE.DirectionalLight(0xffffff, 0.6);
    dirLight.position.set(5, 10, 7);
    scene.add(dirLight);

    // --- 2. MEMBUAT MODEL 3D MOLEKUL (PARTIKEL) ---
    const particlesGroup = new THREE.Group();
    scene.add(particlesGroup);

    const particles = [];
    const particleCount = 27; // Grid 3x3x3
    const basePositions = [];

    // Geometri & Material Partikel Atom/Molekul
    const sphereGeometry = new THREE.SphereGeometry(0.08, 16, 16);
    const particleMaterial = new THREE.MeshStandardMaterial({
      color: 0x3498db, // Biru (Suhu dingin awal)
      roughness: 0.3,
      metalness: 0.2
    });

    // Susun partikel dalam grid 3D
    let index = 0;
    const spacing = 0.25;
    for (let x = -1; x <= 1; x++) {
      for (let y = -1; y <= 1; y++) {
        for (let z = -1; z <= 1; z++) {
          const particle = new THREE.Mesh(sphereGeometry, particleMaterial.clone());
          const posX = x * spacing;
          const posY = y * spacing + 0.5; // Agak terangkat di atas bidang
          const posZ = z * spacing;

          particle.position.set(posX, posY, posZ);
          particlesGroup.add(particle);

          particles.push(particle);
          basePositions.push(new THREE.Vector3(posX, posY, posZ));
        }
      }
    }

    // --- 3. KONTROL INTERAKTIF & LOGIKA FISIKA ---
    let currentTemp = 25;
    let vibrationIntensity = 0.005; // Intensitas getaran berdasarkan suhu

    const tempSlider = document.getElementById('temp-slider');
    const tempValueDisplay = document.getElementById('temp-value');
    const materialSelect = document.getElementById('material-select');
    const formulaInfo = document.getElementById('formula-info');

    // Update gerakan & warna partikel berdasarkan suhu
    function updatePhysics() {
      currentTemp = parseFloat(tempSlider.value);
      tempValueDisplay.innerText = currentTemp;

      // Hitung intensitas getaran kinetik berdasarkan suhu
      vibrationIntensity = (currentTemp / 100) * 0.03 + 0.002;

      // Mengubah warna partikel dari dingin (Biru) ke panas (Merah/Oranye)
      const heatRatio = currentTemp / 100;
      particles.forEach(p => {
        p.material.color.setHSL(0.6 * (1 - heatRatio), 0.8, 0.5); // 0.6 = Biru, 0.0 = Merah
      });

      // Update Teks Informasi
      formulaInfo.innerText = `Suhu: ${currentTemp}°C | Energi Kinetik: ${currentTemp > 60 ? 'Tinggi (Partikel Bergetar Cepat)' : 'Rendah (Partikel Stabil)'}`;
    }

    tempSlider.addEventListener('input', updatePhysics);
    materialSelect.addEventListener('change', (e) => {
      alert(`Material diubah ke: ${e.target.value.toUpperCase()}`);
    });

    // --- 4. LOOP ANIMASI & SIMULASI GETARAN MOLEKUL ---
    const clock = new THREE.Clock();

    function animate() {
      renderer.setAnimationLoop(() => {
        const time = clock.getElapsedTime();

        // Animasi getaran kinetik molekul/partikel
        particles.forEach((p, i) => {
          const base = basePositions[i];
          p.position.x = base.x + (Math.sin(time * 20 + i) * vibrationIntensity);
          p.position.y = base.y + (Math.cos(time * 25 + i) * vibrationIntensity);
          p.position.z = base.z + (Math.sin(time * 18 + i) * vibrationIntensity);
        });

        // Putar sedikit seluruh grup molekul agar terlihat 3D
        particlesGroup.rotation.y += 0.005;

        renderer.render(scene, camera);
      });
    }

    animate();

    // --- 5. LOGIKA AR MARKERLESS (WEBXR) ---
    const arButton = document.getElementById('ar-button');
    
    if ('xr' in navigator) {
      navigator.xr.isSessionSupported('immersive-ar').then((supported) => {
        if (supported) {
          arButton.addEventListener('click', () => {
            navigator.xr.requestSession('immersive-ar', {
              requiredFeatures: ['hit-test', 'dom-overlay'],
              domOverlay: { root: document.body }
            }).then((session) => {
              renderer.xr.setSession(session);
              arButton.style.display = 'none';
            });
          });
        } else {
          arButton.innerText = "AR Tidak Didukung di Perangkat Ini";
        }
      });
    } else {
      arButton.innerText = "WebXR Tidak Tersedia";
    }

    // Adjust Canvas saat Window Resize
    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });
  </script>
</body>
</html>

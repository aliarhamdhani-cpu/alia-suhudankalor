<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>LKPD & WebAR Efek Doppler - Suhu dan Kalor</title>
  
  <!-- Import Three.js & KaTeX untuk Rumus -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.css">
  <script src="https://cdn.jsdelivr.net/npm/katex@0.16.8/dist/katex.min.js"></script>

  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; height: 100vh; overflow: hidden; }

    /* Layout Split Screen: Kiri (LKPD), Kanan (WebAR) */
    .container { display: flex; width: 100vw; height: 100vh; }
    
    .panel-left {
      width: 45%;
      height: 100%;
      background: #f8f9fa;
      border-right: 3px solid #dee2e6;
      padding: 20px;
      overflow-y: auto;
    }

    .panel-right {
      width: 55%;
      height: 100%;
      position: relative;
      background: #111;
    }

    /* WebGL Canvas */
    #webgl-container { width: 100%; height: 100%; display: block; }

    /* Floating Control AR Overlay */
    #ar-controls {
      position: absolute;
      bottom: 15px;
      left: 50%;
      transform: translateX(-50%);
      width: 90%;
      background: rgba(255, 255, 255, 0.9);
      backdrop-filter: blur(8px);
      padding: 15px;
      border-radius: 12px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.3);
      font-size: 13px;
    }

    .control-row { display: flex; align-items: center; justify-content: space-between; margin-bottom: 8px; }
    .control-row label { font-weight: bold; width: 40%; }
    .control-row input { width: 55%; }

    .math-card {
      background: #e9ecef;
      padding: 10px;
      border-radius: 8px;
      margin-top: 10px;
      font-size: 13px;
    }

    /* LKPD Styling */
    h2 { color: #2c3e50; margin-bottom: 10px; font-size: 20px; }
    h3 { color: #34495e; margin-top: 15px; margin-bottom: 8px; font-size: 15px; border-bottom: 2px solid #007bff; padding-bottom: 4px; }
    p, li { font-size: 13px; color: #495057; line-height: 1.5; margin-bottom: 8px; }
    textarea, input[type="text"] {
      width: 100%;
      padding: 8px;
      border: 1px solid #ced4da;
      border-radius: 6px;
      margin-top: 4px;
      margin-bottom: 10px;
      font-family: inherit;
    }

    #ar-btn {
      position: absolute;
      top: 15px;
      right: 15px;
      padding: 10px 16px;
      background: #28a745;
      color: white;
      border: none;
      border-radius: 20px;
      font-weight: bold;
      cursor: pointer;
      z-index: 10;
    }

    @media (max-width: 768px) {
      .container { flex-direction: column; }
      .panel-left, .panel-right { width: 100%; height: 50vh; }
    }
  </style>
</head>
<body>

<div class="container">
  
  <!-- PANEL KIRI: LKPD INTERAKTIF -->
  <div class="panel-left">
    <h2>LKPD Digital: Efek Doppler & Suhu Medium</h2>
    <p><strong>Mata Pelajaran:</strong> Fisika | <strong>Materi:</strong> Gelombang Bunyi, Suhu & Kalor</p>
    <hr style="margin: 10px 0;">

    <h3>A. Tujuan Percobaan</h3>
    <ol style="margin-left: 20px;">
      <li>Menganalisis pengaruh suhu ($T$) medium gas terhadap cepat rambat bunyi ($v$).</li>
      <li>Mengamati fenomena Efek Doppler (perubahan frekuensi terdeteksi) akibat gerakan sumber bunyi dan pengaruh suhu udara.</li>
    </ol>

    <h3>B. Identitas Siswa</h3>
    <label>Nama Siswa:</label>
    <input type="text" placeholder="Masukkan Nama Lengkap">
    <label>Kelas / Kelompok:</label>
    <input type="text" placeholder="Contoh: XI-MIPA 1 / Kelompok 2">

    <h3>C. Orientasi Masalah</h3>
    <p>Mengapa suara sirine ambulan terdengar lebih melengking saat mendekati kita di siang hari yang panas dibandingkan malam hari yang dingin? Bagaimana hubungan antara kalor, suhu medium, dan frekuensi bunyi yang terdengar?</p>

    <h3>D. Pengumpulan Data Simulasi</h3>
    <p>Atur kontrol pada WebAR di sebelah kanan dan catat hasilnya:</p>
    
    <label>1. Catat Nilai Kecepatan Bunyi ($v$) saat Suhu Udara = 0°C dan 50°C:</label>
    <textarea rows="2" placeholder="Tuliskan nilai v (m/s) hasil pengukuran simulasi..."></textarea>

    <label>2. Bandingkan Frekuensi Pendengar ($f_p$) saat Sumber Mendekat vs Menjauh:</label>
    <textarea rows="3" placeholder="Bagaimana perubahan nilai fp saat sumber bergerak mendekati dan menjauhi pendengar?"></textarea>

    <h3>E. Analisis & Kesimpulan</h3>
    <label>Tuliskan kesimpulan hubungan antara Suhu ($T$), Cepat Rambat ($v$), dan Frekuensi Doppler ($f_p$):</label>
    <textarea rows="4" placeholder="Tuliskan kesimpulan percobaan di sini..."></textarea>
  </div>

  <!-- PANEL KANAN: WEBAR & SIMULASI 3D -->
  <div class="panel-right">
    <button id="ar-btn">Masuk Mode AR</button>
    <div id="webgl-container"></div>

    <!-- UI Overlay Kontrol Input Simulasi -->
    <div id="ar-controls">
      <div class="control-row">
        <label>Suhu Udara ($T$): <span id="val-T">25</span> °C</label>
        <input type="range" id="input-T" min="-20" max="80" value="25">
      </div>
      <div class="control-row">
        <label>Kecepatan Sumber ($v_s$): <span id="val-vs">20</span> m/s</label>
        <input type="range" id="input-vs" min="0" max="60" value="20">
      </div>
      <div class="control-row">
        <label>Frekuensi Asli ($f_s$): <span id="val-fs">500</span> Hz</label>
        <input type="range" id="input-fs" min="200" max="1000" value="500">
      </div>

      <div class="math-card">
        <div><strong>Cepat Rambat ($v$):</strong> <span id="out-v">0</span> m/s</div>
        <div><strong>Frekuensi Pendengar ($f_p$):</strong> <span id="out-fp">0</span> Hz</div>
      </div>
    </div>
  </div>

</div>

<script>
  // --- 1. SETUP SCENE THREE.JS ---
  const container = document.getElementById('webgl-container');
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(60, container.clientWidth / container.clientHeight, 0.1, 1000);
  camera.position.set(0, 3, 6);
  camera.lookAt(0, 0, 0);

  const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(container.clientWidth, container.clientHeight);
  renderer.xr.enabled = true;
  container.appendChild(renderer.domElement);

  // Lighting
  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(5, 10, 5);
  scene.add(light);
  scene.add(new THREE.AmbientLight(0xdddddd));

  // --- 2. OBJEK SIMULASI (PENDENGAR & SUMBER BUNYI) ---
  // Pendengar (Objek Biru)
  const listenerGeo = new THREE.CylinderGeometry(0.2, 0.2, 0.6, 16);
  const listenerMat = new THREE.MeshStandardMaterial({ color: 0x007bff });
  const listenerMesh = new THREE.Mesh(listenerGeo, listenerMat);
  listenerMesh.position.set(-2, 0.3, 0);
  scene.add(listenerMesh);

  // Sumber Bunyi / Mobil Sirine (Objek Merah)
  const sourceGeo = new THREE.BoxGeometry(0.5, 0.4, 0.8);
  const sourceMat = new THREE.MeshStandardMaterial({ color: 0xd9534f });
  const sourceMesh = new THREE.Mesh(sourceGeo, sourceMat);
  sourceMesh.position.set(2, 0.2, 0);
  scene.add(sourceMesh);

  // Visualisasi Gelombang (Lingkaran-lingkaran Muka Gelombang)
  const waveRings = [];
  const maxRings = 6;
  for (let i = 0; i < maxRings; i++) {
    const ringGeo = new THREE.RingGeometry(0.05, 0.08, 32);
    ringGeo.rotateX(-Math.PI / 2);
    const ringMat = new THREE.MeshBasicMaterial({ color: 0xffcc00, side: THREE.DoubleSide, transparent: true, opacity: 0.7 });
    const ring = new THREE.Mesh(ringGeo, ringMat);
    ring.visible = false;
    scene.add(ring);
    waveRings.push({ mesh: ring, radius: 0, active: false });
  }

  // --- 3. KALKULASI FISIKA (SUHU, KALOR & EFEK DOPPLER) ---
  const inputT = document.getElementById('input-T');
  const inputVs = document.getElementById('input-vs');
  const inputFs = document.getElementById('input-fs');

  let T_celsius = 25;
  let v_s = 20; // m/s
  let f_s = 500; // Hz
  let v_sound = 343; // m/s
  let f_p = 500;

  function recalculatePhysics() {
    T_celsius = parseFloat(inputT.value);
    v_s = parseFloat(inputVs.value);
    f_s = parseFloat(inputFs.value);

    document.getElementById('val-T').innerText = T_celsius;
    document.getElementById('val-vs').innerText = v_s;
    document.getElementById('val-fs').innerText = f_s;

    // Rumus Cepat Rambat Bunyi terhadap Suhu: v = 331 * sqrt(1 + T/273)
    const T_kelvin = T_celsius + 273.15;
    v_sound = 331.3 * Math.sqrt(T_kelvin / 273.15);

    // Efek Doppler (Asumsi Sumber Mendekati Pendengar)
    f_p = (v_sound / (v_sound - v_s)) * f_s;

    document.getElementById('out-v').innerText = v_sound.toFixed(1);
    document.getElementById('out-fp').innerText = Math.round(f_p);
  }

  inputT.addEventListener('input', recalculatePhysics);
  inputVs.addEventListener('input', recalculatePhysics);
  inputFs.addEventListener('input', recalculatePhysics);
  recalculatePhysics();

  // --- 4. ANIMASI LOOP ---
  let clock = new THREE.Clock();
  let waveTimer = 0;

  function animate() {
    renderer.setAnimationLoop(() => {
      const delta = clock.getDelta();
      const time = clock.getElapsedTime();

      // Animasi Sumber Gerak Bolak-balik (Simulasi Gerak Sirine)
      const posX = Math.sin(time * 1.5) * 2.5 + 0.5;
      sourceMesh.position.x = posX;

      // Pemicu Muka Gelombang Bunyi
      waveTimer += delta;
      if (waveTimer > 0.4) {
        waveTimer = 0;
        let ring = waveRings.find(r => !r.active);
        if (ring) {
          ring.active = true;
          ring.mesh.position.set(sourceMesh.position.x, 0.05, sourceMesh.position.z);
          ring.radius = 0.1;
          ring.mesh.visible = true;
        }
      }

      // Perambatan Gelombang (Dipengaruhi oleh Nilai v Kecepatan Suhu)
      const expansionSpeed = (v_sound / 340) * 1.5; 
      waveRings.forEach(r => {
        if (r.active) {
          r.radius += delta * expansionSpeed;
          r.mesh.scale.set(r.radius, r.radius, 1);
          r.mesh.material.opacity = Math.max(0, 1 - (r.radius / 3));

          if (r.radius > 3) {
            r.active = false;
            r.mesh.visible = false;
          }
        }
      });

      renderer.render(scene, camera);
    });
  }

  animate();

  // --- 5. LOGIKA WEBAR (WEBXR) ---
  const arBtn = document.getElementById('ar-btn');
  if ('xr' in navigator) {
    navigator.xr.isSessionSupported('immersive-ar').then(supported => {
      if (supported) {
        arBtn.addEventListener('click', () => {
          navigator.xr.requestSession('immersive-ar', {
            requiredFeatures: ['hit-test', 'dom-overlay'],
            domOverlay: { root: document.body }
          }).then(session => {
            renderer.xr.setSession(session);
          });
        });
      } else {
        arBtn.innerText = "AR Mode 3D Only";
      }
    });
  }

  window.addEventListener('resize', () => {
    camera.aspect = container.clientWidth / container.clientHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(container.clientWidth, container.clientHeight);
  });
</script>

</body>
</html>

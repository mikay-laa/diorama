<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Reference-Matched Sustainable City Diorama</title>
  <link rel="stylesheet" href="style.css">
  
  <!-- Three.js and OrbitControls -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body>

  <div id="canvas-container"></div>

  <!-- UI Control Panel -->
  <div id="ui-panel">
    <h1>Sustainable City</h1>
    <div class="control-group">
      <label for="speedSlider">Day/Night Speed <span id="speedVal">1.0x</span></label>
      <input type="range" id="speedSlider" min="0" max="3" step="0.1" value="1">
    </div>
    <button id="toggleRain" class="btn">Toggle Rain</button>
  </div>

  <!-- Interactive Info Card -->
  <div id="info-card" class="info-card hidden">
    <button id="close-btn" class="close-btn">&times;</button>
    <div class="card-tag" id="card-tag">TAG</div>
    <h2 id="card-title">Infrastructure</h2>
    <p id="card-desc">Description text.</p>
    <div class="card-stats">
      <div class="stat">
        <span class="stat-value" id="stat-1">-</span>
        <span class="stat-label" id="stat-1-label">Metric</span>
      </div>
      <div class="stat">
        <span class="stat-value" id="stat-2">-</span>
        <span class="stat-label" id="stat-2-label">Metric</span>
      </div>
    </div>
  </div>

  <script>
    // --- 1. SCENE, CAMERA, & RENDERER SETUP ---
    const container = document.getElementById('canvas-container');
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x87ceeb);

    const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(38, 30, 38);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    container.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;
    controls.dampingFactor = 0.05;
    controls.maxPolarAngle = Math.PI / 2 - 0.02;

    // --- 2. LIGHTING & GLOBAL ARRAYS ---
    const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
    scene.add(ambientLight);

    const sunLight = new THREE.DirectionalLight(0xfffaed, 1.6);
    sunLight.position.set(35, 45, 35);
    sunLight.castShadow = true;
    sunLight.shadow.mapSize.width = 2048;
    sunLight.shadow.mapSize.height = 2048;
    const d = 26;
    sunLight.shadow.camera.left = -d;
    sunLight.shadow.camera.right = d;
    sunLight.shadow.camera.top = d;
    sunLight.shadow.camera.bottom = -d;
    scene.add(sunLight);

    const interactiveObjects = [];
    const windTurbines = [];
    const animatedVehicles = [];
    const grazingAnimals = [];
    const nighttimeEmissives = [];
    const nighttimeLights = [];

    const daySky = new THREE.Color(0x87ceeb);
    const sunsetSky = new THREE.Color(0xff7f50);
    const nightSky = new THREE.Color(0x0a0a1a);

    const mat = (col, rough = 0.8, metal = 0.1) => 
      new THREE.MeshStandardMaterial({ color: col, flatShading: true, roughness: rough, metalness: metal });

    // Glass Material for Modern Building
    const glassMat = new THREE.MeshPhysicalMaterial({
      color: 0x38bdf8,
      metalness: 0.1,
      roughness: 0.1,
      transparent: true,
      opacity: 0.75,
      transmission: 0.8
    });

    // Dark Blue Solar Panel Material (Image Reference 3)
    const solarMat = new THREE.MeshStandardMaterial({
      color: 0x1e1b4b,
      metalness: 0.9,
      roughness: 0.1,
      flatShading: true
    });

    // --- 3. TERRAIN & RIVER ---
    const terrain = new THREE.Group();

    const baseCrust = new THREE.Mesh(new THREE.BoxGeometry(32, 4, 32), mat(0x451a03));
    baseCrust.position.y = -3.0;
    terrain.add(baseCrust);

    const westBank = new THREE.Mesh(new THREE.BoxGeometry(13, 1, 32), mat(0x16a34a));
    westBank.position.set(-9.5, -0.5, 0);
    westBank.receiveShadow = true;
    terrain.add(westBank);

    const eastBank = new THREE.Mesh(new THREE.BoxGeometry(13, 1, 32), mat(0x16a34a));
    eastBank.position.set(9.5, -0.5, 0);
    eastBank.receiveShadow = true;
    terrain.add(eastBank);

    const river = new THREE.Mesh(
      new THREE.BoxGeometry(6, 0.8, 32),
      new THREE.MeshStandardMaterial({ color: 0x0284c7, roughness: 0.1, transparent: true, opacity: 0.85 })
    );
    river.position.set(0, -0.6, 0);
    terrain.add(river);

    scene.add(terrain);

    // --- 4. WOODEN ARCH PEDESTRIAN BRIDGE (IMAGE REFERENCE 4) ---
    function createWoodenArchBridge(x, z) {
      const g = new THREE.Group();
      const woodMat = mat(0x92400e, 0.9, 0.05);

      // Curved Deck
      const deckGeo = new THREE.BoxGeometry(6.5, 0.25, 2.6);
      const deck = new THREE.Mesh(deckGeo, woodMat);
      deck.position.set(0, 0.2, 0);
      deck.castShadow = true;
      g.add(deck);

      // Curved Arched Handrails
      for (let zOff of [-1.2, 1.2]) {
        const arch = new THREE.Mesh(new THREE.TorusGeometry(3.2, 0.1, 8, 16, Math.PI), woodMat);
        arch.rotation.x = Math.PI;
        arch.position.set(0, 0.3, zOff);
        g.add(arch);

        // Baluster Posts
        for (let xOff = -2.5; xOff <= 2.5; xOff += 1.0) {
          const post = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.08, 0.8), woodMat);
          post.position.set(xOff, 0.6, zOff);
          g.add(post);
        }
      }

      g.position.set(x, 0, z);

      const meta = {
        tag: "INFRASTRUCTURE",
        title: "Traditional Wooden Arch Bridge",
        description: "Handcrafted timber pedestrian bridge providing clean foot and bicycle transit across the river.",
        stat1: "100%", stat1Label: "Sustainable Wood", stat2: "Zero", stat2Label: "Carbon Footprint"
      };
      g.traverse(c => { if (c.isMesh) { c.userData = meta; interactiveObjects.push(c); } });
      scene.add(g);
    }

    createWoodenArchBridge(0, 0);

    // Roadway
    const mainRoad = new THREE.Mesh(new THREE.BoxGeometry(28, 0.02, 2.6), mat(0x1e293b));
    mainRoad.position.set(0, 0.02, 0);
    scene.add(mainRoad);

    // --- 5. MODERN COMMERCIAL GLASS BUILDING (IMAGE REFERENCE 1) ---
    function createModernGlassBuilding(x, z) {
      const g = new THREE.Group();
      const orangeMat = mat(0xea580c); // Orange trim

      // Main Glass Body with Grid Panes
      const glassBody = new THREE.Mesh(new THREE.BoxGeometry(4.8, 3.8, 3.8), glassMat);
      glassBody.position.y = 1.9;
      glassBody.castShadow = true;
      g.add(glassBody);

      // Orange Frame Outer Casing
      const outerFrame = new THREE.Mesh(new THREE.BoxGeometry(5.0, 4.0, 0.4), orangeMat);
      outerFrame.position.set(0, 2.0, 1.8);
      g.add(outerFrame);

      // Central Entrance Overhang Canopy
      const canopy = new THREE.Mesh(new THREE.BoxGeometry(2.0, 0.2, 1.2), orangeMat);
      canopy.position.set(0, 1.2, 2.2);
      g.add(canopy);

      // Round Topiary Trees (As seen in Reference 1)
      const treeMat = mat(0x22c55e);
      const trunkMat = mat(0x78350f);
      for (let tx of [-2.8, 2.8]) {
        const trunk = new THREE.Mesh(new THREE.CylinderGeometry(0.08, 0.12, 1.2), trunkMat);
        trunk.position.set(tx, 0.6, 2.0);
        g.add(trunk);

        const foliage = new THREE.Mesh(new THREE.SphereGeometry(0.7, 12, 12), treeMat);
        foliage.position.set(tx, 1.5, 2.0);
        g.add(foliage);
      }

      g.position.set(x, 0, z);

      const meta = {
        tag: "COMMERCIAL",
        title: "Modern Glass Eco-Center",
        description: "Energy-efficient commercial building featuring glass curtain walls and insulated orange framing.",
        stat1: "-40%", stat1Label: "Energy Use", stat2: "A+", stat2Label: "Thermal Rating"
      };
      g.traverse(c => { if (c.isMesh) { c.userData = meta; interactiveObjects.push(c); } });
      scene.add(g);
    }

    createModernGlassBuilding(10, -8);

    // --- 6. INTEGRATED ECO-FARM COMPLEX (IMAGE REFERENCE 2) ---
    function createEcoFarmComplex(x, z) {
      const farmGroup = new THREE.Group();
      const barnMat = mat(0xd97706); // Red/Orange Barn
      const roofMat = mat(0x475569); // Metallic roof

      // 1. Red Barn Structure
      const barn = new THREE.Mesh(new THREE.BoxGeometry(3.5, 2.5, 2.8), barnMat);
      barn.position.set(0, 1.25, 0);
      barn.castShadow = true;
      farmGroup.add(barn);

      const barnRoof = new THREE.Mesh(new THREE.ConeGeometry(2.6, 1.2, 4), roofMat);
      barnRoof.rotation.y = Math.PI / 4;
      barnRoof.position.set(0, 3.1, 0);
      farmGroup.add(barnRoof);

      // 2. Traditional Red Windmill with Animated Blades
      const millBody = new THREE.Mesh(new THREE.CylinderGeometry(0.6, 0.9, 3.5, 8), barnMat);
      millBody.position.set(-3.2, 1.75, -1.0);
      farmGroup.add(millBody);

      const millCap = new THREE.Mesh(new THREE.ConeGeometry(0.8, 0.8, 8), mat(0xef4444));
      millCap.position.set(-3.2, 3.9, -1.0);
      farmGroup.add(millCap);

      const rotorGroup = new THREE.Group();
      rotorGroup.position.set(-3.2, 3.6, -0.4);

      for (let i = 0; i < 4; i++) {
        const blade = new THREE.Mesh(new THREE.BoxGeometry(0.1, 1.6, 0.02), mat(0xffffff));
        blade.rotation.z = (i * Math.PI) / 2;
        rotorGroup.add(blade);
      }
      farmGroup.add(rotorGroup);
      windTurbines.push({ rotor: rotorGroup, speed: 1.8 });

      // 3. Greenhouse Tunnel & Orchard Grid
      const glassTunnel = new THREE.Mesh(
        new THREE.CylinderGeometry(0.8, 0.8, 2.5, 12, 1, false, 0, Math.PI),
        glassMat
      );
      glassTunnel.rotation.z = Math.PI / 2;
      glassTunnel.position.set(2.5, 0.4, -1.0);
      farmGroup.add(glassTunnel);

      // Dark Furrow Fields
      const furrow = new THREE.Mesh(new THREE.BoxGeometry(3.0, 0.1, 2.5), mat(0x3d2314));
      furrow.position.set(2.5, 0.05, 1.8);
      farmGroup.add(furrow);

      farmGroup.position.set(x, 0, z);

      const meta = {
        tag: "AGRICULTURE",
        title: "Traditional Eco-Farm Complex",
        description: "Combines classic farm architecture with clean energy windmills, greenhouses, and organic crop furrows.",
        stat1: "100%", stat1Label: "Organic", stat2: "Zero", stat2Label: "Chemical Use"
      };
      farmGroup.traverse(c => { if (c.isMesh) { c.userData = meta; interactiveObjects.push(c); } });
      scene.add(farmGroup);
    }

    createEcoFarmComplex(-10, 8);

    // --- 7. HOUSES WITH SOLAR PANELS (IMAGE REFERENCE 3) ---
    function createSolarHouse(x, z) {
      const g = new THREE.Group();

      // White Body
      const body = new THREE.Mesh(new THREE.BoxGeometry(2.6, 1.6, 2.2), mat(0xfff7ed));
      body.position.y = 0.8;
      body.castShadow = true;
      g.add(body);

      // Orange Pitched Roof
      const roof = new THREE.Mesh(new THREE.ConeGeometry(2.0, 1.1, 4), mat(0xea580c));
      roof.rotation.y = Math.PI / 4;
      roof.position.y = 2.15;
      roof.castShadow = true;
      g.add(roof);

      // Dark Blue Solar Panel Array mounted on roof slope
      for (let px of [-0.5, 0.5]) {
        const panel = new THREE.Mesh(new THREE.BoxGeometry(0.7, 0.04, 1.0), solarMat);
        panel.rotation.x = Math.PI / 6;
        panel.position.set(px, 2.1, 0.5);
        g.add(panel);
      }

      g.position.set(x, 0, z);

      const meta = {
        tag: "RESIDENTIAL",
        title: "Solar-Roof Eco Home",
        description: "Residential house fitted with angled rooftop photovoltaic solar panels producing clean household electricity.",
        stat1: "350 kWh", stat1Label: "Monthly Solar Output", stat2: "100%", stat2Label: "Self-Powered"
      };
      g.traverse(c => { if (c.isMesh) { c.userData = meta; interactiveObjects.push(c); } });
      scene.add(g);
    }

    // Spawn Solar Homes in Residential Sector
    createSolarHouse(-10, -11);
    createSolarHouse(-7, -11);
    createSolarHouse(-10, -7);
    createSolarHouse(-7, -7);

    // --- 8. OTHER MUNICIPAL BUILDINGS & SCHOOL WITH PH FLAG ---
    // Signboard Generator
    function createSignboard(text, width = 2.2, height = 0.6) {
      const canvas = document.createElement('canvas');
      canvas.width = 256; canvas.height = 64;
      const ctx = canvas.getContext('2d');
      ctx.fillStyle = '#0f172a'; ctx.fillRect(0, 0, 256, 64);
      ctx.strokeStyle = '#38bdf8'; ctx.lineWidth = 6; ctx.strokeRect(3, 3, 250, 58);
      ctx.fillStyle = '#ffffff'; ctx.font = 'bold 28px sans-serif';
      ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
      ctx.fillText(text, 128, 32);

      const texture = new THREE.CanvasTexture(canvas);
      return new THREE.Mesh(new THREE.PlaneGeometry(width, height), new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide }));
    }

    // Philippine Flag
    function createPhilippineFlag() {
      const g = new THREE.Group();
      const pole = new THREE.Mesh(new THREE.CylinderGeometry(0.04, 0.04, 3.5, 8), mat(0xd1d5db));
      pole.position.y = 1.75;
      g.add(pole);

      const canvas = document.createElement('canvas');
      canvas.width = 300; canvas.height = 150;
      const ctx = canvas.getContext('2d');
      ctx.fillStyle = '#0038a8'; ctx.fillRect(0, 0, 300, 75);
      ctx.fillStyle = '#ce1126'; ctx.fillRect(0, 75, 300, 75);
      ctx.fillStyle = '#ffffff'; ctx.beginPath(); ctx.moveTo(0, 0); ctx.lineTo(130, 75); ctx.lineTo(0, 150); ctx.closePath(); ctx.fill();

      const texture = new THREE.CanvasTexture(canvas);
      const flagMesh = new THREE.Mesh(new THREE.PlaneGeometry(1.4, 0.7), new THREE.MeshBasicMaterial({ map: texture, side: THREE.DoubleSide }));
      flagMesh.position.set(0.7, 3.0, 0);
      g.add(flagMesh);
      return g;
    }

    function createSchool(x, z) {
      const g = new THREE.Group();
      const body = new THREE.Mesh(new THREE.BoxGeometry(5.0, 3.0, 3.5), mat(0xfef3c7));
      body.position.y = 1.5;
      g.add(body);

      const sign = createSignboard("SCHOOL", 2.2, 0.6);
      sign.position.set(0, 3.4, 1.76);
      g.add(sign);

      const phFlag = createPhilippineFlag();
      phFlag.position.set(-1.8, 3.0, 0);
      g.add(phFlag);

      g.position.set(x, 0, z);
      scene.add(g);
    }

    createSchool(10, 2);

    // --- 9. ANIMATION LOOP & WEATHER ---
    const rainCount = 800;
    const rainGeo = new THREE.BufferGeometry();
    const rainPos = new Float32Array(rainCount * 3);

    for (let i = 0; i < rainCount * 3; i += 3) {
      rainPos[i] = (Math.random() - 0.5) * 32;
      rainPos[i + 1] = Math.random() * 20 + 2;
      rainPos[i + 2] = (Math.random() - 0.5) * 32;
    }

    rainGeo.setAttribute('position', new THREE.BufferAttribute(rainPos, 3));
    const rain = new THREE.Points(rainGeo, new THREE.PointsMaterial({ color: 0x93c5fd, size: 0.12, transparent: true, opacity: 0.65 }));
    rain.visible = false;
    scene.add(rain);

    let dayTime = 0;
    let timeSpeedMultiplier = 1.0;
    const dayDuration = 30;

    function updateDayNightCycle(dt) {
      dayTime = (dayTime + (dt * timeSpeedMultiplier) / dayDuration) % 1;
      const angle = dayTime * Math.PI * 2;
      const sunHeight = Math.sin(angle);

      sunLight.position.x = Math.cos(angle) * 45;
      sunLight.position.y = sunHeight * 45;

      if (sunHeight > 0.2) {
        scene.background.lerpColors(sunsetSky, daySky, (sunHeight - 0.2) / 0.8);
        sunLight.intensity = 1.6 * sunHeight;
        ambientLight.intensity = 0.5;
      } else {
        scene.background.lerp(nightSky, 0.1);
        sunLight.intensity = 0.1;
        ambientLight.intensity = 0.08;
      }
    }

    // Raycaster Info Cards
    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();
    const infoCard = document.getElementById('info-card');

    window.addEventListener('pointerdown', (e) => {
      mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;

      raycaster.setFromCamera(mouse, camera);
      const intersects = raycaster.intersectObjects(interactiveObjects, false);

      if (intersects.length > 0) {
        const data = intersects[0].object.userData;
        if (data && data.title) {
          document.getElementById('card-tag').textContent = data.tag;
          document.getElementById('card-title').textContent = data.title;
          document.getElementById('card-desc').textContent = data.description;
          document.getElementById('stat-1').textContent = data.stat1;
          document.getElementById('stat-1-label').textContent = data.stat1Label;
          document.getElementById('stat-2').textContent = data.stat2;
          document.getElementById('stat-2-label').textContent = data.stat2Label;
          infoCard.classList.remove('hidden');
        }
      }
    });

    document.getElementById('close-btn').addEventListener('click', () => infoCard.classList.add('hidden'));

    document.getElementById('speedSlider').addEventListener('input', (e) => {
      timeSpeedMultiplier = parseFloat(e.target.value);
      document.getElementById('speedVal').textContent = timeSpeedMultiplier.toFixed(1) + 'x';
    });

    document.getElementById('toggleRain').addEventListener('click', () => {
      rain.visible = !rain.visible;
    });

    window.addEventListener('resize', () => {
      camera.aspect = window.innerWidth / window.innerHeight;
      camera.updateProjectionMatrix();
      renderer.setSize(window.innerWidth, window.innerHeight);
    });

    // --- 10. MAIN ANIMATION LOOP ---
    const clock = new THREE.Clock();

    function animate() {
      requestAnimationFrame(animate);

      const dt = clock.getDelta();

      updateDayNightCycle(dt);

      // Rotate Windmill and Wind Turbines
      windTurbines.forEach(wt => { wt.rotor.rotation.z += wt.speed * dt; });

      // Animate Rain
      if (rain.visible) {
        const positions = rain.geometry.attributes.position.array;
        for (let i = 1; i < rainCount * 3; i += 3) {
          positions[i] -= 22 * dt;
          if (positions[i] < 0) positions[i] = 20;
        }
        rain.geometry.attributes.position.needsUpdate = true;
      }

      controls.update();
      renderer.render(scene, camera);
    }

    animate();
  </script>
</body>
</html>

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pak Racer: City Rush</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        body { margin: 0; overflow: hidden; background: #000; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; touch-action: none; }
        #ui-layer { position: absolute; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; display: flex; flex-direction: column; justify-content: space-between; }
        
        /* Menu Styling */
        #menu { position: absolute; width: 100%; height: 100%; background: rgba(0,0,0,0.85); display: flex; flex-direction: column; align-items: center; justify-content: center; z-index: 100; pointer-events: auto; text-align: center; color: #fff; }
        .btn { padding: 15px 40px; margin: 10px; font-size: 20px; background: #01411C; color: white; border: 2px solid #fff; border-radius: 30px; cursor: pointer; transition: 0.3s; }
        .btn:active { transform: scale(0.9); background: #026b2e; }
        h1 { font-size: 3rem; margin-bottom: 5px; color: #fff; text-shadow: 2px 2px #01411C; }
        p { margin-bottom: 30px; color: #ccc; }

        /* Game Controls */
        #controls { position: absolute; bottom: 20px; width: 100%; display: flex; justify-content: space-between; padding: 0 20px; box-sizing: border-box; pointer-events: auto; }
        .control-group { display: flex; gap: 15px; }
        .touch-btn { width: 80px; height: 80px; background: rgba(255,255,255,0.2); border: 2px solid #fff; border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; font-size: 24px; user-select: none; -webkit-user-select: none; }
        .touch-btn:active { background: rgba(255,255,255,0.5); }

        #stats { position: absolute; top: 20px; left: 20px; color: white; font-size: 24px; text-shadow: 2px 2px 4px black; }
        #city-indicator { position: absolute; top: 20px; right: 20px; color: #ffd700; font-size: 20px; font-weight: bold; }
    </style>
</head>
<body>

<div id="menu">
    <h1>PAK RACER</h1>
    <p>Select Your Waseela (Vehicle)</p>
    <button class="btn" onclick="startGame('rickshaw')">🛺 Rickshaw</button>
    <button class="btn" onclick="startGame('truck')">🚚 Truck Art</button>
    <button class="btn" onclick="startGame('car')">🚗 Mehran</button>
    <p style="font-size: 12px; margin-top: 20px;">Touch to Steer & Drive</p>
</div>

<div id="ui-layer">
    <div id="stats">Speed: <span id="speed">0</span> km/h</div>
    <div id="city-indicator">Entering: LAHORE</div>
    <div id="controls" style="display: none;">
        <div class="control-group">
            <div class="touch-btn" id="btn-left">←</div>
            <div class="touch-btn" id="btn-right">→</div>
        </div>
        <div class="control-group">
            <div class="touch-btn" id="btn-brake" style="background: rgba(255,0,0,0.3);">B</div>
            <div class="touch-btn" id="btn-gas" style="background: rgba(0,255,0,0.3);">↑</div>
        </div>
    </div>
</div>

<script>
    let scene, camera, renderer, vehicle, road, buildings = [];
    let speed = 0, targetSpeed = 0, rotation = 0;
    let gameActive = false;
    let clock = new THREE.Clock();

    // Input state
    const input = { left: false, right: false, gas: false, brake: false };

    function init() {
        scene = new THREE.Scene();
        scene.background = new THREE.Color(0x87ceeb); // Daytime sky
        scene.fog = new THREE.Fog(0x87ceeb, 20, 100);

        camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(0, 5, 10);
        camera.lookAt(0, 0, 0);

        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(window.devicePixelRatio);
        document.body.appendChild(renderer.domElement);

        // Lights
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.6);
        scene.add(ambientLight);
        const sunLight = new THREE.DirectionalLight(0xffffff, 0.8);
        sunLight.position.set(10, 20, 10);
        scene.add(sunLight);

        createRoad();
        createScenery();
        
        window.addEventListener('resize', onWindowResize, false);
        setupTouchControls();
    }

    function createRoad() {
        const roadGeo = new THREE.PlaneGeometry(20, 2000);
        const roadMat = new THREE.MeshStandardMaterial({ color: 0x333333 });
        road = new THREE.Mesh(roadGeo, roadMat);
        road.rotation.x = -Math.PI / 2;
        scene.add(road);

        // Center line
        const lineGeo = new THREE.PlaneGeometry(0.5, 2000);
        const lineMat = new THREE.MeshBasicMaterial({ color: 0xffffff });
        const line = new THREE.Mesh(lineGeo, lineMat);
        line.position.y = 0.01;
        road.add(line);
    }

    function createVehicle(type) {
        if (vehicle) scene.remove(vehicle);
        vehicle = new THREE.Group();

        if (type === 'rickshaw') {
            // Rickshaw Body
            const body = new THREE.Mesh(new THREE.BoxGeometry(1.5, 1.5, 2), new THREE.MeshStandardMaterial({color: 0xffff00}));
            body.position.y = 1;
            vehicle.add(body);
            // Black roof
            const roof = new THREE.Mesh(new THREE.BoxGeometry(1.6, 0.1, 2.1), new THREE.MeshStandardMaterial({color: 0x111111}));
            roof.position.y = 1.8;
            vehicle.add(roof);
        } else if (type === 'truck') {
            // Pak Truck Art - Vibrant colors
            const body = new THREE.Mesh(new THREE.BoxGeometry(2, 2.5, 5), new THREE.MeshStandardMaterial({color: 0xcc2222}));
            body.position.y = 1.5;
            vehicle.add(body);
            // Painted Patterns (Simplified)
            const decos = new THREE.Mesh(new THREE.BoxGeometry(2.1, 1.5, 3), new THREE.MeshStandardMaterial({color: 0x2288ff}));
            decos.position.y = 1.5;
            decos.position.z = 0.5;
            vehicle.add(decos);
        } else {
            // Mehran / Small car
            const body = new THREE.Mesh(new THREE.BoxGeometry(1.8, 1, 3.5), new THREE.MeshStandardMaterial({color: 0xeeeeee}));
            body.position.y = 0.8;
            vehicle.add(body);
            const top = new THREE.Mesh(new THREE.BoxGeometry(1.4, 0.6, 1.8), new THREE.MeshStandardMaterial({color: 0xeeeeee}));
            top.position.y = 1.5;
            vehicle.add(top);
        }

        // Common wheels
        const wheelGeo = new THREE.CylinderGeometry(0.4, 0.4, 0.3, 12);
        const wheelMat = new THREE.MeshStandardMaterial({color: 0x222222});
        for(let i=0; i<4; i++) {
            let w = new THREE.Mesh(wheelGeo, wheelMat);
            w.rotation.z = Math.PI/2;
            w.position.set(i < 2 ? 0.9 : -0.9, 0.4, i % 2 === 0 ? 1 : -1);
            vehicle.add(w);
        }

        scene.add(vehicle);
    }

    function createScenery() {
        // Buildings along the road
        for(let i = 0; i < 40; i++) {
            const h = 5 + Math.random() * 15;
            const w = 4 + Math.random() * 4;
            const geo = new THREE.BoxGeometry(w, h, w);
            // Random Pakistani building colors (sandy, white, brick)
            const colors = [0xd2b48c, 0xffffff, 0xa52a2a, 0x808080];
            const mat = new THREE.MeshStandardMaterial({ color: colors[Math.floor(Math.random()*colors.length)] });
            const b = new THREE.Mesh(geo, mat);
            
            const side = i % 2 === 0 ? 1 : -1;
            b.position.set(side * (15 + Math.random() * 5), h/2, -i * 50);
            scene.add(b);
            buildings.push(b);
        }

        // Add a "Badshahi Mosque" style landmark
        const domeGeo = new THREE.SphereGeometry(6, 32, 32, 0, Math.PI * 2, 0, Math.PI/2);
        const domeMat = new THREE.MeshStandardMaterial({color: 0xffffff});
        const dome = new THREE.Mesh(domeGeo, domeMat);
        dome.position.set(-40, 0, -300);
        scene.add(dome);
    }

    function setupTouchControls() {
        const bind = (id, key) => {
            const el = document.getElementById(id);
            el.addEventListener('touchstart', (e) => { e.preventDefault(); input[key] = true; });
            el.addEventListener('touchend', (e) => { e.preventDefault(); input[key] = false; });
        };
        bind('btn-left', 'left');
        bind('btn-right', 'right');
        bind('btn-gas', 'gas');
        bind('btn-brake', 'brake');
    }

    function startGame(type) {
        document.getElementById('menu').style.display = 'none';
        document.getElementById('controls').style.display = 'flex';
        createVehicle(type);
        gameActive = true;
    }

    function update() {
        if (!gameActive) return;

        const delta = clock.getDelta();

        // Driving Logic
        if (input.gas) speed += 20 * delta;
        else speed -= 5 * delta;
        
        if (input.brake) speed -= 40 * delta;
        
        speed = Math.max(0, Math.min(speed, 120)); // Cap speed at 120

        // Steering
        if (input.left) vehicle.position.x -= speed * 0.1 * delta;
        if (input.right) vehicle.position.x += speed * 0.1 * delta;
        
        // Bounds
        vehicle.position.x = Math.max(-8, Math.min(8, vehicle.position.x));

        // Movement forward
        vehicle.position.z -= speed * 0.2 * delta;

        // Camera follow
        camera.position.z = vehicle.position.z + 8;
        camera.position.x = vehicle.position.x * 0.5;
        camera.lookAt(vehicle.position.x, 1, vehicle.position.z - 5);

        // Update UI
        document.getElementById('speed').innerText = Math.floor(speed);

        // Endless Road recycling (simple teleport for buildings)
        buildings.forEach(b => {
            if (b.position.z > camera.position.z + 20) {
                b.position.z -= 1000;
            }
        });

        // City zone feedback
        const dist = Math.abs(vehicle.position.z);
        const indicator = document.getElementById('city-indicator');
        if (dist < 1000) indicator.innerText = "City: LAHORE";
        else if (dist < 2000) indicator.innerText = "City: KARACHI";
        else indicator.innerText = "City: ISLAMABAD";
    }

    function onWindowResize() {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    }

    function animate() {
        requestAnimationFrame(animate);
        update();
        renderer.render(scene, camera);
    }

    window.onload = () => {
        init();
        animate();
    };
</script>
</body>
</html>

# Ppak-driver-
Welcome to Pakistan 

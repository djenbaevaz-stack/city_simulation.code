<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>City Simulator - Fixed Traffic & Controls</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            user-select: none;
        }

        body, html {
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #000;
            font-family: 'Segoe UI', Roboto, sans-serif;
        }

        #canvas-container {
            width: 100%;
            height: 100%;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }

        /* --- HUD --- */
        #hud {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            pointer-events: none;
            padding: 25px;
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            color: #fff;
            font-family: monospace;
        }

        .hud-panel {
            background: rgba(10, 15, 30, 0.75);
            padding: 20px;
            border-radius: 16px;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            pointer-events: auto;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }

        .left-controls {
            width: 270px;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .stat-group {
            display: flex;
            flex-direction: column;
            gap: 4px;
        }

        .stat-label {
            font-size: 11px;
            color: #94a3b8;
            font-weight: bold;
            letter-spacing: 1px;
        }

        .bar-outer {
            width: 100%;
            height: 14px;
            background: rgba(255,255,255,0.1);
            border-radius: 7px;
            overflow: hidden;
            border: 1px solid rgba(0,0,0,0.5);
        }

        .bar-inner {
            height: 100%;
            width: 100%;
            transition: width 0.1s ease;
        }

        #hp-bar { background: #22c55e; box-shadow: 0 0 10px #22c55e; }
        #fuel-bar { background: #eab308; box-shadow: 0 0 10px #eab308; }

        .telemetry {
            display: flex;
            justify-content: space-between;
            align-items: baseline;
            margin-top: 10px;
        }

        #speedometer {
            font-size: 36px;
            font-weight: bold;
            color: #f8fafc;
        }

        #speedometer span {
            font-size: 14px;
            color: #64748b;
        }

        #gear-indicator {
            font-size: 24px;
            font-weight: bold;
            color: #38bdf8;
            background: rgba(56, 189, 248, 0.1);
            padding: 2px 10px;
            border-radius: 8px;
        }

        #action-prompt {
            background: rgba(34, 197, 94, 0.25);
            border: 1px solid #22c55e;
            color: #4ade80;
            padding: 8px;
            text-align: center;
            font-size: 12px;
            border-radius: 6px;
            font-weight: bold;
            display: none;
        }

        .drift-btn-container {
            margin-top: 5px;
            width: 100%;
        }

        #drift-toggle-btn {
            background: #ef4444; 
            color: white;
            border: 1px solid rgba(255, 255, 255, 0.2);
            padding: 8px;
            font-size: 11px;
            font-weight: bold;
            border-radius: 8px;
            cursor: pointer;
            width: 100%;
        }

        #drift-toggle-btn.active {
            background: #22c55e; 
        }

        .right-controls {
            width: 180px;
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            gap: 10px;
        }

        #clock { font-size: 24px; font-weight: bold; color: #f1f5f9; }
        #weather-info { font-size: 11px; color: #cbd5e1; background: rgba(255,255,255,0.1); padding: 3px 8px; border-radius: 4px;}
        #control-mode { font-size: 12px; color: #38bdf8; font-weight: bold; letter-spacing: 1px; }

        #minimap {
            width: 130px;
            height: 130px;
            background: rgba(0,0,0,0.6);
            border: 2px solid rgba(255,255,255,0.2);
            border-radius: 50%;
            overflow: hidden;
        }
    </style>
</head>
<body>

    <div id="canvas-container"></div>

    <div id="hud">
        <div class="hud-panel left-controls">
            <div id="control-mode">REJIM: MASHINA (DRIVING)</div>
            <div id="action-prompt">MOSHINADAN TUSHISH UCHUN [F] BOSING</div>
            <div class="stat-group">
                <div class="stat-label">AVTOMOBIL HOLATI (HP)</div>
                <div class="bar-outer"><div id="hp-bar" class="bar-inner"></div></div>
            </div>
            <div class="stat-group">
                <div class="stat-label">BENZIN (FUEL)</div>
                <div class="bar-outer"><div id="fuel-bar" class="bar-inner"></div></div>
            </div>
            <div class="telemetry">
                <div id="speedometer">0 <span>KM/H</span></div>
                <div id="gear-indicator">P</div>
            </div>
            <div class="drift-btn-container">
                <button id="drift-toggle-btn">DRIFT MODE: OFF</button>
            </div>
        </div>

        <div class="hud-panel right-controls">
            <div id="clock">12:00</div>
            <div id="weather-info">QUYOSHLI</div>
            <div id="minimap">
                <canvas id="minimap-canvas" width="130" height="130"></canvas>
            </div>
        </div>
    </div>

    <script>
        const container = document.getElementById('canvas-container');
        const mCanvas = document.getElementById('minimap-canvas');
        const mctx = mCanvas.getContext('2d');

        let scene, camera, renderer, dirLight, ambientLight;
        let clock = new THREE.Clock();
        
        let playerCar; 
        let playerPed; 
        let trafficCars = [];
        let npcPeople = []; 
        let trafficLights = [];
        let buildings = [];
        let particles = [];

        let state = {
            paused: false,
            gameOver: false,
            driftMode: false,
            inCar: true, 
            activeVehicle: null,
            trafficPhase: 'X_GREEN', // Fazalar: 'X_GREEN' va 'Z_GREEN'
            trafficLightTimer: 0,
            gameTime: 12 * 60 // 12:00 dan boshlanadi (minut hisobida)
        };

        const WORLD_SIZE = 1200;
        const ROAD_WIDTH = 24;      
        const SIDEWALK_WIDTH = 4;   
        const GRID_SPACING = 200;   

        const keys = { w: false, a: false, s: false, d: false };
        
        window.addEventListener('keydown', (e) => {
            let k = e.key.toLowerCase();
            if (keys.hasOwnProperty(k)) keys[k] = true;
            
            if (k === 'f' && state.inCar) handleVehicleExit();
            if (k === 'g' && !state.inCar) handleVehicleEnter();
        });
        window.addEventListener('keyup', (e) => {
            let k = e.key.toLowerCase();
            if (keys.hasOwnProperty(k)) keys[k] = false;
        });

        const driftBtn = document.getElementById('drift-toggle-btn');
        driftBtn.addEventListener('click', () => {
            state.driftMode = !state.driftMode;
            driftBtn.innerText = state.driftMode ? "DRIFT MODE: ON" : "DRIFT MODE: OFF";
            driftBtn.classList.toggle('active', state.driftMode);
        });

        const materials = {
            asphalt: new THREE.MeshStandardMaterial({ color: 0x22252a, roughness: 0.85 }),
            sidewalk: new THREE.MeshStandardMaterial({ color: 0x64748b, roughness: 0.7 }), 
            grass: new THREE.MeshStandardMaterial({ color: 0x112211, roughness: 0.95 }),
            building: new THREE.MeshStandardMaterial({ color: 0x334155, roughness: 0.6 }),
            windowLight: new THREE.MeshBasicMaterial({ color: 0xfde047 }), 
            windowDark: new THREE.MeshBasicMaterial({ color: 0x1e293b }),  
            shopFront: new THREE.MeshStandardMaterial({ color: 0x0284c7, emissive: 0x075985 }),
            bmwBody: new THREE.MeshStandardMaterial({ color: 0x0a192f, roughness: 0.15, metalness: 0.85 }),
            aiBody: new THREE.MeshStandardMaterial({ color: 0xb91c1c, roughness: 0.3 }),
            chrome: new THREE.MeshStandardMaterial({ color: 0xcccccc, metalness: 0.9, roughness: 0.1 }),
            glass: new THREE.MeshStandardMaterial({ color: 0x0f172a, transparent: true, opacity: 0.65 }),
            wheel: new THREE.MeshStandardMaterial({ color: 0x090d16, roughness: 0.9 }),
            pedSkin: new THREE.MeshStandardMaterial({ color: 0x0ea5e9 }), 
            npcSkin: new THREE.MeshStandardMaterial({ color: 0xec4899 }), 
            smoke: new THREE.MeshBasicMaterial({ color: 0xcccccc, transparent: true, opacity: 0.25 })
        };

        class SmokeParticle {
            constructor(x, y, z) {
                this.mesh = new THREE.Mesh(new THREE.SphereGeometry(0.3, 4, 4), materials.smoke);
                this.mesh.position.set(x, y, z);
                scene.add(this.mesh);
                this.vy = 0.5 + Math.random();
                this.life = 1.0;
            }
            update(delta) {
                this.mesh.position.y += this.vy * delta;
                this.life -= delta * 2;
                this.mesh.scale.multiplyScalar(1 + delta * 0.4);
                if (this.life <= 0) { scene.remove(this.mesh); return false; }
                return true;
            }
        }

        class Pedestrian {
            constructor(x, z, isPlayer = false) {
                this.x = x;
                this.z = z;
                this.y = 0.1;
                this.angle = 0;
                this.speed = 3.5;
                this.isPlayer = isPlayer;

                this.roadLine = Math.random() > 0.5 ? 'X' : 'Z';
                this.fixedOffset = (Math.floor(Math.random() * 4) - 2) * GRID_SPACING + (Math.random() > 0.5 ? 14 : -14);

                this.mesh = new THREE.Group();
                let body = new THREE.Mesh(new THREE.CylinderGeometry(0.25, 0.25, 1.3), isPlayer ? materials.pedSkin : materials.npcSkin);
                body.position.y = 0.85;
                body.castShadow = true;
                this.mesh.add(body);

                let head = new THREE.Mesh(new THREE.SphereGeometry(0.22), isPlayer ? materials.pedSkin : materials.npcSkin);
                head.position.y = 1.6;
                this.mesh.add(head);

                this.mesh.position.set(this.x, this.y, this.z);
                if (!isPlayer) scene.add(this.mesh);
            }

            update(delta) {
                if (this.isPlayer) {
                    let moveX = 0;
                    let moveZ = 0;
                    if (keys.w) moveZ = -1;
                    if (keys.s) moveZ = 1;
                    if (keys.a) moveX = -1;
                    if (keys.d) moveX = 1;

                    if (moveX !== 0 || moveZ !== 0) {
                        this.angle = Math.atan2(moveX, moveZ);
                        this.x += Math.sin(this.angle) * this.speed * delta;
                        this.z += Math.cos(this.angle) * this.speed * delta;
                        this.mesh.rotation.y = this.angle;
                    }
                } else {
                    if (this.roadLine === 'X') {
                        this.z = this.fixedOffset; 
                        this.x += this.speed * 0.4 * (this.angle === 0 ? 1 : -1) * delta;
                        this.mesh.rotation.y = this.angle === 0 ? Math.PI/2 : -Math.PI/2;
                        if (Math.abs(this.x) > WORLD_SIZE / 2) this.angle = this.angle === 0 ? Math.PI : 0;
                    } else {
                        this.x = this.fixedOffset; 
                        this.z += this.speed * 0.4 * (this.angle === 0 ? 1 : -1) * delta;
                        this.mesh.rotation.y = this.angle === 0 ? 0 : Math.PI;
                        if (Math.abs(this.z) > WORLD_SIZE / 2) this.angle = this.angle === 0 ? Math.PI : 0;
                    }
                }
                this.mesh.position.set(this.x, this.y, this.z);
            }
        }

        class Vehicle {
            constructor(x, z, isPlayerCar = false, fixedLine = null) {
                this.mesh = new THREE.Group();
                this.isPlayerCar = isPlayerCar;
                
                this.x = x;
                this.z = z;
                this.y = 0.4;
                
                this.fixedLine = fixedLine; 
                this.fixedCoord = (fixedLine === 'X') ? z : x; 

                this.angle = isPlayerCar ? 0 : (fixedLine === 'X' ? 0 : Math.PI/2);
                this.speed = isPlayerCar ? 0 : 8; 
                
                this.maxSpeed = isPlayerCar ? 26 : 8.5;
                this.accel = 0.3;
                this.brakeForce = 0.7;
                this.steerAngle = 0;

                this.hp = 100;
                this.fuel = 100;
                this.gear = 'P';

                this.createGeometry();
                scene.add(this.mesh);
            }

            createGeometry() {
                let mat = this.isPlayerCar ? materials.bmwBody : materials.aiBody;
                if(!this.isPlayerCar && Math.random() > 0.5) {
                    mat = new THREE.MeshStandardMaterial({color: 0x1e3a8a, roughness: 0.3});
                }
                let body = new THREE.Mesh(new THREE.BoxGeometry(4.3, 0.95, 1.8), mat);
                body.position.y = 0.5;
                body.castShadow = true;
                this.mesh.add(body);

                let glass = new THREE.Mesh(new THREE.BoxGeometry(2.1, 0.55, 1.45), materials.glass);
                glass.position.set(-0.1, 1.15, 0);
                this.mesh.add(glass);

                const wGeom = new THREE.CylinderGeometry(0.38, 0.38, 0.3, 12).rotateX(Math.PI/2);
                this.wheels = [];
                const wPos = [{x:1.2, z:0.85}, {x:1.2, z:-0.85}, {x:-1.2, z:0.85}, {x:-1.2, z:-0.85}];
                wPos.forEach(p => {
                    let w = new THREE.Mesh(wGeom, materials.wheel);
                    w.position.set(p.x, 0.38, p.z);
                    this.mesh.add(w);
                    this.wheels.push(w);
                });
            }

            update(delta) {
                if (state.inCar && state.activeVehicle === this) {
                    if (this.hp <= 0 || this.fuel <= 0) { this.speed *= 0.92; this.applyMovement(); return; }
                    this.fuel = Math.max(0, this.fuel - delta * 0.03);

                    if (keys.w) {
                        this.speed = Math.min(this.maxSpeed, this.speed + this.accel);
                        this.gear = 'D';
                    } else if (keys.s) {
                        this.speed = Math.max(-7, this.speed - this.brakeForce);
                        this.gear = 'R';
                    } else {
                        this.speed *= 0.94;
                        this.gear = 'N';
                    }

                    if (keys.a) this.steerAngle = Math.min(0.4, this.steerAngle + 0.035);
                    else if (keys.d) this.steerAngle = Math.max(-0.4, this.steerAngle - 0.035);
                    else this.steerAngle *= 0.72;

                    this.wheels[0].rotation.y = this.steerAngle;
                    this.wheels[1].rotation.y = this.steerAngle;

                    if (Math.abs(this.speed) > 0.5) {
                        let driftMult = state.driftMode ? 1.7 : 1.0;
                        this.angle += this.steerAngle * (this.speed / 24) * driftMult;
                    }

                    if (state.driftMode && Math.abs(this.steerAngle) > 0.25 && this.speed > 8) {
                        particles.push(new SmokeParticle(this.x, 0.2, this.z));
                    }

                    this.x += Math.cos(this.angle) * this.speed * delta;
                    this.z -= Math.sin(this.angle) * this.speed * delta;

                } else {
                    this.gear = 'D';
                    let shouldStop = false;

                    trafficLights.forEach(tl => {
                        if (this.fixedLine === 'X') {
                            let dist = tl.x - this.x;
                            if (state.trafficPhase === 'Z_GREEN' && dist > 4 && dist < 22 && Math.abs(this.z - tl.z) < 15) {
                                shouldStop = true;
                            }
                        } else if (this.fixedLine === 'Z') {
                            let dist = tl.z - this.z;
                            if (state.trafficPhase === 'X_GREEN' && dist > 4 && dist < 22 && Math.abs(this.x - tl.x) < 15) {
                                shouldStop = true;
                            }
                        }
                    });

                    trafficCars.forEach(other => {
                        if (other !== this && other.fixedLine === this.fixedLine) {
                            if (this.fixedLine === 'X' && other.x > this.x && (other.x - this.x) < 15) {
                                shouldStop = true;
                            }
                            if (this.fixedLine === 'Z' && other.z > this.z && (other.z - this.z) < 15) {
                                shouldStop = true;
                            }
                        }
                    });

                    if (shouldStop) {
                        this.speed = Math.max(0, this.speed - 0.5); 
                    } else {
                        this.speed = Math.min(this.maxSpeed, this.speed + 0.2); 
                    }

                    if (this.fixedLine === 'X') {
                        this.z = this.fixedCoord; 
                        this.x += this.speed * delta;
                        this.angle = 0;
                        if (this.x > WORLD_SIZE / 2) this.x = -WORLD_SIZE / 2; 
                    } else {
                        this.x = this.fixedCoord; 
                        this.z += this.speed * delta;
                        this.angle = -Math.PI / 2;
                        if (this.z > WORLD_SIZE / 2) this.z = -WORLD_SIZE / 2;
                    }
                }

                this.applyMovement();
            }

            applyMovement() {
                this.mesh.position.set(this.x, 0, this.z);
                this.mesh.rotation.y = this.angle;
                this.wheels.forEach(w => w.rotation.z -= this.speed * 0.12);
            }
        }

        function handleVehicleExit() {
            state.inCar = false;
            playerPed.x = state.activeVehicle.x + 3.5; 
            playerPed.z = state.activeVehicle.z;
            scene.add(playerPed.mesh); 
            state.activeVehicle = null;
            
            document.getElementById('control-mode').innerText = "REJIM: PIYODA (WALKING)";
            document.getElementById('control-mode').style.color = "#f43f5e";
        }

        function handleVehicleEnter() {
            let closestVehicle = null;
            let maxRadius = 15; 

            let dStart = Math.hypot(playerPed.x - playerCar.x, playerPed.z - playerCar.z);
            if (dStart < maxRadius) {
                closestVehicle = playerCar;
                maxRadius = dStart;
            }

            trafficCars.forEach(car => {
                let d = Math.hypot(playerPed.x - car.x, playerPed.z - car.z);
                if (d < maxRadius) {
                    closestVehicle = car;
                    maxRadius = d;
                }
            });

            if (closestVehicle) {
                state.inCar = true;
                state.activeVehicle = closestVehicle;
                scene.remove(playerPed.mesh); 
                
                document.getElementById('control-mode').innerText = "REJIM: MASHINA (DRIVING)";
                document.getElementById('control-mode').style.color = "#38bdf8";
            }
        }

        function createTrafficLight(x, z) {
            let pole = new THREE.Group();
            pole.add(new THREE.Mesh(new THREE.CylinderGeometry(0.12, 0.12, 5.5), materials.chrome));
            
            let box = new THREE.Mesh(new THREE.BoxGeometry(0.35, 1.1, 0.35), materials.wheel);
            box.position.y = 5.2;
            pole.add(box);

            let greenLight = new THREE.Mesh(new THREE.SphereGeometry(0.15), new THREE.MeshBasicMaterial({color: 0x003300}));
            greenLight.position.set(0, 4.8, 0.19);
            let redLight = new THREE.Mesh(new THREE.SphereGeometry(0.15), new THREE.MeshBasicMaterial({color: 0x330000}));
            redLight.position.set(0, 5.4, 0.19);

            pole.add(greenLight, redLight);
            pole.position.set(x, 0, z);
            scene.add(pole);
            trafficLights.push({ x: x, z: z, g: greenLight, r: redLight });
        }

        function createComplexBuilding(x, z) {
            let h = 25 + Math.random() * 35;
            let w = 26;
            let d = 26;

            let mainMesh = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), materials.building);
            mainMesh.position.set(x, h/2, z);
            mainMesh.castShadow = true;
            scene.add(mainMesh);
            buildings.push({ x: x, z: z, w: w, h: h, d: d });

            let shop = new THREE.Mesh(new THREE.BoxGeometry(w + 0.1, 3.8, d + 0.1), materials.shopFront);
            shop.position.set(x, 1.9, z);
            scene.add(shop);

            for (let floor = 5; floor < h - 2; floor += 4) {
                for (let side = -8; side <= 8; side += 4) {
                    let win = new THREE.Mesh(new THREE.PlaneGeometry(1.1, 1.6), Math.random() > 0.4 ? materials.windowLight : materials.windowDark);
                    win.position.set(x + side, floor, z + d/2 + 0.02);
                    scene.add(win);
                }
            }
        }

        function initWorld() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x070a13);
            scene.fog = new THREE.FogExp2(0x070a13, 0.0035);

            camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000);
            renderer = new THREE.WebGLRenderer({ antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.shadowMap.enabled = true;
            container.appendChild(renderer.domElement);

            let ground = new THREE.Mesh(new THREE.PlaneGeometry(WORLD_SIZE, WORLD_SIZE), materials.grass);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            scene.add(ground);

            for (let pos = -WORLD_SIZE / 2; pos <= WORLD_SIZE / 2; pos += GRID_SPACING) {
                if (pos === 0) continue;

                let roadX = new THREE.Mesh(new THREE.PlaneGeometry(WORLD_SIZE, ROAD_WIDTH), materials.asphalt);
                roadX.rotation.x = -Math.PI / 2; roadX.position.set(0, 0.02, pos); scene.add(roadX);

                let roadZ = new THREE.Mesh(new THREE.PlaneGeometry(ROAD_WIDTH, WORLD_SIZE), materials.asphalt);
                roadZ.rotation.x = -Math.PI / 2; roadZ.position.set(pos, 0.02, 0); scene.add(roadZ);

                let swX1 = new THREE.Mesh(new THREE.PlaneGeometry(WORLD_SIZE, SIDEWALK_WIDTH), materials.sidewalk);
                swX1.rotation.x = -Math.PI / 2; swX1.position.set(0, 0.06, pos + ROAD_WIDTH/2 + SIDEWALK_WIDTH/2); scene.add(swX1);
                let swX2 = new THREE.Mesh(new THREE.PlaneGeometry(WORLD_SIZE, SIDEWALK_WIDTH), materials.sidewalk);
                swX2.rotation.x = -Math.PI / 2; swX2.position.set(0, 0.06, pos - ROAD_WIDTH/2 - SIDEWALK_WIDTH/2); scene.add(swX2);

                let swZ1 = new THREE.Mesh(new THREE.PlaneGeometry(SIDEWALK_WIDTH, WORLD_SIZE), materials.sidewalk);
                swZ1.rotation.x = -Math.PI / 2; swZ1.position.set(pos + ROAD_WIDTH/2 + SIDEWALK_WIDTH/2, 0.06, 0); scene.add(swZ1);
                let swZ2 = new THREE.Mesh(new THREE.PlaneGeometry(SIDEWALK_WIDTH, WORLD_SIZE), materials.sidewalk);
                swZ2.rotation.x = -Math.PI / 2; swZ2.position.set(pos - ROAD_WIDTH/2 - SIDEWALK_WIDTH/2, 0.06, 0); scene.add(swZ2);

                createTrafficLight(pos - 16, pos + 16);
                createTrafficLight(pos + 16, -pos - 16);
            }

            for (let x = -350; x <= 350; x += 75) {
                for (let z = -350; z <= 350; z += 75) {
                    if (Math.abs(x) % GRID_SPACING > 35 && Math.abs(z) % GRID_SPACING > 35) {
                        createComplexBuilding(x, z);
                    }
                }
            }

            dirLight = new THREE.DirectionalLight(0xfffaed, 1.0);
            dirLight.position.set(100, 180, 80);
            scene.add(dirLight);
            ambientLight = new THREE.AmbientLight(0xffffff, 0.4);
            scene.add(ambientLight);

            playerCar = new Vehicle(0, 204, true);    
            playerPed = new Pedestrian(0, 204, true);  

            state.activeVehicle = playerCar; 
            state.inCar = true; 

            for (let i = 0; i < 20; i++) {
                let randomGrid = (Math.floor(Math.random() * 3) - 1) * GRID_SPACING;
                if(randomGrid === 0) randomGrid = GRID_SPACING;

                let carX = new Vehicle((i * -50) + 100, randomGrid + 4, false, 'X');
                trafficCars.push(carX);

                let carZ = new Vehicle(randomGrid - 4, (i * -50) + 100, false, 'Z');
                trafficCars.push(carZ);
            }

            for (let i = 0; i < 35; i++) {
                let npc = new Pedestrian((Math.random() - 0.5) * 500, (Math.random() - 0.5) * 500, false);
                npcPeople.push(npc);
            }
        }

        function updateTrafficLights(delta) {
            state.trafficLightTimer += delta;
            
            if (state.trafficLightTimer > 6.0) {
                state.trafficPhase = (state.trafficPhase === 'X_GREEN') ? 'Z_GREEN' : 'X_GREEN';
                state.trafficLightTimer = 0;
            }

            trafficLights.forEach(tl => {
                if (state.trafficPhase === 'X_GREEN') {
                    tl.g.material.color.setHex(0x00ff00); 
                    tl.r.material.color.setHex(0x330000); 
                } else {
                    tl.g.material.color.setHex(0x003300); 
                    tl.r.material.color.setHex(0xff0000); 
                }
            });
        }

        function updateHUD(delta) {
            // --- 1. SOAT TIZIMI (Dinamik yangilanish) ---
            state.gameTime += delta * 2; 
            let hours = Math.floor(state.gameTime / 60) % 24;
            let minutes = Math.floor(state.gameTime % 60);
            document.getElementById('clock').innerText = 
                `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`;

            // --- 2. MINI-PROMPT VA STATUS BARLAR ---
            let prompt = document.getElementById('action-prompt');
            let currentX = state.inCar ? state.activeVehicle.x : playerPed.x;
            let currentZ = state.inCar ? state.activeVehicle.z : playerPed.z;

            if (!state.inCar) {
                let dist = Math.hypot(playerPed.x - playerCar.x, playerPed.z - playerCar.z);
                let showPrompt = (dist < 15);
                
                trafficCars.forEach(c => {
                    if(Math.hypot(playerPed.x - c.x, playerPed.z - c.z) < 15) showPrompt = true;
                });
                
                prompt.style.display = showPrompt ? 'block' : 'none';
                prompt.innerText = "MOSHINAGA QAYTA O'TIRISH UCHUN [G] BOSING";
                
                document.getElementById('hp-bar').style.width = `100%`;
                document.getElementById('fuel-bar').style.width = `100%`;
                document.getElementById('speedometer').innerHTML = `0 <span>KM/H</span>`;
            } else {
                prompt.style.display = 'block';
                prompt.innerText = "MOSHINADAN TUSHISH UCHUN [F] BOSING";
                
                let v = state.activeVehicle;
                document.getElementById('hp-bar').style.width = `${v.hp}%`;
                document.getElementById('fuel-bar').style.width = `${v.fuel}%`;
                document.getElementById('speedometer').innerHTML = `${Math.round(Math.abs(v.speed) * 6.8)} <span>KM/H</span>`;
                document.getElementById('gear-indicator').innerText = v.gear;
            }

            // --- 3. TO'LIQ FUNKSIONAL MINIMAP CHIZISH ---
            mctx.clearRect(0, 0, 130, 130);

            mctx.fillStyle = 'rgba(11, 15, 26, 0.85)';
            mctx.beginPath(); 
            mctx.arc(65, 65, 65, 0, Math.PI * 2); 
            mctx.fill();

            mctx.save();
            mctx.beginPath();
            mctx.arc(65, 65, 63, 0, Math.PI * 2);
            mctx.clip(); 

            let zoom = 0.4; 
            mctx.translate(65, 65);

            mctx.fillStyle = 'rgba(50, 55, 65, 0.6)';
            for (let pos = -WORLD_SIZE / 2; pos <= WORLD_SIZE / 2; pos += GRID_SPACING) {
                if (pos === 0) continue;
                let rx = (0 - currentX) * zoom;
                let ry = (pos - currentZ) * zoom;
                mctx.fillRect(rx - (WORLD_SIZE/2)*zoom, ry - (ROAD_WIDTH/2)*zoom, WORLD_SIZE * zoom, ROAD_WIDTH * zoom);

                let zx = (pos - currentX) * zoom;
                let zy = (0 - currentZ) * zoom;
                mctx.fillRect(zx - (ROAD_WIDTH/2)*zoom, zy - (WORLD_SIZE/2)*zoom, ROAD_WIDTH * zoom, WORLD_SIZE * zoom);
            }

            mctx.fillStyle = 'rgba(255, 255, 255, 0.08)';
            buildings.forEach(b => {
                let bx = (b.x - currentX) * zoom;
                let bz = (b.z - currentZ) * zoom;
                mctx.fillRect(bx - (b.w/2)*zoom, bz - (b.d/2)*zoom, b.w * zoom, b.d * zoom);
            });

            mctx.restore();

            mctx.fillStyle = '#38bdf8';
            mctx.beginPath(); 
            mctx.arc(65, 65, 4.5, 0, Math.PI * 2); 
            mctx.fill();
            
            let currentAngle = state.inCar ? state.activeVehicle.angle : playerPed.angle;
            mctx.strokeStyle = '#38bdf8';
            mctx.lineWidth = 2;
            mctx.beginPath();
            mctx.moveTo(65, 65);
            mctx.lineTo(65 + Math.cos(currentAngle) * 10, 65 - Math.sin(currentAngle) * 10);
            mctx.stroke();
        }

        function updateCamera() {
            if (state.inCar) {
                let v = state.activeVehicle;
                camera.position.set(v.x - Math.cos(v.angle)*9.5, v.y + 4.0, v.z + Math.sin(v.angle)*9.5);
                camera.lookAt(v.x + Math.cos(v.angle)*2, v.y, v.z - Math.sin(v.angle)*2);
            } else {
                camera.position.set(playerPed.x - Math.sin(playerPed.angle)*6.5, playerPed.y + 3.2, playerPed.z - Math.cos(playerPed.angle)*6.5);
                camera.lookAt(playerPed.x, playerPed.y + 0.8, playerPed.z);
            }
        }

        // --- MANA SHU YERDAN KODINGIZ TUZATILDI VA OXIRIGA YETKAZILDI ---
        function tick() {
            let delta = Math.min(clock.getDelta(), 0.1);

            // O'yin pauza yoki tugagan bo'lsa, tsiklni to'xtatish
            if (state.paused || state.gameOver) {
                requestAnimationFrame(tick);
                return;
            }

            // O'yinchi holatini yangilash (Piyoda yoki Mashinada)
            if (state.inCar && state.activeVehicle) {
                state.activeVehicle.update(delta);
            } else {
                playerPed.update(delta);
            }

            // Svetoforlar va yo'l harakatini yangilash
            updateTrafficLights(delta);

            // Boshqa AI mashinalarni yangilash
            trafficCars.forEach(car => car.update(delta));

            // Ko'chadagi NPC odamlarni yangilash
            npcPeople.forEach(npc => npc.update(delta));

            // Smoke (Drift tutunlari) effektini yangilash
            for (let i = particles.length - 1; i >= 0; i--) {
                let alive = particles[i].update(delta);
                if (!alive) particles.splice(i, 1);
            }

            // Kamera, Minimap va HUD interfeyslarini yangilash
            updateCamera();
            updateHUD(delta);

            // Three.js 3D sahnani ekranga chizish (Render)
            renderer.render(scene, camera);

            // Keyingi kadrni chaqirish
            requestAnimationFrame(tick);
        }

        // O'yin oynasi o'lchami o'zgarganda kamerani to'g'rilash
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        // Dunyoni yaratish va animatsiyani boshlash
        initWorld();
        tick();
    </script>
</body>
</html>

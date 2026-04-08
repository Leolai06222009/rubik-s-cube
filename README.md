<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3x3x3 魔術方塊</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #222; font-family: sans-serif; }
        #ui {
            position: absolute;
            top: 20px;
            left: 20px;
            color: white;
            z-index: 100;
            pointer-events: none;
        }
        .controls {
            pointer-events: auto;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        button {
            padding: 10px 15px;
            font-size: 16px;
            cursor: pointer;
            background: #444;
            color: white;
            border: 1px solid #666;
            border-radius: 5px;
            transition: background 0.2s;
        }
        button:hover { background: #666; }
        button:disabled { opacity: 0.5; cursor: not-allowed; }
        #message {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #0f0;
            font-size: 48px;
            font-weight: bold;
            display: none;
            text-shadow: 0 0 10px rgba(0,255,0,0.5);
            pointer-events: none;
            z-index: 200;
        }
        .instruction {
            font-size: 14px;
            margin-top: 10px;
            color: #aaa;
        }
    </style>
</head>
<body>

<div id="ui">
    <h1>3x3x3 魔術方塊</h1>
    <div class="controls">
        <button id="scrambleBtn">打亂方塊 (Scramble)</button>
        <div style="margin-top:10px;">
            <div>旋轉控制 (Rotations):</div>
            <div style="display:grid; grid-template-columns: repeat(3, 1fr); gap: 5px; margin-top:5px;">
                <button onclick="rotate('x', 0, 1)">L</button>
                <button onclick="rotate('x', 1, 1)">M</button>
                <button onclick="rotate('x', 2, 1)">R'</button>
                <button onclick="rotate('x', 0, -1)">L'</button>
                <button onclick="rotate('x', 1, -1)">M'</button>
                <button onclick="rotate('x', 2, -1)">R</button>
            </div>
            <div style="display:grid; grid-template-columns: repeat(3, 1fr); gap: 5px; margin-top:5px;">
                <button onclick="rotate('y', 2, 1)">U</button>
                <button onclick="rotate('y', 1, 1)">E</button>
                <button onclick="rotate('y', 0, 1)">D'</button>
                <button onclick="rotate('y', 2, -1)">U'</button>
                <button onclick="rotate('y', 1, -1)">E'</button>
                <button onclick="rotate('y', 0, -1)">D</button>
            </div>
            <div style="display:grid; grid-template-columns: repeat(3, 1fr); gap: 5px; margin-top:5px;">
                <button onclick="rotate('z', 2, 1)">F</button>
                <button onclick="rotate('z', 1, 1)">S</button>
                <button onclick="rotate('z', 0, 1)">B'</button>
                <button onclick="rotate('z', 2, -1)">F'</button>
                <button onclick="rotate('z', 1, -1)">S'</button>
                <button onclick="rotate('z', 0, -1)">B</button>
            </div>
        </div>
        <div class="instruction">使用滑鼠拖曳來旋轉視角</div>
    </div>
</div>

<div id="message">恭喜通關！</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>

<script>
    // --- Setup Scene ---
    const scene = new THREE.Scene();
    const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(5, 5, 10);

    const renderer = new THREE.WebGLRenderer({ antialias: true });
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.setPixelRatio(window.devicePixelRatio);
    document.body.appendChild(renderer.domElement);

    const controls = new THREE.OrbitControls(camera, renderer.domElement);
    controls.enableDamping = true;

    const ambientLight = new THREE.AmbientLight(0xffffff, 0.8);
    scene.add(ambientLight);

    const directionalLight = new THREE.DirectionalLight(0xffffff, 0.5);
    directionalLight.position.set(10, 20, 10);
    scene.add(directionalLight);

    // --- Cube Constants ---
    const COLORS = {
        white:  0xffffff, // Up
        yellow: 0xffff00, // Down
        red:    0xff0000, // Left
        orange: 0xffa500, // Right
        green:  0x00ff00, // Front
        blue:   0x0000ff  // Back
    };

    // --- Create Cubelets ---
    const cubelets = [];
    const size = 1;
    const gap = 0.05;

    function createCubelet(x, y, z) {
        const geometry = new THREE.BoxGeometry(size, size, size);
        
        // Define materials for each side
        // order: pos-x, neg-x, pos-y, neg-y, pos-z, neg-z
        // R (orange), L (red), U (white), D (yellow), F (green), B (blue)
        const materials = [
            new THREE.MeshLambertMaterial({ color: (x === 1)  ? COLORS.orange : 0x111111 }),
            new THREE.MeshLambertMaterial({ color: (x === -1) ? COLORS.red    : 0x111111 }),
            new THREE.MeshLambertMaterial({ color: (y === 1)  ? COLORS.white  : 0x111111 }),
            new THREE.MeshLambertMaterial({ color: (y === -1) ? COLORS.yellow : 0x111111 }),
            new THREE.MeshLambertMaterial({ color: (z === 1)  ? COLORS.green  : 0x111111 }),
            new THREE.MeshLambertMaterial({ color: (z === -1) ? COLORS.blue   : 0x111111 })
        ];

        const cubelet = new THREE.Mesh(geometry, materials);
        cubelet.position.set(x * (size + gap), y * (size + gap), z * (size + gap));
        
        // Custom property to track original face colors for win detection
        // We'll use the final orientation instead.
        
        scene.add(cubelet);
        cubelets.push(cubelet);
    }

    for (let x = -1; x <= 1; x++) {
        for (let y = -1; y <= 1; y++) {
            for (let z = -1; z <= 1; z++) {
                if (x === 0 && y === 0 && z === 0) continue; // Hollow center
                createCubelet(x, y, z);
            }
        }
    }

    // --- Rotation Logic ---
    let isRotating = false;

    /**
     * @param {string} axis - 'x', 'y', or 'z'
     * @param {number} layerIndex - -1, 0, or 1 (mapped from 0, 1, 2)
     * @param {number} direction - 1 (CW) or -1 (CCW)
     */
    function rotate(axis, layerIndex, direction, speed = 15) {
        if (isRotating) return;
        isRotating = true;

        // Map layer index (0,1,2) to internal coordinates (-1,0,1)
        const targetLayer = layerIndex - 1;
        
        const group = new THREE.Group();
        scene.add(group);

        const movingCubelets = [];
        const threshold = 0.1;

        cubelets.forEach(c => {
            // Check position in world space
            const worldPos = new THREE.Vector3();
            c.getWorldPosition(worldPos);
            
            let match = false;
            if (axis === 'x' && Math.abs(worldPos.x / (size + gap) - targetLayer) < threshold) match = true;
            if (axis === 'y' && Math.abs(worldPos.y / (size + gap) - targetLayer) < threshold) match = true;
            if (axis === 'z' && Math.abs(worldPos.z / (size + gap) - targetLayer) < threshold) match = true;

            if (match) {
                movingCubelets.push(c);
            }
        });

        movingCubelets.forEach(c => {
            group.attach(c);
        });

        const targetRotation = (Math.PI / 2) * direction;
        let currentRotation = 0;
        const step = (targetRotation / speed);

        function animateRotation() {
            if (Math.abs(currentRotation) < Math.abs(targetRotation)) {
                group.rotation[axis] += step;
                currentRotation += step;
                requestAnimationFrame(animateRotation);
            } else {
                // Snap to final rotation
                group.rotation[axis] = targetRotation;
                
                // Finalize positions
                const toUngroup = [...movingCubelets];
                toUngroup.forEach(c => {
                    scene.attach(c);
                });
                
                scene.remove(group);
                isRotating = false;
                checkSolved();
            }
        }

        animateRotation();
    }

    // --- Scramble ---
    const scrambleBtn = document.getElementById('scrambleBtn');
    scrambleBtn.addEventListener('click', async () => {
        if (isRotating) return;
        scrambleBtn.disabled = true;
        document.getElementById('message').style.display = 'none';

        const moves = 20;
        const axes = ['x', 'y', 'z'];
        for (let i = 0; i < moves; i++) {
            const axis = axes[Math.floor(Math.random() * 3)];
            const layer = Math.floor(Math.random() * 3);
            const dir = Math.random() > 0.5 ? 1 : -1;
            
            rotate(axis, layer, dir, 5); // Faster rotation for scramble
            
            // Wait for rotation to finish
            while (isRotating) {
                await new Promise(r => setTimeout(r, 10));
            }
        }
        scrambleBtn.disabled = false;
    });

    // --- Win Detection ---
    function checkSolved() {
        // A simple way to check if solved:
        // Each cubelet should have its rotation close to a multiple of 90 degrees
        // and aligned such that all outward stickers match.
        // Even simpler for this implementation: check if the normals of the colored faces
        // are all aligned correctly in world space.
        
        const faces = [
            { normal: new THREE.Vector3(1, 0, 0), color: COLORS.orange },
            { normal: new THREE.Vector3(-1, 0, 0), color: COLORS.red },
            { normal: new THREE.Vector3(0, 1, 0), color: COLORS.white },
            { normal: new THREE.Vector3(0, -1, 0), color: COLORS.yellow },
            { normal: new THREE.Vector3(0, 0, 1), color: COLORS.green },
            { normal: new THREE.Vector3(0, 0, -1), color: COLORS.blue }
        ];

        let allSolved = true;

        for (const face of faces) {
            // Find all cubelets that are on this face in current world space
            const faceCubelets = cubelets.filter(c => {
                const worldPos = new THREE.Vector3();
                c.getWorldPosition(worldPos);
                const dot = worldPos.dot(face.normal);
                return dot > (size + gap) * 0.9;
            });

            // For each cubelet on this face, check if the material facing this direction has the correct color
            for (const c of faceCubelets) {
                // We need to check which of its 6 faces is currently pointing towards `face.normal`
                let materialFound = false;
                
                // Get world normals of the cubelet faces
                // Index 0: +x, 1: -x, 2: +y, 3: -y, 4: +z, 5: -z
                const localNormals = [
                    new THREE.Vector3(1, 0, 0), new THREE.Vector3(-1, 0, 0),
                    new THREE.Vector3(0, 1, 0), new THREE.Vector3(0, -1, 0),
                    new THREE.Vector3(0, 0, 1), new THREE.Vector3(0, 0, -1)
                ];

                for (let i = 0; i < 6; i++) {
                    const worldNormal = localNormals[i].clone().applyQuaternion(c.quaternion);
                    if (worldNormal.dot(face.normal) > 0.9) {
                        // This is the material facing our face direction
                        if (c.material[i].color.getHex() !== face.color) {
                            allSolved = false;
                            break;
                        }
                        materialFound = true;
                        break;
                    }
                }
                if (!allSolved) break;
            }
            if (!allSolved) break;
        }

        if (allSolved) {
            document.getElementById('message').style.display = 'block';
        }
    }

    // --- Animation Loop ---
    function animate() {
        requestAnimationFrame(animate);
        controls.update();
        renderer.render(scene, camera);
    }

    window.addEventListener('resize', () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    });

    animate();
</script>

</body>
</html>

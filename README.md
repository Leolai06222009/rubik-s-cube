<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <title>3D 魔術方塊 - 互動版</title>
    <style>
        body { margin: 0; background: #1a1a1a; color: white; font-family: sans-serif; overflow: hidden; }
        #ui { position: absolute; top: 10px; left: 10px; z-index: 10; background: rgba(0,0,0,0.7); padding: 15px; border-radius: 8px; }
        button { margin: 5px; padding: 8px 12px; cursor: pointer; background: #444; color: white; border: 1px solid #666; border-radius: 4px; }
        button:hover { background: #666; }
        .status { margin-top: 10px; color: #0f0; font-weight: bold; }
        #info { position: absolute; bottom: 10px; width: 100%; text-align: center; pointer-events: none; color: #888; }
    </style>
</head>
<body>

<div id="ui">
    <h3>控制面板</h3>
    <div>
        <strong>旋轉軸：</strong><br>
        X 軸 (列): <button onclick="rotateLayer('x', 0)">左層</button><button onclick="rotateLayer('x', 1)">中層</button><button onclick="rotateLayer('x', 2)">右層</button><br>
        Y 軸 (行): <button onclick="rotateLayer('y', 0)">下層</button><button onclick="rotateLayer('y', 1)">中層</button><button onclick="rotateLayer('y', 2)">上層</button><br>
        Z 軸: <button onclick="rotateLayer('z', 0)">前層</button><button onclick="rotateLayer('z', 1)">中層</button><button onclick="rotateLayer('z', 2)">後層</button>
    </div>
    <div id="win-msg" class="status"></div>
    <button onclick="shuffleCube()" style="background: #28a745;">重新打亂</button>
</div>

<div id="info">滑鼠左鍵拖拽旋轉視角</div>

<script type="importmap">
    { "imports": { "three": "https://unpkg.com/three@0.160.0/build/three.module.js" } }
</script>

<script type="module">
    import * as THREE from 'three';
    import { OrbitControls } from 'https://unpkg.com/three@0.160.0/examples/jsm/controls/OrbitControls.js';

    let scene, camera, renderer, controls;
    let cubies = [];
    const step = 105; // 方塊間距
    let isRotating = false;

    init();

    function init() {
        scene = new THREE.Scene();
        camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
        camera.position.set(5, 5, 10);

        renderer = new THREE.WebGLRenderer({ antialias: true });
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.body.appendChild(renderer.domElement);

        controls = new OrbitControls(camera, renderer.domElement);
        
        createCube();
        animate();
        
        // 初始動畫打亂
        setTimeout(shuffleCube, 1000);
    }

    function createCube() {
        const colors = [0xffffff, 0xffff00, 0xffa500, 0xff0000, 0x00ff00, 0x0000ff]; // R, L, U, D, F, B
        const materials = colors.map(c => new THREE.MeshBasicMaterial({ color: c, side: THREE.DoubleSide }));
        
        // 清除舊方塊
        cubies.forEach(c => scene.remove(c));
        cubies = [];

        for (let x = 0; x < 3; x++) {
            for (let y = 0; y < 3; y++) {
                for (let z = 0; z < 3; z++) {
                    const geometry = new THREE.BoxGeometry(0.9, 0.9, 0.9);
                    const mesh = new THREE.Mesh(geometry, materials);
                    // 設置初始邏輯位置 (0,1,2)
                    mesh.position.set(x - 1, y - 1, z - 1);
                    scene.add(mesh);
                    cubies.push(mesh);
                }
            }
        }
    }

    // 核心旋轉邏輯
    window.rotateLayer = function(axis, layerIndex) {
        if (isRotating) return;
        isRotating = true;

        const group = new THREE.Group();
        scene.add(group);

        const targetLayer = cubies.filter(c => {
            const pos = new THREE.Vector3();
            c.getWorldPosition(pos);
            const val = Math.round(pos[axis] + 1);
            return val === layerIndex;
        });

        targetLayer.forEach(c => group.attach(c));

        const rotateAxis = new THREE.Vector3();
        rotateAxis[axis] = 1;
        
        let progress = 0;
        const speed = 0.1;
        
        function animateRotate() {
            if (progress < Math.PI / 2) {
                group.rotateOnAxis(rotateAxis, speed);
                progress += speed;
                requestAnimationFrame(animateRotate);
            } else {
                // 修正最終角度
                group.rotation[axis] = Math.PI / 2;
                
                // 拆解 Group 回到 Scene
                const children = [...group.children];
                children.forEach(c => {
                    const pos = new THREE.Vector3();
                    const quat = new THREE.Quaternion();
                    c.getWorldPosition(pos);
                    c.getWorldQuaternion(quat);
                    scene.attach(c);
                    // 物理修正座標
                    c.position.set(Math.round(pos.x), Math.round(pos.y), Math.round(pos.z));
                });
                
                scene.remove(group);
                isRotating = false;
                checkWin();
            }
        }
        animateRotate();
    }

    window.shuffleCube = async function() {
        document.getElementById('win-msg').innerText = "打亂中...";
        const axes = ['x', 'y', 'z'];
        for (let i = 0; i < 15; i++) {
            const axis = axes[Math.floor(Math.random() * 3)];
            const layer = Math.floor(Math.random() * 3);
            rotateLayer(axis, layer);
            await new Promise(r => setTimeout(r, 300));
        }
        document.getElementById('win-msg').innerText = "開始挑戰！";
    }

    function checkWin() {
        // 簡化檢測：檢查所有方塊的方向是否一致
        const firstQuat = cubies[0].quaternion;
        const won = cubies.every(c => c.quaternion.equals(firstQuat));
        if (won) {
            document.getElementById('win-msg').innerText = "🎉 恭喜通關！";
            document.getElementById('win-msg').style.color = "#ffff00";
        }
    }

    function animate() {
        requestAnimationFrame(animate);
        renderer.render(scene, camera);
    }

    window.addEventListener('resize', () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
    });
</script>
</body>
</html>

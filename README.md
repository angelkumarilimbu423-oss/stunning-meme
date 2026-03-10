# stunning-meme
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HIGGI-AI | 3D Generative Architecture</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
</head>
<body class="bg-[#0a0a0c] text-slate-300 font-sans selection:bg-blue-500/30">

    <nav class="h-16 border-b border-white/5 flex items-center justify-between px-6 bg-[#0a0a0c]/80 backdrop-blur-xl sticky top-0 z-50">
        <div class="flex items-center gap-3">
            <div class="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center font-bold text-white shadow-lg shadow-blue-600/20">H</div>
            <span class="font-bold tracking-tight text-white uppercase text-sm">Higgi <span class="text-blue-500 font-mono">.AI</span></span>
        </div>
        <div class="flex items-center gap-6">
            <div class="hidden md:flex gap-4 text-[10px] font-mono uppercase tracking-widest text-slate-500">
                <span>GPU Acceleration: On</span>
                <span class="text-green-500">● AI Engine Linked</span>
            </div>
            <button class="bg-white/5 hover:bg-white/10 border border-white/10 text-white px-4 py-1.5 rounded-full text-xs transition-all">Export glTF</button>
        </div>
    </nav>

    <main class="flex flex-col lg:flex-row h-[calc(100vh-64px)] overflow-hidden">
        
        <aside class="w-full lg:w-80 border-r border-white/5 p-6 space-y-8 bg-[#0d0d0f] overflow-y-auto">
            <div>
                <label class="text-[10px] uppercase tracking-widest text-slate-500 font-bold block mb-4">Core Dimensions</label>
                <div class="space-y-6">
                    <div>
                        <div class="flex justify-between text-xs mb-2"><span>Floors</span><span id="floor-val" class="text-blue-400 font-mono">15</span></div>
                        <input type="range" id="floors" min="2" max="60" value="15" oninput="rebuildBuilding()" class="w-full h-1 bg-white/5 rounded-lg appearance-none cursor-pointer accent-blue-600">
                    </div>
                    <div>
                        <div class="flex justify-between text-xs mb-2"><span>Width</span><span id="size-val" class="text-blue-400 font-mono">12</span></div>
                        <input type="range" id="size" min="8" max="30" value="12" oninput="rebuildBuilding()" class="w-full h-1 bg-white/5 rounded-lg appearance-none cursor-pointer accent-blue-600">
                    </div>
                </div>
            </div>

            <div class="pt-6 border-t border-white/5">
                <label class="text-[10px] uppercase tracking-widest text-slate-500 font-bold block mb-4">Material Analysis</label>
                <div class="bg-white/5 p-4 rounded-xl border border-white/5">
                    <p id="ai-status" class="text-[11px] leading-relaxed italic text-slate-400">
                        "AI is analyzing structural integrity for a 15-story residential unit..."
                    </p>
                </div>
            </div>

            <button onclick="aiRandomize()" class="w-full bg-blue-600 hover:bg-blue-500 text-white py-3 rounded-xl text-xs font-bold uppercase tracking-widest transition-all shadow-lg shadow-blue-600/20">
                Generate Design
            </button>
        </aside>

        <section class="flex-1 relative bg-black">
            <div id="canvas-container" class="w-full h-full"></div>
            
            <div class="absolute bottom-6 left-6 p-4 bg-black/40 backdrop-blur-md border border-white/5 rounded-lg pointer-events-none">
                <p class="text-[10px] uppercase tracking-tighter text-slate-500 mb-1">Live Mesh Statistics</p>
                <p id="mesh-stats" class="text-xs font-mono text-white">POLY: 1,420 | FPS: 60</p>
            </div>
        </section>
    </main>

    <script>
        let scene, camera, renderer, controls, buildingGroup;

        function init() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x0a0a0c);
            scene.fog = new THREE.Fog(0x0a0a0c, 30, 140);

            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            camera.position.set(40, 40, 40);

            renderer = new THREE.WebGLRenderer({ antialias: true });
            document.getElementById('canvas-container').appendChild(renderer.domElement);

            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;

            // Lights
            const ambient = new THREE.AmbientLight(0xffffff, 0.5);
            scene.add(ambient);
            const directional = new THREE.DirectionalLight(0x3b82f6, 1);
            directional.position.set(10, 20, 10);
            scene.add(directional);

            // Grid
            const grid = new THREE.GridHelper(100, 20, 0x1e293b, 0x0f172a);
            scene.add(grid);

            buildingGroup = new THREE.Group();
            scene.add(buildingGroup);

            handleResize();
            window.addEventListener('resize', handleResize);
            
            rebuildBuilding();
            animate();
        }

        function rebuildBuilding() {
            while(buildingGroup.children.length > 0) buildingGroup.remove(buildingGroup.children[0]);

            const floors = parseInt(document.getElementById('floors').value);
            const size = parseInt(document.getElementById('size').value);
            
            document.getElementById('floor-val').innerText = floors;
            document.getElementById('size-val').innerText = size;

            const floorH = 3.5;
            for (let i = 0; i < floors; i++) {
                const y = i * floorH + floorH/2;

                // Glass Body
                const geo = new THREE.BoxGeometry(size, floorH - 0.1, size);
                const mat = new THREE.MeshPhongMaterial({ color: 0x1e293b, transparent: true, opacity: 0.6, shininess: 100 });
                const mesh = new THREE.Mesh(geo, mat);
                mesh.position.y = y;
                buildingGroup.add(mesh);

                // Internal Core
                const cGeo = new THREE.BoxGeometry(size * 0.4, floorH, size * 0.4);
                const cMat = new THREE.MeshBasicMaterial({ color: 0x000000 });
                const core = new THREE.Mesh(cGeo, cMat);
                core.position.y = y;
                buildingGroup.add(core);

                // Wireframe
                const edges = new THREE.EdgesGeometry(geo);
                const line = new THREE.LineSegments(edges, new THREE.LineBasicMaterial({ color: 0x3b82f6, transparent: true, opacity: 0.5 }));
                line.position.y = y;
                buildingGroup.add(line);
            }
        }

        function aiRandomize() {
            document.getElementById('floors').value = Math.floor(Math.random() * 40) + 5;
            document.getElementById('size').value = Math.floor(Math.random() * 15) + 10;
            rebuildBuilding();
        }

        function handleResize() {
            const container = document.getElementById('canvas-container');
            camera.aspect = container.clientWidth / container.clientHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(container.clientWidth, container.clientHeight);
        }

        function animate() {
            requestAnimationFrame(animate);
            controls.update();
            renderer.render(scene, camera);
        }

        init();
    </script>
</body>
</html>

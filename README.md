# rea-lmetaworld
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>リアル衛星データメタバース</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            overflow: hidden;
            background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        #container {
            position: relative;
            width: 100vw;
            height: 100vh;
        }
        
        #ui {
            position: absolute;
            top: 20px;
            left: 20px;
            z-index: 100;
            background: rgba(0, 0, 0, 0.85);
            padding: 25px;
            border-radius: 20px;
            backdrop-filter: blur(15px);
            border: 1px solid rgba(255, 255, 255, 0.15);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            min-width: 300px;
        }
        
        .control-group {
            margin-bottom: 20px;
            padding: 15px;
            background: rgba(255, 255, 255, 0.05);
            border-radius: 10px;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        
        .control-group h3 {
            color: #4CAF50;
            margin: 0 0 15px 0;
            font-size: 16px;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 1px;
        }
        
        .control {
            margin-bottom: 15px;
            color: white;
        }
        
        .control label {
            display: block;
            margin-bottom: 8px;
            font-size: 13px;
            font-weight: 500;
            color: #e0e0e0;
        }
        
        .control input, .control select {
            width: 100%;
            padding: 10px;
            border: none;
            border-radius: 8px;
            background: rgba(255, 255, 255, 0.1);
            color: white;
            border: 1px solid rgba(255, 255, 255, 0.2);
            font-size: 14px;
            box-sizing: border-box;
        }
        
        .control input:focus, .control select:focus {
            outline: none;
            border-color: #4CAF50;
            box-shadow: 0 0 12px rgba(76, 175, 80, 0.4);
        }
        
        .coord-input {
            display: flex;
            gap: 10px;
        }
        
        .coord-input input {
            flex: 1;
        }
        
        .control button {
            background: linear-gradient(135deg, #4CAF50, #45a049);
            color: white;
            border: none;
            padding: 12px 24px;
            border-radius: 10px;
            cursor: pointer;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 1px;
            transition: all 0.3s ease;
            width: 100%;
            font-size: 14px;
        }
        
        .control button:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(76, 175, 80, 0.4);
            background: linear-gradient(135deg, #5CBF60, #4CAF50);
        }
        
        .control button:disabled {
            background: rgba(128, 128, 128, 0.5);
            cursor: not-allowed;
            transform: none;
            box-shadow: none;
        }
        
        #info {
            position: absolute;
            bottom: 20px;
            right: 20px;
            background: rgba(0, 0, 0, 0.85);
            color: white;
            padding: 20px;
            border-radius: 15px;
            backdrop-filter: blur(15px);
            border: 1px solid rgba(255, 255, 255, 0.15);
            font-size: 13px;
            min-width: 250px;
        }
        
        .info-section {
            margin-bottom: 15px;
        }
        
        .info-section h4 {
            color: #4CAF50;
            margin: 0 0 8px 0;
            font-size: 14px;
        }
        
        #loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 18px;
            z-index: 1000;
            background: rgba(0, 0, 0, 0.9);
            padding: 30px;
            border-radius: 15px;
            backdrop-filter: blur(15px);
            text-align: center;
            border: 1px solid rgba(255, 255, 255, 0.15);
        }
        
        .loading-spinner {
            border: 3px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            border-top: 3px solid #4CAF50;
            width: 30px;
            height: 30px;
            animation: spin 1s linear infinite;
            margin: 20px auto;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .hidden {
            display: none;
        }
        
        .status-indicator {
            display: inline-block;
            width: 10px;
            height: 10px;
            border-radius: 50%;
            margin-right: 8px;
        }
        
        .status-loading { background-color: #ff9800; }
        .status-ready { background-color: #4CAF50; }
        .status-error { background-color: #f44336; }
    </style>
</head>
<body>
    <div id="container">
        <div id="loading">
            <div class="loading-spinner"></div>
            <div id="loadingText">3Dエンジンを初期化中...</div>
        </div>
        
        <div id="ui">
            <div class="control-group">
                <h3>位置設定</h3>
                <div class="control">
                    <label>プリセット位置:</label>
                    <select id="locationPreset">
                        <option value="tokyo">東京 (35.6762, 139.6503)</option>
                        <option value="fuji">富士山 (35.3606, 138.7274)</option>
                        <option value="osaka">大阪 (34.6937, 135.5023)</option>
                        <option value="kyoto">京都 (35.0116, 135.7681)</option>
                        <option value="hiroshima">広島 (34.3853, 132.4553)</option>
                        <option value="custom">カスタム座標</option>
                    </select>
                </div>
                <div class="control">
                    <label>緯度・経度:</label>
                    <div class="coord-input">
                        <input type="number" id="latitude" placeholder="緯度" step="0.0001" value="35.6762">
                        <input type="number" id="longitude" placeholder="経度" step="0.0001" value="139.6503">
                    </div>
                </div>
                <div class="control">
                    <button onclick="loadLocation()">位置を読み込み</button>
                </div>
            </div>
            
            <div class="control-group">
                <h3>表示設定</h3>
                <div class="control">
                    <label>地形詳細度:</label>
                    <select id="terrainDetail">
                        <option value="32">低 (32x32)</option>
                        <option value="64" selected>中 (64x64)</option>
                        <option value="128">高 (128x128)</option>
                        <option value="256">最高 (256x256)</option>
                    </select>
                </div>
                <div class="control">
                    <label>高度倍率: <span id="heightScaleValue">2.0</span></label>
                    <input type="range" id="heightScale" min="0.5" max="5" step="0.1" value="2.0">
                </div>
                <div class="control">
                    <label>視野範囲 (km): <span id="viewRangeValue">5</span></label>
                    <input type="range" id="viewRange" min="1" max="20" step="1" value="5">
                </div>
                <div class="control">
                    <button onclick="toggleWireframe()" id="wireframeBtn">ワイヤーフレーム: OFF</button>
                </div>
            </div>
            
            <div class="control-group">
                <h3>データソース</h3>
                <div class="control">
                    <label>衛星画像プロバイダー:</label>
                    <select id="imageProvider">
                        <option value="osm">OpenStreetMap</option>
                        <option value="satellite">衛星画像 (模擬)</option>
                        <option value="terrain">地形図</option>
                    </select>
                </div>
                <div class="control">
                    <label>標高データ:</label>
                    <select id="elevationProvider">
                        <option value="srtm">SRTM (模擬)</option>
                        <option value="aster">ASTER (模擬)</option>
                        <option value="synthetic">合成データ</option>
                    </select>
                </div>
            </div>
        </div>
        
        <div id="info">
            <div class="info-section">
                <h4>システム状態</h4>
                <div id="systemStatus">
                    <div><span class="status-indicator status-loading"></span>地形データ: 読み込み中</div>
                    <div><span class="status-indicator status-loading"></span>衛星画像: 読み込み中</div>
                    <div><span class="status-indicator status-loading"></span>テクスチャ: 準備中</div>
                </div>
            </div>
            
            <div class="info-section">
                <h4>操作方法</h4>
                <div>マウス: 視点回転</div>
                <div>ホイール: ズーム</div>
                <div>WASD/矢印: 移動</div>
                <div>Shift: 高速移動</div>
            </div>
            
            <div class="info-section">
                <h4>現在位置</h4>
                <div id="currentCoords">緯度: ---, 経度: ---</div>
                <div id="cameraCoords">カメラ: (0, 0, 0)</div>
                <div id="elevation">標高: --- m</div>
            </div>
            
            <div class="info-section">
                <h4>パフォーマンス</h4>
                <div id="fps">FPS: --</div>
                <div id="triangles">三角形: ---</div>
                <div id="lodLevel">LOD: ---</div>
            </div>
        </div>
    </div>

    <script>
        // グローバル変数
        let scene, camera, renderer, terrain, controls;
        let currentLat = 35.6762, currentLng = 139.6503;
        let terrainChunks = [];
        let wireframeMode = false;
        let cameraTarget = new THREE.Vector3(0, 0, 0);
        let cameraPosition = new THREE.Vector3(0, 100, 200);
        let frameCount = 0;
        let lastTime = 0;
        
        // 座標変換クラス
        class CoordinateConverter {
            static deg2rad(deg) {
                return deg * (Math.PI / 180);
            }
            
            static rad2deg(rad) {
                return rad * (180 / Math.PI);
            }
            
            // Web Mercator投影
            static latLngToMercator(lat, lng) {
                const x = lng * 20037508.34 / 180;
                let y = Math.log(Math.tan((90 + lat) * Math.PI / 360)) / (Math.PI / 180);
                y = y * 20037508.34 / 180;
                return { x, y };
            }
            
            // 緯度経度からローカル座標
            static latLngToLocal(lat, lng, centerLat, centerLng, scale = 1000) {
                const earthRadius = 6371000; // m
                const dLat = this.deg2rad(lat - centerLat);
                const dLng = this.deg2rad(lng - centerLng);
                
                const x = dLng * earthRadius * Math.cos(this.deg2rad(centerLat)) / scale;
                const z = -dLat * earthRadius / scale; // Zは北向きを負にする
                
                return { x, z };
            }
            
            // ローカル座標から緯度経度
            static localToLatLng(x, z, centerLat, centerLng, scale = 1000) {
                const earthRadius = 6371000;
                const dLat = -z * scale / earthRadius;
                const dLng = x * scale / (earthRadius * Math.cos(this.deg2rad(centerLat)));
                
                return {
                    lat: centerLat + this.rad2deg(dLat),
                    lng: centerLng + this.rad2deg(dLng)
                };
            }
        }
        
        // DEMデータ生成器（実際のAPIに置き換え可能）
        class DEMDataGenerator {
            static async generateElevationData(lat, lng, size, resolution) {
                // 実際の実装では、SRTM、ASTER GDEMなどのAPIを使用
                const data = [];
                const step = size / resolution;
                
                for (let y = 0; y < resolution; y++) {
                    data[y] = [];
                    for (let x = 0; x < resolution; x++) {
                        const currentLat = lat + (y - resolution / 2) * step / 111320;
                        const currentLng = lng + (x - resolution / 2) * step / (111320 * Math.cos(CoordinateConverter.deg2rad(lat)));
                        
                        // 合成標高データ（実際のDEMデータに置き換え）
                        let elevation = this.generateSyntheticElevation(currentLat, currentLng);
                        
                        // ノイズ追加
                        elevation += (Math.random() - 0.5) * 20;
                        
                        data[y][x] = Math.max(0, elevation);
                    }
                }
                
                return data;
            }
            
            static generateSyntheticElevation(lat, lng) {
                // 富士山周辺の特別処理
                const fujiLat = 35.3606;
                const fujiLng = 138.7274;
                const distToFuji = Math.sqrt(Math.pow(lat - fujiLat, 2) + Math.pow(lng - fujiLng, 2));
                
                if (distToFuji < 0.5) {
                    return Math.max(0, 3776 * (1 - distToFuji / 0.5));
                }
                
                // 一般的な地形
                let elevation = 0;
                elevation += Math.sin(lat * 100) * 200;
                elevation += Math.cos(lng * 150) * 300;
                elevation += Math.sin(lat * 50 + lng * 50) * 150;
                elevation += Math.abs(Math.sin(lat * 200)) * 500;
                
                return Math.max(0, elevation + 100);
            }
        }
        
        // LODマネージャー
        class LODManager {
            constructor() {
                this.lodLevels = [
                    { distance: 0, resolution: 256 },
                    { distance: 1000, resolution: 128 },
                    { distance: 3000, resolution: 64 },
                    { distance: 8000, resolution: 32 }
                ];
            }
            
            getLODLevel(distance) {
                for (let i = this.lodLevels.length - 1; i >= 0; i--) {
                    if (distance >= this.lodLevels[i].distance) {
                        return this.lodLevels[i];
                    }
                }
                return this.lodLevels[0];
            }
        }
        
        // 衛星タイル管理
        class SatelliteTileManager {
            constructor() {
                this.tileCache = new Map();
                this.baseUrls = {
                    osm: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
                    satellite: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
                    terrain: 'https://tile.opentopomap.org/{z}/{x}/{y}.png'
                };
            }
            
            async loadTile(provider, z, x, y) {
                const cacheKey = `${provider}_${z}_${x}_${y}`;
                
                if (this.tileCache.has(cacheKey)) {
                    return this.tileCache.get(cacheKey);
                }
                
                try {
                    // 実際の実装では、指定されたURLからタイルを取得
                    // ここでは模擬データを生成
                    const canvas = document.createElement('canvas');
                    canvas.width = 256;
                    canvas.height = 256;
                    const ctx = canvas.getContext('2d');
                    
                    // プロバイダーに応じた色彩生成
                    switch (provider) {
                        case 'satellite':
                            this.generateSatelliteTexture(ctx, x, y, z);
                            break;
                        case 'terrain':
                            this.generateTerrainTexture(ctx, x, y, z);
                            break;
                        default:
                            this.generateOSMTexture(ctx, x, y, z);
                    }
                    
                    const texture = new THREE.CanvasTexture(canvas);
                    this.tileCache.set(cacheKey, texture);
                    return texture;
                } catch (error) {
                    console.error('タイル読み込みエラー:', error);
                    return null;
                }
            }
            
            generateSatelliteTexture(ctx, x, y, z) {
                // 衛星画像風テクスチャ
                const gradient = ctx.createLinearGradient(0, 0, 256, 256);
                gradient.addColorStop(0, `hsl(${(x + y) * 20 % 360}, 30%, 40%)`);
                gradient.addColorStop(1, `hsl(${(x + y) * 30 % 360}, 50%, 60%)`);
                
                ctx.fillStyle = gradient;
                ctx.fillRect(0, 0, 256, 256);
                
                // 詳細パターン追加
                ctx.fillStyle = 'rgba(0, 100, 0, 0.3)';
                for (let i = 0; i < 50; i++) {
                    const px = Math.random() * 256;
                    const py = Math.random() * 256;
                    const size = Math.random() * 20 + 5;
                    ctx.fillRect(px, py, size, size);
                }
            }
            
            generateTerrainTexture(ctx, x, y, z) {
                // 地形図風テクスチャ
                ctx.fillStyle = '#f5f5dc';
                ctx.fillRect(0, 0, 256, 256);
                
                // 等高線パターン
                ctx.strokeStyle = '#8B4513';
                ctx.lineWidth = 1;
                for (let i = 0; i < 10; i++) {
                    ctx.beginPath();
                    ctx.moveTo(0, i * 25);
                    ctx.lineTo(256, i * 25 + Math.sin(x + y) * 20);
                    ctx.stroke();
                }
            }
            
            generateOSMTexture(ctx, x, y, z) {
                // OpenStreetMap風テクスチャ
                ctx.fillStyle = '#f2efe9';
                ctx.fillRect(0, 0, 256, 256);
                
                // 道路パターン
                ctx.strokeStyle = '#ffffff';
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.moveTo(128, 0);
                ctx.lineTo(128, 256);
                ctx.moveTo(0, 128);
                ctx.lineTo(256, 128);
                ctx.stroke();
            }
        }
        
        // メインアプリケーション初期化
        async function init() {
            updateLoadingText('3Dエンジンを初期化中...');
            
            // シーンの初期化
            scene = new THREE.Scene();
            scene.fog = new THREE.Fog(0x87CEEB, 100, 20000);
            
            // カメラの初期化
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 1, 50000);
            camera.position.copy(cameraPosition);
            camera.lookAt(cameraTarget);
            
            // レンダラーの初期化
            renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setClearColor(0x87CEEB, 1);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            renderer.gammaOutput = true;
            document.getElementById('container').appendChild(renderer.domElement);
            
            // ライティング設定
            setupLighting();
            
            // イベントリスナー設定
            setupEventListeners();
            
            // 初期位置読み込み
            await loadLocation();
            
            // システム状態更新
            updateSystemStatus();
            
            // ローディング画面を非表示
            document.getElementById('loading').classList.add('hidden');
            
            // アニメーションループ開始
            animate();
        }
        
        function setupLighting() {
            // 環境光
            const ambientLight = new THREE.AmbientLight(0x404040, 0.4);
            scene.add(ambientLight);
            
            // 太陽光（方向光）
            const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8);
            directionalLight.position.set(1000, 2000, 1000);
            directionalLight.castShadow = true;
            directionalLight.shadow.mapSize.width = 4096;
            directionalLight.shadow.mapSize.height = 4096;
            directionalLight.shadow.camera.near = 0.5;
            directionalLight.shadow.camera.far = 10000;
            directionalLight.shadow.camera.left = -5000;
            directionalLight.shadow.camera.right = 5000;
            directionalLight.shadow.camera.top = 5000;
            directionalLight.shadow.camera.bottom = -5000;
            scene.add(directionalLight);
            
            // 空の色グラデーション
            const skyGeometry = new THREE.SphereGeometry(40000, 32, 32);
            const skyMaterial = new THREE.ShaderMaterial({
                vertexShader: `
                    varying vec3 vWorldPosition;
                    void main() {
                        vec4 worldPosition = modelMatrix * vec4(position, 1.0);
                        vWorldPosition = worldPosition.xyz;
                        gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
                    }
                `,
                fragmentShader: `
                    uniform vec3 topColor;
                    uniform vec3 bottomColor;
                    uniform float offset;
                    uniform float exponent;
                    varying vec3 vWorldPosition;
                    void main() {
                        float h = normalize(vWorldPosition + offset).y;
                        gl_FragColor = vec4(mix(bottomColor, topColor, max(pow(max(h, 0.0), exponent), 0.0)), 1.0);
                    }
                `,
                uniforms: {
                    topColor: { value: new THREE.Color(0x0077ff) },
                    bottomColor: { value: new THREE.Color(0xffffff) },
                    offset: { value: 33 },
                    exponent: { value: 0.6 }
                },
                side: THREE.BackSide
            });
            
            const sky = new THREE.Mesh(skyGeometry, skyMaterial);
            scene.add(sky);
        }
        
        async function loadLocation() {
            const lat = parseFloat(document.getElementById('latitude').value);
            const lng = parseFloat(document.getElementById('longitude').value);
            
            if (isNaN(lat) || isNaN(lng)) {
                alert('有效な緯度・経度を入力してください');
                return;
            }
            
            currentLat = lat;
            currentLng = lng;
            
            updateLoadingText('地形データを読み込み中...');
            document.getElementById('loading').classList.remove('hidden');
            
            try {
                // 既存の地形を削除
                terrainChunks.forEach(chunk => scene.remove(chunk));
                terrainChunks = [];
                
                // 新しい地形を生成
                await generateTerrain(lat, lng);
                
                // カメラ位置をリセット
                cameraPosition.set(0, 500, 1000);
                cameraTarget.set(0, 0, 0);
                camera.position.copy(cameraPosition);
                camera.lookAt(cameraTarget);
                
                updateSystemStatus();
                
            } catch (error) {
                console.error('地形読み込みエラー:', error);
                alert('地形データの読み込みに失敗しました');
            } finally {
                document.getElementById('loading').classList.add('hidden');
            }
        }
        
        async function generateTerrain(centerLat, centerLng) {
            const viewRange = parseInt(document.getElementById('viewRange').value) * 1000;
            const detail = parseInt(document.getElementById('terrainDetail').value);
            const heightScale = parseFloat(document.getElementById('heightScale').value);
            const imageProvider = document.getElementById('imageProvider').value;
            
            // DEMデータ取得
            updateLoadingText('標高データを取得中...');
            const elevationData = await DEMDataGenerator.generateElevationData(
                centerLat, centerLng, viewRange, detail
            );
            
            // 衛星タイル取得
            updateLoadingText('衛星画像を取得中...');
            const tileManager = new SatelliteTileManager();
            
            // 地形ジオメトリ作成
            const geometry = new THREE.PlaneGeometry(viewRange, viewRange, detail - 1, detail - 1);
            const vertices = geometry.attributes.position.array;
            
            // 標高データを頂点に適用
            for (let i = 0; i < vertices.length / 3; i++) {
                const x = Math.floor(i % detail);
                const y = Math.floor(i / detail);
                if (elevationData[y] && elevationData[y][x] !== undefined) {
                    vertices[i * 3 + 2] = elevationData[y][x] * heightScale;
                }
            }
            
            geometry.attributes.position.needsUpdate = true;
            geometry.computeVertexNormals();
            
            // テクスチャ取得と適用
            updateLoadingText('テクスチャを生成中...');
            const texture = await tileManager.loadTile(imageProvider, 10, 100, 100);
            
            // マテリアル作成
            const material = new THREE.MeshLambertMaterial({
                map: texture,
                wireframe: wireframeMode
            });
            
            // メッシュ作成
            const terrainMesh = new THREE.Mesh(geometry, material);
            terrainMesh.rotation.x = -Math.PI / 2;
            terrainMesh.receiveShadow = true;
            terrainMesh.castShadow = true;
            
            scene.add(terrainMesh);
            terrainChunks.push(terrainMesh);
            
            // 追加の詳細要素
            await addDetailElements(centerLat, centerLng, viewRange);
        }
        
        async function addDetailElements(centerLat, centerLng, viewRange) {
            // 水面
            if (centerLat > 34 && centerLat < 36 && centerLng > 138 && centerLng < 140) {
                addWaterBodies();
            }
            
            // 建物（都市部）
            if (isUrbanArea(centerLat, centerLng)) {
                addBuildings(viewRange);
            }
            
            // 植生
            addVegetation(viewRange);
        }
        
        function addWaterBodies() {
            const waterGeometry = new THREE.PlaneGeometry(2000, 1000);
            const waterMaterial = new THREE.MeshPhongMaterial({
                color: 0x006994,
                transparent: true,
                opacity: 0.6,
                shininess: 100
            });
            
            const water = new THREE.Mesh(waterGeometry, waterMaterial);
            water.rotation.x = -Math.PI / 2;
            water.position.y = 5;
            scene.add(water);
            terrainChunks.push(water);
        }
        
        function isUrbanArea(lat, lng) {
            // 主要都市の座標範囲をチェック
            const cities = [
                { name: '東京', lat: 35.6762, lng: 139.6503, radius: 0.5 },
                { name: '大阪', lat: 34.6937, lng: 135.5023, radius: 0.3 },
                { name: '名古屋', lat: 35.1815, lng: 136.9066, radius: 0.2 }
            ];
            
            return cities.some(city => {
                const distance = Math.sqrt(Math.pow(lat - city.lat, 2) + Math.pow(lng - city.lng, 2));
                return distance < city.radius;
            });
        }
        
        function addBuildings(viewRange) {
            const buildingCount = Math.floor(viewRange / 100);
            
            for (let i = 0; i < buildingCount; i++) {
                const height = Math.random() * 200 + 20;
                const width = Math.random() * 50 + 20;
                const depth = Math.random() * 50 + 20;
                
                const buildingGeometry = new THREE.BoxGeometry(width, height, depth);
                const buildingMaterial = new THREE.MeshLambertMaterial({
                    color: new THREE.Color().setHSL(0.1, 0.2, Math.random() * 0.5 + 0.5)
                });
                
                const building = new THREE.Mesh(buildingGeometry, buildingMaterial);
                building.position.x = (Math.random() - 0.5) * viewRange * 0.8;
                building.position.z = (Math.random() - 0.5) * viewRange * 0.8;
                building.position.y = height / 2;
                building.castShadow = true;
                building.receiveShadow = true;
                
                scene.add(building);
                terrainChunks.push(building);
            }
        }
        
        function addVegetation(viewRange) {
            const treeCount = Math.floor(viewRange / 50);
            
            for (let i = 0; i < treeCount; i++) {
                // 木の幹
                const trunkGeometry = new THREE.CylinderGeometry(2, 4, 20, 8);
                const trunkMaterial = new THREE.MeshLambertMaterial({ color: 0x8B4513 });
                const trunk = new THREE.Mesh(trunkGeometry, trunkMaterial);
                
                // 木の葉
                const foliageGeometry = new THREE.SphereGeometry(15, 8, 6);
                const foliageMaterial = new THREE.MeshLambertMaterial({ color: 0x228B22 });
                const foliage = new THREE.Mesh(foliageGeometry, foliageMaterial);
                foliage.position.y = 25;
                
                const tree = new THREE.Group();
                tree.add(trunk);
                tree.add(foliage);
                
                tree.position.x = (Math.random() - 0.5) * viewRange * 0.9;
                tree.position.z = (Math.random() - 0.5) * viewRange * 0.9;
                tree.position.y = 10;
                
                tree.castShadow = true;
                scene.add(tree);
                terrainChunks.push(tree);
            }
        }
        
        function setupEventListeners() {
            // ウィンドウリサイズ
            window.addEventListener('resize', onWindowResize);
            
            // キーボード操作
            document.addEventListener('keydown', onKeyDown);
            document.addEventListener('keyup', onKeyUp);
            
            // マウス操作
            document.addEventListener('mousedown', onMouseDown);
            document.addEventListener('mousemove', onMouseMove);
            document.addEventListener('mouseup', onMouseUp);
            document.addEventListener('wheel', onMouseWheel);
            
            // UI コントロール
            document.getElementById('locationPreset').addEventListener('change', onLocationPresetChange);
            document.getElementById('heightScale').addEventListener('input', onHeightScaleChange);
            document.getElementById('viewRange').addEventListener('input', onViewRangeChange);
            document.getElementById('terrainDetail').addEventListener('change', onTerrainDetailChange);
            document.getElementById('imageProvider').addEventListener('change', onImageProviderChange);
            document.getElementById('elevationProvider').addEventListener('change', onElevationProviderChange);
        }
        
        // イベントハンドラー
        const keys = {};
        let mouse = { x: 0, y: 0, lastX: 0, lastY: 0, down: false };
        
        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }
        
        function onKeyDown(event) {
            keys[event.code] = true;
        }
        
        function onKeyUp(event) {
            keys[event.code] = false;
        }
        
        function onMouseDown(event) {
            mouse.down = true;
            mouse.lastX = event.clientX;
            mouse.lastY = event.clientY;
        }
        
        function onMouseMove(event) {
            mouse.x = event.clientX;
            mouse.y = event.clientY;
            
            if (mouse.down) {
                const deltaX = mouse.x - mouse.lastX;
                const deltaY = mouse.y - mouse.lastY;
                
                // カメラの回転
                const spherical = new THREE.Spherical();
                spherical.setFromVector3(cameraPosition.clone().sub(cameraTarget));
                
                spherical.theta -= deltaX * 0.01;
                spherical.phi += deltaY * 0.01;
                spherical.phi = Math.max(0.1, Math.min(Math.PI - 0.1, spherical.phi));
                
                cameraPosition.setFromSpherical(spherical).add(cameraTarget);
                camera.position.copy(cameraPosition);
                camera.lookAt(cameraTarget);
                
                mouse.lastX = mouse.x;
                mouse.lastY = mouse.y;
            }
        }
        
        function onMouseUp(event) {
            mouse.down = false;
        }
        
        function onMouseWheel(event) {
            const distance = cameraPosition.distanceTo(cameraTarget);
            const zoomSpeed = distance * 0.1;
            const newDistance = Math.max(10, Math.min(5000, distance + event.deltaY * zoomSpeed * 0.01));
            
            const direction = cameraPosition.clone().sub(cameraTarget).normalize();
            cameraPosition = cameraTarget.clone().add(direction.multiplyScalar(newDistance));
            camera.position.copy(cameraPosition);
        }
        
        function onLocationPresetChange(event) {
            const presets = {
                tokyo: { lat: 35.6762, lng: 139.6503 },
                fuji: { lat: 35.3606, lng: 138.7274 },
                osaka: { lat: 34.6937, lng: 135.5023 },
                kyoto: { lat: 35.0116, lng: 135.7681 },
                hiroshima: { lat: 34.3853, lng: 132.4553 }
            };
            
            const preset = presets[event.target.value];
            if (preset) {
                document.getElementById('latitude').value = preset.lat;
                document.getElementById('longitude').value = preset.lng;
            }
        }
        
        function onHeightScaleChange(event) {
            const value = parseFloat(event.target.value);
            document.getElementById('heightScaleValue').textContent = value.toFixed(1);
            
            // リアルタイム更新は重いので、少し遅延させる
            clearTimeout(window.heightScaleTimeout);
            window.heightScaleTimeout = setTimeout(() => {
                loadLocation();
            }, 500);
        }
        
        function onViewRangeChange(event) {
            const value = parseInt(event.target.value);
            document.getElementById('viewRangeValue').textContent = value;
            
            clearTimeout(window.viewRangeTimeout);
            window.viewRangeTimeout = setTimeout(() => {
                loadLocation();
            }, 500);
        }
        
        function onTerrainDetailChange() {
            loadLocation();
        }
        
        function onImageProviderChange() {
            loadLocation();
        }
        
        function onElevationProviderChange() {
            loadLocation();
        }
        
        function toggleWireframe() {
            wireframeMode = !wireframeMode;
            document.getElementById('wireframeBtn').textContent = 
                `ワイヤーフレーム: ${wireframeMode ? 'ON' : 'OFF'}`;
            
            terrainChunks.forEach(chunk => {
                if (chunk.material) {
                    chunk.material.wireframe = wireframeMode;
                }
            });
        }
        
        function updateSystemStatus() {
            const statusElement = document.getElementById('systemStatus');
            statusElement.innerHTML = `
                <div><span class="status-indicator status-ready"></span>地形データ: 読み込み完了</div>
                <div><span class="status-indicator status-ready"></span>衛星画像: 読み込み完了</div>
                <div><span class="status-indicator status-ready"></span>テクスチャ: 適用完了</div>
            `;
        }
        
        function updateLoadingText(text) {
            const loadingTextElement = document.getElementById('loadingText');
            if (loadingTextElement) {
                loadingTextElement.textContent = text;
            }
        }
        
        function updateCameraMovement() {
            const speed = keys['ShiftLeft'] || keys['ShiftRight'] ? 50 : 20;
            const forward = new THREE.Vector3(0, 0, -1).applyQuaternion(camera.quaternion);
            const right = new THREE.Vector3(1, 0, 0).applyQuaternion(camera.quaternion);
            
            if (keys['KeyW'] || keys['ArrowUp']) {
                cameraTarget.add(forward.clone().multiplyScalar(speed));
                cameraPosition.add(forward.clone().multiplyScalar(speed));
            }
            if (keys['KeyS'] || keys['ArrowDown']) {
                cameraTarget.add(forward.clone().multiplyScalar(-speed));
                cameraPosition.add(forward.clone().multiplyScalar(-speed));
            }
            if (keys['KeyA'] || keys['ArrowLeft']) {
                cameraTarget.add(right.clone().multiplyScalar(-speed));
                cameraPosition.add(right.clone().multiplyScalar(-speed));
            }
            if (keys['KeyD'] || keys['ArrowRight']) {
                cameraTarget.add(right.clone().multiplyScalar(speed));
                cameraPosition.add(right.clone().multiplyScalar(speed));
            }
            
            camera.position.copy(cameraPosition);
            camera.lookAt(cameraTarget);
        }
        
        function updateInfo() {
            // 現在の緯度経度を計算
            const localCoord = CoordinateConverter.localToLatLng(
                cameraTarget.x, cameraTarget.z, currentLat, currentLng, 1000
            );
            
            document.getElementById('currentCoords').textContent = 
                `緯度: ${localCoord.lat.toFixed(6)}, 経度: ${localCoord.lng.toFixed(6)}`;
            
            document.getElementById('cameraCoords').textContent = 
                `カメラ: (${Math.round(camera.position.x)}, ${Math.round(camera.position.y)}, ${Math.round(camera.position.z)})`;
            
            document.getElementById('elevation').textContent = 
                `標高: ${Math.round(camera.position.y)} m`;
            
            // パフォーマンス情報
            const fps = Math.round(1000 / (performance.now() - lastTime));
            document.getElementById('fps').textContent = `FPS: ${fps}`;
            
            const triangles = renderer.info.render.triangles;
            document.getElementById('triangles').textContent = `三角形: ${triangles.toLocaleString()}`;
            
            const distance = cameraPosition.distanceTo(cameraTarget);
            const lodManager = new LODManager();
            const lodLevel = lodManager.getLODLevel(distance);
            document.getElementById('lodLevel').textContent = `LOD: ${lodLevel.resolution}`;
        }
        
        function animate() {
            requestAnimationFrame(animate);
            
            const currentTime = performance.now();
            
            // カメラ移動処理
            updateCameraMovement();
            
            // 情報更新（60FPSでは重いので、30FPSで更新）
            if (frameCount % 2 === 0) {
                updateInfo();
            }
            
            // レンダリング
            renderer.render(scene, camera);
            
            frameCount++;
            lastTime = currentTime;
        }
        
        // 初期化実行
        document.addEventListener('DOMContentLoaded', () => {
            init().catch(error => {
                console.error('初期化エラー:', error);
                document.getElementById('loadingText').textContent = '初期化に失敗しました';
            })
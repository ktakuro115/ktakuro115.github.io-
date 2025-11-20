<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0, user-scalable=yes">
    <title>GB Camera V16 (1080p) - Final</title>
    
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">

    <style>
        /* ----- BASIC SETUP ----- */
        :root {
            --gb-body: #c0c0c0;
            --gb-screen-bg: #0f380f;
            --font-main: 'Press Start 2P', monospace;
        }

        body {
            background-color: #151515;
            background-image: 
                linear-gradient(rgba(50, 50, 50, 0.5) 1px, transparent 1px),
                linear-gradient(90deg, rgba(50, 50, 50, 0.5) 1px, transparent 1px);
            background-size: 30px 30px;
            font-family: var(--font-main);
            height: 100vh;
            width: 100vw;
            margin: 0;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            touch-action: none;
            user-select: none;
            -webkit-user-select: none;
        }

        /* ----- TRANSFORM CONTAINER ----- */
        #gbContainer {
            width: 340px;
            height: 600px; 
            transform-origin: center center;
            transition: transform 0.05s linear;
        }

        /* ----- GAMEBOY BODY ----- */
        .gb-body {
            background: var(--gb-body);
            width: 100%;
            height: 100%;
            border-radius: 15px 15px 40px 15px;
            box-shadow: 
                inset 3px 3px 5px rgba(255,255,255,0.6),
                inset -3px -3px 5px rgba(0,0,0,0.2),
                15px 15px 40px rgba(0,0,0,0.6);
            display: flex;
            flex-direction: column;
            align-items: center;
            padding-top: 25px;
            box-sizing: border-box;
            position: relative;
            border-right: 4px solid #999;
            border-bottom: 4px solid #999;
        }

        .logo-text {
            position: absolute; top: 10px; left: 25px;
            color: #666; font-size: 10px; font-weight: bold;
            font-style: italic; letter-spacing: 1px;
            text-shadow: 0 1px 0 rgba(255,255,255,0.4);
        }

        /* SCREEN AREA (SQUARE) */
        .screen-lens {
            background: #555; 
            width: 290px; 
            height: 260px;
            border-radius: 10px 10px 40px 10px; 
            position: relative;
            display: flex; justify-content: center; align-items: center;
            box-shadow: inset 0 0 15px rgba(0,0,0,0.8); 
            margin-bottom: 10px; 
        }

        .screen-cover {
            width: 220px; 
            height: 220px; 
            position: relative;
            overflow: hidden; 
            background: #000;
            border: 2px solid #333;
        }

        /* CANVAS: 実際は1080pxだがCSSで小さく表示 */
        canvas {
            display: block; 
            width: 100%; 
            height: 100%;
            background-color: var(--gb-screen-bg); 
            image-rendering: pixelated; 
            box-sizing: border-box;
            transition: opacity 0.1s;
        }

        .scanlines {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(to bottom, rgba(0,0,0,0), rgba(0,0,0,0.1) 50%, rgba(0,0,0,0) 50%);
            background-size: 100% 4px; pointer-events: none; z-index: 5; opacity: 0.4;
        }

        #toast {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            background: rgba(0,0,0,0.8); color: #fff; padding: 8px 12px;
            font-size: 10px; display: none; pointer-events: none; white-space: nowrap; border-radius: 4px;
            font-family: var(--font-main); border: 1px solid #fff; text-transform: uppercase; z-index: 10;
        }

        /* ----- PREVIEW OVERLAY STYLES ----- */
        #previewContainer {
            position: absolute;
            top: 50%;   
            left: 50%;  
            transform: translate(-50%, -50%);
            width: 220px;
            height: 220px;
            background: #000;
            z-index: 20;
            display: none; 
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-family: sans-serif;
            color: white;
            padding: 0;
            box-sizing: border-box;
            background-color: #222;
        }

        #previewMediaImg, #previewMediaVideo {
            max-width: 100%;
            max-height: 100%;
            object-fit: contain;
            display: none;
            image-rendering: pixelated;
            background-color: #000;
        }

        #previewControls {
            position: absolute;
            bottom: 5px;
            width: 100%;
            display: flex;
            justify-content: space-around;
            padding: 5px 0;
            background: rgba(0, 0, 0, 0.5);
            font-family: var(--font-main);
        }

        #previewControls button {
            background: #cc3333;
            color: #fff;
            border: 2px solid #fff;
            padding: 5px 10px;
            font-family: var(--font-main);
            font-size: 8px;
            cursor: pointer;
            box-shadow: 2px 2px 4px rgba(0,0,0,0.4);
            text-transform: uppercase;
        }

        /* --- ボタンの凹みUIの核心部分 --- */
        #previewControls button:active {
            transform: translate(1px, 1px);
            box-shadow: inset 1px 1px 3px rgba(0,0,0,0.6);
        }
        /* ---------------------------------- */

        /* CONTROLS */
        .controls { width: 100%; height: 250px; position: relative; } 

        /* D-PAD */
        .dpad-area { 
            position: absolute; 
            top: 10px; 
            left: 45px; 
            width: 100px; 
            height: 100px; 
        } 
        .cross { 
            background: #222; position: absolute; border-radius: 4px; box-shadow: 2px 2px 5px rgba(0,0,0,0.4); 
            pointer-events: none; transition: transform 0.05s ease, box-shadow 0.05s ease; 
        }
        .c-h { width: 90px; height: 30px; top: 30px; left: 0; }
        .c-v { width: 30px; height: 90px; top: 0; left: 30px; }
        .c-c { 
            width: 30px; height: 30px; top: 30px; left: 30px; background:none; z-index: 2; 
            box-shadow:inset 2px 2px 5px rgba(0,0,0,0.5); border-radius: 50%; 
        }
        .d-btn { position: absolute; z-index: 10; cursor: pointer; -webkit-tap-highlight-color: transparent; }
        .d-up { width: 40px; height: 45px; top: -10px; left: 25px; }
        .d-down { width: 40px; height: 45px; bottom: -10px; left: 25px; }
        .d-left { width: 45px; height: 40px; top: 25px; left: -10px; }
        .d-right { width: 45px; height: 40px; top: 25px; right: -10px; }
        
        /* D-PAD 凹み効果 */
        .d-up:active ~ .cross.c-v { transform: translateY(-1px); box-shadow: inset 1px 1px 3px rgba(0,0,0,0.5); }
        .d-down:active ~ .cross.c-v { transform: translateY(1px); box-shadow: inset 1px 1px 3px rgba(0,0,0,0.5); }
        .d-left:active ~ .cross.c-h { transform: translateX(-1px); box-shadow: inset 1px 1px 3px rgba(0,0,0,0.5); }
        .d-right:active ~ .cross.c-h { transform: translateX(1px); box-shadow: inset 1px 1px 3px rgba(0,0,0,0.5); }
        .d-btn:active ~ .cross.c-c { box-shadow: inset 2px 2px 5px rgba(0,0,0,0.8); }


        /* AB BUTTONS */
        .ab-area { 
            position: absolute; 
            top: 10px;   
            right: 25px; 
            width: 130px; 
            height: 60px; 
            transform: rotate(-25deg); 
            display: flex; 
            gap: 15px; 
            align-items: flex-end; 
        }
        .btn-wrapper-ab { display: flex; flex-direction: column; align-items: center; position: relative; }
        .btn-round { 
            width: 45px; height: 45px; background: #cc3333; border-radius: 50%; 
            box-shadow: 0px 4px 6px rgba(0,0,0,0.6), inset 0px 1px 1px rgba(255,255,255,0.4);
            display: flex; justify-content: center; align-items: center; font-size: 10px; color: rgba(0,0,0,0.2); 
            cursor: pointer; position: relative; 
            transition: transform 0.05s ease, box-shadow 0.05s ease;
        }
        /* A/B ボタンの凹み効果 */
        .btn-round:active { 
            transform: translate(1px, 1px);
            box-shadow: inset 0 0 8px rgba(0,0,0,0.8), 0 0 2px rgba(0,0,0,0.2);
        }
        .btn-round.pressing { background: #ff5555; box-shadow: 0 0 10px #ff0000; }

        /* START/SELECT */
        .meta-area { position: absolute; bottom: 80px; left: 50%; transform: translateX(-50%); display: flex; gap: 25px; } 
        .btn-wrapper { display: flex; flex-direction: column; align-items: center; position: relative; }
        .btn-oval { 
            width: 50px; height: 12px; background: #666; border-radius: 10px; transform: rotate(-25deg); 
            box-shadow: 1px 1px 3px rgba(0,0,0,0.4); cursor: pointer; position: relative; 
            transition: transform 0.05s ease, box-shadow 0.05s ease;
        }
        /* START/SELECT ボタンの凹み効果 */
        .btn-oval:active { 
            transform: rotate(-25deg) translate(1px, 1px); 
            box-shadow: inset 1px 1px 5px rgba(0,0,0,0.7);
        }
        .meta-label { margin-top: 8px; font-size: 8px; color: #444; font-weight: bold; letter-spacing: 1px; transform: rotate(-25deg); }

        /* ZOOM SLIDER */
        .zoom-integrator {
            position: absolute; bottom: 10px; left: 50%; transform: translateX(-50%); 
            width: 240px; height: 60px; background: transparent;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }
        .zoom-label { font-size: 8px; color: #666; letter-spacing: 2px; margin-bottom: 5px; text-shadow: 1px 1px 0 rgba(255,255,255,0.5); font-weight: bold; }
        .slider-container {
            display: flex; align-items: center; gap: 10px; background: #b6b6b6; padding: 5px 15px;
            border-radius: 30px; box-shadow: inset 2px 2px 5px rgba(0,0,0,0.3), inset -1px -1px 2px rgba(255,255,255,0.5); border-bottom: 1px solid rgba(255,255,255,0.5);
        }
        input[type=range] { -webkit-appearance: none; width: 140px; background: transparent; }
        input[type=range]:focus { outline: none; }
        input[type=range]::-webkit-slider-runnable-track { width: 100%; height: 6px; cursor: pointer; background: #333; border-radius: 3px; box-shadow: inset 1px 1px 2px rgba(0,0,0,0.8); }
        input[type=range]::-webkit-slider-thumb {
            height: 18px; width: 28px; border-radius: 4px; background: #555; cursor: pointer; -webkit-appearance: none; margin-top: -6px;
            box-shadow: inset 1px 1px 1px rgba(255,255,255,0.3), inset -1px -1px 1px rgba(0,0,0,0.5), 2px 2px 4px rgba(0,0,0,0.5);
            background-image: linear-gradient(90deg, transparent 50%, rgba(0,0,0,0.2) 50%); background-size: 4px 100%;
        }
        #btnResetZoom, #btnReload {
            width: 20px; height: 20px; border-radius: 50%; background: #888; border: none;
            box-shadow: 2px 2px 4px rgba(0,0,0,0.4), inset 1px 1px 2px rgba(255,255,255,0.4); cursor: pointer; font-size: 0; position: relative;
            transition: transform 0.05s ease, box-shadow 0.05s ease;
            margin-left: 5px;
        }
        /* Zoom/Reload ボタンの凹み効果 */
        #btnResetZoom:active, #btnReload:active { 
            transform: scale(0.9);
            box-shadow: inset 0 0 8px rgba(0,0,0,0.6);
        }

        #btnResetZoom::after { content: "R"; font-size: 8px; color: #333; font-weight: bold; position: absolute; top:50%; left:50%; transform:translate(-50%,-50%); }
        #btnReload::after {
            content: "↻"; 
            font-size: 14px; 
            color: #333; 
            font-weight: bold; 
            position: absolute; 
            top:50%; left:50%; 
            transform:translate(-50%,-50%); 
            line-height: 1;
        }

        .battery-led { position: absolute; left: 10px; top: 40%; width: 8px; height: 8px; background: #444; border-radius: 50%; transition: 0.1s; }
        .battery-led.on { background: #f00; box-shadow: 0 0 8px #f00; }

        video { display: none; }
    </style>
</head>
<body>

    <div id="gbContainer">
        <div class="gb-body">
            <div class="logo-text">NINTENDO GAME BOY™</div>
            
            <div class="screen-lens">
                <div class="battery-led" id="led"></div>
                <div class="screen-cover">
                    <canvas id="gbCanvas" width="1080" height="1080"></canvas>
                    <div class="scanlines"></div>
                    <div id="toast">BOOTING...</div>

                    <div id="previewContainer">
                        <img id="previewMediaImg" style="display:none; max-width: 100%; max-height: 100%; object-fit: contain; image-rendering: pixelated;">
                        <video id="previewMediaVideo" controls autoplay loop style="display:none; max-width: 100%; max-height: 100%; object-fit: contain; image-rendering: pixelated;"></video>
                        
                        <div id="previewControls">
                            <button id="btnCancel">CANCEL</button>
                            <button id="btnSave">SAVE TO DEVICE</button>
                        </div>
                    </div>
                </div>
            </div>

            <div class="controls">
                <div class="dpad-area">
                    <div class="cross c-h"></div><div class="cross c-v"></div><div class="cross c-c"></div>
                    <div class="d-btn d-up" id="dUp"></div><div class="d-btn d-down" id="dDown"></div>
                    <div class="d-btn d-left" id="dLeft"></div><div class="d-btn d-right" id="dRight"></div>
                </div>
                <div class="ab-area">
                    <div class="btn-wrapper-ab"><div class="btn-round" id="btnB">B</div></div>
                    <div class="btn-wrapper-ab" style="margin-top:-20px;"><div class="btn-round" id="btnA">A</div></div>
                </div>
                
                <div class="meta-area">
                    <div class="btn-wrapper"><div class="btn-oval" id="btnSelect"></div><div class="meta-label">SELECT</div></div>
                    <div class="btn-wrapper"><div class="btn-oval" id="btnStart"></div><div class="meta-label">START</div></div>
                </div>

                <div class="zoom-integrator">
                    <div class="zoom-label">ZOOM LEVEL</div> 
                    <div class="slider-container">
                        <input type="range" id="zoomSlider" min="1.0" max="2.0" step="0.01" value="1.0"> 
                        <button id="btnResetZoom" title="Reset Zoom"></button> 
                        <button id="btnReload" title="Reload Page"></button> 
                    </div>
                </div>
            </div>
        </div>
    </div>

    <video id="video" autoplay playsinline muted></video>

    <script>
        const container = document.getElementById('gbContainer');
        const zoomSlider = document.getElementById('zoomSlider');
        const btnResetZoom = document.getElementById('btnResetZoom'); 
        const btnReload = document.getElementById('btnReload'); 
        
        function autoFitScreen() {
            const baseW = 340; const baseH = 600;
            const s = window.innerHeight / baseH; 
            container.style.transform = `scale(${s})`;
        }
        window.addEventListener('load', autoFitScreen); window.addEventListener('resize', autoFitScreen);

        const GB_RES = 135; 
        const FINAL_RES = 1080;

        const config = { paletteIdx: 0, frameIdx: 0, brightness: 0, contrast: 2, camFacing: 'environment', zoomLevel: 1.0, locationName: "SHIBUYA, TOKYO" }; 

        const palettes = [
            { name: "CLASSIC GREEN", colors: [[15,56,15], [48,98,48], [139,172,15], [155,188,15]], border: "#306230" },
            { name: "GREY SCALE", colors: [[20,20,20], [80,80,80], [160,160,160], [240,240,240]], border: "#555" },
            { name: "VIRTUAL RED", colors: [[40,0,0], [100,0,0], [180,0,0], [255,50,50]], border: "#800" },
            { name: "CYBER BLUE", colors: [[0,20,40], [0,70,110], [0,140,190], [180,230,255]], border: "#004070" },
            { name: "16 COLOR", colors: [], border: "#000" } 
        ];
        const frames = ["OFF", "LOCATION TAG", "DATETIME", "FILM", "SCANLINE", "WHITE BORDER"]; 
        const bayerMatrix = [[0, 8, 2, 10],[12, 4, 14, 6],[3, 11, 1, 9],[15, 7, 13, 5]];

        const canvas = document.getElementById('gbCanvas');
        const ctx = canvas.getContext('2d', { willReadFrequently: true });
        const offCanvas = document.createElement('canvas');
        offCanvas.width = GB_RES; offCanvas.height = GB_RES;
        const offCtx = offCanvas.getContext('2d', { willReadFrequently: true });

        const video = document.getElementById('video');
        const led = document.getElementById('led');
        const toast = document.getElementById('toast');
        const previewContainer = document.getElementById('previewContainer');
        const previewMediaImg = document.getElementById('previewMediaImg');
        const previewMediaVideo = document.getElementById('previewMediaVideo');
        const btnSave = document.getElementById('btnSave');
        const btnCancel = document.getElementById('btnCancel');
        let currentMediaBlob = null;
        let currentMediaExt = null;
        
        let isRecording = false, mediaRecorder, recordedChunks = [], stream = null, longPressTimer, isLongPress = false;

        let recMimeType = 'video/webm'; let recExt = 'webm';
        if (MediaRecorder.isTypeSupported('video/mp4')) { recMimeType = 'video/mp4'; recExt = 'mp4'; }
        else if (MediaRecorder.isTypeSupported('video/mp4;codecs=h264')) { recMimeType = 'video/mp4;codecs=h264'; recExt = 'mp4'; }

        async function initCamera() {
            if (stream) stream.getTracks().forEach(t => t.stop());
            try {
                stream = await navigator.mediaDevices.getUserMedia({ video: { width: {ideal: 1080}, height: {ideal: 1080}, facingMode: config.camFacing }, audio: false });
                video.srcObject = stream;
                video.onloadedmetadata = () => { video.play(); requestAnimationFrame(loop); };
                
                const recStream = canvas.captureStream(30);
                mediaRecorder = new MediaRecorder(recStream, { mimeType: recMimeType, videoBitsPerSecond: 5000000 }); 
                mediaRecorder.ondataavailable = e => { if(e.data.size>0) recordedChunks.push(e.data); };
                mediaRecorder.onstop = showVideoPreview; 
            } catch(e) { console.error(e); showToast("CAMERA ERROR"); }
        }

        function applyZoom(zoom) { 
            config.zoomLevel = parseFloat(zoom); 
            zoomSlider.value = config.zoomLevel;
            showToast(`ZOOM: x${config.zoomLevel.toFixed(1)}`);
        }
        
        zoomSlider.addEventListener('input', (e) => applyZoom(e.target.value));
        btnResetZoom.addEventListener('click', () => applyZoom(1.0));
        
        btnReload.addEventListener('click', (e) => { 
            e.preventDefault(); 
            e.stopPropagation(); 
            location.reload(); 
        }); 

        function showImagePreview(dataURL) {
            previewMediaImg.src = dataURL;
            previewMediaImg.style.display = 'block';
            previewMediaVideo.style.display = 'none';
            previewContainer.style.display = 'flex';
            currentMediaExt = 'png';
            currentMediaBlob = dataURL; 
            showToast("PREVIEW PHOTO");
        }

        function showVideoPreview() { 
            const blob = new Blob(recordedChunks, {type: recMimeType});
            const url = URL.createObjectURL(blob);
            
            previewMediaVideo.src = url;
            previewMediaVideo.style.display = 'block';
            previewMediaImg.style.display = 'none';
            previewContainer.style.display = 'flex';
            
            currentMediaBlob = blob; 
            currentMediaExt = recExt;
            recordedChunks = []; 
            showToast("PREVIEW VIDEO");
        }

        function saveMedia() {
            if (!currentMediaBlob) return;

            const a = document.createElement('a');
            const now = Date.now();
            let url = '';
            let filename = '';

            if (currentMediaExt === 'png') {
                url = currentMediaBlob;
                filename = `gb-photo-${now}.${currentMediaExt}`;
            } else {
                url = URL.createObjectURL(currentMediaBlob);
                filename = `gb-video-${now}.${currentMediaExt}`;
            }

            a.href = url;
            a.download = filename;
            a.click();
            
            if (currentMediaExt !== 'png') URL.revokeObjectURL(url);
            
            hidePreview();
            showToast("SAVED");
        }
        
        function hidePreview() {
            previewContainer.style.display = 'none';
            previewMediaImg.src = '';
            previewMediaVideo.src = '';
            currentMediaBlob = null;
            currentMediaExt = null;
            if (previewMediaVideo.src) URL.revokeObjectURL(previewMediaVideo.src);
        }

        btnSave.addEventListener('click', saveMedia);
        btnCancel.addEventListener('click', hidePreview);

        function updateLocation() {
            config.locationName = "GETTING LOCATION...";
            setTimeout(() => {
                config.locationName = "SHIBUYA, TOKYO"; 
                showToast("LOCATION READY");
            }, 1000);
        }

        function drawCurrentFrameOnCanvas() {
            const vw = video.videoWidth, vh = video.videoHeight;
            const minDim = Math.min(vw, vh);
            const zoomFactor = 1.0 / config.zoomLevel; 
            let cropDim = Math.min(minDim, minDim * zoomFactor); 
            let sx = Math.max(0, (vw - cropDim) / 2);
            let sy = Math.max(0, (vh - cropDim) / 2);
            
            if (sx + cropDim > vw) cropDim = vw - sx;
            if (sy + cropDim > vh) cropDim = vh - sy;
            
            // 1. 低解像度キャンバスに描画（クロップ、リサイズ）
            offCtx.drawImage(video, sx, sy, cropDim, cropDim, 0, 0, GB_RES, GB_RES);
            
            const is16Color = palettes[config.paletteIdx].name === "16 COLOR";
            const imgData = offCtx.getImageData(0,0,GB_RES,GB_RES); 
            const d = imgData.data;
            const cF = (259*(config.contrast*10+255))/(255*(259-config.contrast*10));
            const bV = config.brightness*10;
            const Q_STEPS = 4; const Q_MAX = Q_STEPS - 1; const Q_FACTOR = 255 / Q_MAX; 

            let x = 0; let y = 0;
            for(let i=0; i<d.length; i+=4) {
                if (is16Color) {
                    let r = cF * (d[i] - 128) + 128 + bV;
                    let g = cF * (d[i+1] - 128) + 128 + bV;
                    let b = cF * (d[i+2] - 128) + 128 + bV;
                    r = Math.round(r / 255 * Q_MAX) * Q_FACTOR;
                    g = Math.round(g / 255 * Q_MAX) * Q_FACTOR;
                    b = Math.round(b / 255 * Q_MAX) * Q_FACTOR;
                    d[i] = Math.min(255, Math.max(0, r));
                    d[i+1] = Math.min(255, Math.max(0, g));
                    d[i+2] = Math.min(255, Math.max(0, b));
                } else {
                    const pal = palettes[config.paletteIdx].colors;
                    let g = 0.299*d[i] + 0.587*d[i+1] + 0.114*d[i+2];
                    g = cF*(g-128)+128+bV;
                    const dither = (bayerMatrix[y%4][x%4]-8)*4; 
                    g += dither;
                    let idx = 0; if (g>=64 && g<128) idx=1; else if (g>=128 && g<192) idx=2; else if (g>=192) idx=3;
                    if (g<0) idx=0; if (g>255) idx=3;
                    d[i]=pal[idx][0]; d[i+1]=pal[idx][1]; d[i+2]=pal[idx][2];
                }
                x++; if(x>=GB_RES){ x=0; y++; }
            }
            offCtx.putImageData(imgData,0,0);

            // 2. メインキャンバスに拡大描画
            ctx.imageSmoothingEnabled = false;
            ctx.drawImage(offCanvas, 0, 0, FINAL_RES, FINAL_RES);

            // 3. フレームの描画
            drawFrame(palettes[config.paletteIdx].colors, ctx); 
        }

        function loop() {
            if (video.readyState === 4) {
                if (previewContainer.style.display === 'flex') {
                    requestAnimationFrame(loop);
                    return;
                }
                
                // 動画ストリームのラグを防ぐため、常に描画
                drawCurrentFrameOnCanvas(); 

                const is16Color = palettes[config.paletteIdx].name === "16 COLOR";
                canvas.parentElement.querySelector('.scanlines').style.opacity = is16Color ? 0.1 : 0.4;
            }
            requestAnimationFrame(loop);
        }

        function drawFrame(pal, ctx) {
            const type = frames[config.frameIdx]; 
            const is16Color = palettes[config.paletteIdx].name === "16 COLOR";
            const dk = pal.length > 0 && !is16Color ? `rgb(${pal[0].join(',')})` : '#000', 
                  lt = pal.length > 0 && !is16Color ? `rgb(${pal[3].join(',')})` : '#FFF';
            ctx.fillStyle = dk;
            const borderSize = 80; 

            if (type==="FILM") { 
                ctx.fillRect(0,0,1080,60); ctx.fillRect(0,1020,1080,60); 
                ctx.fillRect(0,0,60,1080); ctx.fillRect(1020,0,60,1080); 
                ctx.fillStyle=lt; ctx.font = "40px 'Press Start 2P'"; ctx.fillText("POCKET CAM", 80, 45); 
            } 
            else if (type==="SCANLINE") { 
                ctx.fillStyle="rgba(0,0,0,0.2)"; 
                for(let y=0;y<1080;y+=8) ctx.fillRect(0,y,1080,4); 
            } 
            else if (type==="DATETIME") {
                const now = new Date();
                const dateStr = `${now.getFullYear()}/${(now.getMonth()+1).toString().padStart(2, '0')}/${now.getDate().toString().padStart(2, '0')}`;
                const timeStr = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
                
                ctx.fillStyle=dk;
                ctx.fillRect(0,0,1080,80); 
                
                ctx.fillStyle=lt; ctx.font = "40px 'Press Start 2P'"; ctx.fillText(dateStr, 80, 40);
                ctx.fillText(timeStr, 80, 75);
            }
            else if (type==="WHITE BORDER") {
                ctx.fillStyle="white"; 
                ctx.fillRect(0, 0, 1080, borderSize); 
                ctx.fillRect(0, 1080 - borderSize, 1080, borderSize);
                ctx.fillRect(0, borderSize, borderSize, 1080 - 2 * borderSize);
                ctx.fillRect(1080 - borderSize, borderSize, borderSize, 1080 - 2 * borderSize);
            }
            else if (type==="LOCATION TAG") {
                ctx.fillStyle=dk;
                ctx.fillRect(0, 0, 1080, 80);
                
                ctx.fillStyle=lt; 
                ctx.font = "40px 'Press Start 2P'"; ctx.fillText(config.locationName, 80, 55); 
            }
        }

        function showToast(msg) { toast.innerText = msg; toast.style.display = 'block'; clearTimeout(toast.timer); toast.timer = setTimeout(() => toast.style.display = 'none', 1000); }
        function handleDpad(key) {
            if (previewContainer.style.display === 'flex') return;

            if (key === 'up') config.brightness = Math.min(5, config.brightness + 1);
            if (key === 'down') config.brightness = Math.max(-5, config.brightness - 1);
            if (key === 'right') config.contrast = Math.min(5, config.contrast + 1);
            if (key === 'left') config.contrast = Math.max(-5, config.contrast - 1);
            if (key === 'up' || key === 'down') showToast(`BRIGHT: ${config.brightness}`);
            if (key === 'left' || key === 'right') showToast(`CONTRAST: ${config.contrast}`);
        }

        document.getElementById('btnB').addEventListener('click', (e) => {
            e.preventDefault(); 
            if (previewContainer.style.display === 'flex') {
                hidePreview(); 
                return;
            }
            config.paletteIdx = (config.paletteIdx + 1) % palettes.length; 
            showToast(palettes[config.paletteIdx].name);
        });

        document.getElementById('btnSelect').addEventListener('click', (e) => {
            e.preventDefault(); 
            if (previewContainer.style.display === 'flex') return;
            config.frameIdx = (config.frameIdx + 1) % frames.length; 
            showToast(`FRAME: ${frames[config.frameIdx]}`);
            
            if (frames[config.frameIdx] === "LOCATION TAG") {
                updateLocation();
            }
        });

        document.getElementById('btnStart').addEventListener('click', (e) => {
            e.preventDefault(); 
            if (previewContainer.style.display === 'flex') return;
            config.camFacing = (config.camFacing === 'user') ? 'environment' : 'user';
            initCamera(); showToast("SWITCH CAM");
        });

        const btnA = document.getElementById('btnA');
        function startPressA(e) {
            e.preventDefault(); 
            if (isRecording || previewContainer.style.display === 'flex') return;
            isLongPress = false; btnA.classList.add('pressing');
            longPressTimer = setTimeout(() => {
                isLongPress = true; mediaRecorder.start(); isRecording = true; led.classList.add('on'); showToast("REC 1080P");
            }, 400);
        }
        function endPressA(e) {
            e.preventDefault(); clearTimeout(longPressTimer); btnA.classList.remove('pressing');
            
            if (previewContainer.style.display === 'flex') {
                saveMedia();
                return;
            }

            if (isLongPress) {
                if(isRecording) { mediaRecorder.stop(); isRecording = false; led.classList.remove('on'); }
            } else {
                // ★修正: 静止画撮影時、フレームを再描画してからキャプチャ
                drawCurrentFrameOnCanvas(); 
                const dataURL = canvas.toDataURL('image/png', 1.0);
                showImagePreview(dataURL);
                canvas.style.opacity = 0; setTimeout(() => canvas.style.opacity = 1, 100);
            }
            isLongPress = false;
        }
        btnA.addEventListener('mousedown', startPressA); btnA.addEventListener('touchstart', startPressA);
        btnA.addEventListener('mouseup', endPressA); btnA.addEventListener('touchend', endPressA);

        ['Up','Down','Left','Right'].forEach(d => document.getElementById('d'+d).addEventListener('click', (e)=>{ e.preventDefault(); handleDpad(d.toLowerCase()); }));
        
        updateLocation(); 
        initCamera(); 
        applyZoom(config.zoomLevel); 
        showToast("READY (1080P)");
    </script>
</body>
</html>

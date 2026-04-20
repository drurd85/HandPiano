<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hand Tracking Piano</title>
    
    <!-- MediaPipe Hands CDN -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/control_utils/control_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>

    <style>
        :root {
            --primary: #1a1a2e;
            --secondary: #16213e;
            --accent: #0f3460;
            --gold: #e94560;
            --text: #ffffff;
        }

        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, var(--primary) 0%, var(--secondary) 100%);
            color: var(--text);
            overflow: hidden;
            height: 100vh;
        }

        .container {
            display: flex;
            height: 100vh;
            gap: 20px;
            padding: 20px;
        }

        .camera-section {
            flex: 1;
            position: relative;
            border-radius: 15px;
            overflow: hidden;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            background: #000;
        }

        .input_video {
            position: absolute;
            width: 100%;
            height: 100%;
            object-fit: cover;
            transform: scaleX(-1);
        }

        #canvas {
            position: absolute;
            width: 100%;
            height: 100%;
            transform: scaleX(-1);
        }

        .piano-section {
            flex: 1;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .title-bar {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            padding: 20px;
            border-radius: 15px;
            text-align: center;
        }

        .title-bar h1 {
            font-size: 2.5em;
            font-weight: 300;
            letter-spacing: 3px;
            margin-bottom: 10px;
            text-shadow: 0 0 20px rgba(233, 69, 96, 0.5);
        }

        .title-bar p {
            opacity: 0.7;
            font-size: 0.9em;
        }

        .piano-keyboard {
            flex: 1;
            background: linear-gradient(135deg, #1f1f3d 0%, #2a2a4e 100%);
            border-radius: 15px;
            padding: 20px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            display: flex;
            flex-direction: column;
            gap: 15px;
        }

        .keys-container {
            display: flex;
            gap: 5px;
            flex: 1;
            justify-content: center;
            align-items: center;
        }

        .piano-key {
            flex: 1;
            background: linear-gradient(145deg, #ffffff 0%, #e8e8e8 100%);
            border: 2px solid #999;
            border-radius: 8px 8px 12px 12px;
            cursor: pointer;
            transition: all 0.05s ease;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: flex-end;
            padding: 10px;
            min-height: 150px;
            position: relative;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2), inset 0 -2px 5px rgba(0, 0, 0, 0.1);
            user-select: none;
        }

        .piano-key:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3), inset 0 -2px 5px rgba(0, 0, 0, 0.1);
        }

        .piano-key.active {
            background: linear-gradient(145deg, #f0f0f0 0%, #ffffff 100%);
            transform: translateY(3px);
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2), inset 0 -2px 5px rgba(0, 0, 0, 0.1);
        }

        .piano-key.white-key {
            color: #333;
        }

        .piano-key.black-key {
            background: linear-gradient(145deg, #333 0%, #1a1a1a 100%);
            color: #fff;
            border-color: #000;
        }

        .piano-key.black-key.active {
            background: linear-gradient(145deg, #555 0%, #333 100%);
        }

        .key-label {
            font-size: 0.85em;
            font-weight: bold;
            opacity: 0.6;
        }

        .key-frequency {
            font-size: 0.7em;
            opacity: 0.4;
            margin-top: 5px;
        }

        .controls {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }

        .control-panel {
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.2);
            padding: 15px;
            border-radius: 10px;
        }

        .control-panel h3 {
            font-size: 0.9em;
            margin-bottom: 10px;
            opacity: 0.8;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .control-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 8px;
            font-size: 0.85em;
        }

        .control-item:last-child {
            margin-bottom: 0;
        }

        .control-label {
            opacity: 0.7;
        }

        .control-value {
            color: var(--gold);
            font-weight: bold;
        }

        input[type="range"] {
            width: 100%;
            margin: 8px 0;
            cursor: pointer;
        }

        .start-overlay {
            position: fixed;
            inset: 0;
            background: rgba(0, 0, 0, 0.9);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 1000;
            backdrop-filter: blur(5px);
        }

        .start-overlay.hidden {
            display: none;
        }

        .start-overlay h1 {
            font-size: 3em;
            margin-bottom: 20px;
            text-shadow: 0 0 30px rgba(233, 69, 96, 0.6);
            letter-spacing: 3px;
        }

        .start-overlay p {
            font-size: 1.1em;
            margin-bottom: 40px;
            opacity: 0.8;
            max-width: 500px;
            text-align: center;
        }

        .start-btn {
            padding: 16px 50px;
            font-size: 1.2em;
            background: linear-gradient(135deg, var(--gold) 0%, #ff6b6b 100%);
            border: none;
            color: white;
            border-radius: 50px;
            cursor: pointer;
            font-weight: bold;
            text-transform: uppercase;
            letter-spacing: 2px;
            transition: all 0.3s ease;
            box-shadow: 0 8px 30px rgba(233, 69, 96, 0.4);
        }

        .start-btn:hover {
            transform: scale(1.05);
            box-shadow: 0 12px 40px rgba(233, 69, 96, 0.6);
        }

        .stats {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }

        .stat-item {
            background: rgba(233, 69, 96, 0.1);
            border-left: 3px solid var(--gold);
            padding: 8px 12px;
            border-radius: 5px;
            font-size: 0.85em;
        }

        .stat-label {
            opacity: 0.7;
            display: block;
            margin-bottom: 3px;
        }

        .stat-value {
            font-size: 1.1em;
            font-weight: bold;
            color: var(--gold);
        }
    </style>
</head>
<body>

    <div class="start-overlay" id="startOverlay">
        <h1>🎹 HAND PIANO</h1>
        <p>Grant camera permissions and click below to start playing the piano with your hands</p>
        <button class="start-btn" id="startBtn">Start Playing</button>
    </div>

    <div class="container">
        <!-- Camera Feed -->
        <div class="camera-section">
            <video class="input_video" autoplay playsinline></video>
            <canvas id="canvas"></canvas>
        </div>

        <!-- Piano Interface -->
        <div class="piano-section">
            <div class="title-bar">
                <h1>HAND PIANO</h1>
                <p>Move your fingers over the keys to play</p>
            </div>

            <div class="piano-keyboard">
                <div class="keys-container" id="keysContainer"></div>

                <div class="controls">
                    <div class="control-panel">
                        <h3>📊 Detection</h3>
                        <div class="stats">
                            <div class="stat-item">
                                <span class="stat-label">Hands</span>
                                <span class="stat-value" id="handsCount">0</span>
                            </div>
                            <div class="stat-item">
                                <span class="stat-label">Active Keys</span>
                                <span class="stat-value" id="activeKeys">0</span>
                            </div>
                            <div class="stat-item">
                                <span class="stat-label">FPS</span>
                                <span class="stat-value" id="fps">0</span>
                            </div>
                            <div class="stat-item">
                                <span class="stat-label">Volume</span>
                                <span class="stat-value" id="volumeDisplay">50%</span>
                            </div>
                        </div>
                    </div>

                    <div class="control-panel">
                        <h3>⚙️ Settings</h3>
                        <div class="control-item">
                            <span class="control-label">Volume:</span>
                            <input type="range" id="volumeControl" min="0" max="100" value="50" style="flex: 1; margin-left: 10px;">
                        </div>
                        <div class="control-item">
                            <span class="control-label">Octave:</span>
                            <div style="display: flex; gap: 5px;">
                                <button id="octaveDown" style="flex: 1; padding: 5px; border: none; background: rgba(255,255,255,0.2); color: white; border-radius: 5px; cursor: pointer;">−</button>
                                <span class="control-value" id="octaveValue" style="flex: 1; text-align: center;">C4</span>
                                <button id="octaveUp" style="flex: 1; padding: 5px; border: none; background: rgba(255,255,255,0.2); color: white; border-radius: 5px; cursor: pointer;">+</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <script>
        /**
         * PIANO KEY DEFINITIONS
         */
        const NOTES = [
            { name: 'C', freq: 261.63 },
            { name: 'D', freq: 293.66 },
            { name: 'E', freq: 329.63 },
            { name: 'F', freq: 349.23 },
            { name: 'G', freq: 392.00 },
            { name: 'A', freq: 440.00 },
            { name: 'B', freq: 493.88 }
        ];

        let currentOctave = 4;
        let volume = 0.5;
        let audioCtx = null;
        let activeNotes = {};
        let currentHands = [];
        let framesThisSecond = 0;
        let lastFpsTime = performance.now();

        const videoElement = document.querySelector('.input_video');
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');

        // UI Elements
        const handsCountDisplay = document.getElementById('handsCount');
        const activeKeysDisplay = document.getElementById('activeKeys');
        const fpsDisplay = document.getElementById('fps');
        const volumeControl = document.getElementById('volumeControl');
        const volumeDisplay = document.getElementById('volumeDisplay');
        const octaveValue = document.getElementById('octaveValue');

        /**
         * INITIALIZATION
         */
        function resizeCanvas() {
            canvas.width = videoElement.clientWidth;
            canvas.height = videoElement.clientHeight;
        }
        window.addEventListener('resize', resizeCanvas);

        function createPiano() {
            const container = document.getElementById('keysContainer');
            container.innerHTML = '';

            NOTES.forEach((note, idx) => {
                const key = document.createElement('div');
                key.className = 'piano-key white-key';
                key.dataset.note = note.name;
                key.dataset.index = idx;
                key.innerHTML = `
                    <div class="key-label">${note.name}</div>
                    <div class="key-frequency">${(note.freq * Math.pow(2, currentOctave - 4)).toFixed(1)} Hz</div>
                `;
                container.appendChild(key);
            });
        }

        document.getElementById('volumeControl').addEventListener('input', (e) => {
            volume = e.target.value / 100;
            volumeDisplay.textContent = e.target.value + '%';
        });

        document.getElementById('octaveUp').addEventListener('click', () => {
            if (currentOctave < 8) {
                currentOctave++;
                octaveValue.textContent = `C${currentOctave}`;
                createPiano();
            }
        });

        document.getElementById('octaveDown').addEventListener('click', () => {
            if (currentOctave > 1) {
                currentOctave--;
                octaveValue.textContent = `C${currentOctave}`;
                createPiano();
            }
        });

        document.getElementById('startBtn').addEventListener('click', () => {
            document.getElementById('startOverlay').classList.add('hidden');
            initAudio();
            initMediaPipe();
            resizeCanvas();
            createPiano();
            requestAnimationFrame(renderLoop);
        });

        /**
         * AUDIO ENGINE
         */
        function initAudio() {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }

        function playNote(noteName, index) {
            if (!audioCtx) return;

            const noteObj = NOTES[index];
            const frequency = noteObj.freq * Math.pow(2, currentOctave - 4);
            const key = `${noteName}-${currentOctave}-${index}`;

            if (activeNotes[key]) return; // Already playing

            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();

            osc.type = 'sine';
            osc.frequency.value = frequency;

            gain.gain.setValueAtTime(volume * 0.3, audioCtx.currentTime);

            osc.connect(gain);
            gain.connect(audioCtx.destination);

            osc.start(audioCtx.currentTime);

            activeNotes[key] = { osc, gain, startTime: audioCtx.currentTime };
        }

        function stopNote(noteName, index) {
            if (!audioCtx) return;

            const key = `${noteName}-${currentOctave}-${index}`;
            if (!activeNotes[key]) return;

            const { osc, gain } = activeNotes[key];

            gain.gain.setTargetAtTime(0, audioCtx.currentTime, 0.1);
            osc.stop(audioCtx.currentTime + 0.1);

            delete activeNotes[key];
        }

        function stopAllNotes() {
            Object.keys(activeNotes).forEach(key => {
                const { osc, gain } = activeNotes[key];
                gain.gain.setTargetAtTime(0, audioCtx.currentTime, 0.05);
                osc.stop(audioCtx.currentTime + 0.05);
            });
            activeNotes = {};
        }

        /**
         * HAND TRACKING & KEY DETECTION
         */
        function getKeyboardBounds() {
            const container = document.getElementById('keysContainer');
            const rect = container.getBoundingClientRect();
            return {
                left: rect.left,
                top: rect.top,
                width: rect.width,
                height: rect.height,
                keyWidth: rect.width / NOTES.length
            };
        }

        function checkFingerOverKey(fingerPos) {
            const bounds = getKeyboardBounds();
            const canvasRect = canvas.getBoundingClientRect();

            // Convert normalized hand coordinates to canvas coordinates
            const canvasX = fingerPos.x * canvas.width;
            const canvasY = fingerPos.y * canvas.height;

            // Convert canvas coordinates to screen coordinates
            const screenX = canvasRect.left + canvasX;
            const screenY = canvasRect.top + canvasY;

            // Check if within keyboard bounds
            if (screenX < bounds.left || screenX > bounds.left + bounds.width ||
                screenY < bounds.top || screenY > bounds.top + bounds.height) {
                return null;
            }

            // Calculate which key
            const keyIndex = Math.floor((screenX - bounds.left) / bounds.keyWidth);
            return Math.max(0, Math.min(keyIndex, NOTES.length - 1));
        }

        const FINGER_TIPS = [4, 8, 12, 16, 20];
        let activeKeysByFinger = {};

        function updateKeyPress() {
            const newActiveKeys = {};

            currentHands.forEach((hand, handIdx) => {
                FINGER_TIPS.forEach((tipIdx, fingerIdx) => {
                    const fingerId = `${handIdx}-${fingerIdx}`;
                    const fingerPos = hand[tipIdx];
                    const keyIndex = checkFingerOverKey(fingerPos);

                    if (keyIndex !== null) {
                        const noteName = NOTES[keyIndex].name;
                        newActiveKeys[fingerId] = keyIndex;

                        // Play note if this is a new key
                        if (!activeKeysByFinger[fingerId] || activeKeysByFinger[fingerId] !== keyIndex) {
                            if (activeKeysByFinger[fingerId] !== undefined) {
                                stopNote(NOTES[activeKeysByFinger[fingerId]].name, activeKeysByFinger[fingerId]);
                            }
                            playNote(noteName, keyIndex);
                        }
                    } else if (activeKeysByFinger[fingerId] !== undefined) {
                        // Finger left the keyboard
                        stopNote(NOTES[activeKeysByFinger[fingerId]].name, activeKeysByFinger[fingerId]);
                    }
                });
            });

            // Stop notes for fingers that are no longer active
            Object.keys(activeKeysByFinger).forEach(fingerId => {
                if (!newActiveKeys[fingerId]) {
                    stopNote(NOTES[activeKeysByFinger[fingerId]].name, activeKeysByFinger[fingerId]);
                }
            });

            activeKeysByFinger = newActiveKeys;
            activeKeysDisplay.textContent = Object.keys(newActiveKeys).length;
        }

        /**
         * VISUALIZATION
         */
        function drawVisualization() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            if (currentHands.length === 0) return;

            currentHands.forEach((hand, handIdx) => {
                const color = handIdx === 0 ? '#00ff88' : '#ff0088';

                // Draw finger tips
                FINGER_TIPS.forEach((tipIdx, fingerIdx) => {
                    const pt = hand[tipIdx];
                    const x = pt.x * canvas.width;
                    const y = pt.y * canvas.height;
                    const keyIndex = checkFingerOverKey(pt);

                    // Draw circle for each finger
                    ctx.beginPath();
                    ctx.arc(x, y, 12, 0, Math.PI * 2);
                    ctx.fillStyle = keyIndex !== null ? '#ffff00' : color;
                    ctx.fill();
                    ctx.strokeStyle = '#fff';
                    ctx.lineWidth = 2;
                    ctx.stroke();

                    // Draw label
                    ctx.fillStyle = '#fff';
                    ctx.font = 'bold 10px Arial';
                    ctx.textAlign = 'center';
                    ctx.textBaseline = 'middle';
                    ctx.fillText(fingerIdx + 1, x, y);
                });

                // Draw palm
                if (hand[0]) {
                    const pt = hand[0];
                    const x = pt.x * canvas.width;
                    const y = pt.y * canvas.height;

                    ctx.beginPath();
                    ctx.arc(x, y, 8, 0, Math.PI * 2);
                    ctx.fillStyle = color;
                    ctx.fill();
                }
            });
        }

        /**
         * MEDIAPIPE INITIALIZATION
         */
        function initMediaPipe() {
            const hands = new Hands({
                locateFile: (file) => {
                    return `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`;
                }
            });

            hands.setOptions({
                maxNumHands: 2,
                modelComplexity: 1,
                minDetectionConfidence: 0.7,
                minTrackingConfidence: 0.7
            });

            hands.onResults((results) => {
                if (!audioCtx) return;

                handsCountDisplay.textContent = results.multiHandLandmarks ? results.multiHandLandmarks.length : 0;
                currentHands = results.multiHandLandmarks || [];
                updateKeyPress();
            });

            const camera = new Camera(videoElement, {
                onFrame: async () => {
                    await hands.sen*

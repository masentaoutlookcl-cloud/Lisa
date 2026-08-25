<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Guardian de Energía</title>

<style>
*{
    box-sizing:border-box;
    touch-action: none;
}

html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#02030b;
    cursor:none;
    user-select: none;
    -webkit-user-select: none;
}

canvas{
    position:fixed;
    inset:0;
    width:100%;
    height:100%;
}

:root{
    --color:#00eaff;
}

/* HUD */
#hud{
    position:fixed;
    top:18px;
    left:20px;
    z-index:10;
    color:white;
    font-family:Arial,sans-serif;
    font-size:18px;
    line-height:1.55;
    text-shadow: 0 0 8px var(--color), 0 0 18px var(--color);
}

.value{
    color:var(--color);
    font-weight:bold;
}

.bar{
    width:210px;
    height:11px;
    margin:3px 0 5px;
    background:#111525;
    border:1px solid #4b5370;
    border-radius:20px;
    overflow:hidden;
}

#shieldBar{
    width:100%;
    height:100%;
    background:var(--color);
    box-shadow: 0 0 12px var(--color);
    transition: width .2s, background 1s;
}

/* BOTÓN DE OPCIONES */
#settingsBtn{
    position:fixed;
    top:20px;
    right:20px;
    z-index:30;
    background:rgba(2, 3, 11, 0.75);
    border:2px solid var(--color);
    color:white;
    padding:10px 18px;
    border-radius:12px;
    font-size:16px;
    font-weight:bold;
    font-family:Arial,sans-serif;
    cursor:pointer;
    box-shadow:0 0 10px rgba(0,0,0,0.5);
    transition:transform 0.2s, background 0.2s;
}

#settingsBtn:active{
    transform:scale(0.95);
}

/* PANEL LATERAL DE OPCIONES */
#settingsPanel{
    position:fixed;
    top:0;
    right:-320px;
    width:300px;
    height:100%;
    background:rgba(6, 10, 26, 0.95);
    border-left:2px solid var(--color);
    box-shadow: -5px 0 25px rgba(0, 0, 0, 0.8);
    z-index:40;
    padding:30px 20px;
    color:white;
    font-family:Arial,sans-serif;
    transition:right 0.3s ease-in-out;
    display:flex;
    flex-direction:column;
    gap:20px;
}

#settingsPanel.open{
    right:0;
}

#settingsPanel h2{
    margin:0 0 5px;
    color:var(--color);
    text-shadow: 0 0 10px var(--color);
    font-size:24px;
    text-align:center;
}

.setting-group{
    display:flex;
    flex-direction:column;
    gap:8px;
}

.setting-group label{
    font-size:15px;
    display:flex;
    justify-content:space-between;
}

.setting-group input[type="range"]{
    accent-color: var(--color);
    cursor:pointer;
    height:6px;
}

/* BOTÓN DE PAUSA Y CERRAR */
.panel-btn{
    padding:12px;
    border:none;
    font-weight:bold;
    border-radius:8px;
    cursor:pointer;
    font-size:16px;
}

#pauseGameBtn{
    background:rgba(0, 234, 255, 0.15);
    color:var(--color);
    border:1px solid var(--color);
    box-shadow: 0 0 8px rgba(0,234,255,0.2);
}

#closeSettingsBtn{
    margin-top:auto;
    background:var(--color);
    color:#001018;
}

/* MENSAJE DE PAUSA EN PANTALLA */
#pauseOverlay{
    position:fixed;
    inset:0;
    background:rgba(2, 3, 11, 0.65);
    backdrop-filter: blur(4px);
    z-index:25;
    display:none;
    align-items:center;
    justify-content:center;
    color:white;
    font-family:Arial,sans-serif;
    font-size:45px;
    font-weight:bold;
    text-shadow: 0 0 20px var(--color);
    pointer-events:none;
    letter-spacing: 2px;
}

/* MENSAJE DE NIVEL */
#levelMessage{
    position:fixed;
    left:50%;
    top:38%;
    transform: translate(-50%,-50%) scale(.5);
    z-index:20;
    color:white;
    font-family:Arial,sans-serif;
    font-size:65px;
    font-weight:bold;
    opacity:0;
    text-shadow: 0 0 15px var(--color), 0 0 40px var(--color), 0 0 80px var(--color);
    pointer-events:none;
}

.showLevel{
    animation:levelAnimation 1.3s ease-out;
}

@keyframes levelAnimation{
    0%{ opacity:0; transform: translate(-50%,-50%) scale(.4); }
    30%{ opacity:1; transform: translate(-50%,-50%) scale(1.2); }
    70%{ opacity:1; transform: translate(-50%,-50%) scale(1); }
    100%{ opacity:0; transform: translate(-50%,-50%) scale(1.5); }
}

/* GAME OVER */
#gameOver{
    position:fixed;
    inset:0;
    display:none;
    align-items:center;
    justify-content:center;
    flex-direction:column;
    z-index:50;
    background:rgba(0,0,10,.9);
    color:white;
    font-family:Arial,sans-serif;
    text-align:center;
}

#gameOver h1{
    font-size:60px;
    margin:10px;
    color:var(--color);
    text-shadow: 0 0 30px var(--color);
}

#restart{
    margin-top:20px;
    padding:14px 32px;
    border:0;
    border-radius:12px;
    background:var(--color);
    color:#001018;
    font-size:19px;
    font-weight:bold;
    cursor:pointer;
}
</style>
</head>

<body>

<canvas id="gameCanvas"></canvas>

<div id="hud">
    ⭐ Puntos: <span class="value" id="score">0</span><br>
    🔥 Nivel: <span class="value" id="level">1</span><br>
    ⚡ Combo: <span class="value" id="combo">0</span><br>
    🛡️ Escudo
    <div class="bar">
        <div id="shieldBar"></div>
    </div>
</div>

<div id="pauseOverlay">JUEGO PAUSADO</div>

<button id="settingsBtn">⚙️ Opciones</button>

<!-- PANEL DE CONFIGURACIÓN -->
<div id="settingsPanel">
    <h2>OPCIONES</h2>
    
    <div class="setting-group">
        <label>🎵 Música: <span id="musicVolVal">30%</span></label>
        <input type="range" id="musicVolSlider" min="0" max="100" value="30">
    </div>

    <div class="setting-group">
        <label>🔊 Efectos (SFX): <span id="sfxVolVal">70%</span></label>
        <input type="range" id="sfxVolSlider" min="0" max="100" value="70">
    </div>

    <hr style="border:0; border-top:1px solid #28345c; margin:5px 0;">

    <div class="setting-group">
        <button id="pauseGameBtn" class="panel-btn">⏸️ Pausar Juego</button>
    </div>

    <button id="closeSettingsBtn" class="panel-btn">Cerrar</button>
</div>

<div id="levelMessage">
    NIVEL <span id="levelNumber">1</span>
</div>

<div id="gameOver">
    <h1>FIN DEL JUEGO</h1>
    <div id="finalStats"></div>
    <button id="restart">JUGAR OTRA VEZ</button>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

let W = innerWidth;
let H = innerHeight;

function resize(){
    W = canvas.width = innerWidth;
    H = canvas.height = innerHeight;
}
window.addEventListener("resize", resize);
resize();

/* =====================================
   CONTROL DE AUDIO Y ESTADOS
===================================== */
const AudioContext = window.AudioContext || window.webkitAudioContext;
let audioCtx = null;
let musicTimer = null;
let musicStep = 0;

let musicVolume = 0.03;
let sfxVolume = 0.7;
let isPaused = false;

const musicSlider = document.getElementById("musicVolSlider");
const sfxSlider = document.getElementById("sfxVolSlider");
const musicVolVal = document.getElementById("musicVolVal");
const sfxVolVal = document.getElementById("sfxVolVal");
const pauseGameBtn = document.getElementById("pauseGameBtn");
const pauseOverlay = document.getElementById("pauseOverlay");

musicSlider.addEventListener("input", (e) => {
    const val = e.target.value;
    musicVolVal.textContent = val + "%";
    musicVolume = (val / 100) * 0.1;
});

sfxSlider.addEventListener("input", (e) => {
    const val = e.target.value;
    sfxVolVal.textContent = val + "%";
    sfxVolume = val / 100;
});

// Función de Pausa
pauseGameBtn.addEventListener("click", () => {
    isPaused = !isPaused;
    if (isPaused) {
        pauseGameBtn.textContent = "▶️ Reanudar";
        pauseOverlay.style.display = "flex";
        if (audioCtx && audioCtx.state === 'running') {
            audioCtx.suspend();
        }
    } else {
        pauseGameBtn.textContent = "⏸️ Pausar Juego";
        pauseOverlay.style.display = "none";
        if (audioCtx && audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
        lastTime = performance.now();
        requestAnimationFrame(gameLoop);
    }
});

// Frecuencias musicales
const NOTES = {
    C3: 130.81, D3: 146.83, E3: 164.81, F3: 174.61, G3: 196.00, A3: 220.00, B3: 246.94,
    C4: 261.63, D4: 293.66, E4: 329.63, F4: 349.23, G4: 392.00, A4: 440.00, B4: 493.88,
    C5: 523.25, D5: 587.33, E5: 659.25, G5: 783.99, A5: 880.00, OFF: 0
};

const TRACKS = [
    {
        bpm: 120,
        melody: [
            NOTES.C4, NOTES.E4, NOTES.G4, NOTES.B4, NOTES.C5, NOTES.B4, NOTES.G4, NOTES.E4,
            NOTES.A3, NOTES.C4, NOTES.E4, NOTES.G4, NOTES.A4, NOTES.G4, NOTES.E4, NOTES.C4
        ]
    },
    {
        bpm: 140,
        melody: [
            NOTES.E4, NOTES.E4, NOTES.G4, NOTES.E4, NOTES.A4, NOTES.E4, NOTES.B4, NOTES.E4,
            NOTES.C5, NOTES.B4, NOTES.A4, NOTES.G4, NOTES.F4, NOTES.E4, NOTES.D4, NOTES.B3
        ]
    },
    {
        bpm: 160,
        melody: [
            NOTES.A4, NOTES.C5, NOTES.E5, NOTES.A5, NOTES.G5, NOTES.E5, NOTES.C5, NOTES.G4,
            NOTES.F4, NOTES.A4, NOTES.C5, NOTES.F5, NOTES.E5, NOTES.C5, NOTES.G4, NOTES.E4
        ]
    }
];

function initAudio() {
    if (!audioCtx) {
        audioCtx = new AudioContext();
        startMusic();
    }
    if (audioCtx.state === 'suspended' && !isPaused) {
        audioCtx.resume();
    }
}

function startMusic() {
    if (musicTimer) clearInterval(musicTimer);
    
    const trackIndex = Math.min(Math.floor((level - 1) / 2), TRACKS.length - 1);
    const currentTrack = TRACKS[trackIndex];
    const stepDuration = (60 / currentTrack.bpm) * 1000 / 2;

    musicTimer = setInterval(() => {
        if (!running || isPaused || !audioCtx || musicVolume === 0) return;

        const freq = currentTrack.melody[musicStep % currentTrack.melody.length];
        if (freq > 0) {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();

            osc.type = "triangle";
            osc.frequency.setValueAtTime(freq, audioCtx.currentTime);

            gain.gain.setValueAtTime(musicVolume, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + (stepDuration/1000));

            osc.connect(gain);
            gain.connect(audioCtx.destination);

            osc.start();
            osc.stop(audioCtx.currentTime + (stepDuration/1000));
        }

        musicStep++;
    }, stepDuration);
}

function playScoreSound() {
    if (!audioCtx || sfxVolume === 0 || isPaused) return;
    const osc = audioCtx.createOscillator();
    const gain = audioCtx.createGain();
    
    osc.type = "sine";
    osc.frequency.setValueAtTime(587.33, audioCtx.currentTime);
    osc.frequency.exponentialRampToValueAtTime(880, audioCtx.currentTime + 0.08);
    
    gain.gain.setValueAtTime(0.15 * sfxVolume, audioCtx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.08);
    
    osc.connect(gain);
    gain.connect(audioCtx.destination);
    
    osc.start();
    osc.stop(audioCtx.currentTime + 0.08);
}

function playSpecialSound() {
    if (!audioCtx || sfxVolume === 0 || isPaused) return;

    const freqs = [523.25, 659.25, 783.99, 1046.50];
    freqs.forEach((freq, index) => {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();

        osc.type = "triangle";
        osc.frequency.setValueAtTime(freq, audioCtx.currentTime + (index * 0.04));

        gain.gain.setValueAtTime(0.12 * sfxVolume, audioCtx.currentTime + (index * 0.04));
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + (index * 0.04) + 0.15);

        osc.connect(gain);
        gain.connect(audioCtx.destination);

        osc.start(audioCtx.currentTime + (index * 0.04));
        osc.stop(audioCtx.currentTime + (index * 0.04) + 0.15);
    });
}

function playLevelUpSound() {
    if (!audioCtx || sfxVolume === 0 || isPaused) return;

    const notes = [440, 554.37, 659.25, 880];
    notes.forEach((freq, index) => {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();

        osc.type = "square";
        osc.frequency.setValueAtTime(freq, audioCtx.currentTime + (index * 0.07));

        gain.gain.setValueAtTime(0.08 * sfxVolume, audioCtx.currentTime + (index * 0.07));
        gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + (index * 0.07) + 0.2);

        osc.connect(gain);
        gain.connect(audioCtx.destination);

        osc.start(audioCtx.currentTime + (index * 0.07));
        osc.stop(audioCtx.currentTime + (index * 0.07) + 0.2);
    });
}

/* =====================================
   INTERFAZ PANEL OPCIONES
===================================== */
const settingsPanel = document.getElementById("settingsPanel");
const settingsBtn = document.getElementById("settingsBtn");
const closeSettingsBtn = document.getElementById("closeSettingsBtn");

settingsBtn.addEventListener("click", () => {
    initAudio();
    settingsPanel.classList.toggle("open");
});

closeSettingsBtn.addEventListener("click", () => {
    settingsPanel.classList.remove("open");
});

const colors = [
    "#00eaff", "#8a2be2", "#ff2bd6", "#ff7417", "#ffe600",
    "#26ff55", "#1976ff", "#00ffd5", "#d42bff", "#ff1744"
];

let currentColor = colors[0];
let currentRGB = "0,234,255";

function hexToRGB(hex){
    hex = hex.replace("#","");
    const r = parseInt(hex.substring(0,2), 16);
    const g = parseInt(hex.substring(2,4), 16);
    const b = parseInt(hex.substring(4,6), 16);
    return `${r},${g},${b}`;
}

function changeColor(){
    const index = (level - 1) % colors.length;
    currentColor = colors[index];
    currentRGB = hexToRGB(currentColor);
    document.documentElement.style.setProperty("--color", currentColor);
    flash = 1;
}

let score = 0;
let level = 1;
let combo = 0;
let shieldPower = 100;
let running = true;
let flash = 0;
let mapOffset = 0;
let blockTimer = 0;
let lastTime = performance.now();

const player = {
    x: W/2,
    y: H-140,
    targetX: W/2,
    targetY: H-140,
    vx: 0,
    vy: 0,
    radius: 27
};

/* CONTROLES TÁCTILES */
let touchActive = false;
let touchStartX = 0;
let touchStartY = 0;
let playerStartX = 0;
let playerStartY = 0;

window.addEventListener("mousemove", e => {
    initAudio();
    if (!touchActive && !isPaused) {
        player.targetX = e.clientX;
        player.targetY = e.clientY;
    }
});

window.addEventListener("touchstart", e => {
    if(e.target.closest('#settingsPanel') || e.target.closest('#settingsBtn')) return;
    initAudio();
    if (e.touches.length > 0 && !isPaused) {
        touchActive = true;
        touchStartX = e.touches[0].clientX;
        touchStartY = e.touches[0].clientY;
        playerStartX = player.x;
        playerStartY = player.y;
    }
}, { passive: false });

window.addEventListener("touchmove", e => {
    if (e.touches.length > 0 && touchActive && !isPaused) {
        e.preventDefault();
        const deltaX = e.touches[0].clientX - touchStartX;
        const deltaY = e.touches[0].clientY - touchStartY;

        player.targetX = playerStartX + deltaX;
        player.targetY = playerStartY + deltaY;
    }
}, { passive: false });

window.addEventListener("touchend", () => {
    touchActive = false;
});

let blocks = [];

function createBlock(){
    const size = 30 + Math.random()*25;
    const rand = Math.random();
    const special = rand < 0.12;
    const multiColor = !special && rand < 0.35;

    blocks.push({
        x: Math.random() * (W-size),
        y: -size,
        size: size,
        speed: 2.5 + Math.random()*2 + level*.3,
        rotation: Math.random() * Math.PI,
        rotationSpeed: -.04 + Math.random()*.08,
        special: special,
        multiColor: multiColor,
        hue: Math.random() * 360
    });
}

let particles = [];

function explosion(x, y, color){
    for(let i=0; i<12; i++){
        const angle = Math.random() * Math.PI*2;
        const speed = 1 + Math.random()*5;
        particles.push({
            x: x, y: y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            life: 1,
            color: color
        });
    }
}

function updatePlayer(){
    const dx = player.targetX - player.x;
    const dy = player.targetY - player.y;

    player.vx += dx * 0.08;
    player.vy += dy * 0.08;

    player.vx *= 0.75;
    player.vy *= 0.75;

    player.x += player.vx;
    player.y += player.vy;

    player.x = Math.max(30, Math.min(W - 30, player.x));
    player.y = Math.max(30, Math.min(H - 30, player.y));
}

function drawMap(){
    const gradient = ctx.createRadialGradient(
        W/2, H*.55, 50, W/2, H*.55, Math.max(W,H)*.75
    );

    gradient.addColorStop(0, `rgba(${currentRGB},.25)`);
    gradient.addColorStop(.45, `rgba(${currentRGB},.09)`);
    gradient.addColorStop(1, "#02030b");

    ctx.fillStyle = gradient;
    ctx.fillRect(0, 0, W, H);

    mapOffset += .3;
    const gridSize = 70;

    for(let x=-gridSize; x<W+gridSize; x+=gridSize){
        const wave = Math.sin((x+mapOffset)/100)*10;
        ctx.strokeStyle = `rgba(${currentRGB},.08)`;
        ctx.lineWidth = 1;
        ctx.beginPath();
        ctx.moveTo(x+wave, 0);
        ctx.lineTo(x-wave, H);
        ctx.stroke();
    }

    for(let y=-gridSize; y<H+gridSize; y+=gridSize){
        ctx.strokeStyle = `rgba(${currentRGB},.06)`;
        ctx.beginPath();
        ctx.moveTo(0, y);
        ctx.lineTo(W, y);
        ctx.stroke();
    }

    if(flash>0){
        ctx.fillStyle = `rgba(255,255,255,${flash*.35})`;
        ctx.fillRect(0, 0, W, H);
        flash *= .88;
    }
}

function drawPlayer(){
    const x = player.x;
    const y = player.y;
    const tilt = player.vx * .015;

    ctx.save();
    ctx.translate(x, y);
    ctx.rotate(tilt);

    const aura = ctx.createRadialGradient(0, 0, 5, 0, 0, 75);
    aura.addColorStop(0, `rgba(${currentRGB},.3)`);
    aura.addColorStop(1, "rgba(0,0,0,0)");
    ctx.fillStyle = aura;
    ctx.beginPath();
    ctx.arc(0, 0, 75, 0, Math.PI*2);
    ctx.fill();

    ctx.strokeStyle = currentColor;
    ctx.lineWidth = 3;
    ctx.shadowBlur = 25;
    ctx.shadowColor = currentColor;
    ctx.beginPath();
    ctx.arc(0, 0, 62, 0, Math.PI*2);
    ctx.stroke();

    const head = ctx.createRadialGradient(-7, -10, 3, 0, 0, 25);
    head.addColorStop(0, "#ffffff");
    head.addColorStop(.45, currentColor);
    head.addColorStop(1, "#35106b");
    ctx.fillStyle = head;
    ctx.shadowBlur = 20;
    ctx.shadowColor = currentColor;
    ctx.beginPath();
    ctx.arc(0, -30, 22, 0, Math.PI*2);
    ctx.fill();

    ctx.shadowBlur = 0;
    ctx.fillStyle = "#06101a";
    ctx.beginPath();
    ctx.ellipse(-7, -31, 3, 6, 0, 0, Math.PI*2);
    ctx.fill();
    ctx.beginPath();
    ctx.ellipse(7, -31, 3, 6, 0, 0, Math.PI*2);
    ctx.fill();

    const body = ctx.createLinearGradient(-20, -5, 20, 35);
    body.addColorStop(0, currentColor);
    body.addColorStop(1, "#6b00ff");
    ctx.fillStyle = body;
    ctx.beginPath();
    ctx.roundRect(-19, -8, 38, 48, 12);
    ctx.fill();

    ctx.fillStyle = "#ffffff";
    ctx.shadowBlur = 25;
    ctx.shadowColor = currentColor;
    ctx.beginPath();
    ctx.arc(0, 14, 7, 0, Math.PI*2);
    ctx.fill();

    ctx.strokeStyle = currentColor;
    ctx.lineWidth = 10;
    ctx.lineCap = "round";
    ctx.beginPath();
    ctx.moveTo(-10, 36);
    ctx.lineTo(-13, 58);
    ctx.stroke();
    ctx.beginPath();
    ctx.moveTo(10, 36);
    ctx.lineTo(13, 58);
    ctx.stroke();

    ctx.restore();
}

function drawBlock(block){
    ctx.save();
    const cx = block.x + block.size/2;
    const cy = block.y + block.size/2;

    ctx.translate(cx, cy);
    ctx.rotate(block.rotation);

    if(block.special){
        const pulse = Math.sin(performance.now()/100);
        ctx.shadowBlur = 45 + pulse*15;
        ctx.shadowColor = "#ffffff";

        const aura = ctx.createRadialGradient(0, 0, 2, 0, 0, block.size*2);
        aura.addColorStop(0, "#ffffff");
        aura.addColorStop(.25, currentColor);
        aura.addColorStop(.7, `rgba(${currentRGB},.25)`);
        aura.addColorStop(1, "rgba(0,0,0,0)");

        ctx.fillStyle = aura;
        ctx.beginPath();
        ctx.arc(0, 0, block.size*2, 0, Math.PI*2);
        ctx.fill();

        const specialGradient = ctx.createLinearGradient(-block.size/2, -block.size/2, block.size/2, block.size/2);
        specialGradient.addColorStop(0, "#ffffff");
        specialGradient.addColorStop(.35, currentColor);
        specialGradient.addColorStop(1, "#ffffff");

        ctx.fillStyle = specialGradient;
        ctx.fillRect(-block.size/2, -block.size/2, block.size, block.size);
        ctx.strokeStyle = "#ffffff";
        ctx.lineWidth = 3;
        ctx.strokeRect(-block.size/2, -block.size/2, block.size, block.size);
    } 
    else if(block.multiColor) {
        block.hue = (block.hue + 2.5) % 360;
        const dynamicColor = `hsl(${block.hue}, 100%, 60%)`;
        const altColor = `hsl(${(block.hue + 120) % 360}, 100%, 60%)`;

        ctx.shadowBlur = 25;
        ctx.shadowColor = dynamicColor;

        const multiGrad = ctx.createLinearGradient(-block.size/2, -block.size/2, block.size/2, block.size/2);
        multiGrad.addColorStop(0, dynamicColor);
        multiGrad.addColorStop(0.5, altColor);
        multiGrad.addColorStop(1, `hsl(${(block.hue + 240) % 360}, 100%, 60%)`);

        ctx.fillStyle = multiGrad;
        ctx.fillRect(-block.size/2, -block.size/2, block.size, block.size);

        ctx.strokeStyle = "#ffffff";
        ctx.lineWidth = 2;
        ctx.strokeRect(-block.size/2, -block.size/2, block.size, block.size);

        const pulse = Math.sin(performance.now()/150) * 2;
        ctx.save();
        ctx.rotate(-block.rotation * 2);

        ctx.fillStyle = "rgba(255, 255, 255, 0.9)";
        ctx.shadowBlur = 15;
        ctx.shadowColor = "#ffffff";
        ctx.beginPath();
        ctx.arc(0, 0, Math.max(3, block.size/5 + pulse), 0, Math.PI*2);
        ctx.fill();

        ctx.strokeStyle = dynamicColor;
        ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.strokeRect(-block.size/4, -block.size/4, block.size/2, block.size/2);
        ctx.restore();
    } 
    else {
        const gradient = ctx.createLinearGradient(-block.size/2, -block.size/2, block.size/2, block.size/2);
        gradient.addColorStop(0, currentColor);
        gradient.addColorStop(1, "#ffffff");

        ctx.fillStyle = gradient;
        ctx.shadowBlur = 20;
        ctx.shadowColor = currentColor;
        ctx.fillRect(-block.size/2, -block.size/2, block.size, block.size);
        ctx.strokeStyle = "rgba(255,255,255,.8)";
        ctx.lineWidth = 2;
        ctx.strokeRect(-block.size/2, -block.size/2, block.size, block.size);
    }

    ctx.restore();
}

function danger(block){
    const distanceLimit = 38;
    const cx = block.x + block.size/2;
    const cy = block.y + block.size/2;
    const dx = cx - player.x;
    const dy = cy - player.y;
    return Math.sqrt(dx*dx + dy*dy) <= distanceLimit;
}

function updateBlocks(){
    for(let i = blocks.length-1; i>=0; i--){
        const b = blocks[i];
        b.y += b.speed;
        b.rotation += b.rotationSpeed;

        if(danger(b)){
            if(b.special){
                score += 5;
                combo += 3;
                shieldPower = Math.min(100, shieldPower + 30);
                explosion(b.x+b.size/2, b.y+b.size/2, "#ffffff");
                explosion(b.x+b.size/2, b.y+b.size/2, currentColor);
                playSpecialSound();
            } else if(b.multiColor) {
                score += 3;
                combo += 2;
                explosion(b.x+b.size/2, b.y+b.size/2, `hsl(${b.hue}, 100%, 60%)`);
                explosion(b.x+b.size/2, b.y+b.size/2, "#ffffff");
                playSpecialSound();
            } else {
                score += 1;
                combo += 1;
                explosion(b.x+b.size/2, b.y+b.size/2, currentColor);
                playScoreSound();
            }

            blocks.splice(i, 1);
            const newLevel = Math.floor(score/10) + 1;

            if(newLevel > level){
                level = newLevel;
                document.getElementById("level").textContent = level;
                document.getElementById("levelNumber").textContent = level;
                changeColor();
                levelMessage();
                playLevelUpSound();
                startMusic();
            }
            continue;
        }

        if(b.y > H+100){
            blocks.splice(i, 1);
        }
    }
}

function updateParticles(){
    for(let i = particles.length-1; i>=0; i--){
        const p = particles[i];
        p.x += p.vx;
        p.y += p.vy;
        p.vx *= .97;
        p.vy *= .97;
        p.life -= .025;
        if(p.life <= 0){
            particles.splice(i, 1);
        }
    }
}

function drawParticles(){
    for(const p of particles){
        ctx.globalAlpha = p.life;
        ctx.fillStyle = p.color;
        ctx.shadowBlur = 10;
        ctx.shadowColor = p.color;
        ctx.beginPath();
        ctx.arc(p.x, p.y, 3, 0, Math.PI*2);
        ctx.fill();
    }
    ctx.globalAlpha = 1;
}

const levelMessageEl = document.getElementById("levelMessage");

function levelMessage(){
    document.getElementById("levelNumber").textContent = level;
    levelMessageEl.classList.remove("showLevel");
    void levelMessageEl.offsetWidth;
    levelMessageEl.classList.add("showLevel");
}

function updateHUD(){
    document.getElementById("score").textContent = score;
    document.getElementById("level").textContent = level;
    document.getElementById("combo").textContent = combo;
    document.getElementById("shieldBar").style.width = shieldPower + "%";
}

document.getElementById("restart").addEventListener("click", () => {
    initAudio();
    restart();
});

function restart(){
    isPaused = false;
    pauseGameBtn.textContent = "⏸️ Pausar Juego";
    pauseOverlay.style.display = "none";
    settingsPanel.classList.remove("open");
    
    blocks = [];
    particles = [];
    score = 0;
    level = 1;
    combo = 0;
    shieldPower = 100;
    player.x = W/2;
    player.y = H-140;
    player.targetX = W/2;
    player.targetY = H-140;
    changeColor();
    document.getElementById("gameOver").style.display = "none";
    running = true;
    startMusic();
    lastTime = performance.now();
    requestAnimationFrame(gameLoop);
}

function gameLoop(time){
    if(!running || isPaused) return;

    const dt = Math.min(32, time-lastTime);
    lastTime = time;

    blockTimer += dt;
    const interval = Math.max(180, 700 - level*32);

    if(blockTimer > interval){
        createBlock();
        blockTimer = 0;
    }

    updatePlayer();
    updateBlocks();
    updateParticles();

    ctx.clearRect(0, 0, W, H);
    drawMap();

    for(const block of blocks){
        drawBlock(block);
    }

    drawParticles();
    drawPlayer();
    updateHUD();

    requestAnimationFrame(gameLoop);
}

changeColor();
requestAnimationFrame(gameLoop);
</script>

</body>
</html>

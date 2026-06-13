<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
<title>VOID DRIFT — 虚空漂移</title>
<style>
  :root {
    --bg: #080810;
    --ink: #E8E4F0;
    --accent: #C8FF00;
    --ghost: rgba(200,255,0,0.08);
    --mid: #1a1a2e;
    --danger: #FF3A5C;
  }
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    background: var(--bg);
    color: var(--ink);
    font-family: 'Space Mono', monospace;
    height: 100dvh;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    overflow: hidden;
    touch-action: none;
  }
  #stars { position: fixed; inset: 0; pointer-events: none; z-index: 0; }
  .screen {
    position: relative; z-index: 2;
    display: flex; flex-direction: column;
    align-items: center; justify-content: center;
    width: 100%; height: 100%; padding: 24px;
  }
  .hidden { display: none !important; }
  #start-screen .eyebrow {
    font-size: 10px; letter-spacing: 0.3em;
    color: var(--accent); text-transform: uppercase;
    margin-bottom: 20px; opacity: 0.7;
  }
  #start-screen h1 {
    font-size: clamp(2.8rem, 10vw, 5rem);
    font-weight: 700; line-height: 0.9;
    text-align: center; letter-spacing: -0.02em; margin-bottom: 8px;
  }
  #start-screen h1 span {
    display: block; color: var(--accent);
    font-size: 0.45em; letter-spacing: 0.25em; font-weight: 400;
  }
  #start-screen .tagline {
    font-size: 13px; color: var(--ink); opacity: 0.45;
    margin-top: 16px; margin-bottom: 48px; letter-spacing: 0.1em;
  }
  .btn-start {
    background: var(--accent); color: var(--bg);
    border: none; padding: 16px 48px;
    font-family: monospace; font-size: 14px; font-weight: 700;
    letter-spacing: 0.15em; cursor: pointer;
    clip-path: polygon(8px 0%, 100% 0%, calc(100% - 8px) 100%, 0% 100%);
    transition: transform 0.1s, opacity 0.1s;
  }
  .btn-start:active { transform: scale(0.97); opacity: 0.85; }
  .hint {
    margin-top: 32px; font-size: 10px; color: var(--ink);
    opacity: 0.25; letter-spacing: 0.15em; text-align: center; line-height: 1.8;
  }
  #game-screen { padding: 0; position: relative; }
  #hud {
    position: absolute; top: 0; left: 0; right: 0; z-index: 10;
    display: flex; justify-content: space-between; align-items: flex-start;
    padding: 18px 20px 0; pointer-events: none;
  }
  .hud-label { font-size: 9px; letter-spacing: 0.25em; color: var(--accent); opacity: 0.6; margin-bottom: 2px; }
  .hud-value { font-size: 22px; font-weight: 700; color: var(--ink); line-height: 1; }
  #lives-display { display: flex; gap: 6px; margin-top: 4px; }
  .life-dot {
    width: 8px; height: 8px; background: var(--accent);
    clip-path: polygon(50% 0%, 100% 38%, 82% 100%, 18% 100%, 0% 38%);
    transition: opacity 0.3s;
  }
  .life-dot.lost { opacity: 0.15; }
  canvas { display: block; touch-action: none; }
  #joystick-zone {
    position: absolute; bottom: 0; left: 0; right: 0; height: 200px;
    display: flex; align-items: center; justify-content: center; z-index: 5;
  }
  #joystick-base {
    width: 110px; height: 110px; border-radius: 50%;
    border: 1.5px solid rgba(200,255,0,0.2);
    background: rgba(200,255,0,0.04);
    position: relative; display: flex; align-items: center; justify-content: center;
  }
  #joystick-knob {
    width: 42px; height: 42px; border-radius: 50%;
    background: radial-gradient(circle at 35% 35%, rgba(200,255,0,0.6), rgba(200,255,0,0.15));
    border: 1px solid rgba(200,255,0,0.5);
    position: absolute;
    box-shadow: 0 0 20px rgba(200,255,0,0.2);
  }
  #gameover-screen { gap: 0; }
  #gameover-screen .go-label {
    font-size: 10px; letter-spacing: 0.3em;
    color: var(--danger); margin-bottom: 16px; opacity: 0.8;
  }
  #gameover-screen h2 {
    font-size: clamp(2rem, 8vw, 3.5rem); font-weight: 700;
    line-height: 1; margin-bottom: 40px; text-align: center;
  }
  .score-row {
    display: flex; justify-content: space-between; width: 240px;
    border-bottom: 1px solid rgba(255,255,255,0.08);
    padding: 12px 0; font-size: 13px;
  }
  .score-row .label { opacity: 0.4; letter-spacing: 0.1em; }
  .score-row .val { color: var(--accent); font-weight: 700; }
  .btn-restart {
    margin-top: 40px; background: transparent; color: var(--accent);
    border: 1px solid rgba(200,255,0,0.4); padding: 14px 40px;
    font-family: monospace; font-size: 12px; font-weight: 700;
    letter-spacing: 0.2em; cursor: pointer; transition: background 0.2s;
  }
  .btn-restart:active { background: var(--ghost); }
  body::after {
    content: ''; position: fixed; inset: 0;
    background: repeating-linear-gradient(0deg, transparent, transparent 2px, rgba(0,0,0,0.03) 2px, rgba(0,0,0,0.03) 4px);
    pointer-events: none; z-index: 100;
  }
</style>
</head>
<body>
<canvas id="stars"></canvas>
<div id="start-screen" class="screen">
  <p class="eyebrow">· VOID DRIFT ·</p>
  <h1>虚空<span>VOID DRIFT</span>漂移</h1>
  <p class="tagline">在虚空中闪避，越走越快，永不停歇</p>
  <button class="btn-start" onclick="startGame()">开始漂移</button>
  <p class="hint">触摸摇杆操控飞船<br>躲避红色晶体碎片</p>
</div>
<div id="game-screen" class="screen hidden">
  <div id="hud">
    <div class="hud-block">
      <div class="hud-label">SCORE</div>
      <div class="hud-value" id="score-display">0</div>
    </div>
    <div class="hud-block" style="text-align:right">
      <div class="hud-label">LIVES</div>
      <div id="lives-display">
        <div class="life-dot" id="l1"></div>
        <div class="life-dot" id="l2"></div>
        <div class="life-dot" id="l3"></div>
      </div>
    </div>
  </div>
  <canvas id="gameCanvas"></canvas>
  <div id="joystick-zone">
    <div id="joystick-base">
      <div id="joystick-knob"></div>
    </div>
  </div>
</div>
<div id="gameover-screen" class="screen hidden">
  <p class="go-label">— SIGNAL LOST —</p>
  <h2>漂移<br>终止</h2>
  <div class="score-row"><span class="label">SCORE</span><span class="val" id="final-score">0</span></div>
  <div class="score-row"><span class="label">BEST</span><span class="val" id="best-score">0</span></div>
  <div class="score-row"><span class="label">WAVE</span><span class="val" id="final-wave">1</span></div>
  <button class="btn-restart" onclick="startGame()">再次漂移</button>
</div>
<script>
const starsCanvas = document.getElementById('stars');
const sCtx = starsCanvas.getContext('2d');
let starsArr = [];
function resizeStars() {
  starsCanvas.width = window.innerWidth;
  starsCanvas.height = window.innerHeight;
  starsArr = Array.from({length: 80}, () => ({
    x: Math.random() * starsCanvas.width,
    y: Math.random() * starsCanvas.height,
    r: Math.random() * 1.2 + 0.2,
    speed: Math.random() * 0.4 + 0.1,
    opacity: Math.random() * 0.5 + 0.1
  }));
}
function drawStars() {
  sCtx.clearRect(0, 0, starsCanvas.width, starsCanvas.height);
  starsArr.forEach(s => {
    s.y += s.speed;
    if (s.y > starsCanvas.height) { s.y = 0; s.x = Math.random() * starsCanvas.width; }
    sCtx.beginPath();
    sCtx.arc(s.x, s.y, s.r, 0, Math.PI * 2);
    sCtx.fillStyle = `rgba(200,255,0,${s.opacity})`;
    sCtx.fill();
  });
  requestAnimationFrame(drawStars);
}
resizeStars(); drawStars();
window.addEventListener('resize', resizeStars);

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let gameState = {};
let animId = null;
let bestScore = 0;

function setupCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
}

function startGame() {
  setupCanvas();
  document.getElementById('start-screen').classList.add('hidden');
  document.getElementById('gameover-screen').classList.add('hidden');
  document.getElementById('game-screen').classList.remove('hidden');
  if (animId) cancelAnimationFrame(animId);
  const cx = canvas.width / 2;
  const cy = canvas.height * 0.35;
  gameState = {
    ship: { x: cx, y: cy, vx: 0, vy: 0, radius: 14, trail: [] },
    obstacles: [], particles: [],
    score: 0, lives: 3, wave: 1,
    speed: 2.8, spawnRate: 90, frame: 0, invincible: 0, running: true
  };
  updateLivesUI();
  document.getElementById('score-display').textContent = '0';
  loop();
}

function loop() {
  if (!gameState.running) return;
  update(); draw();
  animId = requestAnimationFrame(loop);
}

function update() {
  const gs = gameState;
  gs.frame++;
  gs.score += 1;
  document.getElementById('score-display').textContent = gs.score;
  if (gs.frame % 400 === 0) {
    gs.wave++;
    gs.speed = Math.min(2.8 + gs.wave * 0.4, 7);
    gs.spawnRate = Math.max(90 - gs.wave * 8, 30);
  }
  const jDx = joystickDelta.x;
  const jDy = joystickDelta.y;
  const deadzone = 0.05, accel = 0.45, friction = 0.82, maxSpeed = 7;
  if (Math.abs(jDx) > deadzone) gs.ship.vx += jDx * accel; else gs.ship.vx *= friction;
  if (Math.abs(jDy) > deadzone) gs.ship.vy += jDy * accel; else gs.ship.vy *= friction;
  gs.ship.vx = Math.max(-maxSpeed, Math.min(maxSpeed, gs.ship.vx));
  gs.ship.vy = Math.max(-maxSpeed, Math.min(maxSpeed, gs.ship.vy));
  gs.ship.x = Math.max(gs.ship.radius, Math.min(canvas.width - gs.ship.radius, gs.ship.x + gs.ship.vx));
  gs.ship.y = Math.max(gs.ship.radius + 60, Math.min(canvas.height - 220, gs.ship.y + gs.ship.vy));
  gs.ship.trail.unshift({ x: gs.ship.x, y: gs.ship.y });
  if (gs.ship.trail.length > 18) gs.ship.trail.pop();
  if (gs.frame % gs.spawnRate === 0) spawnObstacle();
  gs.obstacles.forEach(o => { o.y += o.vy; o.x += o.vx; o.rot += o.rotSpeed; });
  gs.obstacles = gs.obstacles.filter(o => o.y < canvas.height + 60 && o.x > -60 && o.x < canvas.width + 60);
  if (gs.invincible > 0) { gs.invincible--; }
  else {
    gs.obstacles.forEach(o => {
      const dx = gs.ship.x - o.x, dy = gs.ship.y - o.y;
      if (Math.sqrt(dx*dx + dy*dy) < gs.ship.radius + o.radius * 0.7) hit(o);
    });
  }
  gs.particles.forEach(p => { p.x += p.vx; p.y += p.vy; p.life--; p.vy += 0.05; });
  gs.particles = gs.particles.filter(p => p.life > 0);
  if (gs.frame % 2 === 0) {
    gs.particles.push({
      x: gs.ship.x + (Math.random()-0.5)*6, y: gs.ship.y + 12,
      vx: (Math.random()-0.5)*0.8, vy: gs.speed * 0.3 + Math.random()*0.5,
      life: 18 + Math.random()*10, maxLife: 28,
      color: Math.random() > 0.5 ? '#C8FF00' : '#80FF00',
      size: Math.random()*2+0.5, type: 'engine'
    });
  }
}

function spawnObstacle() {
  const gs = gameState;
  const shapes = ['crystal', 'shard', 'ring'];
  const margin = 40;
  gs.obstacles.push({
    x: margin + Math.random() * (canvas.width - margin * 2), y: -40,
    vy: gs.speed + Math.random() * 1.5, vx: (Math.random() - 0.5) * 1.2,
    radius: 16 + Math.random() * 14,
    rot: Math.random() * Math.PI * 2, rotSpeed: (Math.random() - 0.5) * 0.06,
    shape: shapes[Math.floor(Math.random() * shapes.length)],
    hue: Math.random() > 0.3 ? 0 : 280
  });
}

function hit(obstacle) {
  const gs = gameState;
  for (let i = 0; i < 22; i++) {
    const angle = (Math.PI * 2 / 22) * i + Math.random() * 0.3;
    const spd = 1.5 + Math.random() * 3;
    gs.particles.push({
      x: obstacle.x, y: obstacle.y,
      vx: Math.cos(angle) * spd, vy: Math.sin(angle) * spd,
      life: 25 + Math.random()*15, maxLife: 40,
      color: '#FF3A5C', size: Math.random()*3+1, type: 'explosion'
    });
  }
  gs.obstacles = gs.obstacles.filter(o => o !== obstacle);
  gs.lives--; gs.invincible = 90;
  updateLivesUI();
  if (gs.lives <= 0) { gs.running = false; gameOver(); }
}

function updateLivesUI() {
  for (let i = 1; i <= 3; i++) {
    document.getElementById(`l${i}`).classList.toggle('lost', i > gameState.lives);
  }
}

function draw() {
  const gs = gameState;
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  gs.particles.forEach(p => {
    ctx.save(); ctx.globalAlpha = p.life / p.maxLife;
    ctx.fillStyle = p.color;
    ctx.beginPath(); ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
    ctx.fill(); ctx.restore();
  });
  gs.obstacles.forEach(o => drawObstacle(o));
  gs.ship.trail.forEach((t, i) => {
    ctx.save();
    ctx.globalAlpha = (1 - i / gs.ship.trail.length) * 0.25;
    ctx.fillStyle = '#C8FF00';
    ctx.beginPath(); ctx.arc(t.x, t.y, (1 - i / gs.ship.trail.length) * 6, 0, Math.PI * 2);
    ctx.fill(); ctx.restore();
  });
  const blink = gs.invincible > 0 && Math.floor(gs.invincible / 6) % 2 === 0;
  if (!blink) drawShip(gs.ship);
}

function drawShip(ship) {
  ctx.save(); ctx.translate(ship.x, ship.y);
  const grd = ctx.createRadialGradient(0, 0, 4, 0, 0, 22);
  grd.addColorStop(0, 'rgba(200,255,0,0.25)'); grd.addColorStop(1, 'rgba(200,255,0,0)');
  ctx.fillStyle = grd; ctx.beginPath(); ctx.arc(0, 0, 22, 0, Math.PI * 2); ctx.fill();
  ctx.strokeStyle = '#C8FF00'; ctx.lineWidth = 1.8; ctx.lineJoin = 'round';
  ctx.fillStyle = 'rgba(200,255,0,0.1)';
  ctx.beginPath();
  ctx.moveTo(0, -14); ctx.lineTo(10, 6); ctx.lineTo(5, 3);
  ctx.lineTo(0, 8); ctx.lineTo(-5, 3); ctx.lineTo(-10, 6);
  ctx.closePath(); ctx.fill(); ctx.stroke();
  ctx.fillStyle = 'rgba(200,255,0,0.5)';
  ctx.beginPath(); ctx.ellipse(0, -3, 2.5, 4, 0, 0, Math.PI * 2); ctx.fill();
  ctx.restore();
}

function drawObstacle(o) {
  ctx.save(); ctx.translate(o.x, o.y); ctx.rotate(o.rot);
  const baseColor = o.hue === 0 ? '#FF3A5C' : '#CC44FF';
  const glowColor = o.hue === 0 ? 'rgba(255,58,92,0.3)' : 'rgba(200,68,255,0.3)';
  const grd = ctx.createRadialGradient(0, 0, 2, 0, 0, o.radius * 1.4);
  grd.addColorStop(0, glowColor); grd.addColorStop(1, 'transparent');
  ctx.fillStyle = grd; ctx.beginPath(); ctx.arc(0, 0, o.radius * 1.4, 0, Math.PI * 2); ctx.fill();
  ctx.strokeStyle = baseColor; ctx.lineWidth = 1.5;
  ctx.fillStyle = o.hue === 0 ? 'rgba(255,58,92,0.12)' : 'rgba(200,68,255,0.12)';
  if (o.shape === 'crystal') {
    ctx.beginPath();
    ctx.moveTo(0, -o.radius); ctx.lineTo(o.radius*0.6, 0);
    ctx.lineTo(0, o.radius); ctx.lineTo(-o.radius*0.6, 0);
    ctx.closePath(); ctx.fill(); ctx.stroke();
  } else if (o.shape === 'shard') {
    ctx.beginPath();
    for (let i = 0; i < 5; i++) {
      const angle = (Math.PI*2/5)*i - Math.PI/2;
      const r = o.radius * (0.7 + Math.sin(i*1.3)*0.3);
      if (i===0) ctx.moveTo(Math.cos(angle)*r, Math.sin(angle)*r);
      else ctx.lineTo(Math.cos(angle)*r, Math.sin(angle)*r);
    }
    ctx.closePath(); ctx.fill(); ctx.stroke();
  } else {
    ctx.beginPath(); ctx.arc(0, 0, o.radius, 0, Math.PI*2); ctx.stroke();
    ctx.beginPath(); ctx.arc(0, 0, o.radius*0.5, 0, Math.PI*2); ctx.fill(); ctx.stroke();
    for (let i = 0; i < 4; i++) {
      const a = (Math.PI/2)*i;
      ctx.beginPath();
      ctx.moveTo(Math.cos(a)*o.radius*0.5, Math.sin(a)*o.radius*0.5);
      ctx.lineTo(Math.cos(a)*o.radius, Math.sin(a)*o.radius);
      ctx.strokeStyle = baseColor+'88'; ctx.stroke();
    }
  }
  ctx.restore();
}

function gameOver() {
  cancelAnimationFrame(animId);
  if (gameState.score > bestScore) bestScore = gameState.score;
  document.getElementById('final-score').textContent = gameState.score;
  document.getElementById('best-score').textContent = bestScore;
  document.getElementById('final-wave').textContent = gameState.wave;
  setTimeout(() => {
    document.getElementById('game-screen').classList.add('hidden');
    document.getElementById('gameover-screen').classList.remove('hidden');
  }, 600);
}

const joystickBase = document.getElementById('joystick-base');
const joystickKnob = document.getElementById('joystick-knob');
let joystickDelta = { x: 0, y: 0 };
let joystickActive = false;
let joystickOrigin = { x: 0, y: 0 };
const joystickRadius = 34;

function getJoystickCenter() {
  const rect = joystickBase.getBoundingClientRect();
  return { x: rect.left + rect.width/2, y: rect.top + rect.height/2 };
}

joystickBase.addEventListener('touchstart', e => {
  e.preventDefault(); joystickActive = true;
  joystickOrigin = getJoystickCenter();
}, { passive: false });

document.addEventListener('touchmove', e => {
  if (!joystickActive) return; e.preventDefault();
  const touch = e.touches[0];
  let dx = touch.clientX - joystickOrigin.x;
  let dy = touch.clientY - joystickOrigin.y;
  const dist = Math.sqrt(dx*dx + dy*dy);
  if (dist > joystickRadius) { dx = (dx/dist)*joystickRadius; dy = (dy/dist)*joystickRadius; }
  joystickDelta = { x: dx/joystickRadius, y: dy/joystickRadius };
  joystickKnob.style.transform = `translate(${dx}px, ${dy}px)`;
}, { passive: false });

document.addEventListener('touchend', () => {
  joystickActive = false; joystickDelta = { x: 0, y: 0 };
  joystickKnob.style.transform = 'translate(0,0)';
});

const keys = {};
document.addEventListener('keydown', e => keys[e.key] = true);
document.addEventListener('keyup', e => keys[e.key] = false);
function updateKeyboardJoystick() {
  if (!gameState.running) return;
  let x = 0, y = 0;
  if (keys['ArrowLeft'] || keys['a'] || keys['A']) x = -1;
  if (keys['ArrowRight'] || keys['d'] || keys['D']) x = 1;
  if (keys['ArrowUp'] || keys['w'] || keys['W']) y = -1;
  if (keys['ArrowDown'] || keys['s'] || keys['S']) y = 1;
  if (!joystickActive) joystickDelta = { x, y };
  requestAnimationFrame(updateKeyboardJoystick);
}
updateKeyboardJoystick();
window.addEventListener('resize', () => { if (gameState.running) setupCanvas(); });
</script>
</body>
</html>

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const overlay = document.getElementById('overlay');
const startBtn = document.getElementById('startBtn');
const scoreEl = document.getElementById('score');
const livesEl = document.getElementById('lives');
const targetEl = document.getElementById('target');

const targetScore = 10;
const keys = {};

let player;
let enemies;
let pickups;
let score;
let lives;
let gameStarted;
let gameOver;
let won;
let spawnTimer;
let pickupTimer;

function resetGame() {
  player = {
    x: canvas.width / 2,
    y: canvas.height - 38,
    radius: 18,
    speed: 6.4,
  };

  enemies = [];
  pickups = [];
  score = 0;
  lives = 3;
  gameStarted = false;
  gameOver = false;
  won = false;
  spawnTimer = 0.8;
  pickupTimer = 1.3;

  scoreEl.textContent = score;
  livesEl.textContent = lives;
  targetEl.textContent = targetScore;
}

function setOverlay(title, message, buttonText) {
  overlay.classList.add('visible');
  overlay.innerHTML = `
    <div class="panel">
      <h1>${title}</h1>
      <p>${message}</p>
      <button id="startBtn">${buttonText}</button>
    </div>
  `;

  document.getElementById('startBtn').addEventListener('click', () => {
    startGame();
  });
}

function startGame() {
  resetGame();
  gameStarted = true;
  overlay.classList.remove('visible');
}

function spawnEnemy() {
  const radius = 18 + Math.random() * 18;
  const side = Math.random() < 0.5 ? -1 : 1;
  const x = Math.random() * (canvas.width - radius * 2) + radius;
  const y = -radius - 10;
  const drift = (Math.random() * 1.5 + 0.6) * side;
  const speed = 2 + Math.random() * 2.4;

  enemies.push({ x, y, radius, speed, drift, hue: 340 + Math.random() * 50 });
}

function spawnPickup() {
  const radius = 12;
  const x = 30 + Math.random() * (canvas.width - 60);
  const y = -radius - 10;
  pickups.push({ x, y, radius, speed: 2 + Math.random() * 1.8, pulse: Math.random() * Math.PI * 2 });
}

function handleInput() {
  if (!gameStarted || gameOver || won) return;

  if (keys['ArrowLeft'] || keys['a'] || keys['A']) player.x -= player.speed;
  if (keys['ArrowRight'] || keys['d'] || keys['D']) player.x += player.speed;
  if (keys['ArrowUp'] || keys['w'] || keys['W']) player.y -= player.speed;
  if (keys['ArrowDown'] || keys['s'] || keys['S']) player.y += player.speed;

  player.x = Math.max(player.radius, Math.min(canvas.width - player.radius, player.x));
  player.y = Math.max(player.radius, Math.min(canvas.height - player.radius, player.y));
}

function update(delta) {
  if (!gameStarted || gameOver || won) return;

  spawnTimer -= delta;
  pickupTimer -= delta;

  if (spawnTimer <= 0) {
    spawnEnemy();
    spawnTimer = 0.7 + Math.random() * 0.8;
  }

  if (pickupTimer <= 0) {
    spawnPickup();
    pickupTimer = 1.4 + Math.random() * 1.6;
  }

  for (let i = enemies.length - 1; i >= 0; i--) {
    const enemy = enemies[i];
    enemy.y += enemy.speed;
    enemy.x += enemy.drift;

    if (enemy.x < enemy.radius || enemy.x > canvas.width - enemy.radius) {
      enemy.drift *= -1;
    }

    if (enemy.y - enemy.radius > canvas.height) {
      enemies.splice(i, 1);
      lives -= 1;
      livesEl.textContent = lives;
      if (lives <= 0) {
        gameOver = true;
        setOverlay('Game Over', 'The resonance field collapsed. Press the button to try again.', 'Retry');
      }
      continue;
    }

    const dx = enemy.x - player.x;
    const dy = enemy.y - player.y;
    const distance = Math.hypot(dx, dy);
    if (distance < enemy.radius + player.radius) {
      enemies.splice(i, 1);
      lives -= 1;
      livesEl.textContent = lives;
      if (lives <= 0) {
        gameOver = true;
        setOverlay('Game Over', 'The pulse drones overwhelmed you. Press the button to try again.', 'Retry');
      }
    }
  }

  for (let i = pickups.length - 1; i >= 0; i--) {
    const pickup = pickups[i];
    pickup.y += pickup.speed;
    pickup.pulse += 0.08;

    if (pickup.y - pickup.radius > canvas.height) {
      pickups.splice(i, 1);
      continue;
    }

    const dx = pickup.x - player.x;
    const dy = pickup.y - player.y;
    const distance = Math.hypot(dx, dy);
    if (distance < pickup.radius + player.radius) {
      pickups.splice(i, 1);
      score += 1;
      scoreEl.textContent = score;

      if (score >= targetScore) {
        won = true;
        setOverlay('You Win!', 'The resonance network is stable again. Press the button for another run.', 'Play Again');
      }
    }
  }
}

function drawBackground() {
  const gradient = ctx.createRadialGradient(
    canvas.width / 2,
    canvas.height / 2,
    10,
    canvas.width / 2,
    canvas.height / 2,
    canvas.width
  );
  gradient.addColorStop(0, '#1d2b4d');
  gradient.addColorStop(1, '#070b12');
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.strokeStyle = 'rgba(110, 231, 255, 0.08)';
  ctx.lineWidth = 1;
  for (let x = 0; x < canvas.width; x += 40) {
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, canvas.height);
    ctx.stroke();
  }
  for (let y = 0; y < canvas.height; y += 40) {
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(canvas.width, y);
    ctx.stroke();
  }
}

function drawPlayer() {
  ctx.save();
  ctx.translate(player.x, player.y);

  const glow = ctx.createRadialGradient(0, 0, 2, 0, 0, player.radius + 10);
  glow.addColorStop(0, '#d9ffff');
  glow.addColorStop(0.5, '#6ee7ff');
  glow.addColorStop(1, 'rgba(110, 231, 255, 0)');
  ctx.fillStyle = glow;
  ctx.beginPath();
  ctx.arc(0, 0, player.radius + 12, 0, Math.PI * 2);
  ctx.fill();

  ctx.fillStyle = '#8af7b3';
  ctx.beginPath();
  ctx.arc(0, 0, player.radius, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();
}

function drawEnemy(enemy) {
  ctx.save();
  ctx.translate(enemy.x, enemy.y);
  const pulse = 10 + Math.sin(Date.now() * 0.01 + enemy.x) * 4;

  const glow = ctx.createRadialGradient(0, 0, 2, 0, 0, enemy.radius + pulse);
  glow.addColorStop(0, `hsla(${enemy.hue}, 95%, 72%, 1)`);
  glow.addColorStop(0.5, `hsla(${enemy.hue}, 90%, 60%, 0.7)`);
  glow.addColorStop(1, `hsla(${enemy.hue}, 95%, 50%, 0)`);
  ctx.fillStyle = glow;
  ctx.beginPath();
  ctx.arc(0, 0, enemy.radius + pulse, 0, Math.PI * 2);
  ctx.fill();

  ctx.fillStyle = `hsla(${enemy.hue}, 95%, 64%, 0.95)`;
  ctx.beginPath();
  ctx.arc(0, 0, enemy.radius, 0, Math.PI * 2);
  ctx.fill();
  ctx.restore();
}

function drawPickup(pickup) {
  ctx.save();
  ctx.translate(pickup.x, pickup.y);
  const size = pickup.radius + Math.sin(pickup.pulse) * 3;

  ctx.shadowColor = '#ffd166';
  ctx.shadowBlur = 18;
  ctx.fillStyle = '#ffd166';
  ctx.beginPath();
  ctx.moveTo(0, -size);
  ctx.lineTo(size * 0.8, 0);
  ctx.lineTo(0, size);
  ctx.lineTo(-size * 0.8, 0);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}

function render() {
  drawBackground();

  if (gameStarted) {
    for (const pickup of pickups) drawPickup(pickup);
    for (const enemy of enemies) drawEnemy(enemy);
    drawPlayer();
  } else {
    drawPlayer();
  }
}

let lastFrame = 0;
function frame(timestamp) {
  const delta = Math.min((timestamp - lastFrame) / 1000 || 0.016, 0.03);
  lastFrame = timestamp;

  handleInput();
  update(delta);
  render();

  requestAnimationFrame(frame);
}

window.addEventListener('keydown', (event) => {
  keys[event.key] = true;

  if (event.code === 'Space' && !gameStarted) {
    startGame();
  }

  if (event.key.toLowerCase() === 'r' && (gameOver || won)) {
    startGame();
  }
});

window.addEventListener('keyup', (event) => {
  keys[event.key] = false;
});

resetGame();
render();
requestAnimationFrame(frame);

setOverlay('Resonenes: Echo Escape', 'Collect 10 shards, avoid the pulse drones, and keep the network alive.', 'Start Game');

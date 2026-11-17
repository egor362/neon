<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<title>NEON BLOCKS</title>

<script src="https://telegram.org/js/telegram-web-app.js"></script>

<style>
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  background: #0b1020;
  color: white;
  font-family: Arial, sans-serif;
  text-align: center;
  touch-action: manipulation;
}

header {
  color: #00eaff;
  font-size: 28px;
  font-weight: bold;
  padding: 18px 5px;
  letter-spacing: 2px;
}

.stats {
  display: flex;
  justify-content: space-around;
  padding: 10px 2px;
  font-size: 14px;
}

canvas {
  display: block;
  width: min(94vw, 400px);
  height: auto;
  margin: 10px auto;
  background: #111d32;
  border: 2px solid #00dfff;
  border-radius: 12px;
  touch-action: none;
}

button {
  border: none;
  border-radius: 10px;
  background: #00dfff;
  color: #07101b;
  padding: 13px 18px;
  margin: 5px;
  font-size: 15px;
  font-weight: bold;
  cursor: pointer;
}

button:disabled {
  opacity: .5;
}

button:active {
  transform: scale(.96);
}

#message {
  min-height: 28px;
  color: #00eaff;
  padding: 7px;
}

.panel {
  width: min(94vw, 400px);
  margin: 12px auto;
  padding: 15px;
  background: #14223a;
  border-radius: 12px;
}

.hidden {
  display: none;
}

.skin {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #20314d;
  padding: 10px;
  margin: 8px 0;
  border-radius: 9px;
}

.dot {
  display: inline-block;
  width: 17px;
  height: 17px;
  border-radius: 50%;
  margin-right: 8px;
  vertical-align: middle;
}

#shop {
  position: relative;
}
</style>
</head>

<body>

<header>NEON BLOCKS</header>

<div class="stats">
  <span>Уровень: <b id="level">1</b></span>
  <span>Монеты: <b id="coins">0</b> 🪙</span>
  <span>Жизни: <b id="lives">3</b> ❤️</span>
</div>

<canvas id="game" width="400" height="550"></canvas>

<div id="message">Нажми «Играть»</div>

<div>
  <button id="startBtn">Играть</button>
  <button id="pauseBtn">Пауза</button>
  <button id="shopBtn">Магазин</button>
</div>

<div id="shop" class="panel hidden">
  <h2>МАГАЗИН СКИНОВ</h2>
  <p>Покупай скины за монеты</p>
  <div id="skinList"></div>
  <button id="closeShopBtn">Закрыть магазин</button>
</div>

<div class="panel">
  <b>Как играть?</b>
  <p>Двигай платформу пальцем или мышкой.</p>
  <p>Отбивай мяч и разбивай все блоки.</p>
</div>

<script>
"use strict";

// Telegram
try {
  if (window.Telegram && window.Telegram.WebApp) {
    window.Telegram.WebApp.ready();
    window.Telegram.WebApp.expand();
  }
} catch (e) {
  console.log("Telegram SDK недоступен");
}

// Элементы
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

const levelText = document.getElementById("level");
const coinsText = document.getElementById("coins");
const livesText = document.getElementById("lives");
const message = document.getElementById("message");

const shop = document.getElementById("shop");
const skinList = document.getElementById("skinList");

// Скины
const skins = [
  {id:"blue", name:"Неон", color:"#00eaff", price:0},
  {id:"red", name:"Огонь", color:"#ff405d", price:50},
  {id:"green", name:"Изумруд", color:"#38ff91", price:100},
  {id:"purple", name:"Космос", color:"#b56bff", price:150},
  {id:"gold", name:"Золото", color:"#ffd447", price:250}
];

// Сохранение
function readSave() {
  try {
    return JSON.parse(localStorage.getItem("neon_blocks_v2")) || {};
  } catch (e) {
    return {};
  }
}

const save = readSave();

let coins = Math.max(0, Number(save.coins) || 0);
let unlocked = Array.isArray(save.unlocked)
  ? save.unlocked.filter(id => skins.some(s => s.id === id))
  : ["blue"];

if (!unlocked.includes("blue")) {
  unlocked.push("blue");
}

let selectedSkin = unlocked.includes(save.selectedSkin)
  ? save.selectedSkin
  : "blue";

function saveGame() {
  try {
    localStorage.setItem("neon_blocks_v2", JSON.stringify({
      coins: coins,
      unlocked: unlocked,
      selectedSkin: selectedSkin
    }));
  } catch (e) {
    console.log("Не удалось сохранить прогресс");
  }
}

// Состояние игры
let level = 1;
let lives = 3;
let score = 0;

let state = "ready";
// ready, playing, paused, over

const paddle = {
  x: 160,
  y: 510,
  w: 80,
  h: 12
};

const ball = {
  x: 200,
  y: 470,
  r: 7,
  dx: 3,
  dy: -4
};

let bricks = [];

// Разные схемы уровней
const patterns = [
  [
    "111111",
    "111111",
    "111111"
  ],
  [
    "001100",
    "011110",
    "111111",
    "111111"
  ],
  [
    "110011",
    "110011",
    "110011",
    "110011"
  ],
  [
    "101010",
    "010101",
    "101010",
    "010101",
    "101010"
  ],
  [
    "111111",
    "100001",
    "111111",
    "100001",
    "111111"
  ]
];

function currentColor() {
  const skin = skins.find(s => s.id === selectedSkin);
  return skin ? skin.color : "#00eaff";
}

function updateStats() {
  levelText.textContent = level;
  coinsText.textContent = coins;
  livesText.textContent = lives;
}

// Создание уровня
function createLevel() {
  bricks = [];

  const pattern = patterns[(level - 1) % patterns.length];

  const brickW = 55;
  const brickH = 20;
  const gap = 7;
  const startX = 14;
  const startY = 65;

  for (let r = 0; r < pattern.length; r++) {
    for (let c = 0; c < pattern[r].length; c++) {
      if (pattern[r][c] === "1") {
        bricks.push({
          x: startX + c * (brickW + gap),
          y: startY + r * (brickH + gap),
          w: brickW,
          h: brickH,
          alive: true
        });
      }
    }
  }

  resetBall();
  updateStats();
}

// Сброс мяча
function resetBall() {
  ball.x = paddle.x + paddle.w / 2;
  ball.y = paddle.y - 15;

  const direction = Math.random() > 0.5 ? 1 : -1;

  ball.dx = direction * (2.8 + Math.min(level * 0.12, 2));
  ball.dy = -(3.5 + Math.min(level * 0.15, 2));
}

// Начать игру
function startGame() {
  level = 1;
  lives = 3;
  score = 0;

  paddle.x = 160;

  createLevel();

  state = "playing";

  message.textContent = "Игра началась!";
  document.getElementById("startBtn").textContent = "Заново";

  updateStats();
}

// Пауза
function togglePause() {
  if (state === "playing") {
    state = "paused";
    message.textContent = "Пауза";
  } else if (state === "paused") {
    state = "playing";
    message.textContent = "Продолжаем!";
  }
}

// Движение платформы
function movePaddle(clientX) {
  const rect = canvas.getBoundingClientRect();

  const position = (clientX - rect.left) * (400 / rect.width);

  paddle.x = position - paddle.w / 2;

  if (paddle.x < 0) paddle.x = 0;
  if (paddle.x + paddle.w > 400) {
    paddle.x = 400 - paddle.w;
  }
}

canvas.addEventListener("pointermove", function(e) {
  movePaddle(e.clientX);
});

canvas.addEventListener("pointerdown", function(e) {
  movePaddle(e.clientX);
});

// Клавиатура
document.addEventListener("keydown", function(e) {
  if (e.key === "ArrowLeft" || e.key === "a") {
    paddle.x -= 25;
  }

  if (e.key === "ArrowRight" || e.key === "d") {
    paddle.x += 25;
  }

  paddle.x = Math.max(0, Math.min(400 - paddle.w, paddle.x));
});

// Проверка столкновений
function updateGame() {
  if (state !== "playing") return;

  const oldX = ball.x;
  const oldY = ball.y;

  ball.x += ball.dx;
  ball.y += ball.dy;

  // Левая и правая стены
  if (ball.x - ball.r < 0) {
    ball.x = ball.r;
    ball.dx = Math.abs(ball.dx);
  }

  if (ball.x + ball.r > 400) {
    ball.x = 400 - ball.r;
    ball.dx = -Math.abs(ball.dx);
  }

  // Верхняя стена
  if (ball.y - ball.r < 0) {
    ball.y = ball.r;
    ball.dy = Math.abs(ball.dy);
  }

  // Платформа
  if (
    ball.dy > 0 &&
    oldY + ball.r <= paddle.y &&
    ball.y + ball.r >= paddle.y &&
    ball.x >= paddle.x &&
    ball.x <= paddle.x + paddle.w
  ) {
    const hit = (ball.x - (paddle.x + paddle.w / 2)) / (paddle.w / 2);

    ball.dy = -Math.abs(ball.dy);
    ball.dx = hit * 5;
    ball.y = paddle.y - ball.r;
  }

  // Блоки
  for (let i = 0; i < bricks.length; i++) {
    const b = bricks[i];

    if (!b.alive) continue;

    const hit =
      ball.x + ball.r > b.x &&
      ball.x - ball.r < b.x + b.w &&
      ball.y + ball.r > b.y &&
      ball.y - ball.r < b.y + b.h;

    if (hit) {
      b.alive = false;

      // Выбираем направление отскока
      if (oldY + ball.r <= b.y || oldY - ball.r >= b.y + b.h) {
        ball.dy *= -1;
      } else {
        ball.dx *= -1;
      }

      coins += 2;
      score += 10;

      saveGame();
      updateStats();

      break;
    }
  }

  // Мяч упал
  if (ball.y - ball.r > 550) {
    lives--;
    updateStats();

    if (lives <= 0) {
      state = "over";
      message.textContent = "Игра окончена! Нажми «Играть».";
    } else {
      resetBall();
      message.textContent = "Осталось жизней: " + lives;
    }
    return;
  }

  // Проверка победы
  if (bricks.length > 0 && bricks.every(b => !b.alive)) {
    level++;
    coins += 20;

    saveGame();
    createLevel();

    message.textContent = "Уровень пройден! +20 монет";
  }
}

// Отрисовка
function drawGame() {
  ctx.clearRect(0, 0, 400, 550);

  ctx.fillStyle = "#111d32";
  ctx.fillRect(0, 0, 400, 550);

  // Блоки
  for (const b of bricks) {
    if (!b.alive) continue;

    ctx.fillStyle = currentColor();
    ctx.fillRect(b.x, b.y, b.w, b.h);
  }

  // Платформа
  ctx.fillStyle = currentColor();
  ctx.fillRect(paddle.x, paddle.y, paddle.w, paddle.h);

  // Мяч
  ctx.beginPath();
  ctx.arc(ball.x, ball.y, ball.r, 0, Math.PI * 2);
  ctx.fillStyle = "white";
  ctx.fill();

  // Подсказки
  if (state === "ready") {
    ctx.fillStyle = "white";
    ctx.font = "bold 22px Arial";
    ctx.textAlign = "center";
    ctx.fillText("НАЖМИ ИГРАТЬ", 200, 300);
  }

  if (state === "paused") {
    ctx.fillStyle = "rgba(0,0,0,0.65)";
    ctx.fillRect(0, 0, 400, 550);

    ctx.fillStyle = "white";
    ctx.font = "bold 30px Arial";
    ctx.textAlign = "center";
    ctx.fillText("ПАУЗА", 200, 280);
  }

  if (state === "over") {
    ctx.fillStyle = "rgba(0,0,0,0.7)";
    ctx.fillRect(0, 0, 400, 550);

    ctx.fillStyle = "#ff405d";
    ctx.font = "bold 24px Arial";
    ctx.textAlign = "center";
    ctx.fillText("ИГРА ОКОНЧЕНА", 200, 280);
  }
}

// Главный игровой цикл
function gameLoop() {
  updateGame();
  drawGame();

  requestAnimationFrame(gameLoop);
}

// Магазин
function openShop() {
  shop.classList.remove("hidden");

  if (state === "playing") {
    state = "paused";
  }

  renderShop();
}

function closeShop() {
  shop.classList.add("hidden");

  if (state === "paused") {
    state = "playing";
  }
}

function renderShop() {
  skinList.innerHTML = "";

  for (const skin of skins) {
    const owned = unlocked.includes(skin.id);
    const selected = selectedSkin === skin.id;

    const item = document.createElement("div");
    item.className = "skin";

    const name = document.createElement("span");

    const dot = document.createElement("span");
    dot.className = "dot";
    dot.style.background = skin.color;

    name.appendChild(dot);
    name.appendChild(document.createTextNode(skin.name));

    const btn = document.createElement("button");

    if (selected) {
      btn.textContent = "Выбран";
      btn.disabled = true;
    } else if (owned) {
      btn.textContent = "Использовать";
      btn.addEventListener("click", function() {
        selectedSkin = skin.id;
        saveGame();
        renderShop();
      });
    } else {
      btn.textContent = skin.price + " 🪙";

      btn.addEventListener("click", function() {
        if (coins < skin.price) {
          alert("Недостаточно монет!");
          return;
        }

        coins -= skin.price;
        unlocked.push(skin.id);

        saveGame();
        updateStats();
        renderShop();
      });
    }

    item.appendChild(name);
    item.appendChild(btn);

    skinList.appendChild(item);
  }
}

// Кнопки
document.getElementById("startBtn").addEventListener("click", startGame);
document.getElementById("pauseBtn").addEventListener("click", togglePause);
document.getElementById("shopBtn").addEventListener("click", openShop);
document.getElementById("closeShopBtn").addEventListener("click", closeShop);

// Первоначальная настройка
createLevel();
updateStats();
gameLoop();

</script>
</body>
</html>

 
 

  

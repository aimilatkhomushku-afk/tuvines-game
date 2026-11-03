 <!DOCTYPE html>

<html lang="ru">

<head>

<meta charset="UTF-8">

<meta name="viewport" content="width=device-width, initial-scale=1">

<title>Тувинец: Большое приключение</title>

<style>

* { box-sizing: border-box; }

body {

  margin: 0;

  background: #172b38;

  color: white;

  font-family: Arial;

  text-align: center;

}

h1 {

  font-size: 23px;

  padding: 10px;

}

#info {

  font-size: 18px;

  margin: 10px;

}

canvas {

  display: block;

  width: 95%;

  max-width: 420px;

  height: auto;

  margin: auto;

  border: 3px solid white;

  border-radius: 12px;

  background: #f4d58b;

  touch-action: none;

}

button {

  background: #e8a43b;

  color: #26190b;

  border: 0;

  border-radius: 12px;

  padding: 14px 22px;

  margin: 6px;

  font-size: 22px;

  font-weight: bold;

}

.controls {

  display: flex;

  justify-content: center;

  gap: 10px;

}

#start {

  font-size: 19px;

}

</style>

</head>

<body>

<h1>ТУВИНЕЦ: БОЛЬШОЕ ПРИКЛЮЧЕНИЕ</h1>

<div id="info">Монеты: 0 | Рекорд: 0</div>

<canvas id="game" width="360" height="520"></canvas>

<button id="start">НАЧАТЬ ИГРУ</button>

<div class="controls">

  <button id="left">←</button>

  <button id="jump">ПРЫЖОК</button>

  <button id="right">→</button>

</div>

<p>Собирай монеты и буузы. Уворачивайся от камней!</p>

<script>

const canvas = document.getElementById("game");

const ctx = canvas.getContext("2d");

let player;

let objects;

let score = 0;

let speed = 3;

let running = false;

let lastTime = 0;

let spawnTimer = 0;

let animation = null;

let highScore = 0;

try {

  highScore = Number(localStorage.getItem("tuvinRecord") || 0);

} catch(e) {}

function restart() {

  cancelAnimationFrame(animation);

  player = {

    x: 180,

    y: 440,

    jump: 0

  };

  objects = [];

  score = 0;

  speed = 3;

  spawnTimer = 0;

  running = true;

  document.getElementById("start").textContent = "ЗАНОВО";

  lastTime = performance.now();

  requestAnimationFrame(loop);

}

function background() {

  let sky = ctx.createLinearGradient(0, 0, 0, 520);

  sky.addColorStop(0, "#70c5e8");

  sky.addColorStop(1, "#f9d58c");

  ctx.fillStyle = sky;

  ctx.fillRect(0, 0, 360, 520);

  // Солнце

  ctx.fillStyle = "#ffe56b";

  ctx.beginPath();

  ctx.arc(290, 75, 35, 0, Math.PI * 2);

  ctx.fill();

  // Горы

  ctx.fillStyle = "#829b77";

  ctx.beginPath();

  ctx.moveTo(0, 300);

  ctx.lineTo(100, 130);

  ctx.lineTo(200, 300);

  ctx.fill();

  ctx.beginPath();

  ctx.moveTo(130, 300);

  ctx.lineTo(260, 115);

  ctx.lineTo(360, 300);

  ctx.fill();

  // Степь

  ctx.fillStyle = "#b8a04c";

  ctx.fillRect(0, 300, 360, 220);

  ctx.fillStyle = "#d4b65d";

  ctx.fillRect(0, 340, 360, 180);

  // Дорога

  ctx.fillStyle = "#9b8549";

  ctx.fillRect(55, 300, 250, 220);

}

function drawPlayer() {

  let y = player.y - player.jump;

  // Тень

  ctx.fillStyle = "rgba(0,0,0,0.2)";

  ctx.beginPath();

  ctx.ellipse(player.x, 460, 23, 7, 0, 0, Math.PI * 2);

  ctx.fill();

  // Ноги

  ctx.fillStyle = "#333";

  ctx.fillRect(player.x - 13, y + 24, 10, 22);

  ctx.fillRect(player.x + 3, y + 24, 10, 22);

  // Тело

  ctx.fillStyle = "#286b53";

  ctx.fillRect(player.x - 17, y - 5, 34, 35);

  // Голова

  ctx.fillStyle = "#d99b68";

  ctx.beginPath();

  ctx.arc(player.x, y - 17, 15, 0, Math.PI * 2);

  ctx.fill();

  // Волосы

  ctx.fillStyle = "#38251b";

  ctx.beginPath();

  ctx.arc(player.x, y - 24, 15, Math.PI, Math.PI * 2);

  ctx.fill();

  // Глаза

  ctx.fillStyle = "#111";

  ctx.fillRect(player.x - 6, y - 19, 3, 3);

  ctx.fillRect(player.x + 4, y - 19, 3, 3);

}

function drawObjects() {

  objects.forEach(o => {

    ctx.font = "30px Arial";

    ctx.textAlign = "center";

    if (o.type === "coin") {

      ctx.fillText("🪙", o.x, o.y);

    }

    if (o.type === "food") {

      ctx.fillText("🥟", o.x, o.y);

    }

    if (o.type === "rock") {

      ctx.fillText("🪨", o.x, o.y);

    }

  });

}

function spawnObject() {

  let chance = Math.random();

  let type =

    chance < 0.45 ? "coin" :

    chance < 0.65 ? "food" : "rock";

  objects.push({

    x: 85 + Math.random() * 190,

    y: 300,

    type: type,

    taken: false

  });

}

function update(dt) {

  speed += dt * 0.00015;

  spawnTimer += dt;

  if (spawnTimer > 850) {

    spawnObject();

    spawnTimer = 0;

  }

  objects.forEach(o => {

    o.y += speed * dt / 16;

    let playerY = player.y - player.jump;

    if (!o.taken &&

        Math.abs(o.x - player.x) < 28 &&

        Math.abs(o.y - playerY) < 30) {

      if (o.type === "coin" || o.type === "food") {

        score += o.type === "food" ? 3 : 1;

        o.taken = true;

      }

      if (o.type === "rock" && player.jump < 25) {

        running = false;

      }

    }

  });

  objects = objects.filter(o => o.y < 540 && !o.taken);

  if (player.jump > 0) {

    player.jump -= 5 * dt / 16;

    if (player.jump < 0) player.jump = 0;

  }

  document.getElementById("info").textContent =

    "Монеты: " + score + " | Рекорд: " + highScore;

}

function draw() {

  background();

  drawObjects();

  drawPlayer();

  if (!running) {

    ctx.fillStyle = "rgba(0,0,0,0.65)";

    ctx.fillRect(0, 0, 360, 520);

    ctx.fillStyle = "white";

    ctx.textAlign = "center";

    ctx.font = "28px Arial";

    ctx.fillText("ИГРА ОКОНЧЕНА", 180, 230);

    ctx.font = "21px Arial";

    ctx.fillText("Собрано: " + score, 180, 275);

  }

}

function loop(time) {

  if (!running) return;

  let dt = Math.min(time - lastTime, 40);

  lastTime = time;

  update(dt);

  draw();

  if (score > highScore) {

    highScore = score;

    try {

      localStorage.setItem("tuvinRecord", highScore);

    } catch(e) {}

  }

  if (running) {

    animation = requestAnimationFrame(loop);

  } else {

    draw();

    document.getElementById("start").textContent = "ИГРАТЬ СНОВА";

  }

}

function move(dir) {

  if (!running) return;

  player.x += dir * 30;

  player.x = Math.max(75, Math.min(285, player.x));

}

function jump() {

  if (running && player.jump === 0) {

    player.jump = 85;

  }

}

document.getElementById("left").addEventListener("click", () => move(-1));

document.getElementById("right").addEventListener("click", () => move(1));

document.getElementById("jump").addEventListener("click", jump);

document.getElementById("start").addEventListener("click", restart);

background();

ctx.font = "25px Arial";

ctx.textAlign = "center";

ctx.fillStyle = "#222";

ctx.fillText("Нажми НАЧАТЬ ИГРУ", 180, 250);

</script>

</body>

</html>

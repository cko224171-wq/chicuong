<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Runner Full</title>
<style>
body { margin:0; overflow:hidden; }
canvas { display:block; background:#87CEEB; }
</style>
</head>
<body>

<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

// ===== GROUND =====
let groundY = canvas.height - 120;

// ===== PLAYER =====
let player = {
  x:120,
  y:groundY - 60,
  w:50,
  h:60,
  dy:0,
  jump:false
};

// ===== GAME =====
let gravity = 1;
let obstacles = [];
let coins = [];
let score = 0;
let best = localStorage.getItem("best") || 0;
let coinCount = 0;
let speed = 6;
let gameOver = false;

// ===== NÚT CHƠI LẠI =====
let restartBtn = {
  x: canvas.width/2 - 80,
  y: canvas.height/2 + 50,
  w: 160,
  h: 50
};

// ===== INPUT =====
canvas.addEventListener("touchstart", function(e){
  let touch = e.touches[0];
  let tx = touch.clientX;
  let ty = touch.clientY;

  if(gameOver){
    if(tx > restartBtn.x && tx < restartBtn.x + restartBtn.w &&
       ty > restartBtn.y && ty < restartBtn.y + restartBtn.h){
        restart();
    }
    return;
  }

  if(!player.jump){
    player.dy = -18;
    player.jump = true;
  }
});

document.addEventListener("keydown", e=>{
  if(e.code==="Space"){
    if(gameOver){
      restart();
    } else if(!player.jump){
      player.dy = -18;
      player.jump = true;
    }
  }
});

// ===== SPAWN =====
function spawnObstacle(){
  if(gameOver) return;

  let h = 40 + Math.random()*50;

  obstacles.push({
    x:canvas.width,
    y:groundY - h,
    w:40,
    h:h
  });
}

function spawnCoin(){
  if(gameOver) return;

  coins.push({
    x:canvas.width,
    y:groundY - 60 - Math.random()*100,
    r:10
  });
}

// ===== UPDATE =====
function update(){
  if(gameOver) return;

  speed += 0.002;

  player.dy += gravity;
  player.y += player.dy;

  if(player.y >= groundY - player.h){
    player.y = groundY - player.h;
    player.dy = 0;
    player.jump = false;
  }

  obstacles.forEach(o=>{
    o.x -= speed;

    if(player.x < o.x+o.w &&
       player.x+player.w > o.x &&
       player.y < o.y+o.h &&
       player.y+player.h > o.y){
        endGame();
    }
  });

  obstacles = obstacles.filter(o=>o.x>-50);

  coins.forEach(c=>{
    c.x -= speed;

    let dx = player.x+player.w/2 - c.x;
    let dy = player.y+player.h/2 - c.y;

    if(Math.sqrt(dx*dx + dy*dy) < 30){
      coinCount++;
      c.x = -100;
    }
  });

  coins = coins.filter(c=>c.x>-50);

  score++;
}

// ===== GAME OVER =====
function endGame(){
  gameOver = true;

  if(score > best){
    best = score;
    localStorage.setItem("best", best);
  }
}

// ===== RESTART =====
function restart(){
  obstacles = [];
  coins = [];
  score = 0;
  coinCount = 0;
  speed = 6;
  gameOver = false;

  player.y = groundY - player.h;
  player.dy = 0;
}

// ===== DRAW =====
function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);
  // ground
  ctx.fillStyle="#654321";
  ctx.fillRect(0,groundY,canvas.width,120);

  ctx.fillStyle="green";
  ctx.fillRect(0,groundY,canvas.width,10);

  // player
  ctx.fillStyle="blue";
  ctx.fillRect(player.x,player.y,player.w,player.h);

  // obstacle
  ctx.fillStyle="black";
  obstacles.forEach(o=>{
    ctx.fillRect(o.x,o.y,o.w,o.h);
  });

  // coin
  coins.forEach(c=>{
    ctx.fillStyle="gold";
    ctx.beginPath();
    ctx.arc(c.x,c.y,c.r,0,Math.PI*2);
    ctx.fill();
  });

  // UI
  ctx.fillStyle="white";
  ctx.font="20px Arial";
  ctx.textAlign="left";
  ctx.fillText("Khoảng cách: "+score,20,40);
  ctx.fillText("Kỷ lục: "+best,20,70);
  ctx.fillText("Xu: "+coinCount,20,100);

  // tên bạn
  ctx.textAlign="right";
  ctx.font="14px Arial";
  ctx.fillText("Chí Cường", canvas.width-10, canvas.height-10);

  // game over + nút
  if(gameOver){
    ctx.textAlign="center";
    ctx.font="40px Arial";
    ctx.fillText("GAME OVER", canvas.width/2, canvas.height/2);

    ctx.fillStyle="red";
    ctx.fillRect(restartBtn.x, restartBtn.y, restartBtn.w, restartBtn.h);

    ctx.fillStyle="white";
    ctx.font="20px Arial";
    ctx.fillText("Chơi lại", canvas.width/2, restartBtn.y + 32);
  }
}

// ===== LOOP =====
function loop(){
  update();
  draw();
  requestAnimationFrame(loop);
}

setInterval(spawnObstacle,1300);
setInterval(spawnCoin,2000);

loop();
</script>

</body>
</html>

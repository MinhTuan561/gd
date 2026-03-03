<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Geometry Dash 2.2 Style</title>
<style>
body {
  margin:0;
  overflow:hidden;
  background:#000;
}
canvas {
  display:block;
  margin:auto;
  background:linear-gradient(#0f0f1f,#000);
}
</style>
</head>
<body>
<canvas id="game"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = 1000;
canvas.height = 500;

/* =======================
   GAME STATE
======================= */
let gravity = 0.8;
let speed = 6;
let mode = "cube"; // cube or ship
let gameRunning = true;
let progress = 0;
let cameraX = 0;

/* =======================
   PLAYER
======================= */
const player = {
  x: 200,
  y: 300,
  size: 40,
  dy: 0,
  gravityDir: 1,
  jumping: false
};

/* =======================
   OBJECT ARRAYS
======================= */
let spikes = [];
let portals = [];
let pads = [];
let coins = [];

/* =======================
   LEVEL GENERATOR
======================= */
function generateLevel(){
  for(let i=600;i<5000;i+=400){
    spikes.push({x:i,y:360,w:40,h:40});
  }

  portals.push({x:1200,type:"ship"});
  portals.push({x:2000,type:"cube"});
  portals.push({x:2600,type:"gravity"});
  portals.push({x:3200,type:"speed"});

  pads.push({x:1500,y:340});
  pads.push({x:2800,y:340});

  coins.push({x:1800,y:250,collected:false});
  coins.push({x:3500,y:220,collected:false});
}
generateLevel();

/* =======================
   INPUT
======================= */
function press(){
  if(mode==="cube"){
    if(!player.jumping){
      player.dy = -15 * player.gravityDir;
      player.jumping = true;
    }
  } else if(mode==="ship"){
    player.dy -= 1.5 * player.gravityDir;
  }
}

document.addEventListener("keydown",e=>{
  if(e.code==="Space") press();
});
document.addEventListener("mousedown",press);

/* =======================
   UPDATE LOOP
======================= */
function update(){

if(!gameRunning) return;

ctx.clearRect(0,0,canvas.width,canvas.height);

progress += 0.05;
cameraX += speed;

/* GROUND */
ctx.fillStyle="#fff";
ctx.fillRect(0,360,canvas.width,5);

/* PLAYER PHYSICS */
if(mode==="cube"){
  player.dy += gravity * player.gravityDir;
}
if(mode==="ship"){
  player.dy += 0.5 * player.gravityDir;
}

player.y += player.dy;

/* GROUND COLLISION */
if(player.gravityDir===1){
  if(player.y>=300){
    player.y=300;
    player.dy=0;
    player.jumping=false;
  }
}else{
  if(player.y<=0){
    player.y=0;
    player.dy=0;
    player.jumping=false;
  }
}

/* DRAW PLAYER */
ctx.save();
ctx.translate(player.x,player.y);
ctx.fillStyle = mode==="cube" ? "#00ffff" : "#ff00ff";
ctx.fillRect(0,0,player.size,player.size);
ctx.restore();

/* SPIKES */
ctx.fillStyle="red";
spikes.forEach(s=>{
  let drawX = s.x - cameraX;
  ctx.beginPath();
  ctx.moveTo(drawX,360);
  ctx.lineTo(drawX+20,320);
  ctx.lineTo(drawX+40,360);
  ctx.fill();

  if(player.x < drawX+40 &&
     player.x+player.size > drawX &&
     player.y+player.size > 320 &&
     player.y < 360){
       gameOver();
  }
});

/* PORTALS */
portals.forEach(p=>{
  let drawX = p.x - cameraX;
  ctx.fillStyle="yellow";
  ctx.fillRect(drawX,250,20,100);

  if(player.x < drawX+20 &&
     player.x+player.size > drawX){
       if(p.type==="ship") mode="ship";
       if(p.type==="cube") mode="cube";
       if(p.type==="gravity") player.gravityDir*=-1;
       if(p.type==="speed") speed+=2;
  }
});

/* JUMP PADS */
pads.forEach(p=>{
  let drawX = p.x - cameraX;
  ctx.fillStyle="lime";
  ctx.fillRect(drawX,p.y,40,20);

  if(player.x < drawX+40 &&
     player.x+player.size > drawX &&
     player.y+player.size >= p.y){
       player.dy = -20 * player.gravityDir;
  }
});

/* COINS */
coins.forEach(c=>{
  if(!c.collected){
    let drawX = c.x - cameraX;
    ctx.fillStyle="gold";
    ctx.beginPath();
    ctx.arc(drawX,c.y,10,0,Math.PI*2);
    ctx.fill();

    if(player.x < drawX+10 &&
       player.x+player.size > drawX-10 &&
       player.y < c.y+10 &&
       player.y+player.size > c.y-10){
         c.collected=true;
    }
  }
});

/* PROGRESS BAR */
ctx.fillStyle="cyan";
ctx.fillRect(0,0,progress*3,10);

if(cameraX>4500){
  alert("LEVEL COMPLETE!");
  gameRunning=false;
}

requestAnimationFrame(update);
}

function gameOver(){
  alert("GAME OVER");
  location.reload();
}

update();
</script>
</body>
</html>

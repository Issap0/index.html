<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>¡Salta!</title>
<style>
body {
  margin: 0;
  padding: 0;
  background: #87CEEB;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 100vh;
  font-family: Arial, sans-serif;
  overflow: hidden;
}
h1 {
  margin: 10px;
  font-size: 28px;
  color: #fff;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
}
canvas {
  border: 3px solid #333;
  background: #87CEEB;
}

/* Modales */
.modal {
  display: none;
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: #fff;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
  text-align: center;
  z-index: 10;
}
.modal button {
  margin-top: 15px;
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  background: #ff4d4d;
  color: #fff;
  font-size: 14px;
  cursor: pointer;
}
.modal button:hover { background: #e63939; }

/* Corazón del modal victoria */
.heart {
  width: 50px;
  height: 50px;
  position: relative;
  margin: 10px auto;
}
.heart:before,
.heart:after {
  content: "";
  position: absolute;
  width: 50px;
  height: 80px;
  background: red;
  border-radius: 50px 50px 0 0;
  top: 0;
}
.heart:before { left: 50px; transform: rotate(-45deg); transform-origin: 0 100%; }
.heart:after { left: 0; transform: rotate(45deg); transform-origin: 100% 100%; }

/* Corazones flotando en victoria */
.floating-heart {
  position: absolute;
  width: 20px;
  height: 20px;
  background: pink;
  transform: rotate(45deg);
  animation: floatUp 2s linear forwards;
}
.floating-heart:before, .floating-heart:after {
  content: "";
  position: absolute;
  width: 20px;
  height: 20px;
  background: pink;
  border-radius: 50%;
}
.floating-heart:before { top: -10px; left: 0; }
.floating-heart:after { top: 0; left: -10px; }

@keyframes floatUp {
  0% { transform: translate(0,0) rotate(45deg); opacity:1; }
  100% { transform: translate(-30px,-100px) rotate(45deg); opacity:0; }
}
</style>
</head>
<body>

<h1>¡Salta!</h1>
<canvas id="juego" width="600" height="250"></canvas>

<!-- Modal derrota -->
<div id="modalDerrota" class="modal">
  <p id="mensajeDerrota"></p>
  <button onclick="reiniciarJuego()">🔄 Reintentar</button>
</div>

<!-- Modal victoria -->
<div id="modalVictoria" class="modal">
  <div class="heart"></div>
  <p>¡Eres increíble! por cierto... me gustan mucho tus ojitos</p>
  <button onclick="reiniciarJuego()">❤️ Volver a jugar</button>
</div>

<!-- Sonidos -->
<audio id="sonidoSalto" src="https://www.soundjay.com/buttons/sounds/button-16.mp3"></audio>
<audio id="sonidoChoque" src="https://www.soundjay.com/button/beep-10.mp3"></audio>
<audio id="sonidoVictoria" src="https://www.soundjay.com/misc/sounds/bell-ringing-05.mp3"></audio>

<script>
const canvas = document.getElementById("juego");
const ctx = canvas.getContext("2d");

const sonidoSalto = document.getElementById("sonidoSalto");
const sonidoChoque = document.getElementById("sonidoChoque");
const sonidoVictoria = document.getElementById("sonidoVictoria");

let carro, obstaculos, frame, score, gameOver;
let clouds = [];

const mensajes = [
  "Ups, saltaste mal... intenta de nuevo 😔",
  "La roca ganó esta vez, pero eres mi fav al volante JAJA 😼",
  "Perdiste de nuevo, pero fue con estilo ¿sí o k? 😎",
  "¿Otra vez? 😑"
];

function init(){
  carro = {x:70, y:180, width:50, height:30, dy:0, jumping:false};
  obstaculos = [];
  frame = 0;
  score = 0;
  gameOver = false;

  clouds = [];
  for(let i=0;i<3;i++) clouds.push({x:i*200+100, y:50+Math.random()*30});

  document.getElementById("modalDerrota").style.display="none";
  document.getElementById("modalVictoria").style.display="none";

  update();
}

function drawBackground(){
  ctx.fillStyle="#87CEEB";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  ctx.fillStyle="white";
  clouds.forEach(cloud=>{
    ctx.beginPath();
    ctx.arc(cloud.x, cloud.y, 20,0,Math.PI*2);
    ctx.arc(cloud.x+20, cloud.y,25,0,Math.PI*2);
    ctx.arc(cloud.x+40, cloud.y,20,0,Math.PI*2);
    ctx.fill();
  });
  clouds.forEach(cloud=>{
    cloud.x -= 0.5;
    if(cloud.x < -50) cloud.x = canvas.width + 50;
  });

  ctx.fillStyle="#555";
  ctx.fillRect(0,210,canvas.width,40);
  ctx.strokeStyle="white";
  ctx.setLineDash([40,20]);
  ctx.beginPath();
  ctx.moveTo(0,230);
  ctx.lineTo(canvas.width,230);
  ctx.stroke();
  ctx.setLineDash([]);
}

function drawCarro(){
  ctx.fillStyle="red";
  ctx.fillRect(carro.x, carro.y, carro.width, carro.height);
  ctx.fillRect(carro.x+10, carro.y-15, 30,15);
  ctx.fillStyle="black";
  ctx.beginPath();
  ctx.arc(carro.x+10, carro.y+carro.height,8,0,Math.PI*2);
  ctx.arc(carro.x+40, carro.y+carro.height,8,0,Math.PI*2);
  ctx.fill();
}

function update(){
  if(gameOver) return;
  frame++;
  ctx.clearRect(0,0,canvas.width,canvas.height);

  drawBackground();

  // GRAVEDAD
  carro.dy += 0.7;
  carro.y += carro.dy;
  if(carro.y >= 180){
    carro.y = 180;
    carro.dy = 0;
    carro.jumping = false;
  }

  drawCarro();

  // Obstáculos
  if(frame % Math.max(60, 100 - Math.floor(score/500)) === 0){
    const size = Math.random()<0.5 ? 15 : 25;
    obstaculos.push({x:600, y:210-size, size});
  }
  obstaculos.forEach((o,idx)=>{
    o.x -= 5 + Math.floor(score/1000);
    ctx.fillStyle="sienna";
    ctx.beginPath();
    ctx.arc(o.x, o.y+o.size, o.size,0,Math.PI*2);
    ctx.fill();

    if(carro.x<o.x+o.size && carro.x+carro.width>o.x-o.size &&
       carro.y<o.y+o.size*2 && carro.y+carro.height>o.y){
      sonidoChoque.play();
      mostrarDerrota();
    }
  });
  obstaculos = obstaculos.filter(o=>o.x>-50);

  // Puntos
  score++;
  ctx.fillStyle="black";
  ctx.font="16px Arial";
  ctx.fillText("Puntos: "+score,480,30);

  if(score>=5000){
    sonidoVictoria.play();
    mostrarVictoria();
    return;
  }

  requestAnimationFrame(update);
}

function jump(){
  if(!carro.jumping){
    carro.dy=-14; // suficiente impulso
    carro.jumping=true;
    sonidoSalto.play();
  }
}

function mostrarDerrota(){
  gameOver=true;
  const mensaje = mensajes[Math.floor(Math.random()*mensajes.length)];
  document.getElementById("mensajeDerrota").textContent=mensaje;
  document.getElementById("modalDerrota").style.display="block";
}

function mostrarVictoria(){
  gameOver=true;
  const modal = document.getElementById("modalVictoria");
  modal.style.display="block";

  for(let i=0;i<10;i++){
    const h=document.createElement("div");
    h.className="floating-heart";
    h.style.left = 50 + Math.random()*80 + "px";
    h.style.top = 50 + Math.random()*40 + "px";
    modal.appendChild(h);
    setTimeout(()=>h.remove(),2000);
  }
}

function reiniciarJuego(){ init(); }

// Eventos
document.addEventListener("keydown", function(e){
  if(e.code === "Space" || e.code === "ArrowUp"){
    e.preventDefault();
    jump();
  }
});
document.addEventListener("click", jump);

init();
</script>

</body>
</html>

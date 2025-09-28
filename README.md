<script>
const canvas = document.getElementById("juego");
const ctx = canvas.getContext("2d");

const sonidoSalto = document.getElementById("sonidoSalto");
const sonidoChoque = document.getElementById("sonidoChoque");
const sonidoVictoria = document.getElementById("sonidoVictoria");

let carro, obstaculos, frame, score, gameOver, clouds, particles;

const mensajes = [
  "Ups, saltaste mal... intenta de nuevo 😔",
  "La roca ganó esta vez, pero eres mi fav al volante JAJA 😼",
  "Perdiste de nuevo, pero fue con estilo ¿sí o k? 😎",
  "¿Otra vez? 😑"
];

function init() {
  carro = { x: 70, y: 180, width: 50, height: 30, jumping: false, jumpFrame: 0 };
  obstaculos = [];
  clouds = [];
  particles = [];
  for (let i = 0; i < 3; i++) clouds.push({ x: i*200 + 100, y: 50 + Math.random()*30 });
  frame = 0;
  score = 0;
  gameOver = false;

  document.getElementById("modalDerrota").style.display = "none";
  document.getElementById("modalVictoria").style.display = "none";

  const heart = document.querySelector(".heart");
  heart.style.position = "absolute";
  heart.style.top = "10px";
  heart.style.left = "50%";
  heart.style.transform = "translateX(-50%) scale(1)";
  heart.style.animation = "beat 1s infinite";

  update();
}

function drawBackground() {
  ctx.fillStyle = "#87CEEB";
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  ctx.fillStyle = "white";
  clouds.forEach(cloud => {
    ctx.beginPath();
    ctx.arc(cloud.x, cloud.y, 20, 0, Math.PI*2);
    ctx.arc(cloud.x+20, cloud.y, 25, 0, Math.PI*2);
    ctx.arc(cloud.x+40, cloud.y, 20, 0, Math.PI*2);
    ctx.fill();
  });

  clouds.forEach(cloud => {
    cloud.x -= 0.5;
    if(cloud.x < -50) cloud.x = canvas.width + 50;
  });

  ctx.fillStyle = "#555";
  ctx.fillRect(0, 210, canvas.width, 40);
  ctx.strokeStyle = "white";
  ctx.setLineDash([40,20]);
  ctx.beginPath();
  ctx.moveTo(0,230);
  ctx.lineTo(canvas.width,230);
  ctx.stroke();
  ctx.setLineDash([]);
}

function drawCarro() {
  ctx.fillStyle = "red";
  ctx.fillRect(carro.x, carro.y, carro.width, carro.height);
  ctx.fillRect(carro.x + 10, carro.y - 15, 30, 15);

  const rot = frame * 0.2;
  ctx.fillStyle = "black";
  [10,40].forEach(offset => {
    ctx.beginPath();
    ctx.arc(carro.x + offset, carro.y + carro.height, 8, rot, rot + Math.PI*2);
    ctx.fill();
  });
}

function createParticles(x, y, color="orange") {
  for(let i=0;i<5;i++){
    particles.push({x,y,dx:(Math.random()-0.5)*4,dy:(Math.random()-1.5)*4,life:30,color, type:"square"});
  }
}

function createHeartParticles(x, y) {
  for(let i=0;i<10;i++){
    particles.push({
      x, y,
      dx: (Math.random()-0.5)*1.5,
      dy: -Math.random()*2-1,
      life: 60 + Math.random()*20,
      color: "pink",
      type: "heart",
      size: 8 + Math.random()*4,
      bounce: true
    });
  }
}

function drawParticles() {
  particles.forEach((p, idx) => {
    if(p.type === "square") {
      ctx.fillStyle = p.color;
      ctx.fillRect(p.x, p.y, 3,3);
    } else if(p.type === "heart") {
      ctx.fillStyle = p.color;
      // corazón simplificado: dos círculos arriba + triángulo abajo
      const s = p.size/2;
      ctx.beginPath();
      ctx.arc(p.x - s, p.y, s, Math.PI, 0);
      ctx.arc(p.x + s, p.y, s, Math.PI, 0);
      ctx.moveTo(p.x - p.size, p.y);
      ctx.lineTo(p.x, p.y + p.size);
      ctx.lineTo(p.x + p.size, p.y);
      ctx.closePath();
      ctx.fill();
    }

    p.x += p.dx;
    p.y += p.dy;

    if(p.type==="heart" && p.bounce && p.dy < 0 && p.life < 20){
      p.dy = -p.dy * 0.5;
      p.bounce = false;
    }

    p.life--;
    if(p.life <= 0) particles.splice(idx,1);
  });
}

function update() {
  if(gameOver) return;
  frame++;
  ctx.clearRect(0,0,canvas.width,canvas.height);
  drawBackground();

  if(carro.jumping){
    carro.jumpFrame++;
    const t = carro.jumpFrame/20;
    if(t<=1){
      carro.y = 180 - Math.sin(t*Math.PI)*100;
    } else {
      carro.jumping = false;
      carro.jumpFrame=0;
      carro.y=180;
    }
  }

  drawCarro();
  drawParticles();

  if(frame % Math.max(60, 100 - Math.floor(score/500)) === 0){
    const size = Math.random() < 0.5 ? 15 : 25;
    const y = 210 - size - (Math.random()<0.3 ? 20:0);
    obstaculos.push({x:600,y,size});
  }

  obstaculos.forEach((o, idx) => {
    o.x -= 5 + Math.floor(score/1000);
    ctx.fillStyle = "sienna";
    ctx.beginPath();
    ctx.arc(o.x, o.y + o.size, o.size,0,Math.PI*2);
    ctx.fill();

    if(carro.x < o.x+o.size &&
       carro.x+carro.width>o.x-o.size &&
       carro.y < o.y+o.size*2 &&
       carro.y+carro.height>o.y){
      sonidoChoque.play();
      createParticles(carro.x+carro.width/2, carro.y+carro.height/2,"red");
      mostrarDerrota();
    }
  });

  obstaculos = obstaculos.filter(o=>o.x>-50);

  score++;
  ctx.fillStyle="black";
  ctx.font="20px Arial";
  ctx.shadowColor="white";
  ctx.shadowBlur=4;
  ctx.fillText("Puntos: "+score,460,30);
  ctx.shadowBlur=0;

  if(score>=5000){
    sonidoVictoria.play();
    createHeartParticles(carro.x+carro.width/2, carro.y+carro.height/2);
    mostrarVictoria();
    return;
  }

  requestAnimationFrame(update);
}

function jump(){
  if(!carro.jumping){
    carro.jumping=true;
    carro.jumpFrame=0;
    sonidoSalto.play();
    createParticles(carro.x+carro.width/2, carro.y+carro.height,"yellow");
  }
}

function mostrarDerrota(){
  gameOver=true;
  const mensaje = mensajes[Math.floor(Math.random()*mensajes.length)];
  document.getElementById("mensajeDerrota").textContent = mensaje;
  document.getElementById("modalDerrota").style.display="block";
}

function mostrarVictoria(){
  gameOver=true;
  document.getElementById("modalVictoria").style.display="block";
}

function reiniciarJuego(){
  init();
}

document.addEventListener("keydown",jump);
document.addEventListener("click",jump);

init();
</script>

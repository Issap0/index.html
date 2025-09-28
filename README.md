<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>¡Santa!</title>
  <style>
    :root{
      --bg:#f0f0f0;
      --panel:#fff;
      --accent:#ff4d4d;
      --text:#222;
    }
    html,body{height:100%;margin:0;background:var(--bg);font-family:Arial,Helvetica,sans-serif;color:var(--text)}
    .center{min-height:100%;display:flex;align-items:center;justify-content:center;padding:12px;box-sizing:border-box}
    canvas{background:#fff;border:3px solid #333;border-radius:8px;display:block}
    /* modales */
    .modal{
      position:fixed;inset:0;display:flex;align-items:center;justify-content:center;pointer-events:none;
    }
    .card{
      pointer-events:auto;
      background:var(--panel);padding:18px;border-radius:12px;box-shadow:0 8px 28px rgba(0,0,0,.25);text-align:center;max-width:340px;
    }
    .hidden{display:none}
    .btn{margin-top:12px;padding:10px 16px;border-radius:10px;border:none;background:var(--accent);color:#fff;font-weight:700;cursor:pointer}
    /* heart */
    .heart{width:110px;height:100px;margin:8px auto;position:relative;transform:scale(0.9);animation:beat 900ms infinite}
    .heart::before,.heart::after{content:"";position:absolute;width:70px;height:110px;background:red;border-radius:50px 50px 0 0;top:0;left:20px}
    .heart::before{transform:rotate(-45deg);transform-origin:0 100%}
    .heart::after{transform:rotate(45deg);transform-origin:100% 100%}
    @keyframes beat{0%,100%{transform:scale(0.95)}50%{transform:scale(1.18)}}
    /* score */
    .score{position:fixed;left:14px;top:14px;background:rgba(255,255,255,0.85);padding:8px 12px;border-radius:8px;border:1px solid rgba(0,0,0,0.06);font-weight:700}
    /* error box */
    #errorBox{position:fixed;left:12px;right:12px;bottom:12px;background:#ffefef;border:1px solid #ff7b7b;padding:10px;border-radius:8px;display:none}
    @media(min-width:720px){canvas{max-width:760px}}
  </style>
</head>
<body>
  <div class="center">
    <canvas id="game"></canvas>
  </div>

  <div class="score" id="score">Puntos: 0</div>

  <!-- derrota -->
  <div class="modal" id="modalLose" style="display:none">
    <div class="card">
      <div id="loseText">...</div>
      <button class="btn" id="btnRetryLose">🔄 Reintentar</button>
    </div>
  </div>

  <!-- victoria -->
  <div class="modal" id="modalWin" style="display:none">
    <div class="card">
      <div class="heart" aria-hidden="true"></div>
      <div style="font-weight:700;margin-top:6px">me gusta tus ojitos</div>
      <div id="finalPoints" style="margin-top:6px;color:#444"></div>
      <button class="btn" id="btnRetryWin">💖 Jugar otra vez</button>
    </div>
  </div>

  <div id="errorBox"></div>

<script>
(function(){
  // Envoltorio para atrapar errores y mostrarlos en pantalla (útil cuando se queda en blanco)
  try {

  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const scoreEl = document.getElementById('score');
  const modalLose = document.getElementById('modalLose');
  const modalWin = document.getElementById('modalWin');
  const loseText = document.getElementById('loseText');
  const finalPoints = document.getElementById('finalPoints');
  const errorBox = document.getElementById('errorBox');

  const mensajes = [
    "Ups, saltaste mal... intenta de nuevo 😔",
    "La roca ganó esta vez, pero eres mi fav al volante JAJA 😼",
    "Perdiste de nuevo, pero fue con estilo ¿sí o k? 😎",
    "¿Otra vez? 😑"
  ];

  // meta y dificultad
  const META = 5000;
  let difficulty = { speed: 220, spawnInterval: 1.2 }; // px/s, segundos
  let deviceRatio = 1;

  // estado
  let player, obstacles, lastTime, accSpawn;
  let running = false;
  let score = 0;
  let rafId = null;

  // ajustar canvas responsive con devicePixelRatio
  function fitCanvas(){
    const maxW = Math.min(window.innerWidth - 24, 760);
    const cssW = Math.max(320, maxW);
    const cssH = Math.round(cssW * 0.56); // relación
    deviceRatio = window.devicePixelRatio || 1;
    canvas.style.width = cssW + 'px';
    canvas.style.height = cssH + 'px';
    canvas.width = Math.floor(cssW * deviceRatio);
    canvas.height = Math.floor(cssH * deviceRatio);
    ctx.setTransform(deviceRatio,0,0,deviceRatio,0,0);
  }
  window.addEventListener('resize', fitCanvas);

  function resetState(){
    fitCanvas();
    const ch = canvas.clientHeight;
    player = {
      x: Math.max(20, canvas.clientWidth * 0.08),
      y: ch - 50,
      w: 56,
      h: 30,
      vy: 0,
      onGround: true,
      jumpVel: -420 // px/s (used with dt)
    };
    obstacles = [];
    lastTime = 0;
    accSpawn = 0;
    running = true;
    score = 0;
    difficulty = { speed: 220, spawnInterval: 1.2 };
    scoreEl.textContent = 'Puntos: 0';
    modalLose.style.display = 'none';
    modalWin.style.display = 'none';
  }

  // spawn obstacle
  function spawnRock(){
    const size = 14 + Math.random() * 18;
    const x = canvas.clientWidth + 20;
    const y = canvas.clientHeight - 36 - size/2;
    obstacles.push({ x, y, size });
  }

  function rectsIntersect(a,b){
    return !(a.x + a.w < b.x || a.x > b.x + b.size || a.y + a.h < b.y || a.y > b.y + b.size);
  }

  // update + draw with time-based physics
  function updateAndDraw(ts){
    if (!running) return;
    if (!lastTime) lastTime = ts;
    const dt = Math.min(0.032, (ts - lastTime) / 1000); // segundos, clamp
    lastTime = ts;

    // background + ground
    const cw = canvas.clientWidth;
    const ch = canvas.clientHeight;
    ctx.clearRect(0,0,cw,ch);
    ctx.fillStyle = '#87ceeb';
    ctx.fillRect(0,0,cw,ch);
    ctx.fillStyle = '#ddd';
    ctx.fillRect(0, ch - 30, cw, 30);

    // player physics (vy in px/s)
    if (!player.onGround){
      player.vy += 1000 * dt; // gravity px/s^2
    }
    player.y += player.vy * dt;
    if (player.y >= ch - 30 - player.h){
      player.y = ch - 30 - player.h;
      player.vy = 0;
      player.onGround = true;
    }

    // spawn logic
    accSpawn += dt;
    if (accSpawn >= difficulty.spawnInterval){
      accSpawn = 0;
      spawnRock();
    }

    // move obstacles
    for (let i = obstacles.length - 1; i >= 0; i--){
      const o = obstacles[i];
      o.x -= difficulty.speed * dt;
      // collision detection (simple)
      const a = { x: player.x, y: player.y, w: player.w, h: player.h };
      if (rectsIntersect(a, o)){
        // lose
        running = false;
        showLose();
        return;
      }
      if (o.x + o.size < -30) obstacles.splice(i,1);
    }

    // difficulty ramp: every 800 points, slight increase
    if (score > 0 && score % 800 === 0){
      difficulty.speed = Math.min(520, difficulty.speed + 12);
      difficulty.spawnInterval = Math.max(0.46, difficulty.spawnInterval - 0.08);
    }

    // draw player (car)
    drawCar();

    // draw obstacles (rocks)
    ctx.fillStyle = 'gray';
    for (const o of obstacles){
      ctx.beginPath();
      ctx.arc(o.x, o.y, o.size, 0, Math.PI*2);
      ctx.fill();
    }

    // update score (points per frame scaled to 60fps)
    score += Math.round(60 * dt);
    if (score > META) score = META;
    scoreEl.textContent = 'Puntos: ' + Math.floor(score);

    // check win
    if (score >= META){
      running = false;
      showWin();
      return;
    }

    rafId = requestAnimationFrame(updateAndDraw);
  }

  function drawCar(){
    ctx.fillStyle = '#ff4d4d';
    ctx.fillRect(player.x, player.y, player.w, player.h);
    ctx.fillStyle = '#b30000';
    ctx.fillRect(player.x + 10, player.y - 12, player.w - 20, 12); // techo
    ctx.fillStyle = '#111';
    const r = Math.max(6, Math.min(12, player.h/1.5));
    ctx.beginPath(); ctx.arc(player.x + 12, player.y + player.h + 4, r, 0, Math.PI*2); ctx.fill();
    ctx.beginPath(); ctx.arc(player.x + player.w - 12, player.y + player.h + 4, r, 0, Math.PI*2); ctx.fill();
  }

  // input
  const keys = { jump:false };
  window.addEventListener('keydown', e => {
    if (e.code === 'Space' || e.code === 'ArrowUp') { e.preventDefault(); doJump(); }
  });
  // click/tap to jump
  canvas.addEventListener('pointerdown', e => { e.preventDefault(); doJump(); });

  function doJump(){
    if (!running) return;
    if (player.onGround){
      player.vy = player.jumpVel; // negative px/s
      player.onGround = false;
    }
  }

  // show lose modal
  function showLose(){
    const idx = Math.floor(Math.random() * mensajes.length);
    loseText.textContent = mensajes[idx];
    modalLose.style.display = 'flex';
    // stop RAF if any
    if (rafId) cancelAnimationFrame(rafId);
  }

  // show win modal
  function showWin(){
    finalPoints.textContent = 'Puntos finales: ' + Math.floor(score);
    modalWin.style.display = 'flex';
    if (rafId) cancelAnimationFrame(rafId);
  }

  // retry handlers
  document.getElementById('btnRetryLose').addEventListener('click', ()=> {
    modalLose.style.display = 'none';
    resetState();
    rafId = requestAnimationFrame(updateAndDraw);
  });
  document.getElementById('btnRetryWin').addEventListener('click', ()=> {
    modalWin.style.display = 'none';
    resetState();
    rafId = requestAnimationFrame(updateAndDraw);
  });

  // simple error display utility
  function showError(msg){
    errorBox.style.display = 'block';
    errorBox.textContent = 'Error: ' + msg;
    console.error(msg);
  }

  // start game
  resetState();
  rafId = requestAnimationFrame(updateAndDraw);

  // safety: catch uncaught errors to show on page (helps cuando queda en blanco)
  window.addEventListener('error', (ev) => {
    showError(ev.message || 'error desconocido');
  });
  window.addEventListener('unhandledrejection', (ev) => {
    showError(ev.reason ? (ev.reason.message || ev.reason) : 'promise rejected');
  });

  } catch(err){
    // si ocurre algo al parsear el script mostramos en pantalla
    const errorBox = document.getElementById('errorBox');
    if (errorBox) { errorBox.style.display = 'block'; errorBox.textContent = 'Error al iniciar: ' + (err && err.message ? err.message : String(err)); }
    console.error(err);
  }
})();
</script>
</body>
</html>

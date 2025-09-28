<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>¡Salta!</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      background: #aee6ff; /* azul pastel */
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
      color: #333;
    }
    canvas {
      border: 3px solid #333;
      background: #aee6ff; /* fondo azul pastel dentro del canvas */
    }
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
    .modal button:hover {
      background: #e63939;
    }
    /* Corazón animado */
    .heart {
      position: relative;
      width: 100px;
      height: 90px;
      margin: 0 auto 20px auto;
      transform: scale(1);
      animation: beat 1s infinite;
    }
    .heart:before,
    .heart:after {
      content: "";
      position: absolute;
      top: 0;
      width: 100px;
      height: 150px;
      border-radius: 50px 50px 0 0;
      background: red;
    }
    .heart:before {
      left: 100px;
      transform: rotate(-45deg);
      transform-origin: 0 100%;
    }
    .heart:after {
      left: 0;
      transform: rotate(45deg);
      transform-origin: 100% 100%;
    }
    @keyframes beat {
      0%, 100% { transform: scale(1); }
      50% { transform: scale(1.2); }
    }
  </style>
</head>
<body>
  <h1>¡Salta!</h1>
  <canvas id="juego" width="600" height="200"></canvas>

  <!-- Modal derrota -->
  <div id="modalDerrota" class="modal">
    <p id="mensajeDerrota"></p>
    <button onclick="reiniciarJuego()">🔄 Reintentar</button>
  </div>

  <!-- Modal victoria -->
  <div id="modalVictoria" class="modal">
    <div class="heart"></div>
    <p>me gusta tus ojitos</p>
    <button onclick="reiniciarJuego()">❤️ Volver a jugar</button>
  </div>

  <script>
    const canvas = document.getElementById("juego");
    const ctx = canvas.getContext("2d");

    let carro, obstaculos, frame, score, gameOver, nubes;

    const mensajes = [
      "Ups, saltaste mal... intenta de nuevo 😔",
      "La roca ganó esta vez, pero eres mi fav al volante JAJA 😼",
      "Perdiste de nuevo, pero fue con estilo ¿sí o k? 😎",
      "¿Otra vez? 😑"
    ];

    function init() {
      carro = { x: 50, y: 150, width: 40, height: 30, dy: 0, jumping: false };
      obstaculos = [];
      nubes = [
        { x: 100, y: 40, r: 20 },
        { x: 250, y: 60, r: 25 },
        { x: 400, y: 30, r: 18 },
        { x: 550, y: 50, r: 22 }
      ];
      frame = 0;
      score = 0;
      gameOver = false;
      document.getElementById("modalDerrota").style.display = "none";
      document.getElementById("modalVictoria").style.display = "none";
      update();
    }

    function drawCarro() {
      ctx.fillStyle = "blue";
      ctx.fillRect(carro.x, carro.y, carro.width, carro.height);
      ctx.fillStyle = "black";
      ctx.fillRect(carro.x + 5, carro.y + carro.height, 10, 10);
      ctx.fillRect(carro.x + 25, carro.y + carro.height, 10, 10);
    }

    function drawObstaculos() {
      ctx.fillStyle = "brown";
      obstaculos.forEach(o => ctx.fillRect(o.x, o.y, o.width, o.height));
    }

    function drawNubes() {
      ctx.fillStyle = "white";
      nubes.forEach(n => {
        ctx.beginPath();
        ctx.arc(n.x, n.y, n.r, 0, Math.PI * 2);
        ctx.arc(n.x + n.r, n.y - 10, n.r * 0.8, 0, Math.PI * 2);
        ctx.arc(n.x + n.r * 2, n.y, n.r, 0, Math.PI * 2);
        ctx.fill();
        n.x -= 0.5; // movimiento lento
        if (n.x < -50) n.x = canvas.width + 50;
      });
    }

    function update() {
      if (gameOver) return;

      frame++;
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // fondo cielo
      ctx.fillStyle = "#aee6ff";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // nubes
      drawNubes();

      // suelo
      ctx.fillStyle = "#ddd";
      ctx.fillRect(0, 180, canvas.width, 20);

      // salto
      carro.y += carro.dy;
      if (carro.y < 150) carro.dy += 1;
      else {
        carro.dy = 0;
        carro.y = 150;
        carro.jumping = false;
      }

      drawCarro();

      // obstáculos con dificultad creciente
      if (frame % Math.max(60, 100 - Math.floor(score / 500)) === 0) {
        obstaculos.push({ x: 600, y: 160, width: 20, height: 20 });
      }

      obstaculos.forEach(o => {
        o.x -= 5 + Math.floor(score / 1000);
        ctx.fillStyle = "brown";
        ctx.fillRect(o.x, o.y, o.width, o.height);

        // colisión
        if (
          carro.x < o.x + o.width &&
          carro.x + carro.width > o.x &&
          carro.y < o.y + o.height &&
          carro.y + carro.height > o.y
        ) {
          mostrarDerrota();
        }
      });

      // limpiar obstáculos viejos
      obstaculos = obstaculos.filter(o => o.x > -20);

      // score dentro del canvas
      ctx.fillStyle = "black";
      ctx.font = "16px Arial";
      ctx.fillText("Puntos: " + score, canvas.width - 120, 30);

      score++;

      // Victoria
      if (score >= 5000) {
        mostrarVictoria();
        return;
      }

      requestAnimationFrame(update);
    }

    function jump() {
      if (!carro.jumping) {
        carro.dy = -15;
        carro.jumping = true;
      }
    }

    function mostrarDerrota() {
      gameOver = true;
      const mensaje = mensajes[Math.floor(Math.random() * mensajes.length)];
      document.getElementById("mensajeDerrota").textContent = mensaje;
      document.getElementById("modalDerrota").style.display = "block";
    }

    function mostrarVictoria() {
      gameOver = true;
      document.getElementById("modalVictoria").style.display = "block";
    }

    function reiniciarJuego() {
      init();
    }

    document.addEventListener("keydown", jump);
    document.addEventListener("click", jump);

    init();
  </script>
</body>
</html>

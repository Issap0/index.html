<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>¡Salta!</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      background: #87ceeb; /* azul cielo */
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
      font-size: 32px;
      color: #333;
    }
    canvas {
      border: 3px solid #333;
      background: #87ceeb;
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
  <canvas id="juego" width="700" height="250"></canvas>

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

    let carro, obstaculos, nubes, frame, score, gameOver;

    const mensajes = [
      "Ups, saltaste mal... intenta de nuevo 😔",
      "La roca ganó esta vez, pero eres mi fav al volante JAJA 😼",
      "Perdiste de nuevo, pero fue con estilo ¿sí o k? 😎",
      "¿Otra vez? 😑"
    ];

    function init() {
      carro = { x: 70, y: 180, width: 50, height: 30, dy: 0, jumping: false };
      obstaculos = [];
      nubes = [
        { x: 100, y: 40, size: 30 },
        { x: 300, y: 60, size: 40 },
        { x: 600, y: 50, size: 25 }
      ];
      frame = 0;
      score = 0;
      gameOver = false;
      document.getElementById("modalDerrota").style.display = "none";
      document.getElementById("modalVictoria").style.display = "none";
      update();
    }

    function drawCarro() {
      // cuerpo rojo
      ctx.fillStyle = "red";
      ctx.fillRect(carro.x, carro.y, carro.width, carro.height);
      // techo
      ctx.fillRect(carro.x + 10, carro.y - 20, 30, 20);
      // llantas
      ctx.fillStyle = "black";
      ctx.beginPath();
      ctx.arc(carro.x + 10, carro.y + carro.height, 10, 0, Math.PI * 2);
      ctx.fill();
      ctx.beginPath();
      ctx.arc(carro.x + 40, carro.y + carro.height, 10, 0, Math.PI * 2);
      ctx.fill();
    }

    function drawObstaculos() {
      ctx.fillStyle = "saddlebrown";
      obstaculos.forEach(o => {
        ctx.beginPath();
        ctx.arc(o.x, o.y, o.size, 0, Math.PI * 2);
        ctx.fill();
      });
    }

    function drawNubes() {
      ctx.fillStyle = "white";
      nubes.forEach(n => {
        ctx.beginPath();
        ctx.arc(n.x, n.y, n.size, 0, Math.PI * 2);
        ctx.arc(n.x + n.size, n.y, n.size * 0.8, 0, Math.PI * 2);
        ctx.arc(n.x + n.size * 2, n.y, n.size, 0, Math.PI * 2);
        ctx.fill();
        n.x -= 1; // mover nubes
        if (n.x < -100) n.x = canvas.width + 100;
      });
    }

    function update() {
      if (gameOver) return;

      frame++;
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // cielo
      ctx.fillStyle = "#87ceeb";
      ctx.fillRect(0, 0, canvas.width, canvas.height);

      // nubes
      drawNubes();

      // carretera
      ctx.fillStyle = "#555";
      ctx.fillRect(0, 210, canvas.width, 40);
      ctx.fillStyle = "white";
      for (let i = 0; i < canvas.width; i += 40) {
        ctx.fillRect(i, 230, 20, 4);
      }

      // salto
      carro.y += carro.dy;
      if (carro.y < 180) carro.dy += 1;
      else {
        carro.dy = 0;
        carro.y = 180;
        carro.jumping = false;
      }

      drawCarro();

      // obstáculos
      if (frame % Math.max(70, 120 - Math.floor(score / 300)) === 0) {
        let size = Math.random() > 0.5 ? 20 : 30;
        obstaculos.push({ x: canvas.width, y: 210, size: size });
      }

      obstaculos.forEach(o => {
        o.x -= 5 + Math.floor(score / 1000);
        ctx.fillStyle = "saddlebrown";
        ctx.beginPath();
        ctx.arc(o.x, o.y, o.size, 0, Math.PI * 2);
        ctx.fill();

        // colisión
        if (
          carro.x < o.x + o.size &&
          carro.x + carro.width > o.x - o.size &&
          carro.y < o.y + o.size &&
          carro.y + carro.height > o.y - o.size
        ) {
          mostrarDerrota();
        }
      });

      obstaculos = obstaculos.filter(o => o.x > -50);

      // score
      score++;
      ctx.fillStyle = "black";
      ctx.font = "16px Arial";
      ctx.fillText("Puntos: " + score, 10, 20);

      // victoria
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

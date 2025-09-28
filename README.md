<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>¡Salta!</title>
  <style>
    body {
      margin: 0;
      background: #87ceeb;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      font-family: Arial, sans-serif;
    }
    canvas {
      border: 3px solid #333;
      background: #dcdcdc;
    }
    #mensajeFinal {
      display: none;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background: #fff;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.3);
      text-align: center;
      font-size: 16px;
      max-width: 320px;
    }
    #mensajeFinal button {
      margin-top: 15px;
      padding: 10px 20px;
      border: none;
      border-radius: 8px;
      background: #ff4d4d;
      color: #fff;
      font-size: 14px;
      cursor: pointer;
    }
    #mensajeFinal button:hover {
      background: #e63939;
    }
  </style>
</head>
<body>
  <canvas id="juego" width="600" height="250"></canvas>
  <div id="mensajeFinal">
    <div id="textoMensaje"></div>
    <button onclick="reiniciarJuego()">🔄 Reintentar</button>
  </div>

  <script>
    const canvas = document.getElementById("juego");
    const ctx = canvas.getContext("2d");

    let carro, obstaculos, frame, score, gameOver;

    const mensajes = [
      "Ups, saltaste tarde, pero igual me agradas demasiado 🫶",
      "La roca ganó esta vez, pero sigues siendo mi fav en el volante 😼",
      "Perdiste de nuevooo, pero fue con estilo, si o k? 😎"
    ];

    function init() {
      carro = { x: 50, y: 200, width: 50, height: 30, dy: 0, jumping: false };
      obstaculos = [];
      frame = 0;
      score = 0;
      gameOver = false;
      document.getElementById("mensajeFinal").style.display = "none";
      update();
    }

    function drawCarro() {
      // Cuerpo
      ctx.fillStyle = "red";
      ctx.fillRect(carro.x, carro.y, carro.width, carro.height);
      // Ventana
      ctx.fillStyle = "white";
      ctx.fillRect(carro.x + 10, carro.y + 5, 20, 15);
      // Ruedas
      ctx.fillStyle = "black";
      ctx.beginPath();
      ctx.arc(carro.x + 10, carro.y + carro.height, 8, 0, Math.PI * 2);
      ctx.fill();
      ctx.beginPath();
      ctx.arc(carro.x + carro.width - 10, carro.y + carro.height, 8, 0, Math.PI * 2);
      ctx.fill();
    }

    function drawObstaculos() {
      ctx.fillStyle = "gray";
      obstaculos.forEach(o => {
        ctx.beginPath();
        ctx.arc(o.x, o.y, o.size, 0, Math.PI * 2);
        ctx.fill();
      });
    }

    function update() {
      if (gameOver) return;

      frame++;
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Suelo
      ctx.fillStyle = "#666";
      ctx.fillRect(0, 230, canvas.width, 20);

      // Movimiento carro
      carro.y += carro.dy;
      if (carro.y < 200) carro.dy += 1;
      else {
        carro.dy = 0;
        carro.y = 200;
        carro.jumping = false;
      }

      drawCarro();

      // Obstáculos (rocas)
      if (frame % 90 === 0) {
        const size = 15 + Math.random() * 15;
        obstaculos.push({ x: 600, y: 230 - size, size });
      }

      obstaculos.forEach(o => {
        o.x -= 5;
        if (
          carro.x < o.x + o.size &&
          carro.x + carro.width > o.x - o.size &&
          carro.y < o.y + o.size &&
          carro.y + carro.height > o.y - o.size
        ) {
          endGame();
        }
      });

      obstaculos = obstaculos.filter(o => o.x > -20);

      drawObstaculos();

      // Puntos
      score++;
      ctx.fillStyle = "#000";
      ctx.font = "16px Arial";
      ctx.fillText("Puntos: " + score, 500, 20);

      requestAnimationFrame(update);
    }

    function jump() {
      if (!carro.jumping) {
        carro.dy = -15;
        carro.jumping = true;
      }
    }

    function endGame() {
      gameOver = true;
      const mensajeAleatorio = mensajes[Math.floor(Math.random() * mensajes.length)];
      document.getElementById("textoMensaje").textContent = mensajeAleatorio;
      document.getElementById("mensajeFinal").style.display = "block";
    }

    function reiniciarJuego() {
      init();
    }

    document.addEventListener("keydown", e => {
      if (e.code === "Space" || e.code === "ArrowUp") jump();
    });
    document.addEventListener("click", jump);

    init();
  </script>
</body>
</html>

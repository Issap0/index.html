<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Hot Wheels Mini Juego</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      background: #f0f0f0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      font-family: Arial, sans-serif;
    }
    canvas {
      border: 3px solid #333;
      background: #fff;
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
  <canvas id="juego" width="600" height="200"></canvas>
  <div id="mensajeFinal">
    me alegra haberte conocido, este detalle es para alguien muy especial c:<br>
    <button onclick="reiniciarJuego()">🔄 Reintentar</button>
  </div>

  <script>
    const canvas = document.getElementById("juego");
    const ctx = canvas.getContext("2d");

    let carro;
    let obstaculos;
    let frame;
    let score;
    let gameOver;

    function init() {
      carro = { x: 50, y: 150, width: 40, height: 30, dy: 0, jumping: false };
      obstaculos = [];
      frame = 0;
      score = 0;
      gameOver = false;
      document.getElementById("mensajeFinal").style.display = "none";
      update();
    }

    function drawCarro() {
      ctx.fillStyle = "red";
      ctx.fillRect(carro.x, carro.y, carro.width, carro.height);
    }

    function drawObstaculos() {
      ctx.fillStyle = "green";
      obstaculos.forEach(o => ctx.fillRect(o.x, o.y, o.width, o.height));
    }

    function update() {
      if (gameOver) return;

      frame++;
      ctx.clearRect(0, 0, canvas.width, canvas.height);

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

      // obstáculos
      if (frame % 100 === 0) {
        obstaculos.push({ x: 600, y: 160, width: 20, height: 20 });
      }

      obstaculos.forEach(o => {
        o.x -= 5;
        ctx.fillStyle = "#000";
        ctx.fillRect(o.x, o.y, o.width, o.height);

        // colisión
        if (
          carro.x < o.x + o.width &&
          carro.x + carro.width > o.x &&
          carro.y < o.y + o.height &&
          carro.y + carro.height > o.y
        ) {
          endGame();
        }
      });

      // limpiar obstáculos viejos
      obstaculos = obstaculos.filter(o => o.x > -20);

      // score
      score++;
      ctx.fillStyle = "#333";
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
      document.getElementById("mensajeFinal").style.display = "block";
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

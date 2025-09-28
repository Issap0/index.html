<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Carrerita 🚗</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background: #222;
      color: white;
      font-family: sans-serif;
      flex-direction: column;
    }

    h1 {
      margin-bottom: 10px;
      font-size: 28px;
      color: #ff4444;
    }

    canvas {
      background: #333;
      border: 3px solid #fff;
    }
  </style>
</head>
<body>
  <h1>Carrerita 🚗</h1>
  <canvas id="gameCanvas" width="400" height="600"></canvas>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    // Carrito 🚗
    const car = {
      x: 180,
      y: 500,
      width: 40,
      height: 40,
      emoji: "🚗"
    };

    // Teclas
    let keys = {};
    document.addEventListener("keydown", (e) => keys[e.key] = true);
    document.addEventListener("keyup", (e) => keys[e.key] = false);

    function drawCar() {
      ctx.font = "35px Arial";
      ctx.fillText(car.emoji, car.x, car.y + car.height);
    }

    function update() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Movimiento
      if (keys["ArrowLeft"] && car.x > 0) {
        car.x -= 5;
      }
      if (keys["ArrowRight"] && car.x < canvas.width - car.width) {
        car.x += 5;
      }

      drawCar();
      requestAnimationFrame(update);
    }

    update();
  </script>
</body>
</html>  

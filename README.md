<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>¡Santa!</title>
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
    #mensajeFinal, #mensajeVictoria {
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
      max-width: 320px;
    }
    #mensajeFinal button, #mensajeVictoria button {
      margin-top: 15px;
      padding: 10px 20px;
      border: none;
      border-radius: 8px;
      background: #ff4d4d;
      color: #fff;
      font-size: 14px;
      cursor: pointer;
    }
    #mensajeFinal button:hover, #mensajeVictoria button:hover {
      background: #e63939;
    }

    /* Corazón animado */
    .heart {
      width: 100px;
      height: 90px;
      position: relative;
      margin: 20px auto;
      animation: beat 1s infinite;
    }
    .heart::before, .heart::after {
      content: "";
      width: 100px;
      height: 90px;
      background: red;
      border-radius: 50px 50px 0 0;
      position: absolute;
      top: 0;
      left: 0;
    }
    .heart::before {
      transform: rotate(-45deg);
      transform-origin: 0 100%;
    }
    .heart::after {
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
  <canvas id="juego" width="600" height="200"></canvas>

  <!-- Ventana de derrota -->
  <div id="mensajeFinal">
    <p id="textoFinal"></p>
    <button onclick="reiniciarJuego()">🔄 Reintentar</button>
  </div>

  <!-- Ventana de victoria -->
  <div id="mensajeVictoria">
    <div class="heart"></div>
    <p>me gusta tus ojitos</p>
    <button onclick="reiniciarJuego()">💖 Jugar otra vez</button>
  </div>

  <script>
    const canvas = document.getElementById("juego");
    const ctx = canvas.getContext("2d");

    let carro;
    let obstaculos;
    let frame;
    let score;
    let gameOver;
    let velocidad = 5;
    let frecuencia = 120;

    const metaPuntos = 5000;

    const mensajes = [
      "Ups, saltaste mal... intenta de nuevo 😔",
      "La roca ganó esta vez, pero eres mi fav al volante JAJA 😼",
      "Perdiste de nuevo, pero fue con estilo ¿sí o k? 😎",
      "¿Otra vez? 😑"
    ];

    function init() {
      carro = { x: 50, y: 150, width: 50, height: 25, dy: 0, jumping: false };
      obstaculos = [];
      frame = 0;
      score = 0;
      gameOver = false;
      velocidad = 5;

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Two-Player Pong</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      background-color: #111827;
      color: #f9fafb;
      font-family: system-ui, -apple-system, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      gap: 16px;
    }

    h1 {
      font-size: 1.75rem;
      letter-spacing: 1px;
    }

    .controls {
      display: flex;
      gap: 32px;
      font-size: 0.95rem;
      color: #9ca3af;
    }

    .controls span {
      color: #f9fafb;
      font-weight: 600;
    }

    canvas {
      background-color: #030712;
      border: 3px solid #374151;
      border-radius: 8px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
    }

    button {
      padding: 10px 20px;
      font-size: 1rem;
      font-weight: 600;
      color: #111827;
      background-color: #38bdf8;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      transition: background-color 0.2s;
    }

    button:hover {
      background-color: #7dd3fc;
    }
  </style>
</head>
<body>
  <h1>Two-Player Pong</h1>
  <div class="controls">
    <div>Player 1: <span>W</span> / <span>S</span></div>
    <div>First to 5 Wins</div>
    <div>Player 2: <span>&uarr;</span> / <span>&darr;</span></div>
  </div>

  <canvas id="gameCanvas" width="800" height="500"></canvas>
  <button id="restartBtn">Restart Game</button>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const restartBtn = document.getElementById("restartBtn");

    const WINNING_SCORE = 5;
    const PADDLE_WIDTH = 12;
    const PADDLE_HEIGHT = 90;
    const PADDLE_SPEED = 6;
    const BALL_RADIUS = 8;

    let gameOver = false;
    let winnerText = "";

    const player1 = {
      x: 20,
      y: canvas.height / 2 - PADDLE_HEIGHT / 2,
      score: 0,
      dy: 0,
    };

    const player2 = {
      x: canvas.width - 20 - PADDLE_WIDTH,
      y: canvas.height / 2 - PADDLE_HEIGHT / 2,
      score: 0,
      dy: 0,
    };

    const ball = {
      x: canvas.width / 2,
      y: canvas.height / 2,
      dx: 5,
      dy: 3,
      speed: 5,
    };

    const keys = {};

    document.addEventListener("keydown", (e) => {
      if (["ArrowUp", "ArrowDown", "Space"].includes(e.code)) {
        e.preventDefault();
      }
      keys[e.key.toLowerCase()] = true;
    });

    document.addEventListener("keyup", (e) => {
      keys[e.key.toLowerCase()] = false;
    });

    restartBtn.addEventListener("click", resetGame);

    function resetBall(direction) {
      ball.x = canvas.width / 2;
      ball.y = canvas.height / 2;
      ball.speed = 5;
      const angle = (Math.random() * Math.PI) / 3 - Math.PI / 6;
      ball.dx = direction * ball.speed * Math.cos(angle);
      ball.dy = ball.speed * Math.sin(angle);
    }

    function resetGame() {
      player1.score = 0;
      player2.score = 0;
      player1.y = canvas.height / 2 - PADDLE_HEIGHT / 2;
      player2.y = canvas.height / 2 - PADDLE_HEIGHT / 2;
      gameOver = false;
      winnerText = "";
      resetBall(Math.random() < 0.5 ? 1 : -1);
    }

    function update() {
      if (gameOver) return;

      if (keys["w"]) player1.y -= PADDLE_SPEED;
      if (keys["s"]) player1.y += PADDLE_SPEED;

      if (keys["arrowup"]) player2.y -= PADDLE_SPEED;
      if (keys["arrowdown"]) player2.y += PADDLE_SPEED;

      player1.y = Math.max(0, Math.min(canvas.height - PADDLE_HEIGHT, player1.y));
      player2.y = Math.max(0, Math.min(canvas.height - PADDLE_HEIGHT, player2.y));

      ball.x += ball.dx;
      ball.y += ball.dy;

      if (ball.y - BALL_RADIUS <= 0 || ball.y + BALL_RADIUS >= canvas.height) {
        ball.dy *= -1;
      }

      function checkPaddleCollision(paddle) {
        return (
          ball.x - BALL_RADIUS < paddle.x + PADDLE_WIDTH &&
          ball.x + BALL_RADIUS > paddle.x &&
          ball.y + BALL_RADIUS > paddle.y &&
          ball.y - BALL_RADIUS < paddle.y + PADDLE_HEIGHT
        );
      }

      if (ball.dx < 0 && checkPaddleCollision(player1)) {
        const hitPoint = (ball.y - (player1.y + PADDLE_HEIGHT / 2)) / (PADDLE_HEIGHT / 2);
        const bounceAngle = hitPoint * (Math.PI / 4);
        ball.speed = Math.min(ball.speed + 0.3, 12);
        ball.dx = ball.speed * Math.cos(bounceAngle);
        ball.dy = ball.speed * Math.sin(bounceAngle);
      }

      if (ball.dx > 0 && checkPaddleCollision(player2)) {
        const hitPoint = (ball.y - (player2.y + PADDLE_HEIGHT / 2)) / (PADDLE_HEIGHT / 2);
        const bounceAngle = hitPoint * (Math.PI / 4);
        ball.speed = Math.min(ball.speed + 0.3, 12);
        ball.dx = -ball.speed * Math.cos(bounceAngle);
        ball.dy = ball.speed * Math.sin(bounceAngle);
      }

      if (ball.x + BALL_RADIUS < 0) {
        player2.score++;
        checkWin();
        if (!gameOver) resetBall(-1);
      } else if (ball.x - BALL_RADIUS > canvas.width) {
        player1.score++;
        checkWin();
        if (!gameOver) resetBall(1);
      }
    }

    function checkWin() {
      if (player1.score >= WINNING_SCORE) {
        gameOver = true;
        winnerText = "Player 1 Wins!";
      } else if (player2.score >= WINNING_SCORE) {
        gameOver = true;
        winnerText = "Player 2 Wins!";
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      ctx.setLineDash([10, 10]);
      ctx.strokeStyle = "#374151";
      ctx.lineWidth = 3;
      ctx.beginPath();
      ctx.moveTo(canvas.width / 2, 0);
      ctx.lineTo(canvas.width / 2, canvas.height);
      ctx.stroke();
      ctx.setLineDash([]);

      ctx.fillStyle = "#4b5563";
      ctx.font = "bold 48px system-ui, sans-serif";
      ctx.textAlign = "center";
      ctx.fillText(player1.score, canvas.width / 4, 70);
      ctx.fillText(player2.score, (canvas.width * 3) / 4, 70);

      ctx.fillStyle = "#38bdf8";
      ctx.fillRect(player1.x, player1.y, PADDLE_WIDTH, PADDLE_HEIGHT);

      ctx.fillStyle = "#f472b6";
      ctx.fillRect(player2.x, player2.y, PADDLE_WIDTH, PADDLE_HEIGHT);

      ctx.fillStyle = "#f9fafb";
      ctx.beginPath();
      ctx.arc(ball.x, ball.y, BALL_RADIUS, 0, Math.PI * 2);
      ctx.fill();

      if (gameOver) {
        ctx.fillStyle = "rgba(3, 7, 18, 0.75)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "#f9fafb";
        ctx.font = "bold 42px system-ui, sans-serif";
        ctx.fillText(winnerText, canvas.width / 2, canvas.height / 2 - 10);

        ctx.fillStyle = "#9ca3af";
        ctx.font = "18px system-ui, sans-serif";
        ctx.fillText("Click 'Restart Game' to play again", canvas.width / 2, canvas.height / 2 + 30);
      }
    }

    function gameLoop() {
      update();
      draw();
      requestAnimationFrame(gameLoop);
    }

    resetBall(1);
    gameLoop();
  </script>
</body>
</html>

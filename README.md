!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Tangkap Bintang</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
    }
    body {
      background: radial-gradient(circle at top, #1e293b, #0f172a);
      color: #f8fafc;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      overflow: hidden;
    }
    .hud {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      max-width: 480px;
      padding: 12px 16px;
      background: rgba(30, 41, 59, 0.8);
      border: 1px solid rgba(148, 163, 184, 0.2);
      border-bottom: none;
      border-radius: 12px 12px 0 0;
      font-weight: bold;
      font-size: 16px;
    }
    .game-container {
      position: relative;
      width: 100%;
      max-width: 480px;
      height: 540px;
      background: #020617;
      border: 1px solid rgba(148, 163, 184, 0.2);
      border-radius: 0 0 12px 12px;
      overflow: hidden;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
    }
    canvas {
      display: block;
      width: 100%;
      height: 100%;
    }
    .overlay {
      position: absolute;
      inset: 0;
      background: rgba(2, 6, 23, 0.85);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      text-align: center;
      padding: 24px;
      gap: 16px;
    }
    .hidden {
      display: none !important;
    }
    h1 {
      font-size: 32px;
      color: #facc15;
    }
    p {
      color: #cbd5e1;
      font-size: 15px;
      line-height: 1.5;
    }
    button {
      background: #facc15;
      color: #0f172a;
      border: none;
      padding: 12px 28px;
      font-size: 16px;
      font-weight: bold;
      border-radius: 8px;
      cursor: pointer;
      transition: transform 0.1s, background 0.2s;
    }
    button:hover {
      background: #fde047;
      transform: scale(1.04);
    }
    button:active {
      transform: scale(0.98);
    }
  </style>
</head>
<body>
  <div class="hud">
    <div>Skor: <span id="score">0</span></div>
    <div>Rekor: <span id="highScore">0</span></div>
    <div>Nyawa: <span id="lives">❤️❤️❤️</span></div>
  </div>

  <div class="game-container">
    <canvas id="gameCanvas" width="480" height="540"></canvas>

    <div id="startScreen" class="overlay">
      <h1>⭐ Tangkap Bintang</h1>
      <p>Gerakkan keranjang menggunakan tombol panah <b>Kiri / Kanan</b>, mouse, atau sentuhan layar.<br><br>
      ⭐ Bintang: +10 poin<br>
      💎 Berlian: +30 poin<br>
      ☄️ Meteor: -1 nyawa</p>
      <button id="startBtn">Mulai Bermain</button>
    </div>

    <div id="gameOverScreen" class="overlay hidden">
      <h1>Permainan Selesai!</h1>
      <p>Skor Akhir Kamu: <b id="finalScore" style="color:#facc15; font-size: 20px;">0</b></p>
      <button id="restartBtn">Main Lagi</button>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const highScoreEl = document.getElementById('highScore');
    const livesEl = document.getElementById('lives');
    const startScreen = document.getElementById('startScreen');
    const gameOverScreen = document.getElementById('gameOverScreen');
    const finalScoreEl = document.getElementById('finalScore');

    let score = 0;
    let highScore = Number(localStorage.getItem('starCatcherHigh') || 0);
    let lives = 3;
    let isPlaying = false;
    let items = [];
    let spawnTimer = 0;
    let animationId;

    highScoreEl.textContent = highScore;

    const player = {
      width: 84,
      height: 18,
      x: canvas.width / 2 - 42,
      y: canvas.height - 36,
      speed: 7,
      dx: 0
    };

    const keys = { left: false, right: false };

    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = true;
      if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = true;
    });

    document.addEventListener('keyup', (e) => {
      if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = false;
      if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = false;
    });

    function movePointer(clientX) {
      if (!isPlaying) return;
      const rect = canvas.getBoundingClientRect();
      const scaleX = canvas.width / rect.width;
      const pointerX = (clientX - rect.left) * scaleX;
      player.x = Math.max(0, Math.min(canvas.width - player.width, pointerX - player.width / 2));
    }

    canvas.addEventListener('mousemove', (e) => movePointer(e.clientX));
    canvas.addEventListener('touchmove', (e) => {
      e.preventDefault();
      movePointer(e.touches[0].clientX);
    }, { passive: false });

    function spawnItem() {
      const rand = Math.random();
      let type = 'star';
      let symbol = '⭐';
      let points = 10;
      let speed = 2.8 + Math.random() * 2 + score * 0.005;

      if (rand > 0.85) {
        type = 'diamond';
        symbol = '💎';
        points = 30;
        speed += 1.2;
      } else if (rand > 0.62) {
        type = 'meteor';
        symbol = '☄️';
        points = 0;
        speed += 0.8;
      }

      items.push({
        x: Math.random() * (canvas.width - 32) + 16,
        y: -24,
        radius: 16,
        type,
        symbol,
        points,
        speed
      });
    }

    function updateHUD() {
      scoreEl.textContent = score;
      livesEl.textContent = '❤️'.repeat(Math.max(0, lives));
      if (score > highScore) {
        highScore = score;
        highScoreEl.textContent = highScore;
        localStorage.setItem('starCatcherHigh', highScore);
      }
    }

    function update() {
      if (keys.left) player.x -= player.speed;
      if (keys.right) player.x += player.speed;
      player.x = Math.max(0, Math.min(canvas.width - player.width, player.x));

      spawnTimer++;
      const spawnInterval = Math.max(22, 55 - Math.floor(score / 25));
      if (spawnTimer >= spawnInterval) {
        spawnItem();
        spawnTimer = 0;
      }

      for (let i = items.length - 1; i >= 0; i--) {
        const item = items[i];
        item.y += item.speed;

        const hitX = item.x + item.radius > player.x && item.x - item.radius < player.x + player.width;
        const hitY = item.y + item.radius > player.y && item.y - item.radius < player.y + player.height;

        if (hitX && hitY) {
          if (item.type === 'meteor') {
            lives--;
          } else {
            score += item.points;
          }
          updateHUD();
          items.splice(i, 1);

          if (lives <= 0) {
            endGame();
            return;
          }
          continue;
        }

        if (item.y - item.radius > canvas.height) {
          items.splice(i, 1);
        }
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Gambar keranjang pemain
      ctx.fillStyle = '#38bdf8';
      ctx.beginPath();
      ctx.roundRect(player.x, player.y, player.width, player.height, 9);
      ctx.fill();

      // Gambar objek jatuh
      ctx.font = '26px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      for (const item of items) {
        ctx.fillText(item.symbol, item.x, item.y);
      }
    }

    function gameLoop() {
      if (!isPlaying) return;
      update();
      draw();
      animationId = requestAnimationFrame(gameLoop);
    }

    function startGame() {
      score = 0;
      lives = 3;
      items = [];
      spawnTimer = 0;
      player.x = canvas.width / 2 - player.width / 2;
      isPlaying = true;
      updateHUD();
      startScreen.classList.add('hidden');
      gameOverScreen.classList.add('hidden');
      gameLoop();
    }

    function endGame() {
      isPlaying = false;
      cancelAnimationFrame(animationId);
      finalScoreEl.textContent = score;
      gameOverScreen.classList.remove('hidden');
    }

    document.getElementById('startBtn').addEventListener('click', startGame);
    document.getElementById('restartBtn').addEventListener('click', startGame);
  </script>
</body>
</html>
</head>
<body>
    
</body>
</html>!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Tangkap Bintang</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
    }
    body {
      background: radial-gradient(circle at top, #1e293b, #0f172a);
      color: #f8fafc;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      overflow: hidden;
    }
    .hud {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      max-width: 480px;
      padding: 12px 16px;
      background: rgba(30, 41, 59, 0.8);
      border: 1px solid rgba(148, 163, 184, 0.2);
      border-bottom: none;
      border-radius: 12px 12px 0 0;
      font-weight: bold;
      font-size: 16px;
    }
    .game-container {
      position: relative;
      width: 100%;
      max-width: 480px;
      height: 540px;
      background: #020617;
      border: 1px solid rgba(148, 163, 184, 0.2);
      border-radius: 0 0 12px 12px;
      overflow: hidden;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
    }
    canvas {
      display: block;
      width: 100%;
      height: 100%;
    }
    .overlay {
      position: absolute;
      inset: 0;
      background: rgba(2, 6, 23, 0.85);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      text-align: center;
      padding: 24px;
      gap: 16px;
    }
    .hidden {
      display: none !important;
    }
    h1 {
      font-size: 32px;
      color: #facc15;
    }
    p {
      color: #cbd5e1;
      font-size: 15px;
      line-height: 1.5;
    }
    button {
      background: #facc15;
      color: #0f172a;
      border: none;
      padding: 12px 28px;
      font-size: 16px;
      font-weight: bold;
      border-radius: 8px;
      cursor: pointer;
      transition: transform 0.1s, background 0.2s;
    }
    button:hover {
      background: #fde047;
      transform: scale(1.04);
    }
    button:active {
      transform: scale(0.98);
    }
  </style>
</head>
<body>
  <div class="hud">
    <div>Skor: <span id="score">0</span></div>
    <div>Rekor: <span id="highScore">0</span></div>
    <div>Nyawa: <span id="lives">❤️❤️❤️</span></div>
  </div>

  <div class="game-container">
    <canvas id="gameCanvas" width="480" height="540"></canvas>

    <div id="startScreen" class="overlay">
      <h1>⭐ Tangkap Bintang</h1>
      <p>Gerakkan keranjang menggunakan tombol panah <b>Kiri / Kanan</b>, mouse, atau sentuhan layar.<br><br>
      ⭐ Bintang: +10 poin<br>
      💎 Berlian: +30 poin<br>
      ☄️ Meteor: -1 nyawa</p>
      <button id="startBtn">Mulai Bermain</button>
    </div>

    <div id="gameOverScreen" class="overlay hidden">
      <h1>Permainan Selesai!</h1>
      <p>Skor Akhir Kamu: <b id="finalScore" style="color:#facc15; font-size: 20px;">0</b></p>
      <button id="restartBtn">Main Lagi</button>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const highScoreEl = document.getElementById('highScore');
    const livesEl = document.getElementById('lives');
    const startScreen = document.getElementById('startScreen');
    const gameOverScreen = document.getElementById('gameOverScreen');
    const finalScoreEl = document.getElementById('finalScore');

    let score = 0;
    let highScore = Number(localStorage.getItem('starCatcherHigh') || 0);
    let lives = 3;
    let isPlaying = false;
    let items = [];
    let spawnTimer = 0;
    let animationId;

    highScoreEl.textContent = highScore;

    const player = {
      width: 84,
      height: 18,
      x: canvas.width / 2 - 42,
      y: canvas.height - 36,
      speed: 7,
      dx: 0
    };

    const keys = { left: false, right: false };

    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = true;
      if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = true;
    });

    document.addEventListener('keyup', (e) => {
      if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = false;
      if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = false;
    });

    function movePointer(clientX) {
      if (!isPlaying) return;
      const rect = canvas.getBoundingClientRect();
      const scaleX = canvas.width / rect.width;
      const pointerX = (clientX - rect.left) * scaleX;
      player.x = Math.max(0, Math.min(canvas.width - player.width, pointerX - player.width / 2));
    }

    canvas.addEventListener('mousemove', (e) => movePointer(e.clientX));
    canvas.addEventListener('touchmove', (e) => {
      e.preventDefault();
      movePointer(e.touches[0].clientX);
    }, { passive: false });

    function spawnItem() {
      const rand = Math.random();
      let type = 'star';
      let symbol = '⭐';
      let points = 10;
      let speed = 2.8 + Math.random() * 2 + score * 0.005;

      if (rand > 0.85) {
        type = 'diamond';
        symbol = '💎';
        points = 30;
        speed += 1.2;
      } else if (rand > 0.62) {
        type = 'meteor';
        symbol = '☄️';
        points = 0;
        speed += 0.8;
      }

      items.push({
        x: Math.random() * (canvas.width - 32) + 16,
        y: -24,
        radius: 16,
        type,
        symbol,
        points,
        speed
      });
    }

    function updateHUD() {
      scoreEl.textContent = score;
      livesEl.textContent = '❤️'.repeat(Math.max(0, lives));
      if (score > highScore) {
        highScore = score;
        highScoreEl.textContent = highScore;
        localStorage.setItem('starCatcherHigh', highScore);
      }
    }

    function update() {
      if (keys.left) player.x -= player.speed;
      if (keys.right) player.x += player.speed;
      player.x = Math.max(0, Math.min(canvas.width - player.width, player.x));

      spawnTimer++;
      const spawnInterval = Math.max(22, 55 - Math.floor(score / 25));
      if (spawnTimer >= spawnInterval) {
        spawnItem();
        spawnTimer = 0;
      }

      for (let i = items.length - 1; i >= 0; i--) {
        const item = items[i];
        item.y += item.speed;

        const hitX = item.x + item.radius > player.x && item.x - item.radius < player.x + player.width;
        const hitY = item.y + item.radius > player.y && item.y - item.radius < player.y + player.height;

        if (hitX && hitY) {
          if (item.type === 'meteor') {
            lives--;
          } else {
            score += item.points;
          }
          updateHUD();
          items.splice(i, 1);

          if (lives <= 0) {
            endGame();
            return;
          }
          continue;
        }

        if (item.y - item.radius > canvas.height) {
          items.splice(i, 1);
        }
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Gambar keranjang pemain
      ctx.fillStyle = '#38bdf8';
      ctx.beginPath();
      ctx.roundRect(player.x, player.y, player.width, player.height, 9);
      ctx.fill();

      // Gambar objek jatuh
      ctx.font = '26px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      for (const item of items) {
        ctx.fillText(item.symbol, item.x, item.y);
      }
    }

    function gameLoop() {
      if (!isPlaying) return;
      update();
      draw();
      animationId = requestAnimationFrame(gameLoop);
    }

    function startGame() {
      score = 0;
      lives = 3;
      items = [];
      spawnTimer = 0;
      player.x = canvas.width / 2 - player.width / 2;
      isPlaying = true;
      updateHUD();
      startScreen.classList.add('hidden');
      gameOverScreen.classList.add('hidden');
      gameLoop();
    }

    function endGame() {
      isPlaying = false;
      cancelAnimationFrame(animationId);
      finalScoreEl.textContent = score;
      gameOverScreen.classList.remove('hidden');
    }

    document.getElementById('startBtn').addEventListener('click', startGame);
    document.getElementById('restartBtn').addEventListener('click', startGame);
  </script>
</body>
</html>
</head>
<body>
    
</body>
</html>!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Tangkap Bintang</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      user-select: none;
    }
    body {
      background: radial-gradient(circle at top, #1e293b, #0f172a);
      color: #f8fafc;
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      overflow: hidden;
    }
    .hud {
      display: flex;
      justify-content: space-between;
      align-items: center;
      width: 100%;
      max-width: 480px;
      padding: 12px 16px;
      background: rgba(30, 41, 59, 0.8);
      border: 1px solid rgba(148, 163, 184, 0.2);
      border-bottom: none;
      border-radius: 12px 12px 0 0;
      font-weight: bold;
      font-size: 16px;
    }
    .game-container {
      position: relative;
      width: 100%;
      max-width: 480px;
      height: 540px;
      background: #020617;
      border: 1px solid rgba(148, 163, 184, 0.2);
      border-radius: 0 0 12px 12px;
      overflow: hidden;
      box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
    }
    canvas {
      display: block;
      width: 100%;
      height: 100%;
    }
    .overlay {
      position: absolute;
      inset: 0;
      background: rgba(2, 6, 23, 0.85);
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      text-align: center;
      padding: 24px;
      gap: 16px;
    }
    .hidden {
      display: none !important;
    }
    h1 {
      font-size: 32px;
      color: #facc15;
    }
    p {
      color: #cbd5e1;
      font-size: 15px;
      line-height: 1.5;
    }
    button {
      background: #facc15;
      color: #0f172a;
      border: none;
      padding: 12px 28px;
      font-size: 16px;
      font-weight: bold;
      border-radius: 8px;
      cursor: pointer;
      transition: transform 0.1s, background 0.2s;
    }
    button:hover {
      background: #fde047;
      transform: scale(1.04);
    }
    button:active {
      transform: scale(0.98);
    }
  </style>
</head>
<body>
  <div class="hud">
    <div>Skor: <span id="score">0</span></div>
    <div>Rekor: <span id="highScore">0</span></div>
    <div>Nyawa: <span id="lives">❤️❤️❤️</span></div>
  </div>

  <div class="game-container">
    <canvas id="gameCanvas" width="480" height="540"></canvas>

    <div id="startScreen" class="overlay">
      <h1>⭐ Tangkap Bintang</h1>
      <p>Gerakkan keranjang menggunakan tombol panah <b>Kiri / Kanan</b>, mouse, atau sentuhan layar.<br><br>
      ⭐ Bintang: +10 poin<br>
      💎 Berlian: +30 poin<br>
      ☄️ Meteor: -1 nyawa</p>
      <button id="startBtn">Mulai Bermain</button>
    </div>

    <div id="gameOverScreen" class="overlay hidden">
      <h1>Permainan Selesai!</h1>
      <p>Skor Akhir Kamu: <b id="finalScore" style="color:#facc15; font-size: 20px;">0</b></p>
      <button id="restartBtn">Main Lagi</button>
    </div>
  </div>

  <script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const highScoreEl = document.getElementById('highScore');
    const livesEl = document.getElementById('lives');
    const startScreen = document.getElementById('startScreen');
    const gameOverScreen = document.getElementById('gameOverScreen');
    const finalScoreEl = document.getElementById('finalScore');

    let score = 0;
    let highScore = Number(localStorage.getItem('starCatcherHigh') || 0);
    let lives = 3;
    let isPlaying = false;
    let items = [];
    let spawnTimer = 0;
    let animationId;

    highScoreEl.textContent = highScore;

    const player = {
      width: 84,
      height: 18,
      x: canvas.width / 2 - 42,
      y: canvas.height - 36,
      speed: 7,
      dx: 0
    };

    const keys = { left: false, right: false };

    document.addEventListener('keydown', (e) => {
      if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = true;
      if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = true;
    });

    document.addEventListener('keyup', (e) => {
      if (e.key === 'ArrowLeft' || e.key === 'a' || e.key === 'A') keys.left = false;
      if (e.key === 'ArrowRight' || e.key === 'd' || e.key === 'D') keys.right = false;
    });

    function movePointer(clientX) {
      if (!isPlaying) return;
      const rect = canvas.getBoundingClientRect();
      const scaleX = canvas.width / rect.width;
      const pointerX = (clientX - rect.left) * scaleX;
      player.x = Math.max(0, Math.min(canvas.width - player.width, pointerX - player.width / 2));
    }

    canvas.addEventListener('mousemove', (e) => movePointer(e.clientX));
    canvas.addEventListener('touchmove', (e) => {
      e.preventDefault();
      movePointer(e.touches[0].clientX);
    }, { passive: false });

    function spawnItem() {
      const rand = Math.random();
      let type = 'star';
      let symbol = '⭐';
      let points = 10;
      let speed = 2.8 + Math.random() * 2 + score * 0.005;

      if (rand > 0.85) {
        type = 'diamond';
        symbol = '💎';
        points = 30;
        speed += 1.2;
      } else if (rand > 0.62) {
        type = 'meteor';
        symbol = '☄️';
        points = 0;
        speed += 0.8;
      }

      items.push({
        x: Math.random() * (canvas.width - 32) + 16,
        y: -24,
        radius: 16,
        type,
        symbol,
        points,
        speed
      });
    }

    function updateHUD() {
      scoreEl.textContent = score;
      livesEl.textContent = '❤️'.repeat(Math.max(0, lives));
      if (score > highScore) {
        highScore = score;
        highScoreEl.textContent = highScore;
        localStorage.setItem('starCatcherHigh', highScore);
      }
    }

    function update() {
      if (keys.left) player.x -= player.speed;
      if (keys.right) player.x += player.speed;
      player.x = Math.max(0, Math.min(canvas.width - player.width, player.x));

      spawnTimer++;
      const spawnInterval = Math.max(22, 55 - Math.floor(score / 25));
      if (spawnTimer >= spawnInterval) {
        spawnItem();
        spawnTimer = 0;
      }

      for (let i = items.length - 1; i >= 0; i--) {
        const item = items[i];
        item.y += item.speed;

        const hitX = item.x + item.radius > player.x && item.x - item.radius < player.x + player.width;
        const hitY = item.y + item.radius > player.y && item.y - item.radius < player.y + player.height;

        if (hitX && hitY) {
          if (item.type === 'meteor') {
            lives--;
          } else {
            score += item.points;
          }
          updateHUD();
          items.splice(i, 1);

          if (lives <= 0) {
            endGame();
            return;
          }
          continue;
        }

        if (item.y - item.radius > canvas.height) {
          items.splice(i, 1);
        }
      }
    }

    function draw() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      // Gambar keranjang pemain
      ctx.fillStyle = '#38bdf8';
      ctx.beginPath();
      ctx.roundRect(player.x, player.y, player.width, player.height, 9);
      ctx.fill();

      // Gambar objek jatuh
      ctx.font = '26px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      for (const item of items) {
        ctx.fillText(item.symbol, item.x, item.y);
      }
    }

    function gameLoop() {
      if (!isPlaying) return;
      update();
      draw();
      animationId = requestAnimationFrame(gameLoop);
    }

    function startGame() {
      score = 0;
      lives = 3;
      items = [];
      spawnTimer = 0;
      player.x = canvas.width / 2 - player.width / 2;
      isPlaying = true;
      updateHUD();
      startScreen.classList.add('hidden');
      gameOverScreen.classList.add('hidden');
      gameLoop();
    }

    function endGame() {
      isPlaying = false;
      cancelAnimationFrame(animationId);
      finalScoreEl.textContent = score;
      gameOverScreen.classList.remove('hidden');
    }

    document.getElementById('startBtn').addEventListener('click', startGame);
    document.getElementById('restartBtn').addEventListener('click', startGame);
  </script>
</body>
</html>
</head>
<body>
    
</body>
</html>

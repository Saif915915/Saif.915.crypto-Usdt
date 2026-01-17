<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>موقع تعدين – لعبة</title>

  <style>
    body {
      font-family: Arial, sans-serif;
      direction: rtl;
      margin: 0;
      padding: 0;

      /* خلفية ستايل Red Dead */
      background:
        linear-gradient(rgba(0,0,0,0.55), rgba(0,0,0,0.55)),
        radial-gradient(circle at bottom, #ff8c00, #b22222, #2b1b0e);

      background-attachment: fixed;
      min-height: 100vh;
    }

    header {
      background: rgba(20, 40, 80, 0.85);
      color: white;
      padding: 20px;
      text-align: center;
      backdrop-filter: blur(6px);
    }

    nav ul {
      list-style: none;
      padding: 0;
      display: flex;
      justify-content: center;
      gap: 20px;
    }

    nav a {
      color: white;
      text-decoration: none;
      font-weight: bold;
    }

    main {
      padding: 20px;
      max-width: 900px;
      margin: auto;
    }

    .dashboard, .game, .info {
      background: rgba(255, 255, 255, 0.88);
      padding: 20px;
      border-radius: 15px;
      margin-bottom: 20px;
      backdrop-filter: blur(8px);
      box-shadow: 0 10px 30px rgba(0,0,0,0.4);
    }

    h2 {
      margin-top: 0;
      text-align: center;
    }

    button {
      padding: 12px 22px;
      margin: 10px;
      cursor: pointer;
      border: none;
      border-radius: 8px;
      background-color: #1e90ff;
      color: white;
      font-size: 16px;
    }

    button:hover {
      background-color: #0f6fc5;
    }

    .game {
      background: rgba(255, 230, 150, 0.9);
      text-align: center;
    }

    #mineButton {
      background-color: #c45a00;
      font-size: 18px;
      padding: 15px 35px;
    }

    #mineButton:hover {
      background-color: #9e4500;
    }

    footer {
      background: rgba(0,0,0,0.6);
      color: white;
      text-align: center;
      padding: 15px;
      margin-top: 30px;
    }
  </style>
</head>

<body>

<header>
  <h1>موقع التعدين</h1>
  <nav>
    <ul>
      <li><a href="#">الرئيسية</a></li>
      <li><a href="#">لوحة التحكم</a></li>
      <li><a href="#">اللعبة</a></li>
    </ul>
  </nav>
</header>

<main>

  <!-- لوحة التحكم -->
  <section class="dashboard">
    <h2>لوحة التحكم</h2>
    <p>قوة التعدين: <strong><span id="hashRate">0</span> H/s</strong></p>
    <p>العملات المكتسبة: <strong><span id="coins">0</span></strong></p>
    <button id="startMining">بدء التعدين</button>
    <button id="stopMining">إيقاف التعدين</button>
  </section>

  <!-- معلومات -->
  <section class="info">
    <h2>كيف يعمل الموقع</h2>
    <p>
      هذا نموذج لعبة تعدين تفاعلية.  
      التعدين هنا تعليمي وتجريبي على شكل لعبة.
    </p>
  </section>

  <!-- لعبة التعدين -->
  <section class="game">
    <h2>🎮 لعبة التعدين</h2>
    <p>العملات المكتسبة: <strong><span id="gameCoins">0</span></strong></p>
    <button id="mineButton">⛏ اضغط للتعدين</button>
  </section>

</main>

<footer>
  حقوق النشر © 2026
</footer>

<script>
  /* التعدين التلقائي */
  let mining = false;
  let hashRate = 0;
  let coins = 0;
  let interval;

  document.getElementById("startMining").addEventListener("click", () => {
    if (!mining) {
      mining = true;
      interval = setInterval(() => {
        hashRate = Math.floor(Math.random() * 150) + 50;
        coins += 0.02;
        document.getElementById("hashRate").textContent = hashRate;
        document.getElementById("coins").textContent = coins.toFixed(2);
      }, 1000);
    }
  });

  document.getElementById("stopMining").addEventListener("click", () => {
    mining = false;
    clearInterval(interval);
  });

  /* لعبة التعدين */
  let gameCoins = 0;
  document.getElementById("mineButton").addEventListener("click", () => {
    gameCoins += 0.1;
    document.getElementById("gameCoins").textContent = gameCoins.toFixed(2);
  });
</script>

</body>
</html>

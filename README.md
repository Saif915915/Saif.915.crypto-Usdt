<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>لعبة تعدين USDT</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<style>
body {
  font-family: Arial, sans-serif;
  direction: rtl;
  margin: 0;
  padding: 0;
  min-height: 100vh;
  background:
    linear-gradient(rgba(0,0,0,.55), rgba(0,0,0,.55)),
    url('https://i.imgur.com/qyQ1S5r.jpg') no-repeat center center fixed;
  background-size: cover;
  color: #fff;
}

header {
  background: rgba(20,40,80,.85);
  text-align: center;
  padding: 25px 10px;
  font-size: 24px;
  font-weight: bold;
}

main {
  max-width: 900px;
  margin: 20px auto;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.box {
  background: rgba(0,0,0,.6);
  padding: 20px;
  border-radius: 15px;
  box-shadow: 0 8px 25px rgba(0,0,0,.5);
}

h2 {
  margin-top: 0;
  color: #ffcc00;
  font-size: 20px;
}

button {
  padding: 12px 25px;
  margin: 6px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  background: #1e90ff;
  color: white;
  font-weight: bold;
  transition: 0.2s;
}

button:hover {
  background: #0f6fc5;
}

#start, #stop {
  width: 100px;
}

.mine-shaft {
  width: 200px;
  height: 200px;
  margin: 20px auto;
  background: linear-gradient(#6b4f3d, #3a261a);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: bold;
  color: #ffd700;
  cursor: pointer;
  box-shadow: inset 0 0 20px rgba(0,0,0,.5), 0 6px 15px rgba(0,0,0,.3);
  transition: transform 0.1s;
  position: relative;
}

.mine-shaft span {
  transition: transform 0.1s;
}

.mine-shaft:active span {
  transform: translateY(5px) rotate(-5deg);
}

/* تأثير رسومي "حفر" */
.hole {
  width: 40px;
  height: 20px;
  background: #3a261a;
  border-radius: 50%;
  position: absolute;
  bottom: 20px;
  animation: fall 0.3s ease-out forwards;
}

@keyframes fall {
  0% { transform: scale(0.5) translateY(0); opacity: 0.7; }
  50% { transform: scale(1.2) translateY(10px); opacity: 1; }
  100% { transform: scale(1) translateY(30px); opacity: 0; }
}

footer {
  text-align: center;
  padding: 15px;
  background: rgba(0,0,0,.7);
  font-size: 12px;
  color: #ccc;
}

.warning {
  font-size: 13px;
  color: #ff9999;
  text-align: center;
  margin-top: 10px;
}

.level-bar-container {
  background:#444;
  border-radius:8px;
  height:20px;
  width:100%;
  margin-top:10px;
}

#levelBar {
  background:#1e90ff;
  height:100%;
  width:0%;
  border-radius:8px;
}

@keyframes pop {
  0% { transform: scale(0.5); opacity: 0; }
  50% { transform: scale(1.2); opacity: 1; }
  100% { transform: scale(1); }
}

#levelModal div {
  animation: pop 0.3s ease-out;
}
</style>
</head>

<body>

<header>
🎮 لعبة تعدين USDT التعليمية
</header>

<main>

<div class="box">
  <h2>🔐 المحفظة</h2>
  <p>العنوان: <strong><span id="wallet">غير متصل</span></strong></p>
  <button id="connect">ربط MetaMask</button>
</div>

<div class="box">
  <h2>📊 مستواك</h2>
  <p>المستوى: <strong><span id="level">1</span></strong></p>
  <p>USDT: <strong><span id="usdt">0</span></strong></p>
  <p>الهدف القادم: <span id="target">5</span> USDT</p>

  <div class="level-bar-container">
    <div id="levelBar"></div>
  </div>
</div>

<div class="box mining-section">
  <div id="mineArea" style="flex:1;">
    <h2>🎮 منجم التعدين</h2>
    <div id="mineBtn" class="mine-shaft">
      <span>⛏ احفر هنا!</span>
    </div>
  </div>

  <div id="autoArea" style="flex:1;">
    <h2>⛏ التعدين التلقائي</h2>
    <p>السرعة: <span id="speed">0.02</span> USDT / ثانية</p>
    <button id="start">بدء</button>
    <button id="stop">إيقاف</button>
  </div>
</div>

<div class="warning">
⚠️ هذا الموقع تعليمي فقط. العملة افتراضية ولا تمثل USDT الحقيقي.
</div>

</main>

<footer>
مشروع تعليمي – لعبة Web3
</footer>

<!-- Modal التهنئة -->
<div id="levelModal" style="
    display:none;
    position:fixed;
    top:0; left:0; right:0; bottom:0;
    background:rgba(0,0,0,0.7);
    display:flex;
    justify-content:center;
    align-items:center;
    z-index:1000;
">
  <div style="
      background:#222;
      padding:30px;
      border-radius:15px;
      text-align:center;
      max-width:300px;
      color:#fff;
      box-shadow:0 8px 25px rgba(0,0,0,0.6);
  ">
    <h2 id="modalTitle">🎉 تهانينا!</h2>
    <p id="modalMsg">لقد وصلت لمستوى جديد!</p>
    <button id="closeModal" style="
        padding:10px 20px;
        margin-top:15px;
        border:none;
        border-radius:8px;
        background:#1e90ff;
        color:white;
        font-weight:bold;
        cursor:pointer;
    ">حسناً</button>
  </div>
</div>

<script>
let wallet=null;
let usdt=0;
let level=1;
let speed=0.02;
let interval=null; // التأكد من التهيئة

const levels = [
  { need: 5, reward: 1 },
  { need: 15, reward: 2 },
  { need: 30, reward: 3 },
  { need: 60, reward: 5 },
  { need: 100, reward: 8 }
];

document.getElementById("connect").onclick = async ()=>{
  if(!window.ethereum){ alert("افتح من MetaMask"); return; }
  const acc = await ethereum.request({method:"eth_requestAccounts"});
  wallet = acc[0];
  document.getElementById("wallet").textContent = wallet;
  load();
};

document.getElementById("start").onclick = ()=>{
  if(interval) clearInterval(interval);
  interval = setInterval(()=>{
    usdt += speed;
    checkLevel();
    update();
  },1000);
};

document.getElementById("stop").onclick = ()=>{
  if(interval){
    clearInterval(interval);
    interval = null;
  }
};

document.getElementById("mineBtn").onclick = ()=>{
  usdt += 0.1;
  createHoleEffect();
  checkLevel();
  update();
};

function checkLevel(){
  const cfg = levels[level-1];
  if(cfg && usdt >= cfg.need){
    level++;
    usdt += cfg.reward;
    speed += 0.01;

    update();

    // عرض نافذة التهنئة
    document.getElementById("modalTitle").textContent = `🎉 تهانينا!`;
    document.getElementById("modalMsg").textContent = 
      `لقد وصلت للمستوى ${level} وحصلت على ${cfg.reward} USDT كمكافأة!`;
    document.getElementById("levelModal").style.display = "flex";
  }
}

// إغلاق نافذة التهنئة
document.getElementById("closeModal").onclick = ()=>{
  document.getElementById("levelModal").style.display = "none";
};

function update(){
  document.getElementById("usdt").textContent = usdt.toFixed(2);
  document.getElementById("level").textContent = level;
  document.getElementById("speed").textContent = speed.toFixed(2);
  document.getElementById("target").textContent =
    levels[level-1]?.need ?? "MAX";

  // تحديث شريط تقدم المستوى
  const cfg = levels[level-1];
  if(cfg){
    const percent = Math.min((usdt/cfg.need)*100, 100);
    document.getElementById("levelBar").style.width = percent + "%";
  } else {
    document.getElementById("levelBar").style.width = "100%";
  }

  if(wallet){
    localStorage.setItem(wallet, JSON.stringify({usdt,level,speed}));
  }
}

function load(){
  const d = localStorage.getItem(wallet);
  if(d){
    const s = JSON.parse(d);
    usdt=s.usdt; level=s.level; speed=s.speed;
    update();
  }
}

// تأثير رسومي "حفر" عند الضغط
function createHoleEffect(){
  const mine = document.getElementById("mineBtn");
  const hole = document.createElement("div");
  hole.className = "hole";
  mine.appendChild(hole);
  setTimeout(()=>{ mine.removeChild(hole); },300);
}
</script>

</body>
</html>

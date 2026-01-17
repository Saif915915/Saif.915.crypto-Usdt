#   
  
<!DOCTYPE html>  
<html lang="ar">  
<head>  
<meta charset="UTF-8">  
<title>**لعبة** **تعدين** USDT</title>  
<meta name="viewport" content="width=device-width, initial-scale=1.0">  
  
<style>  
body {  
  font-family: Arial, sans-serif;  
  direction: rtl;  
  margin: 0;  
  background:  
    linear-gradient(rgba(0,0,0,.55), rgba(0,0,0,.55)),  
    radial-gradient(circle at bottom, #ff8c00, #b22222, #2b1b0e);  
  min-height: 100vh;  
}  
  
header {  
  background: rgba(20,40,80,.85);  
  color: white;  
  text-align: center;  
  padding: 20px;  
}  
  
main {  
  max-width: 900px;  
  margin: auto;  
  padding: 20px;  
}  
  
.box {  
  background: rgba(255,255,255,.9);  
  padding: 20px;  
  border-radius: 15px;  
  margin-bottom: 20px;  
  text-align: center;  
}  
  
button {  
  padding: 12px 25px;  
  margin: 6px;  
  border: none;  
  border-radius: 8px;  
  background: #1e90ff;  
  color: white;  
  cursor: pointer;  
}  
  
#mineBtn {  
  background: #c45a00;  
  font-size: 18px;  
}  
  
footer {  
  background: rgba(0,0,0,.6);  
  color: white;  
  text-align: center;  
  padding: 15px;  
}  
</style>  
</head>  
  
<body>  
  
<header>  
<h1>🎮 **لعبة** **تعدين** USDT</h1>  
</header>  
  
<main>  
  
<div class="box">  
<h2>🔐 **المحفظة**</h2>  
<p id="wallet">**غير** **متصل**</p>  
<button id="connect">**ربط** MetaMask</button>  
</div>  
  
<div class="box">  
<h2>📊 **مستواك**</h2>  
<p>**المستوى**: <strong><span id="level">1</span></strong></p>  
<p>USDT: <strong><span id="usdt">0</span></strong></p>  
<p>**الهدف** **القادم**: <span id="target">5</span> USDT</p>  
</div>  
  
<div class="box">  
<h2>⛏ **التعدين** **التلقائي**</h2>  
<p>**السرعة**: <span id="speed">0.02</span> USDT / **ثانية**</p>  
<button id="start">**بدء**</button>  
<button id="stop">**إيقاف**</button>  
</div>  
  
<div class="box">  
<h2>🎮 **لعبة** **التعدين**</h2>  
<button id="mineBtn">⛏ **اضغط** **للتعدين**</button>  
</div>  
  
</main>  
  
<footer>  
USDT **افتراضي** – **مشروع** **تعليمي**  
</footer>  
  
<script>  
let wallet=null;  
let usdt=0;  
let level=1;  
let speed=0.02;  
let interval;  
  
const levels = [  
  { need: 5, reward: 1 },  
  { need: 15, reward: 2 },  
  { need: 30, reward: 3 },  
  { need: 60, reward: 5 },  
  { need: 100, reward: 8 }  
];  
  
document.getElementById("connect").onclick = async ()=>{  
  if(!window.ethereum){ alert("**افتح** **من** MetaMask"); return; }  
  const acc = await ethereum.request({method:"eth_requestAccounts"});  
  wallet = acc[0];  
  document.getElementById("wallet").textContent = wallet;  
  load();  
};  
  
document.getElementById("start").onclick = ()=>{  
  interval = setInterval(()=>{  
    usdt += speed;  
    checkLevel();  
    update();  
  },1000);  
};  
  
document.getElementById("stop").onclick = ()=> clearInterval(interval);  
  
document.getElementById("mineBtn").onclick = ()=>{  
  usdt += 0.1;  
  checkLevel();  
  update();  
};  
  
function checkLevel(){  
  const cfg = levels[level-1];  
  if(cfg && usdt >= cfg.need){  
    level++;  
    usdt += cfg.reward;  
    speed += 0.01;  
    alert(`🎉 **وصلت** **للمستوى** ${level} **وحصلت** **على** ${cfg.reward} USDT`);  
  }  
}  
  
function update(){  
  document.getElementById("usdt").textContent = usdt.toFixed(2);  
  document.getElementById("level").textContent = level;  
  document.getElementById("speed").textContent = speed.toFixed(2);  
  document.getElementById("target").textContent =  
    levels[level-1]?.need ?? "MAX";  
  
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
</script>  
  
</body>  
</html>  

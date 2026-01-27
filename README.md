<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>Neon Mining Game</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<link rel="manifest" href="manifest.json">
<meta name="theme-color" content="#050d16">

<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>

<style>
:root{
  --bg:#050d16;
  --card:#0b1f33;
  --neon:#00ffd5;
  --blue:#1e90ff;
}
*{box-sizing:border-box}

body{
  margin:0;
  font-family:'Segoe UI',Arial;
  direction:rtl;
  background:radial-gradient(circle at top,#0c2b4d,#050d16 70%);
  color:#eafcff;
}
header{
  padding:18px;
  text-align:center;
  font-size:22px;
  font-weight:900;
  background:linear-gradient(90deg,#0b3c5d,#05121f);
  box-shadow:0 0 25px rgba(0,255,213,.3);
}
main{
  max-width:420px;
  margin:16px auto;
  padding:10px;
}
.box{
  background:linear-gradient(180deg,#0d2a44,#071626);
  border-radius:18px;
  padding:16px;
  margin-bottom:14px;
  box-shadow:
    0 0 0 1px rgba(0,255,213,.15),
    0 20px 40px rgba(0,0,0,.6);
}
h3{color:var(--neon);margin:0 0 10px}

.player-bar{
  display:flex;
  justify-content:space-between;
  font-size:14px;
}
.player-bar b{
  color:var(--neon);
  text-shadow:0 0 6px rgba(0,255,213,.7);
}

canvas{
  width:100%;
  height:260px;
  border-radius:16px;
  background:#020a12;
  box-shadow:0 0 30px rgba(0,255,213,.25) inset;
}

button{
  width:100%;
  padding:11px;
  border:none;
  border-radius:14px;
  font-weight:800;
  color:#001018;
  background:linear-gradient(90deg,var(--neon),var(--blue));
  box-shadow:0 0 18px rgba(0,255,213,.6);
}
button:active{transform:scale(.95)}

.actions{
  display:grid;
  grid-template-columns:repeat(3,1fr);
  gap:8px;
  margin-top:10px;
}

.progress{
  height:12px;
  background:#02101d;
  border-radius:10px;
  overflow:hidden;
  box-shadow:inset 0 0 10px rgba(0,255,213,.3);
}
.progress-fill{
  height:100%;
  background:linear-gradient(90deg,#00ffd5,#1e90ff);
}

.shop-item{
  background:#071c30;
  padding:12px;
  border-radius:14px;
  margin-bottom:10px;
}

input{
  width:100%;
  padding:10px;
  border:none;
  border-radius:12px;
  margin-bottom:8px;
  background:#02101d;
  color:#fff;
}

.warning{
  text-align:center;
  font-size:12px;
  color:#ff9a9a;
}
</style>
</head>

<body>

<header>⚡ Neon Mining Game</header>

<main>

<div class="box" id="authBox">
  <h3>تسجيل</h3>
  <input id="email" placeholder="الإيميل">
  <input id="password" type="password" placeholder="كلمة السر">
  <button onclick="createAccount()">إنشاء حساب</button>
  <button onclick="login()">دخول</button>
</div>

<div class="box" id="playerBox" style="display:none">
  <div class="player-bar">
    <span>Lvl <b id="level">1</b></span>
    <span>Points <b id="usdt">0</b></span>
    <span>هدف <b id="target">5</b></span>
  </div>
  <div class="progress" style="margin-top:8px">
    <div class="progress-fill" id="levelBar"></div>
  </div>
</div>

<div class="box" id="gameBox" style="display:none">
  <canvas id="mine3D"></canvas>

  <p style="text-align:center;font-size:13px">اضغط للتعدين</p>

  <p style="font-size:13px">
    سرعة: <b id="speed">0.02</b><br>
    قيد التجميع: <b id="pending">0</b>
  </p>

  <div class="progress">
    <div class="progress-fill" id="pendingBar"></div>
  </div>

  <div class="actions">
    <button id="start">▶</button>
    <button id="collect">📥</button>
    <button id="stop">⏸</button>
  </div>
</div>

<div class="box" id="shopBox" style="display:none">
  <h3>🏪 الترقيات</h3>

  <div class="shop-item">
    ⚡ سرعة التعدين<br>
    السعر: <b id="speedPrice">5</b>
    <button onclick="buySpeed()">شراء</button>
  </div>

  <div class="shop-item">
    👆 قوة الضغطة<br>
    السعر: <b id="clickPrice">3</b>
    <button onclick="buyClick()">شراء</button>
  </div>
</div>

<div class="warning">
⚠️ لعبة تعليمية فقط
</div>

</main>

<script>
// ===== المتغيرات =====
let usdt=0,pending=0,level=1,speed=0.02,clickPower=0.1;
let speedPrice=5,clickPrice=3,interval=null,currentUser=null;

const levels=[{need:5},{need:15},{need:30},{need:60}];

// ===== حساب =====
function createAccount(){
  localStorage.setItem("user_"+email.value,JSON.stringify({
    pass:btoa(password.value),
    usdt:0,pending:0,level:1,speed:0.02,
    clickPower:0.1,speedPrice:5,clickPrice:3
  }));
  alert("تم");
}

function login(){
  const u=JSON.parse(localStorage.getItem("user_"+email.value));
  if(!u||u.pass!==btoa(password.value)) return alert("خطأ");
  currentUser=email.value;
  ({usdt,pending,level,speed,clickPower,speedPrice,clickPrice}=u);
  authBox.style.display="none";
  playerBox.style.display=gameBox.style.display=shopBox.style.display="block";
  update();
}

function save(){
  if(!currentUser) return;
  localStorage.setItem("user_"+currentUser,JSON.stringify({
    pass:JSON.parse(localStorage.getItem("user_"+currentUser)).pass,
    usdt,pending,level,speed,clickPower,speedPrice,clickPrice
  }));
}
setInterval(save,5000);

// ===== أزرار =====
start.onclick=()=>interval||(
  interval=setInterval(()=>{pending+=speed;update()},1000)
);
stop.onclick=()=>{clearInterval(interval);interval=null};
collect.onclick=()=>{
  usdt+=pending; pending=0; checkLevel(); update();
};

// ===== متجر =====
function buySpeed(){
  if(usdt<speedPrice) return;
  usdt-=speedPrice; speed+=0.01; speedPrice+=3; update();
}
function buyClick(){
  if(usdt<clickPrice) return;
  usdt-=clickPrice; clickPower+=0.1; clickPrice+=2; update();
}

// ===== مستوى =====
function checkLevel(){
  if(levels[level-1] && usdt>=levels[level-1].need){
    level++; speed+=0.01;
  }
}

// ===== تحديث =====
function update(){
  usdtEl.textContent=usdt.toFixed(2);
  pendingEl.textContent=pending.toFixed(2);
  levelEl.textContent=level;
  speedEl.textContent=speed.toFixed(2);
  targetEl.textContent=levels[level-1]?.need||"MAX";
  pendingBar.style.width=Math.min(pending/10*100,100)+"%";
  levelBar.style.width=Math.min(usdt/(levels[level-1]?.need||usdt)*100,100)+"%";
  speedPriceEl.textContent=speedPrice;
  clickPriceEl.textContent=clickPrice;
}

// ===== عناصر =====
const email=document.getElementById("email");
const password=document.getElementById("password");
const authBox=document.getElementById("authBox");
const playerBox=document.getElementById("playerBox");
const gameBox=document.getElementById("gameBox");
const shopBox=document.getElementById("shopBox");
const usdtEl=document.getElementById("usdt");
const pendingEl=document.getElementById("pending");
const levelEl=document.getElementById("level");
const speedEl=document.getElementById("speed");
const targetEl=document.getElementById("target");
const pendingBar=document.getElementById("pendingBar");
const levelBar=document.getElementById("levelBar");
const speedPriceEl=document.getElementById("speedPrice");
const clickPriceEl=document.getElementById("clickPrice");

// ===== Three.js =====
const canvas=document.getElementById("mine3D");
const scene=new THREE.Scene();
const camera=new THREE.PerspectiveCamera(75,1,0.1,1000);
const renderer=new THREE.WebGLRenderer({canvas,alpha:true});
renderer.setSize(canvas.clientWidth,canvas.clientHeight);

const light=new THREE.DirectionalLight(0xffffff,1);
light.position.set(5,5,5);
scene.add(light);

const rock=new THREE.Mesh(
  new THREE.DodecahedronGeometry(1),
  new THREE.MeshStandardMaterial({color:0x00ffd5})
);
scene.add(rock);
camera.position.z=5;

(function animate(){
  requestAnimationFrame(animate);
  rock.rotation.y+=0.01;
  renderer.render(scene,camera);
})();

canvas.onclick=()=>{
  pending+=clickPower;
  rock.scale.set(1.1,1.1,1.1);
  setTimeout(()=>rock.scale.set(1,1,1),100);
  update();
};

// ===== PWA =====
if("serviceWorker"in navigator){
  navigator.serviceWorker.register("sw.js");
}
</script>

</body>
</html>

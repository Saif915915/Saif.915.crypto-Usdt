<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>لعبة تعدين USDT</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>

<style>
body {
  font-family: Arial, sans-serif;
  direction: rtl;
  margin:0;
  padding:0;
  min-height:100vh;
  background: url('https://i.imgur.com/R4y1A4P.jpg') no-repeat center center fixed;
  background-size: cover;
  color:#fff;
}
header {
  background: rgba(20,40,80,.85);
  text-align:center;
  padding:25px;
  font-size:24px;
  font-weight:bold;
}
main {
  max-width:900px;
  margin:20px auto;
  display:flex;
  flex-direction:column;
  gap:20px;
}
.box {
  background: rgba(0,0,0,.6);
  padding:20px;
  border-radius:15px;
  box-shadow: 0 8px 25px rgba(0,0,0,.5);
}
h2 { margin-top:0; color:#ffcc00; }
button {
  padding:12px 25px;
  border:none;
  border-radius:8px;
  cursor:pointer;
  background:#1e90ff;
  color:white;
  font-weight:bold;
  transition:0.2s;
}
button:hover { background:#0f6fc5; }
input {
  width:100%;
  padding:10px;
  margin:5px 0;
  border-radius:6px;
  border:none;
}
.warning {
  text-align:center;
  font-size:13px;
  color:#ff9999;
}

/* ===== العداد المرئي ===== */
.progress-box {
  background:#222;
  border-radius:10px;
  overflow:hidden;
  margin:10px 0;
}
.progress-fill {
  height:18px;
  width:0%;
  background:linear-gradient(90deg,#00ffcc,#1e90ff);
  transition:width 0.3s ease;
}
.progress-text {
  text-align:center;
  font-size:13px;
  margin-top:5px;
}

#mine3D {
  width:100%;
  height:300px;
  border-radius:15px;
  cursor:pointer;
}
</style>
</head>

<body>

<header>🎮 لعبة تعدين USDT التعليمية</header>

<main>

<!-- تسجيل الدخول -->
<div class="box" id="authBox">
  <h2>تسجيل الدخول / إنشاء حساب</h2>
  <input id="email" type="email" placeholder="البريد الإلكتروني">
  <input id="password" type="password" placeholder="كلمة السر">
  <button id="loginBtn">تسجيل الدخول</button>
  <button id="createBtn">إنشاء حساب</button>
</div>

<!-- معلومات اللاعب -->
<div class="box" id="playerBox" style="display:none;">
  <h2>📊 معلوماتك</h2>
  <p>المستوى: <strong><span id="level">1</span></strong></p>
  <p>الرصيد: <strong><span id="usdt">0</span></strong> USDT</p>
  <p>الهدف القادم: <span id="target">5</span> USDT</p>
</div>

<!-- التعدين -->
<div class="box" id="gameBox" style="display:none;">
  <h2>⛏ منجم التعدين</h2>
  <canvas id="mine3D"></canvas>
  <p>اضغط على الصخرة للتعدين اليدوي</p>

  <hr>

  <h3>⚙️ التعدين التلقائي</h3>
  <p>السرعة: <span id="speed">0.02</span> USDT / ثانية</p>

  <p>قيد التجميع: <strong><span id="pending">0</span></strong> USDT</p>

  <div class="progress-box">
    <div class="progress-fill" id="pendingBar"></div>
  </div>
  <div class="progress-text">عداد التجميع</div>

  <button id="start">▶️ بدء التعدين</button>
  <button id="stop">⏸ إيقاف</button>
  <button id="collect">📥 تجميع النقاط</button>
</div>

<div class="warning">⚠️ هذه لعبة تعليمية – لا تمثل USDT الحقيقي</div>

</main>

<script>
// ===== متغيرات =====
let usdt=0, pendingUSDT=0, level=1, speed=0.02;
let interval=null, autoMining=false;
let currentUserEmail=null;

const levels=[
  {need:5,reward:1},
  {need:15,reward:2},
  {need:30,reward:3},
  {need:60,reward:5}
];

// ===== الحسابات =====
function createAccount(email,password){
  if(!email||!password) return alert("أدخل البيانات");
  if(localStorage.getItem("user_"+email)) return alert("الحساب موجود");
  localStorage.setItem("user_"+email,JSON.stringify({
    password,
    usdt:0,
    pendingUSDT:0,
    level:1,
    speed:0.02
  }));
  alert("تم إنشاء الحساب");
}

function login(email,password){
  const data=localStorage.getItem("user_"+email);
  if(!data) return alert("الحساب غير موجود");
  const user=JSON.parse(data);
  if(user.password!==password) return alert("كلمة السر خاطئة");

  usdt=user.usdt;
  pendingUSDT=user.pendingUSDT||0;
  level=user.level;
  speed=user.speed;
  currentUserEmail=email;

  document.getElementById("authBox").style.display="none";
  document.getElementById("playerBox").style.display="block";
  document.getElementById("gameBox").style.display="block";
  update();
}

function saveProgress(){
  if(!currentUserEmail) return;
  const user=JSON.parse(localStorage.getItem("user_"+currentUserEmail));
  localStorage.setItem("user_"+currentUserEmail,JSON.stringify({
    password:user.password,
    usdt,
    pendingUSDT,
    level,
    speed
  }));
}

// ===== أزرار =====
createBtn.onclick=()=>createAccount(email.value,password.value);
loginBtn.onclick=()=>login(email.value,password.value);

start.onclick=()=>{
  if(autoMining) return;
  autoMining=true;
  interval=setInterval(()=>{
    pendingUSDT+=speed;
    update();
  },1000);
};

stop.onclick=()=>{
  autoMining=false;
  clearInterval(interval);
};

collect.onclick=()=>{
  if(pendingUSDT<=0) return alert("لا يوجد نقاط");
  usdt+=pendingUSDT;
  pendingUSDT=0;
  checkLevel();
  saveProgress();
  update();
};

// ===== المستويات =====
function checkLevel(){
  const cfg=levels[level-1];
  if(cfg && usdt>=cfg.need){
    level++;
    usdt+=cfg.reward;
    speed+=0.01;
    alert(`🎉 وصلت للمستوى ${level}`);
  }
}

// ===== تحديث الواجهة =====
function update(){
  usdtEl.textContent=usdt.toFixed(2);
  pending.textContent=pendingUSDT.toFixed(2);
  levelEl.textContent=level;
  speedEl.textContent=speed.toFixed(2);
  target.textContent=levels[level-1]?.need||"MAX";

  const percent=Math.min(pendingUSDT/5*100,100);
  pendingBar.style.width=percent+"%";
}

const usdtEl=document.getElementById("usdt");
const pending=document.getElementById("pending");
const pendingBar=document.getElementById("pendingBar");
const levelEl=document.getElementById("level");
const speedEl=document.getElementById("speed");
const target=document.getElementById("target");

// ===== Three.js =====
const canvas=document.getElementById("mine3D");
const scene=new THREE.Scene();
const camera=new THREE.PerspectiveCamera(75,canvas.clientWidth/canvas.clientHeight,0.1,1000);
const renderer=new THREE.WebGLRenderer({canvas,alpha:true});
renderer.setSize(canvas.clientWidth,canvas.clientHeight);

const light=new THREE.DirectionalLight(0xffffff,1);
light.position.set(5,5,5);
scene.add(light);

const rock=new THREE.Mesh(
  new THREE.DodecahedronGeometry(1),
  new THREE.MeshStandardMaterial({color:0x8B4513})
);
scene.add(rock);
camera.position.z=5;

function animate(){
  requestAnimationFrame(animate);
  rock.rotation.y+=0.01;
  renderer.render(scene,camera);
}
animate();

canvas.onclick=()=>{
  if(!currentUserEmail) return alert("سجّل دخول");
  usdt+=0.1;
  checkLevel();
  saveProgress();
  update();
};
</script>

</body>
</html>

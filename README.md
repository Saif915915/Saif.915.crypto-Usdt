<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>لعبة تعدين تعليمية</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>

<style>
body{
  margin:0;
  font-family:Arial;
  direction:rtl;
  background:#0b1d2a;
  color:#fff;
}
header{
  background:#123;
  padding:20px;
  text-align:center;
  font-size:24px;
}
main{
  max-width:900px;
  margin:20px auto;
  padding:10px;
}
.box{
  background:rgba(0,0,0,.6);
  padding:20px;
  border-radius:12px;
  margin-bottom:20px;
}
button{
  padding:10px 20px;
  border:none;
  border-radius:8px;
  cursor:pointer;
  background:#1e90ff;
  color:#fff;
  font-weight:bold;
}
button:hover{background:#0d6efd}
input{
  width:100%;
  padding:10px;
  margin:5px 0;
  border-radius:6px;
  border:none;
}
#mine3D{
  width:100%;
  height:300px;
  border-radius:12px;
  cursor:pointer;
}

/* العداد */
.progress{
  background:#222;
  border-radius:10px;
  overflow:hidden;
  height:18px;
}
.progress-fill{
  height:100%;
  width:0%;
  background:linear-gradient(90deg,#00ffcc,#1e90ff);
  transition:width .3s;
}
.warning{
  text-align:center;
  font-size:13px;
  color:#ffaaaa;
}
</style>
</head>

<body>

<header>🎮 لعبة تعدين تعليمية</header>

<main>

<!-- تسجيل -->
<div class="box" id="authBox">
  <h2>تسجيل / إنشاء حساب</h2>
  <input id="email" placeholder="الإيميل">
  <input id="password" type="password" placeholder="كلمة السر">
  <button onclick="createAccount()">إنشاء حساب</button>
  <button onclick="login()">تسجيل دخول</button>
</div>

<!-- معلومات -->
<div class="box" id="playerBox" style="display:none">
  <p>المستوى: <b id="level">1</b></p>
  <p>الرصيد: <b id="usdt">0</b> USDT</p>
  <p>الهدف القادم: <span id="target">5</span></p>
</div>

<!-- اللعبة -->
<div class="box" id="gameBox" style="display:none">
  <canvas id="mine3D"></canvas>
  <p>اضغط على الصخرة للتعدين اليدوي</p>

  <hr>

  <p>سرعة التعدين: <b id="speed">0.02</b> / ثانية</p>
  <p>قيد التجميع: <b id="pending">0</b></p>

  <div class="progress">
    <div class="progress-fill" id="pendingBar"></div>
  </div>

  <br>

  <button id="start">▶ بدء</button>
  <button id="stop">⏸ إيقاف</button>
  <button id="collect">📥 تجميع</button>
</div>

<div class="warning">⚠️ لعبة تعليمية فقط – لا تمثل USDT الحقيقي</div>

</main>

<script>
// ================= المتغيرات =================
let usdt=0, pendingUSDT=0, level=1, speed=0.02;
let interval=null, currentUser=null;

const levels=[
  {need:5,reward:1},
  {need:15,reward:2},
  {need:30,reward:3},
  {need:60,reward:5}
];

// ================= الحسابات =================
function createAccount(){
  const email=emailInput.value;
  const pass=passwordInput.value;
  if(!email||!pass) return alert("أدخل البيانات");
  if(localStorage.getItem("user_"+email)) return alert("الحساب موجود");
  localStorage.setItem("user_"+email,JSON.stringify({
    password:pass, usdt:0, pending:0, level:1, speed:0.02
  }));
  alert("تم إنشاء الحساب");
}

function login(){
  const email=emailInput.value;
  const pass=passwordInput.value;
  const data=localStorage.getItem("user_"+email);
  if(!data) return alert("الحساب غير موجود");
  const user=JSON.parse(data);
  if(user.password!==pass) return alert("كلمة السر خاطئة");

  currentUser=email;
  usdt=user.usdt;
  pendingUSDT=user.pending;
  level=user.level;
  speed=user.speed;

  authBox.style.display="none";
  playerBox.style.display="block";
  gameBox.style.display="block";
  update();
}

function save(){
  if(!currentUser) return;
  const old=JSON.parse(localStorage.getItem("user_"+currentUser));
  localStorage.setItem("user_"+currentUser,JSON.stringify({
    password:old.password,
    usdt, pending:pendingUSDT, level, speed
  }));
}

// ================= الأزرار =================
start.onclick=()=>{
  if(interval) return;
  interval=setInterval(()=>{
    pendingUSDT+=speed;
    update();
  },1000);
};

stop.onclick=()=>{
  clearInterval(interval);
  interval=null;
};

collect.onclick=()=>{
  if(pendingUSDT<=0) return alert("لا يوجد نقاط");
  usdt+=pendingUSDT;
  pendingUSDT=0;
  checkLevel();
  save();
  update();
};

// ================= المستويات =================
function checkLevel(){
  const cfg=levels[level-1];
  if(cfg && usdt>=cfg.need){
    level++;
    usdt+=cfg.reward;
    speed+=0.01;
    alert("🎉 مستوى جديد!");
  }
}

// ================= التحديث =================
function update(){
  usdtEl.textContent=usdt.toFixed(2);
  pendingEl.textContent=pendingUSDT.toFixed(2);
  levelEl.textContent=level;
  speedEl.textContent=speed.toFixed(2);
  targetEl.textContent=levels[level-1]?.need || "MAX";

  const percent=Math.min(pendingUSDT/10*100,100);
  pendingBar.style.width=percent+"%";
}

// ================= العناصر =================
const emailInput=document.getElementById("email");
const passwordInput=document.getElementById("password");
const authBox=document.getElementById("authBox");
const playerBox=document.getElementById("playerBox");
const gameBox=document.getElementById("gameBox");

const usdtEl=document.getElementById("usdt");
const pendingEl=document.getElementById("pending");
const levelEl=document.getElementById("level");
const speedEl=document.getElementById("speed");
const targetEl=document.getElementById("target");
const pendingBar=document.getElementById("pendingBar");

// ================= Three.js =================
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
  if(!currentUser) return alert("سجّل دخول");
  usdt+=0.1;
  checkLevel();
  save();
  update();
};
</script>

</body>
</html>

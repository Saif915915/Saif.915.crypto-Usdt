 
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
  margin: 0;
  padding: 0;
  min-height: 100vh;
  background: url('https://i.imgur.com/R4y1A4P.jpg') no-repeat center center fixed;
  background-size: cover;
  color: #fff;
}
header { background: rgba(20,40,80,.85); text-align: center; padding: 25px 10px; font-size: 24px; font-weight: bold; }
main { max-width: 900px; margin: 20px auto; display: flex; flex-direction: column; gap: 20px; }
.box { background: rgba(0,0,0,.6); padding: 20px; border-radius: 15px; box-shadow: 0 8px 25px rgba(0,0,0,.5); }
h2 { margin-top: 0; color: #ffcc00; font-size: 20px; }
button { padding: 12px 25px; margin: 6px; border: none; border-radius: 8px; cursor: pointer; background: #1e90ff; color: white; font-weight: bold; transition: 0.2s; }
button:hover { background: #0f6fc5; }
.warning { font-size: 13px; color: #ff9999; text-align: center; margin-top: 10px; }
.level-path { display: flex; gap: 10px; justify-content: space-between; margin-top: 15px; }
.hole { width: 50px; height: 50px; background: #3a261a; border-radius: 50%; position: relative; overflow: hidden; }
.hole .fill { position: absolute; left: 0; bottom: 0; width: 0%; height: 100%; background: #1e90ff; transition: width 0.3s ease; }
#mine3D { width: 100%; height: 300px; background: transparent; border-radius: 15px; cursor: pointer; margin-top: 10px; }
#levelModal div { animation: pop 0.3s ease-out; }
@keyframes pop { 0% { transform: scale(0.5); opacity: 0; } 50% { transform: scale(1.2); opacity: 1; } 100% { transform: scale(1); } }
input { width: calc(100% - 20px); padding: 10px; margin: 5px 0; border-radius: 6px; border: none; }
</style>
</head>
<body>
<header>🎮 لعبة تعدين USDT التعليمية</header>
<main>

<!-- تسجيل الدخول / إنشاء حساب -->
<div id="authBox" class="box">
  <h2>تسجيل الدخول أو إنشاء حساب</h2>
  <input type="email" id="email" placeholder="البريد الإلكتروني" />
  <input type="password" id="password" placeholder="كلمة السر" />
  <button id="loginBtn">تسجيل الدخول</button>
  <button id="createBtn">إنشاء حساب</button>
</div>

<!-- معلومات اللاعب -->
<div class="box" id="playerBox" style="display:none;">
  <h2>📊 مستواك</h2>
  <p>المستوى: <strong><span id="level">1</span></strong></p>
  <p>USDT: <strong><span id="usdt">0</span></strong></p>
  <p>الهدف القادم: <span id="target">5</span> USDT</p>

  <div class="level-path">
    <div class="hole"><div class="fill"></div></div>
    <div class="hole"><div class="fill"></div></div>
    <div class="hole"><div class="fill"></div></div>
    <div class="hole"><div class="fill"></div></div>
    <div class="hole"><div class="fill"></div></div>
  </div>
</div>

<!-- المنجم 3D والتعدين -->
<div class="box mining-section" id="gameBox" style="display:none;">
  <div id="mineArea" style="flex:1;">
    <h2>🎮 منجم التعدين 3D</h2>
    <canvas id="mine3D"></canvas>
    <p>اضغط على المنجم لتحصل على USDT!</p>
  </div>
  <div id="autoArea" style="flex:1;">
    <h2>⛏ التعدين التلقائي</h2>
    <p>السرعة: <span id="speed">0.02</span> USDT / ثانية</p>
    <button id="start">بدء</button>
    <button id="stop">إيقاف</button>
  </div>
</div>

<div class="warning">⚠️ هذا الموقع تعليمي فقط. العملة افتراضية ولا تمثل USDT الحقيقي.</div>
</main>
<footer>مشروع تعليمي – لعبة Web3</footer>

<!-- نافذة التهنئة -->
<div id="levelModal" style="display:none;position:fixed;top:0;left:0;right:0;bottom:0;background:rgba(0,0,0,0.7);display:flex;justify-content:center;align-items:center;z-index:1000;">
  <div style="background:#222;padding:30px;border-radius:15px;text-align:center;max-width:300px;color:#fff;box-shadow:0 8px 25px rgba(0,0,0,0.6);">
    <h2 id="modalTitle">🎉 تهانينا!</h2>
    <p id="modalMsg">لقد وصلت لمستوى جديد!</p>
    <button id="closeModal" style="padding:10px 20px;margin-top:15px;border:none;border-radius:8px;background:#1e90ff;color:white;font-weight:bold;cursor:pointer;">حسناً</button>
  </div>
</div>

<script>
// === متغيرات اللعبة ===
let usdt=0, level=1, speed=0.02, interval=null;
let currentUserEmail=null;
const levels = [{need:5,reward:1},{need:15,reward:2},{need:30,reward:3},{need:60,reward:5},{need:100,reward:8}];

// === إدارة الحسابات ===
function createAccount(email,password){
  if(!email||!password){ alert("الرجاء إدخال الإيميل وكلمة السر"); return; }
  if(localStorage.getItem('user_'+email)){ alert("الحساب موجود مسبقاً"); return; }
  const data = {password, usdt:0, level:1, speed:0.02};
  localStorage.setItem('user_'+email, JSON.stringify(data));
  alert("تم إنشاء الحساب!");
}
function login(email,password){
  if(!email||!password){ alert("الرجاء إدخال الإيميل وكلمة السر"); return; }
  const data = localStorage.getItem('user_'+email);
  if(!data){ alert("الحساب غير موجود"); return; }
  const user = JSON.parse(data);
  if(user.password!==password){ alert("كلمة السر خاطئة"); return; }
  // تحميل بيانات اللاعب
  usdt=user.usdt; level=user.level; speed=user.speed; currentUserEmail=email;
  update();
  document.getElementById("authBox").style.display="none";
  document.getElementById("playerBox").style.display="block";
  document.getElementById("gameBox").style.display="flex";
  alert("تم تسجيل الدخول!");
}
function saveProgress(){
  if(!currentUserEmail) return;
  const data = {password: document.getElementById('password').value, usdt, level, speed};
  localStorage.setItem('user_'+currentUserEmail, JSON.stringify(data));
}

// أحداث تسجيل الدخول / إنشاء حساب
document.getElementById("createBtn").onclick = ()=> createAccount(email.value,password.value);
document.getElementById("loginBtn").onclick = ()=> login(email.value,password.value);

// === التعدين التلقائي ===
document.getElementById("start").onclick = ()=>{
  if(interval) clearInterval(interval);
  interval = setInterval(()=>{
    usdt += speed;
    checkLevel();
    update();
    saveProgress();
  },1000);
};
document.getElementById("stop").onclick = ()=>{ if(interval){ clearInterval(interval); interval=null; } };

// === المستويات ===
function checkLevel(){
  const cfg = levels[level-1];
  if(cfg && usdt >= cfg.need){
    level++;
    usdt += cfg.reward;
    speed += 0.01;
    update();
    saveProgress();
    document.getElementById("modalTitle").textContent=`🎉 تهانينا!`;
    document.getElementById("modalMsg").textContent=`لقد وصلت للمستوى ${level} وحصلت على ${cfg.reward} USDT كمكافأة!`;
    document.getElementById("levelModal").style.display="flex";
  }
}
document.getElementById("closeModal").onclick = ()=> document.getElementById("levelModal").style.display="none";

// === تحديث البيانات ===
function update(){
  document.getElementById("usdt").textContent = usdt.toFixed(2);
  document.getElementById("level").textContent = level;
  document.getElementById("speed").textContent = speed.toFixed(2);
  document.getElementById("target").textContent = levels[level-1]?.need ?? "MAX";
  // شريط المحافر
  const holes = document.querySelectorAll('.level-path .hole');
  const cfg = levels[level-1];
  if(cfg){
    const percent = Math.min((usdt/cfg.need)*100,100);
    holes.forEach((hole,i)=>{
      const part = Math.min(Math.max(percent - i*20,0),20)*5;
      hole.querySelector('.fill').style.width = part+'%';
    });
  }
}

// === Three.js 3D Mine مع جزيئات الذهب ===
const canvas = document.getElementById("mine3D");
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75,canvas.clientWidth/canvas.clientHeight,0.1,1000);
const renderer = new THREE.WebGLRenderer({canvas,alpha:true,antialias:true});
renderer.setSize(canvas.clientWidth,canvas.clientHeight);
renderer.setPixelRatio(window.devicePixelRatio);
const light = new THREE.DirectionalLight(0xffffff,1); light.position.set(5,5,5); scene.add(light);
const geometry = new THREE.DodecahedronGeometry(1,0);
const material = new THREE.MeshStandardMaterial({color:0x8B4513,roughness:0.7,metalness:0.2});
const mineRock = new THREE.Mesh(geometry,material); scene.add(mineRock);
camera.position.z =5;

// جزيئات الذهب
const particleCount=20;
const particles=[];
for(let i=0;i<particleCount;i++){
  const pGeo=new THREE.SphereGeometry(0.05,6,6);
  const pMat=new THREE.MeshStandardMaterial({color:0xFFD700});
  const pMesh=new THREE.Mesh(pGeo,pMat); pMesh.visible=false;
  scene.add(pMesh);
  particles.push({mesh:pMesh,velY:Math.random()*0.05+0.02,velX:(Math.random()-0.5)*0.05});
}

function animate(){
  requestAnimationFrame(animate);
  mineRock.rotation.y += 0.01;
  particles.forEach(p=>{
    if(p.mesh.visible){ p.mesh.position.y+=p.velY; p.mesh.position.x+=p.velX; if(p.mesh.position.y>2)p.mesh.visible=false; }
  });
  renderer.render(scene,camera);
}
animate();

// التفاعل عند الضغط على المنجم
canvas.addEventListener("click", ()=>{
  if(!currentUserEmail){ alert("الرجاء تسجيل الدخول أولاً"); return; }
  usdt+=0.1;
  mineRock.scale.set(1.2,1.2,1.2);
  setTimeout(()=>{ mineRock.scale.set(1,1,1); },100);
  particles.forEach(p=>{ p.mesh.position.set(0,0,0); p.mesh.visible=true; });
  checkLevel();
  update();
  saveProgress();
});
</script>
</body>
</html>

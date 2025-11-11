<!DOCTYPE html>
<html lang="ku">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LA PIZZA — یاری پیتزا خەندەناک</title>
<style>
:root{
  --bg:#0b1020;
  --card:#0f1724;
  --accent1:#1e90ff;
  --accent2:#ff3b3b;
  --muted:#9aa7bf;
  --glass: rgba(255,255,255,0.08);
}
body{margin:0;font-family:Arial,sans-serif;background:var(--bg);color:#fff;}
.container{max-width:480px;margin:0 auto;padding:12px;}
.play-area{position:relative;width:100%;height:400px;background:linear-gradient(180deg,#07101a,#0b1220);border-radius:12px;overflow:hidden;border:2px solid rgba(255,255,255,0.1);}
.hud{position:absolute;top:8px;left:8px;display:flex;gap:6px;z-index:10;}
.pill{background:var(--glass);padding:4px 8px;border-radius:999px;font-weight:600;font-size:13px;}
.center-message{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;pointer-events:none;text-align:center;}
.scorebig{font-size:30px;font-weight:800;}
.input-area{margin-top:8px;}
input, button{width:100%;padding:10px;margin-bottom:6px;border-radius:10px;border:none;font-size:16px;font-weight:600;}
input{background:rgba(255,255,255,0.05);color:white;}
button{background:linear-gradient(90deg,var(--accent1),var(--accent2));color:white;cursor:pointer;}
.pizza{position:absolute;width:60px;height:60px;user-select:none;touch-action:none;z-index:5;}
.pizza img{width:100%;height:100%;display:block;}
.floating{position:absolute;font-weight:800;font-size:16px;color:#ffd700;pointer-events:none;user-select:none;z-index:20;text-shadow:1px 1px 3px #000;}
.hearts{display:flex;gap:4px;align-items:center;}
.heart{width:20px;height:20px;background:red;clip-path: polygon(50% 0%, 61% 15%, 75% 15%, 85% 25%, 85% 40%, 75% 55%, 50% 75%, 25% 55%, 15% 40%, 15% 25%, 25% 15%, 39% 15%);}
.card{margin-top:10px;padding:8px;background:var(--card);border-radius:12px;}
.row{display:flex;justify-content:space-between;padding:6px;border-radius:8px;}
.row:nth-child(odd){background:rgba(255,255,255,0.02);}
</style>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>
</head>
<body>
<div class="container">
  <div class="play-area" id="playArea">
    <div class="hud">
      <div class="pill">کات: <span id="time">60</span>s</div>
      <div class="pill">خاڵ: <span id="score">0</span></div>
      <div class="pill hearts" id="lives"></div>
    </div>
    <div class="center-message" id="centerMessage"></div>
  </div>
  <div class="input-area">
    <input id="playerName" type="text" placeholder="ناوی خۆت بنووسە">
    <button id="startBtn">دەست پێ بکە</button>
  </div>
  <div class="card">
    <h4>باشترین خاڵ 🌎</h4>
    <div id="scoreList"></div>
  </div>
</div>

<script>
// ===== Firebase =====
const firebaseConfig = {
  apiKey: "AIzaSyCPnDNow1BLTFCvxnqO0th6FAnxDRusx9A",
  authDomain: "lapizzagame.firebaseapp.com",
  databaseURL: "https://lapizzagame-default-rtdb.firebaseio.com",
  projectId: "lapizzagame",
  storageBucket: "lapizzagame.firebasestorage.app",
  messagingSenderId: "151514628763",
  appId: "1:151514628763:web:8c6be3aa8e3324d521fc3e"
};
firebase.initializeApp(firebaseConfig);
const db = firebase.database();
const SCORES_REF = db.ref('lapizza-scores');

// ===== Config =====
const CONFIG = {
  GAME_TIME:60,
  START_LIVES:3,
  SPAWN_INTERVAL:600,
  MAX_ON_SCREEN:5,
  PIZZA_SIZE:60,
  SCORE_PER_PIZZA:100,
  COMBO_WINDOW:1200
};

let state = {time:CONFIG.GAME_TIME, score:0, lives:CONFIG.START_LIVES, running:false, combo:0, lastHit:0, pizzas:new Map()};
let pizzaId=0,spawnTimer=null,gameTimer=null,animationHandle=null;

const playArea = document.getElementById('playArea');
const timeEl = document.getElementById('time');
const scoreEl = document.getElementById('score');
const livesEl = document.getElementById('lives');
const centerMessage = document.getElementById('centerMessage');
const playerNameInput = document.getElementById('playerName');
const scoreListEl = document.getElementById('scoreList');
const startBtn = document.getElementById('startBtn');

const PIZZA_DATA='data:image/svg+xml;utf8,' + encodeURIComponent(`<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'><circle cx='100' cy='100' r='96' fill='#f7b733' stroke='#ff3b3b' stroke-width='8'/><circle cx='70' cy='80' r='10' fill='#a52a2a'/><circle cx='120' cy='60' r='10' fill='#a52a2a'/><circle cx='140' cy='120' r='10' fill='#a52a2a'/></svg>`);

// ===== Utilities =====
function rand(min,max){return Math.random()*(max-min)+min;}
function pushScore(name,score){SCORES_REF.push({name:name||'Anon',score:score,ts:Date.now()});}
function listenScores(){
  SCORES_REF.orderByChild('score').limitToLast(10).on('value',snap=>{
    const val=snap.val()||{};
    const arr=Object.values(val).sort((a,b)=>b.score-a.score);
    scoreListEl.innerHTML='';
    arr.forEach((s,i)=>{
      const row=document.createElement('div');
      row.className='row';
      row.innerHTML=`<div>#${i+1} ${s.name}</div><div>${s.score} 🍕</div>`;
      scoreListEl.appendChild(row);
    });
  });
}

// ===== Game Functions =====
function resetGame(){
  state.time=CONFIG.GAME_TIME;
  state.score=0;
  state.lives=CONFIG.START_LIVES;
  state.running=false;
  state.combo=0;
  state.lastHit=0;
  timeEl.textContent=state.time;
  scoreEl.textContent=state.score;
  livesEl.innerHTML='';
  for(let i=0;i<state.lives;i++){
    const h=document.createElement('div');
    h.className='heart';
    livesEl.appendChild(h);
  }
  state.pizzas.forEach(p=>p.el.remove());
  state.pizzas.clear();
  centerMessage.innerHTML='';
}

function spawnPizza(){
  if(!state.running || state.pizzas.size>=CONFIG.MAX_ON_SCREEN) return;
  const rect=playArea.getBoundingClientRect();
  pizzaId++;
  const id='pz'+pizzaId;
  const el=document.createElement('div');
  el.className='pizza';
  el.dataset.id=id;
  el.innerHTML=`<img src="${PIZZA_DATA}">`;
  const size=CONFIG.PIZZA_SIZE*(0.8+Math.random()*0.5);
  const x=rand(10,rect.width-size-10);
  const y=-size;
  el.style.width=size+'px';
  el.style.height=size+'px';
  el.style.left=x+'px';
  el.style.top=y+'px';
  playArea.appendChild(el);
  const vx=(Math.random()-0.5)*1.2;
  const vy=rand(2,4);
  state.pizzas.set(id,{id,el,x,y,vx,vy,size});
  el.addEventListener('pointerdown',()=>{hitPizza(id);});
}

function hitPizza(id){
  const p=state.pizzas.get(id);
  if(!p) return;
  const now=Date.now();
  state.combo=now-state.lastHit<=CONFIG.COMBO_WINDOW?state.combo+1:1;
  state.lastHit=now;
  const gained=Math.round(CONFIG.SCORE_PER_PIZZA*(1+(state.combo-1)*0.4));
  state.score+=gained;
  scoreEl.textContent=state.score;

  const funnyMsgs=['🍕 هەڵچوون!','🧀 پیتزا چاوی!','🤣 پیتزا فڕۆ!','😋 دەمەوێت بخورم!'];
  const msg=funnyMsgs[Math.floor(Math.random()*funnyMsgs.length)];

  const f=document.createElement('div');
  f.className='floating';
  f.textContent=`+${gained} ${msg}`;
  f.style.left=(p.x+30)+'px';
  f.style.top=(p.y)+'px';
  playArea.appendChild(f);
  f.animate([{transform:'translateY(0)',opacity:1},{transform:'translateY(-50px)',opacity:0}],{duration:900,easing:'ease-out'});
  setTimeout(()=>f.remove(),900);

  p.el.remove();
  state.pizzas.delete(id);
}

function tick(){
  if(!state.running) return;
  const rect=playArea.getBoundingClientRect();
  state.pizzas.forEach(p=>{
    p.y+=p.vy;
    p.x+=p.vx;
    p.el.style.left=p.x+'px';
    p.el.style.top=p.y+'px';
    p.el.style.transform=`rotate(${p.vx*20}deg)`;
    if(p.y>rect.height-30){
      p.el.remove();
      state.pizzas.delete(p.id);
      state.lives--;
      livesEl.innerHTML='';
      for(let i=0;i<state.lives;i++){const h=document.createElement('div');h.className='heart';livesEl.appendChild(h);}
      if(state.lives<=0) endGame();
    }
  });
}

function startGame(){
  if(!playerNameInput.value){playerNameInput.focus();return;}
  resetGame();
  state.running=true;
  spawnTimer=setInterval(spawnPizza,CONFIG.SPAWN_INTERVAL);
  gameTimer=setInterval(()=>{
    state.time--;
    timeEl.textContent=state.time;
    if(state.time<=0) endGame();
  },1000);
  animationHandle=setInterval(tick,30);
  centerMessage.innerHTML='<div class="scorebig">بڕۆ! 🍕</div>';
  setTimeout(()=>{centerMessage.innerHTML='';},1000);
}

function endGame(){
  state.running=false;
  clearInterval(spawnTimer);
  clearInterval(gameTimer);
  clearInterval(animationHandle);
  pushScore(playerNameInput.value,state.score);
  const funnyEnds=['😢 پیتزا سڕایەوە!','😂 بۆچی پیتزا فڕۆ؟','🍕 ڕێگە بۆ خاڵ زیاد نەبوو!'];
  const msg=funnyEnds[Math.floor(Math.random()*funnyEnds.length)];
  centerMessage.innerHTML=`<div class="scorebig">یاری کۆتایی هات</div><div>${state.score} خاڵ ${msg}</div>`;
}

startBtn.addEventListener('click',()=>{startGame();});
listenScores();
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="ku">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pizza Flappy Bird 🍕</title>
<style>
body{margin:0;font-family:Arial,sans-serif;background:#0b1020;overflow:hidden;color:white;}
#gameArea{position:relative;width:100%;height:600px;background:linear-gradient(to top,#07101a,#0b1020);overflow:hidden;}
#bird{
  position:absolute;
  width:60px;
  height:60px;
  bottom:0;
  left:50px;
  top:200px;
  border-radius:50%;
  background:#fff;
  display:flex;
  align-items:center;
  justify-content:center;
  overflow:hidden;
  box-shadow: 0 4px 12px rgba(0,0,0,0.5);
}
#bird img{
  width:100%;
  height:100%;
  border-radius:50%;
}
.pipe{position:absolute;width:60px;background:#1e90ff;}
#hud{position:absolute;top:10px;left:10px;background:rgba(255,255,255,0.1);padding:8px;border-radius:10px;font-weight:bold;}
#leaderboard{position:absolute;top:10px;right:10px;background:rgba(255,255,255,0.1);padding:8px;border-radius:10px;width:150px;}
button,input{margin-top:5px;padding:8px;border-radius:10px;border:none;font-weight:bold;width:100%;}
</style>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>
</head>
<body>
<div id="gameArea">
  <div id="bird"><img src="loo.jpg" alt="LA PIZZA"/></div>
  <div id="hud">نمرە: <span id="score">0</span></div>
  <div id="leaderboard"><b>لیستی نمرەکان</b><div id="scoreList"></div></div>
</div>
<div style="padding:10px;max-width:400px;margin:auto;">
  <input type="text" id="playerName" placeholder="ناوی خۆت">
  <button id="startBtn">دەست پێ بکە</button>
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
const SCORES_REF = db.ref('flappy-pizza-scores');

// ===== Game State =====
let state={running:false,birdY:200,birdVY:0,pipes:[],score:0};
const CONFIG={gravity:0.5,flap:-10,pipeSpeed:3,pipeGap:150,pipeInterval:2000};
const gameArea=document.getElementById('gameArea');
const bird=document.getElementById('bird');
const scoreEl=document.getElementById('score');
const scoreListEl=document.getElementById('scoreList');

// ===== Utils =====
function rand(min,max){return Math.random()*(max-min)+min;}
function pushScore(name,score){SCORES_REF.push({name:name||'Anon',score:score,ts:Date.now()});}
function listenScores(){
  SCORES_REF.orderByChild('score').limitToLast(5).on('value',snap=>{
    const val = snap.val()||{};
    const arr=Object.values(val).sort((a,b)=>b.score-a.score);
    scoreListEl.innerHTML='';
    arr.forEach((s,i)=>{scoreListEl.innerHTML+=`<div>${i+1}. ${s.name} ${s.score}</div>`});
  });
}

// ===== Pipes =====
function spawnPipe(){
  if(!state.running) return;
  const topHeight=rand(50,300);
  const bottomHeight=gameArea.offsetHeight-topHeight-CONFIG.pipeGap;
  const topPipe=document.createElement('div');
  topPipe.className='pipe';
  topPipe.style.height=topHeight+'px';
  topPipe.style.top='0px';
  topPipe.style.left=gameArea.offsetWidth+'px';
  gameArea.appendChild(topPipe);

  const bottomPipe=document.createElement('div');
  bottomPipe.className='pipe';
  bottomPipe.style.height=bottomHeight+'px';
  bottomPipe.style.top=(topHeight+CONFIG.pipeGap)+'px';
  bottomPipe.style.left=gameArea.offsetWidth+'px';
  gameArea.appendChild(bottomPipe);

  state.pipes.push({top:topPipe,bottom:bottomPipe,passed:false});
}

// ===== Game Loop =====
function tick(){
  if(!state.running) return;
  state.birdVY+=CONFIG.gravity;
  state.birdY+=state.birdVY;
  if(state.birdY<0) state.birdY=0;
  if(state.birdY>gameArea.offsetHeight-60) state.birdY=gameArea.offsetHeight-60;
  bird.style.top=state.birdY+'px';

  state.pipes.forEach((p,i)=>{
    const left=parseFloat(p.top.style.left);
    p.top.style.left=(left-CONFIG.pipeSpeed)+'px';
    p.bottom.style.left=(left-CONFIG.pipeSpeed)+'px';

    const bRect=bird.getBoundingClientRect();
    const topRect=p.top.getBoundingClientRect();
    const bottomRect=p.bottom.getBoundingClientRect();

    if(!(bRect.right<topRect.left||bRect.left>topRect.right||bRect.bottom<topRect.top||bRect.top>topRect.bottom) ||
       !(bRect.right<bottomRect.left||bRect.left>bottomRect.right||bRect.bottom<bottomRect.top||bRect.top>bottomRect.bottom)){
      endGame();
    }

    if(!p.passed && left+60<50){
      state.score++;
      scoreEl.textContent=state.score;
      p.passed=true;
    }

    if(left+60<0){p.top.remove();p.bottom.remove();state.pipes.splice(i,1);}
  });

  requestAnimationFrame(tick);
}

// ===== Flap =====
function flap(){state.birdVY=CONFIG.flap;}
document.addEventListener('keydown',e=>{if(e.key===' ') flap();});
document.addEventListener('touchstart',flap);

// ===== End Game =====
function endGame(){
  state.running=false;
  alert('کۆتایی یاری! نمرە: '+state.score);
  const name=document.getElementById('playerName').value;
  pushScore(name,state.score);
}

// ===== Start Game =====
document.getElementById('startBtn').addEventListener('click',()=>{
  const name=document.getElementById('playerName');
  if(!name.value){name.focus();return;}
  state.birdY=200;
  state.birdVY=0;
  state.pipes.forEach(p=>{p.top.remove();p.bottom.remove();});
  state.pipes=[];
  state.score=0;
  scoreEl.textContent='0';
  state.running=true;
  requestAnimationFrame(tick);
  setInterval(spawnPipe,CONFIG.pipeInterval);
});

listenScores();
</script>
</body>
</html>

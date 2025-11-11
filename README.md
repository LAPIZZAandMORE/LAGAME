<!doctype html>
<html lang="ku">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>LA PIZZA — Grab The Pizza (Global Leaderboard)</title>
<link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>🍕</text></svg>">

<style>
:root{ --bg:#0b1020; --card:#0f1724; --accent:#ff6b3d; --muted:#9aa7bf; --glass: rgba(255,255,255,0.06); }
*{box-sizing:border-box} html,body{height:100%} body{margin:0; font-family:Arial,Helvetica,sans-serif; background:var(--bg); color:#e6eef9;}
.container{max-width:980px;margin:28px auto;padding:18px}
.header{display:flex;align-items:center;gap:12px}
.logo{display:flex;align-items:center;gap:10px}
.logo .mark{font-size:36px}
.title{font-size:20px;font-weight:700}
.subtitle{color:var(--muted);font-size:13px}
.card{background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);border-radius:14px;padding:18px;margin-top:18px;box-shadow: 0 6px 18px rgba(2,6,23,0.6)}
.flex{display:flex;gap:12px}
.left{flex:1}
.right{width:320px}
.play-area{position:relative;height:460px;border-radius:12px;background:linear-gradient(180deg,#07101a 0%, #0b1220 100%); overflow:hidden; border:1px solid rgba(255,255,255,0.04)}
.hud{position:absolute;left:12px;top:12px;display:flex;gap:8px;align-items:center}
.hud .pill{background:var(--glass);padding:6px 10px;border-radius:999px;font-weight:600;font-size:14px}
.center-message{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;pointer-events:none}
.controls{display:flex;flex-direction:column;gap:8px;padding:12px}
.btn{background:linear-gradient(90deg,var(--accent), #ff8a5a);border:none;padding:10px 14px;border-radius:10px;color:white;font-weight:700;cursor:pointer}
.btn.ghost{background:transparent;border:1px solid rgba(255,255,255,0.06)}
.small{padding:8px 10px;font-size:14px}
.leaderboard{background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent); padding:12px;border-radius:10px}
.leaderboard h4{margin:0 0 8px 0}
.list{max-height:260px;overflow:auto}
.row{display:flex;align-items:center;justify-content:space-between;padding:8px;border-radius:8px}
.row:nth-child(odd){background:rgba(255,255,255,0.01)}
.small-muted{color:var(--muted);font-size:13px}
.footer{margin-top:12px;color:var(--muted);font-size:13px}
.controls input[type=text]{padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:transparent;color:inherit}
.pizza{position:absolute;touch-action:none;user-select:none;width:86px;height:86px;transform-origin:center; will-change:transform, left, top}
.pizza img{width:100%;height:100%;display:block}
.scorebig{font-size:40px;font-weight:800;letter-spacing:-1px}
.badge{display:inline-block;padding:6px 10px;border-radius:999px;background:linear-gradient(90deg,#ffd8c7,#fff1e8);color:#3b1b00;font-weight:800}
@media(max-width:900px){.container{padding:12px}.right{width:100%}.flex{flex-direction:column}.play-area{height:420px}}
@media(max-width:480px){.play-area{height:360px}.pizza{width:72px;height:72px}}
</style>

<!-- Firebase (v8 CDN) -->
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>
</head>
<body>
<div class="container">
  <div class="header">
    <div class="logo">
      <div class="mark">🍕</div>
      <div>
        <div class="title">LA PIZZA — Grab The Pizza</div>
        <div class="subtitle">Click or tap pizzas to score. Highest score wins a free pizza!</div>
      </div>
    </div>
  </div>

  <div class="card flex">
    <div class="left">
      <div class="play-area" id="playArea" aria-label="Game play area">
        <div class="hud" id="hud">
          <div class="pill">Time: <span id="time">60</span>s</div>
          <div class="pill">Score: <span id="score">0</span></div>
          <div class="pill">Lives: <span id="lives">3</span></div>
        </div>
        <div class="center-message" id="centerMessage"></div>
      </div>
      <div style="display:flex;gap:8px;margin-top:12px;align-items:center;flex-wrap:wrap">
        <input id="playerName" type="text" placeholder="Your name (e.g. Ahmed)" />
        <button class="btn" id="startBtn">Start Game</button>
        <button class="btn ghost small" id="howBtn">How to play</button>
      </div>
      <div class="footer">Tip: On mobile, tap pizzas quickly. Combos earn multipliers. Share your score to win!</div>
    </div>

    <div class="right">
      <div class="leaderboard card">
        <h4>Top Scores <span class="small-muted">(Global)</span></h4>
        <div class="list" id="scoreList" role="list"></div>
        <div style="display:flex;gap:8px;margin-top:10px">
          <button class="btn ghost small" id="clearBtn">Clear local view</button>
          <button class="btn small" id="exportBtn">Export score</button>
        </div>
        <div style="margin-top:10px" class="small-muted">Leaderboard uses Firebase Realtime Database. Make sure your Firebase config is correct.</div>
      </div>
    </div>
  </div>
</div>

<!-- Embedded pizza SVG data URI -->
<script>
/* ========================
   CONFIG & FIREBASE INIT
   ======================== */

/* Replace with YOUR Firebase config (you already provided yours) */
const firebaseConfig = {
  apiKey: "AIzaSyCPnDNow1BLTFCvxnqO0th6FAnxDRusx9A",
  authDomain: "lapizzagame.firebaseapp.com",
  databaseURL: "https://lapizzagame-default-rtdb.firebaseio.com",
  projectId: "lapizzagame",
  storageBucket: "lapizzagame.firebasestorage.app",
  messagingSenderId: "151514628763",
  appId: "1:151514628763:web:8c6be3aa8e3324d521fc3e"
};

/* Initialize Firebase (v8 style) */
firebase.initializeApp(firebaseConfig);
const db = firebase.database();
const SCORES_REF = db.ref('lapizza-scores');

/* Game configuration */
const CONFIG = {
  GAME_TIME: 60,
  START_LIVES: 3,
  SPAWN_INTERVAL: 800,
  MAX_ON_SCREEN: 6,
  PIZZA_SIZE: 86,
  SCORE_PER_PIZZA: 100,
  COMBO_WINDOW: 1200,
  WIN_SCORE_THRESHOLD: 1500
};

const PIZZA_DATA = 'data:image/svg+xml;utf8,' + encodeURIComponent(`
<svg xmlns='http://www.w3.org/2000/svg' width='200' height='200' viewBox='0 0 200 200'>
  <circle cx='100' cy='100' r='96' fill='#f7b733' stroke='#f08a2f' stroke-width='8'/>
  <path d='M20 120 Q100 10 180 120' fill='#f08a2f' opacity='0.12'/>
  <circle cx='70' cy='80' r='12' fill='#a52a2a'/>
  <circle cx='120' cy='60' r='12' fill='#a52a2a'/>
  <circle cx='140' cy='120' r='12' fill='#a52a2a'/>
  <path d='M60 140 Q100 180 140 140' stroke='#b04a1d' stroke-width='6' fill='none' stroke-linecap='round'/>
</svg>
`);

/* ========================
   STATE & DOM
   ======================== */
let state = {
  time: CONFIG.GAME_TIME,
  score: 0,
  lives: CONFIG.START_LIVES,
  running: false,
  combo: 0,
  lastHit: 0,
  pizzas: new Map()
};

let pizzaId = 0;
let idleSpawn = null;

const playArea = document.getElementById('playArea');
const startBtn = document.getElementById('startBtn');
const howBtn = document.getElementById('howBtn');
const timeEl = document.getElementById('time');
const scoreEl = document.getElementById('score');
const livesEl = document.getElementById('lives');
const centerMessage = document.getElementById('centerMessage');
const scoreListEl = document.getElementById('scoreList');
const playerNameInput = document.getElementById('playerName');
const clearBtn = document.getElementById('clearBtn');
const exportBtn = document.getElementById('exportBtn');

/* ========================
   UTILITIES
   ======================== */
function rand(min,max){return Math.random()*(max-min)+min}
function escapeHtml(s){return String(s).replace(/[&<>\"']/g,function(c){return{'&':'&amp;','<':'&lt;','>':'&gt;','\"':'&quot;',\"'\":'&#39;'}[c]})}

/* ========================
   FIREBASE: add/load scores
   ======================== */
function pushScoreToFirebase(name, score) {
  const entry = { name: name||'Anon', score: score, ts: Date.now() };
  // Push with a small security measure: attach a random token client-side (not secure)
  entry.clientToken = Math.random().toString(36).slice(2,8);
  return SCORES_REF.push(entry);
}

/* Load top scores (listen for updates) */
function listenForScoresAndRender() {
  // order by score (child) and get last 50, then sort desc client-side
  SCORES_REF.orderByChild('score').limitToLast(50).on('value', snapshot => {
    const val = snapshot.val() || {};
    // convert to array and sort desc
    const arr = Object.keys(val).map(k => val[k]).sort((a,b) => b.score - a.score).slice(0, 10);
    renderScoreList(arr);
  });
}

function renderScoreList(arr) {
  scoreListEl.innerHTML = '';
  if(arr.length === 0) {
    scoreListEl.innerHTML = '<div class="small-muted">No scores yet — be first!</div>';
    return;
  }
  arr.forEach((s, idx) => {
    const row = document.createElement('div');
    row.className = 'row';
    const tsStr = s.ts ? new Date(s.ts).toLocaleString() : '';
    row.innerHTML = `<div><strong>#${idx+1}</strong> <span style="margin-left:8px">${escapeHtml(s.name)}</span><div class="small-muted" style="font-size:11px;margin-top:4px">${tsStr}</div></div><div><strong>${s.score}</strong></div>`;
    scoreListEl.appendChild(row);
  });
}

/* ========================
   GAME: spawn, hit, tick
   ======================== */
function showCenter(text, small='', timeout=null){
  centerMessage.innerHTML = `<div style="text-align:center;pointer-events:auto"><div class="scorebig">${text}</div><div class="small-muted" style="margin-top:6px">${small}</div></div>`;
  if(timeout){ setTimeout(()=>{ centerMessage.innerHTML = '' }, timeout); }
}

function resetGame(){
  state.time = CONFIG.GAME_TIME; state.score = 0; state.lives = CONFIG.START_LIVES;
  state.running = false; state.combo = 0; state.lastHit = 0;
  timeEl.textContent = state.time; scoreEl.textContent = state.score; livesEl.textContent = state.lives;
  clearAllPizzas();
}

function clearAllPizzas(){ state.pizzas.forEach(p => { try{ p.el.remove() }catch(e){} }); state.pizzas.clear(); }

function spawnPizza(){
  if(!state.running) return;
  if(state.pizzas.size >= CONFIG.MAX_ON_SCREEN) return;
  const rect = playArea.getBoundingClientRect();
  pizzaId++;
  const id = 'pz'+pizzaId;
  const el = document.createElement('div');
  el.className = 'pizza';
  el.dataset.id = id;
  el.innerHTML = `<img src="${PIZZA_DATA}" draggable="false" alt="pizza"/>`;
  const size = CONFIG.PIZZA_SIZE * (0.8 + Math.random()*0.6);
  const x = rand(10, rect.width - size - 10);
  const y = rand(20, rect.height - size - 40);
  el.style.width = size+'px'; el.style.height = size+'px';
  el.style.left = x+'px'; el.style.top = y+'px';
  el.style.transition = 'transform 0.12s linear';

  const vx = (Math.random()-0.5) * 1.2;
  const vy = (Math.random()*0.7 + 0.2) * (1 + Math.random()*0.6);
  const rot = (Math.random()-0.5)*40;
  const pizza = { id, el, x, y, vx, vy, rot, spawnTime: Date.now(), size };

  state.pizzas.set(id, pizza);
  playArea.appendChild(el);

  el.addEventListener('pointerdown', function onHit(e){
    e.preventDefault();
    hitPizza(id);
  }, { passive: true });
}

function hitPizza(id){
  const p = state.pizzas.get(id); if(!p) return;
  const now = Date.now();
  if(now - state.lastHit <= CONFIG.COMBO_WINDOW){ state.combo += 1; } else { state.combo = 1; }
  state.lastHit = now;
  const gained = Math.round(CONFIG.SCORE_PER_PIZZA * (1 + (state.combo-1)*0.4));
  state.score += gained; scoreEl.textContent = state.score;
  p.el.style.transform = 'scale(0.85) rotate('+ (p.rot + 30) +'deg)';
  setTimeout(()=>{ try{ p.el.remove() }catch(e){} },150);
  state.pizzas.delete(id);
  showFloating('+ '+gained, p.x + p.size/2, p.y + p.size/2);
}

function showFloating(text,x,y){
  const f = document.createElement('div');
  f.style.position='absolute'; f.style.left=(x-20)+'px'; f.style.top=(y-10)+'px';
  f.style.pointerEvents='none'; f.style.fontWeight='800'; f.style.opacity='1'; f.style.transform='translateY(0)';
  f.textContent=text; playArea.appendChild(f);
  f.animate([{transform:'translateY(0)',opacity:1},{transform:'translateY(-40px)',opacity:0}],{duration:700,easing:'ease-out'});
  setTimeout(()=>f.remove(),800);
}

function tick(){
  if(!state.running) return;
  const rect = playArea.getBoundingClientRect();
  state.pizzas.forEach(p=>{
    p.y += p.vy; p.x += p.vx; p.rot += (p.vx*4);
    p.el.style.left = p.x+'px'; p.el.style.top = p.y+'px'; p.el.style.transform = 'rotate('+p.rot+'deg)';
    if(p.y > rect.height - 30){
      try{ p.el.remove() }catch(e){}
      state.pizzas.delete(p.id);
      loseLife();
    }
  });
}

function loseLife(){ state.lives -= 1; livesEl.textContent = state.lives; if(state.lives <= 0){ endGame(); } }

/* ========================
   TIMERS & GAME FLOW
   ======================== */
let spawnTimer = null, gameTimer = null, animationHandle = null;
function startTimers(){
  state.running = true;
  spawnTimer = setInterval(()=>spawnPizza(), CONFIG.SPAWN_INTERVAL);
  gameTimer = setInterval(()=>{ state.time -= 1; timeEl.textContent = state.time; if(state.time <= 0) endGame(); }, 1000);
  animationHandle = setInterval(tick, 30);
}

function stopTimers(){
  state.running = false;
  clearInterval(spawnTimer); clearInterval(gameTimer); clearInterval(animationHandle);
  spawnTimer = null; gameTimer = null; animationHandle = null;
}

function endGame(){
  stopTimers(); clearAllPizzas();
  showCenter('Game Over', 'Score: '+state.score, 6000);
  // Push to Firebase
  const name = playerNameInput.value.trim() || 'Anon';
  pushScoreToFirebase(name, state.score).then(()=> {
    // success
    setTimeout(()=> {
      // If top or over threshold, show win
      SCORES_REF.orderByChild('score').limitToLast(1).once('value', snap => {
        const v = snap.val(); 
        const topScore = v ? Object.values(v)[0].score : 0;
        if(state.score >= CONFIG.WIN_SCORE_THRESHOLD || state.score >= topScore) {
          showCenter('You won a FREE pizza! 🎉', 'Bring this code to LA PIZZA: WIN-'+Date.now().toString().slice(-6), 8000);
        }
      });
    }, 700);
  }).catch(err=>{
    console.warn('push score failed', err);
  });
}

/* ========================
   UI: buttons, listeners
   ======================== */
startBtn.addEventListener('click', ()=>{
  if(state.running) return;
  // require a name so leaderboard shows who won
  const name = playerNameInput.value.trim();
  if(!name){ playerNameInput.focus(); playerNameInput.placeholder = 'Enter a name to save score'; return; }
  resetGame();
  timeEl.textContent = state.time; scoreEl.textContent = state.score; livesEl.textContent = state.lives;
  showCenter('Get Ready!', 'Grab as many pizzas as you can — 60 seconds', 1200);
  setTimeout(()=>{ showCenter('Go!', '', 600); startTimers(); for(let i=0;i<3;i++) setTimeout(spawnPizza, i*250); },900);
});

howBtn.addEventListener('click', ()=> {
  showCenter('How to play', 'Tap or click pizzas to collect points. Keep combos by hitting pizzas quickly. Missed pizzas cost a life. Top scorer wins a free pizza. Good luck!', 8000);
});

clearBtn.addEventListener('click', ()=> {
  // clear local UI only — cannot delete Firebase from here (you can remove entries in Firebase console)
  if(confirm('Clear visible leaderboard (local list)?')) { scoreListEl.innerHTML = '<div class="small-muted">No scores yet — be first!</div>'; }
});

exportBtn.addEventListener('click', ()=>{
  const name = playerNameInput.value.trim() || 'Anon';
  const text = `LA PIZZA Score — ${name}: ${state.score} points`;
  if(navigator.clipboard) {
    navigator.clipboard.writeText(text).then(()=> alert('Score copied to clipboard — paste into Telegram or chat to share!')).catch(()=> prompt('Copy this:', text));
  } else { prompt('Copy this:', text); }
});

/* idle preview pizzas while not running */
idleSpawn = setInterval(()=>{ if(!state.running && Math.random()>0.6) spawnPizza(); }, 1200);

/* initialize */
resetGame();
listenForScoresAndRender();

/* save before unload - not necessary but harmless */
window.addEventListener('beforeunload', ()=>{ /* nothing to save locally */ });

/* keyboard quickstart */
window.addEventListener('keydown', (e)=>{ if(e.key === ' ' && !state.running) { startBtn.click(); } });

</script>
</body>
</html>

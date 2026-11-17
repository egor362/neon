<!doctype html>
<html lang="ru">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<meta name="theme-color" content="#080d1c">
<title>NEON RIFT — Telegram Mini App</title>
<script src="https://telegram.org/js/telegram-web-app.js"></script>
<style>
:root{color-scheme:dark;--cyan:#58f4ff;--pink:#ff55d9;--panel:#111d36}
*{box-sizing:border-box}
html,body{margin:0;width:100%;height:100%;overflow:hidden;background:#080d1c;color:#effaff;font-family:system-ui,-apple-system,Segoe UI,Roboto,Arial;touch-action:none}
body{background:radial-gradient(ellipse at 50% 0%,#1b2d53,#080d1c 65%);padding:env(safe-area-inset-top) 0 env(safe-area-inset-bottom)}
#app{width:100%;height:100%;max-width:900px;margin:auto;display:flex;flex-direction:column;padding:10px 10px 8px;gap:8px}
header{display:flex;align-items:center;justify-content:space-between;gap:8px;min-height:38px}
.logo{font-size:19px;font-weight:950;letter-spacing:.12em;text-shadow:0 0 14px #48dfff}
.logo span{color:var(--pink)}
.stats{display:flex;gap:5px}
.stat{padding:5px 7px;border:1px solid #29476c;border-radius:9px;background:#111d36;font-size:10px;color:#a9c6e5;white-space:nowrap}
.stat b{display:block;color:#fff;font-size:13px}
#stage{position:relative;flex:1;min-height:180px;border:1px solid #2b4d76;border-radius:15px;overflow:hidden;background:#050a18;box-shadow:0 0 25px #00d9ff16}
canvas{width:100%;height:100%;display:block;touch-action:none}
.overlay{position:absolute;inset:0;display:grid;place-items:center;padding:16px;background:#050916d9;backdrop-filter:blur(6px);z-index:2}
.hidden{display:none!important}
.panel{width:min(100%,390px);padding:22px 18px;text-align:center;border:1px solid #3a638f;border-radius:18px;background:linear-gradient(145deg,#172849,#0a1328);box-shadow:0 15px 45px #0009}
h1{font-size:clamp(27px,8vw,40px);line-height:1;margin:0 0 10px}
h1 span{color:var(--cyan)}
p{font-size:13px;line-height:1.5;color:#bfd0e5;margin:10px 0 18px}
button{border:0;border-radius:11px;padding:12px 17px;background:linear-gradient(100deg,var(--cyan),#a5fff4);color:#06101d;font-weight:850;font-size:13px}
button.secondary{background:#1b2e50;color:#e4f6ff;border:1px solid #42648b}
.row{display:flex;justify-content:center;gap:9px;flex-wrap:wrap}
#footer{display:flex;align-items:center;justify-content:space-between;gap:8px;color:#8da8c8;font-size:11px;min-height:28px}
#controls{display:none;justify-content:center;gap:14px;padding-bottom:2px}
.control{width:82px;height:43px;padding:0;font-size:19px;background:#172b4b;color:#e9fbff;border:1px solid #41648b;touch-action:none}
@media(pointer:coarse){#controls{display:flex}#footer{min-height:24px}}
@media(max-height:520px){#app{padding:5px;gap:4px}header{min-height:29px}.panel{padding:13px}.panel p{margin:6px 0 10px}.control{height:36px}}
</style>
</head>
<body>
<div id="app">
<header>
<div class="logo">NEON <span>RIFT</span></div>
<div class="stats">
<div class="stat">СЧЁТ<b id="score">0</b></div>
<div class="stat">ЖИЗНИ<b id="lives">3</b></div>
<div class="stat">СЕКТОР<b id="level">1/10</b></div>
<div class="stat">РЕКОРД<b id="best">0</b></div>
</div>
</header>

<main id="stage">
<canvas id="game"></canvas>

<section class="overlay" id="menu">
<div class="panel">
<h1>NEON <span>RIFT</span></h1>
<p>
Аркада в стиле неонового космоса. Разбивай блоки, лови бонусы и пройди 10 секторов.<br><br>
Двигай платформу пальцем или клавишами ← →. Нажми на поле, чтобы запустить шар.
</p>
<div class="row">
<button id="play">ИГРАТЬ</button>
<button class="secondary" id="rules">ПРАВИЛА</button>
</div>
</div>
</section>

<section class="overlay hidden" id="pause">
<div class="panel">
<h1>ПАУЗА</h1>
<p>Игра остановлена.</p>
<div class="row">
<button id="resume">ПРОДОЛЖИТЬ</button>
<button class="secondary" id="restart">ЗАНОВО</button>
</div>
</div>
</section>

<section class="overlay hidden" id="result">
<div class="panel">
<h1 id="resultTitle">ПОБЕДА!</h1>
<p id="resultText"></p>
<div class="row">
<button id="again">ЕЩЁ РАЗ</button>
</div>
</div>
</section>
</main>

<div id="footer">
<span>Тап по полю — запуск · Пробел — пауза</span>
<button class="secondary" id="pauseBtn" style="padding:6px 10px;font-size:11px">Ⅱ ПАУЗА</button>
</div>

<div id="controls">
<button class="control" id="left">◀</button>
<button class="control" id="launch">●</button>
<button class="control" id="right">▶</button>
</div>
</div>

<script>
(()=>{
'use strict';

const tg=window.Telegram&&window.Telegram.WebApp;

if(tg){
 try{
  tg.ready();
  tg.expand();
  tg.setHeaderColor('#080d1c');
  tg.setBackgroundColor('#080d1c');
 }catch(e){}
}

const $=id=>document.getElementById(id);
const canvas=$('game');
const ctx=canvas.getContext('2d');

const W=900,H=620;

const ui={
 score:$('score'),
 lives:$('lives'),
 level:$('level'),
 best:$('best'),
 menu:$('menu'),
 pause:$('pause'),
 result:$('result')
};

let dpr=1,scale=1,ox=0,oy=0;
let state='menu';
let score=0,lives=3,level=1,best=0;
let bricks=[],drops=[],sparks=[],stars=[];
let paddle,ball;
let raf=0,last=0,wide=0,slow=0;

const keys={left:false,right:false};

const rand=(a,b)=>a+Math.random()*(b-a);
const clamp=(v,a,b)=>Math.max(a,Math.min(b,v));

try{
 best=Number(localStorage.getItem('neonrift_best'))||0;
}catch(e){}

ui.best.textContent=best;

function resize(){
 const r=canvas.getBoundingClientRect();

 dpr=Math.min(window.devicePixelRatio||1,2);

 canvas.width=Math.max(1,Math.round(r.width*dpr));
 canvas.height=Math.max(1,Math.round(r.height*dpr));

 scale=Math.min(canvas.width/W,canvas.height/H);

 ox=(canvas.width-W*scale)/2;
 oy=(canvas.height-H*scale)/2;

 ctx.setTransform(scale,0,0,scale,ox,oy);
}

new ResizeObserver(resize).observe(canvas);
resize();

stars=Array.from({length:90},()=>({
 x:rand(0,W),
 y:rand(0,H),
 r:rand(.5,1.7),
 a:rand(.15,.65),
 v:rand(5,19)
}));

function updateUI(){
 ui.score.textContent=score;
 ui.lives.textContent=lives;
 ui.level.textContent=level+'/10';
 ui.best.textContent=best;
}

function levelBuild(){
 bricks=[];

 const cols=10;
 const rows=Math.min(5+Math.floor((level-1)/2),7);
 const bw=70,bh=23,gap=7;
 const sx=(W-(cols*bw+(cols-1)*gap))/2;

 for(let r=0;r<rows;r++){
  for(let c=0;c<cols;c++){

   if(level>3&&(r+c+level)%10===0)continue;

   bricks.push({
    x:sx+c*(bw+gap),
    y:75+r*(bh+gap),
    w:bw,
    h:bh,
    hp:level>=3&&r<2&&(r+c)%3===0?2:1,
    color:(r+c+level)%5
   });
  }
 }
}

function resetBall(){
 paddle={
  x:W/2-58,
  y:H-55,
  w:116,
  h:13,
  speed:8
 };

 ball={
  x:W/2,
  y:paddle.y-9,
  r:8,
  vx:rand(-2,2),
  vy:-5.1,
  stuck:true
 };
}

function hideOverlays(){
 ui.menu.classList.add('hidden');
 ui.pause.classList.add('hidden');
 ui.result.classList.add('hidden');
}

function newGame(){
 score=0;
 lives=3;
 level=1;
 wide=slow=0;
 drops=[];
 sparks=[];

 levelBuild();
 resetBall();

 state='ready';

 hideOverlays();
 updateUI();
 draw();
}

function run(){
 if(['menu','won','lost'].includes(state)){
  newGame();
 }

 state='playing';
 ball.stuck=false;

 hideOverlays();

 last=performance.now();

 if(!raf)raf=requestAnimationFrame(loop);
}

function launch(){
 if(state==='ready'){
  state='playing';
  ball.stuck=false;
  last=performance.now();

  if(!raf)raf=requestAnimationFrame(loop);
 }
}

function pauseGame(){
 if(state==='playing'||state==='ready'){
  state='paused';
  ui.pause.classList.remove('hidden');
 }
}

function resume(){
 if(state==='paused'){
  state='playing';
  ui.pause.classList.add('hidden');
  last=performance.now();

  if(!raf)raf=requestAnimationFrame(loop);
 }
}

function finish(win){
 if(score>best){
  best=score;

  try{
   localStorage.setItem('neonrift_best',String(best));
  }catch(e){}
 }

 updateUI();

 state=win?'won':'lost';

 $('resultTitle').textContent=win?'ПОБЕДА!':'ИГРА ОКОНЧЕНА';

 $('resultText').textContent=win
  ?'Ты прошёл все 10 секторов! Счёт: '+score
  :'Счёт: '+score+'. Попробуй ещё раз!';

 ui.result.classList.remove('hidden');

 if(tg){
  try{
   tg.HapticFeedback.notificationOccurred(win?'success':'warning');
  }catch(e){}
 }
}

function nextLevel(){
 if(level>=10){
  finish(true);
  return;
 }

 level++;
 levelBuild();
 resetBall();
 state='ready';
 updateUI();
}

function loseLife(){
 lives--;
 updateUI();

 if(lives<=0){
  finish(false);
  return;
 }

 resetBall();
 state='ready';
}

function burst(x,y,color,n){
 for(let i=0;i<n;i++){
  sparks.push({
   x,y,
   vx:rand(-2.5,2.5),
   vy:rand(-2.5,2.5),
   life:rand(.25,.65),
   max:.65,
   color,
   size:rand(2,4)
  });
 }
}

function maybeDrop(x,y){
 if(Math.random()<.2){
  drops.push({
   x,y,
   vy:2.2,
   w:25,
   h:18,
   type:['wide','slow','life'][Math.floor(Math.random()*3)]
  });
 }
}

function collide(b){
 const nx=clamp(ball.x,b.x,b.x+b.w);
 const ny=clamp(ball.y,b.y,b.y+b.h);

 const dx=ball.x-nx;
 const dy=ball.y-ny;

 if(dx*dx+dy*dy>ball.r*ball.r)return false;

 if(Math.abs(dx)>Math.abs(dy)){
  ball.vx=-ball.vx;
 }else{
  ball.vy=-ball.vy;
 }

 b.hp--;
 score+=b.hp?35:100;

 burst(
  ball.x,
  ball.y,
  ['#58f4ff','#ff55d9','#ffd166','#8cff9c','#a18cff'][b.color],
  b.hp?6:13
 );

 if(!b.hp){
  maybeDrop(b.x+b.w/2,b.y);
  bricks.splice(bricks.indexOf(b),1);
 }

 updateUI();
 return true;
}

function update(dt){
 const f=dt/16.667;

 stars.forEach(s=>{
  s.y+=s.v*dt/1000;

  if(s.y>H){
   s.y=0;
   s.x=rand(0,W);
  }
 });

 if(state!=='playing')return;

 if(wide>0&&((wide-=dt/1000)<=0)){
  paddle.w=116;
  paddle.x=clamp(paddle.x,0,W-paddle.w);
 }

 if(slow>0)slow-=dt/1000;

 paddle.x=clamp(
  paddle.x+((keys.right?1:0)-(keys.left?1:0))*paddle.speed*f,
  0,
  W-paddle.w
 );

 if(ball.stuck){
  ball.x=paddle.x+paddle.w/2;
  ball.y=paddle.y-ball.r-1;
  return;
 }

 const sf=slow>0?.72:1;

 ball.x+=ball.vx*f*sf;
 ball.y+=ball.vy*f*sf;

 if(ball.x<ball.r){
  ball.x=ball.r;
  ball.vx=Math.abs(ball.vx);
 }

 if(ball.x>W-ball.r){
  ball.x=W-ball.r;
  ball.vx=-Math.abs(ball.vx);
 }

 if(ball.y<ball.r+12){
  ball.y=ball.r+12;
  ball.vy=Math.abs(ball.vy);
 }

 if(
  ball.vy>0&&
  ball.y+ball.r>=paddle.y&&
  ball.y-ball.r<=paddle.y+paddle.h&&
  ball.x>=paddle.x-2&&
  ball.x<=paddle.x+paddle.w+2
 ){
  const hit=(ball.x-paddle.x-paddle.w/2)/(paddle.w/2);
  const speed=Math.min(9.2,Math.hypot(ball.vx,ball.vy)+.12);
  const angle=hit*1.05;

  ball.vx=Math.sin(angle)*speed;
  ball.vy=-Math.cos(angle)*speed;
  ball.y=paddle.y-ball.r-1;

  burst(ball.x,paddle.y,'#58f4ff',5);
 }

 for(let i=bricks.length-1;i>=0;i--){
  if(collide(bricks[i]))break;
 }

 if(ball.y>H+20)loseLife();

 for(let i=drops.length-1;i>=0;i--){
  const d=drops[i];
  d.y+=d.vy*f;

  if(
   d.y+d.h>=paddle.y&&
   d.x+d.w>paddle.x&&
   d.x<paddle.x+paddle.w
  ){
   if(d.type==='wide'){
    wide=12;
    paddle.w=158;
   }else if(d.type==='slow'){
    slow=9;
   }else{
    lives=Math.min(5,lives+1);
   }

   burst(d.x,d.y,'#ffd166',12);
   drops.splice(i,1);
   updateUI();

  }else if(d.y>H){
   drops.splice(i,1);
  }
 }

 for(let i=sparks.length-1;i>=0;i--){
  const p=sparks[i];

  p.x+=p.vx*f;
  p.y+=p.vy*f;
  p.life-=dt/1000;

  if(p.life<=0)sparks.splice(i,1);
 }

 if(bricks.length===0)nextLevel();
}

function roundRect(x,y,w,h,r){
 ctx.beginPath();
 ctx.roundRect(x,y,w,h,r);
 ctx.fill();
}

function draw(){
 ctx.clearRect(0,0,W,H);

 const bg=ctx.createLinearGradient(0,0,0,H);
 bg.addColorStop(0,'#0b1732');
 bg.addColorStop(1,'#050916');

 ctx.fillStyle=bg;
 ctx.fillRect(0,0,W,H);

 for(const s of stars){
  ctx.globalAlpha=s.a;
  ctx.fillStyle='#b5edff';
  ctx.beginPath();
  ctx.arc(s.x,s.y,s.r,0,Math.PI*2);
  ctx.fill();
 }

 ctx.globalAlpha=1;
 ctx.strokeStyle='#1b385e';
 ctx.beginPath();
 ctx.moveTo(0,43);
 ctx.lineTo(W,43);
 ctx.stroke();

 const colors=[
  '#36d9ef',
  '#ff4fd8',
  '#ffd166',
  '#8cff9c',
  '#a18cff'
 ];

 for(const b of bricks){
  ctx.shadowBlur=12;
  ctx.shadowColor=colors[b.color];
  ctx.fillStyle=colors[b.color];

  roundRect(b.x,b.y,b.w,b.h,5);

  ctx.shadowBlur=0;

  if(b.hp>1){
   ctx.fillStyle='#071326';
   ctx.font='bold 12px system-ui';
   ctx.textAlign='center';
   ctx.fillText('◆',b.x+b.w/2,b.y+16);
  }
 }

 for(const d of drops){
  const col=d.type==='wide'
   ?'#58f4ff'
   :d.type==='slow'
   ?'#ffd166'
   :'#8cff9c';

  ctx.fillStyle=col;
  ctx.shadowBlur=10;
  ctx.shadowColor=col;

  roundRect(d.x,d.y,d.w,d.h,5);

  ctx.shadowBlur=0;
  ctx.fillStyle='#06101d';
  ctx.font='bold 11px system-ui';
  ctx.textAlign='center';

  ctx.fillText(
   d.type==='wide'?'W':d.type==='slow'?'S':'+',
   d.x+d.w/2,
   d.y+13
  );
 }

 ctx.fillStyle='#58f4ff';
 ctx.shadowBlur=18;
 ctx.shadowColor='#58f4ff';

 roundRect(paddle.x,paddle.y,paddle.w,paddle.h,7);

 ctx.shadowBlur=0;
 ctx.fillStyle='#fff';
 ctx.shadowBlur=20;
 ctx.shadowColor='#fff';

 ctx.beginPath();
 ctx.arc(ball.x,ball.y,ball.r,0,Math.PI*2);
 ctx.fill();

 ctx.shadowBlur=0;

 for(const p of sparks){
  ctx.globalAlpha=Math.max(0,p.life/p.max);
  ctx.fillStyle=p.color;
  ctx.fillRect(p.x,p.y,p.size,p.size);
 }

 ctx.globalAlpha=1;
 ctx.textAlign='left';
 ctx.font='12px system-ui';
 ctx.fillStyle='#7898bf';
 ctx.fillText('NEON RIFT / SECTOR '+level,16,27);

 if(state==='ready'){
  ctx.textAlign='center';
  ctx.fillStyle='#e4fbff';
  ctx.font='bold 16px system-ui';
  ctx.fillText(
   'НАЖМИ НА ПОЛЕ, ЧТОБЫ ЗАПУСТИТЬ',
   W/2,
   H-105
  );
 }
}

function loop(now){
 raf=0;

 const dt=Math.min(34,now-last||16.67);
 last=now;

 update(dt);
 draw();

 if(state==='playing'){
  raf=requestAnimationFrame(loop);
 }
}

function movePointer(e){
 const r=canvas.getBoundingClientRect();
 const px=(e.clientX-r.left)*dpr;
 const world=(px-ox)/scale;

 paddle.x=clamp(world-paddle.w/2,0,W-paddle.w);

 if(ball.stuck){
  ball.x=paddle.x+paddle.w/2;
  ball.y=paddle.y-ball.r-1;
 }
}

canvas.addEventListener('pointermove',movePointer);

canvas.addEventListener('pointerdown',e=>{
 e.preventDefault();
 movePointer(e);

 if(state==='ready')launch();
});

addEventListener('keydown',e=>{
 if(['ArrowLeft','ArrowRight',' ','a','d','A','D'].includes(e.key)){
  e.preventDefault();
 }

 if(e.key==='ArrowLeft'||e.key.toLowerCase()==='a'){
  keys.left=true;
 }

 if(e.key==='ArrowRight'||e.key.toLowerCase()==='d'){
  keys.right=true;
 }

 if(e.key===' '||e.key==='Escape'){
  if(state==='playing'||state==='ready'){
   pauseGame();
  }else{
   resume();
  }
 }

 if(e.key==='Enter'&&state==='menu'){
  run();
 }
});

addEventListener('keyup',e=>{
 if(e.key==='ArrowLeft'||e.key.toLowerCase()==='a'){
  keys.left=false;
 }

 if(e.key==='ArrowRight'||e.key.toLowerCase()==='d'){
  keys.right=false;
 }
});

function hold(id,key){
 const el=$(id);

 el.addEventListener('pointerdown',e=>{
  e.preventDefault();
  keys[key]=true;
  el.setPointerCapture(e.pointerId);
 });

 ['pointerup','pointercancel','lostpointercapture'].forEach(ev=>{
  el.addEventListener(ev,()=>keys[key]=false);
 });
}

hold('left','left');
hold('right','right');

$('launch').addEventListener('pointerdown',e=>{
 e.preventDefault();
 launch();
});

$('play').onclick=run;
$('resume').onclick=resume;

$('restart').onclick=()=>{
 newGame();
 run();
};

$('again').onclick=()=>{
 newGame();
 run();
};

$('pauseBtn').onclick=()=>{
 state==='paused'?resume():pauseGame();
};

$('rules').onclick=()=>{
 $('menu').querySelector('p').innerHTML=
 'Отбивай шар платформой. Бонусы: W — широкая платформа, S — замедление мяча, + — дополнительная жизнь. Блоки с ◆ требуют двух попаданий.';
};

document.addEventListener('visibilitychange',()=>{
 if(document.hidden&&state==='playing'){
  pauseGame();
 }
});

newGame();
state='menu';
ui.menu.classList.remove('hidden');
draw();

})();
</script>
</body>
</html>
    
                                    
            


  
 
   
  

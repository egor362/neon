
    
                                    
            
<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<meta name="theme-color" content="#090d1b">
<title>NEON RIFT — аркада</title>
<script src="https://sad.adsgram.ai/js/sad.min.js"></script>
<style>
:root{--cyan:#51f6ff;--pink:#ff4da6;--bg:#090d1b;--panel:#111a30;--muted:#9eb0d0}
*{box-sizing:border-box}html,body{margin:0;width:100%;height:100%;overflow:hidden;background:var(--bg);color:#f4f8ff;font-family:Inter,system-ui,Arial,sans-serif;touch-action:none}
body{display:flex;align-items:center;justify-content:center;background:radial-gradient(ellipse at 50% 10%,#172448 0%,#090d1b 65%)}
#app{width:min(100vw,520px);height:100dvh;max-height:900px;display:flex;flex-direction:column;padding:10px 12px;gap:9px;position:relative}
header{display:flex;align-items:center;justify-content:space-between;gap:8px;min-height:48px}
.brand{font-weight:1000;letter-spacing:2px;font-size:18px;color:var(--cyan);text-shadow:0 0 15px #51f6ff66}
.brand small{display:block;font-size:9px;letter-spacing:3px;color:#a8b9d9;font-weight:700}
.stats{display:flex;gap:6px;align-items:center}.pill{background:#111b32;border:1px solid #263b62;border-radius:99px;padding:7px 10px;font-size:12px;font-weight:800;white-space:nowrap}
button{font:inherit;cursor:pointer;color:white;border:0}
.ui-btn{background:linear-gradient(135deg,#167d9b,#5140a9);border:1px solid #65eaff66;border-radius:12px;padding:9px 12px;font-weight:800;font-size:12px}
#stage{flex:1;min-height:0;display:flex;align-items:center;justify-content:center;position:relative}
canvas{display:block;width:auto;height:auto;max-width:100%;max-height:100%;aspect-ratio:2/3;background:#0c1427;border:1px solid #31547c;border-radius:15px;box-shadow:0 0 35px #25cfff20;touch-action:none}
#bottom{display:flex;justify-content:space-between;align-items:center;gap:8px;color:var(--muted);font-size:11px;min-height:30px}
#bottom strong{color:var(--cyan)}
.overlay{position:absolute;inset:0;z-index:5;display:none;align-items:center;justify-content:center;padding:16px;background:#050916d9;backdrop-filter:blur(7px)}
.overlay.show{display:flex}
.dialog{width:min(100%,390px);max-height:90%;overflow:auto;background:linear-gradient(160deg,#172542,#0d1428);border:1px solid #3c668e;border-radius:20px;padding:20px;box-shadow:0 15px 60px #0009}
.dialog h1,.dialog h2{margin:0 0 8px;color:var(--cyan);letter-spacing:1px}
.dialog p{color:#b6c5df;font-size:13px;line-height:1.5}
.actions{display:flex;flex-wrap:wrap;gap:8px;margin-top:15px}
.primary{background:linear-gradient(110deg,#00c9d8,#5c61ff);color:#07101e;border-radius:12px;padding:11px 16px;font-weight:900;flex:1}
.secondary{background:#202e4b;border:1px solid #405578;border-radius:12px;padding:11px 14px;font-weight:800}
.shoprow{display:flex;align-items:center;justify-content:space-between;gap:10px;padding:10px;margin:8px 0;background:#111d35;border:1px solid #263b5c;border-radius:12px;font-size:12px}
.shoprow small{display:block;color:#9eb0d0;margin-top:3px}
.shoprow button{background:#174f69;border:1px solid #4bdff0;border-radius:9px;padding:8px 10px;font-weight:800;white-space:nowrap}
.shoprow button:disabled{opacity:.4}
#toast{position:absolute;z-index:8;left:50%;top:65px;transform:translateX(-50%);background:#142847;border:1px solid #51f6ff88;padding:9px 14px;border-radius:30px;font-size:12px;font-weight:800;opacity:0;transition:opacity .2s;pointer-events:none;white-space:nowrap}
#toast.on{opacity:1}
@media(min-height:760px){#app{padding-top:14px;padding-bottom:14px}header{min-height:54px}}
</style>
</head>
<body>
<div id="app">
<header>
  <div class="brand">NEON RIFT<small>ARCADE BREAKER</small></div>
  <div class="stats">
    <div class="pill">УР. <span id="level">1</span>/10</div>
    <div class="pill">◆ <span id="coins">0</span></div>
    <button class="ui-btn" id="shopOpen">МАГАЗИН</button>
  </div>
</header>
<div id="stage">
 <canvas id="game" width="480" height="720" aria-label="Игровое поле"></canvas>

 <div id="toast"></div>

 <section class="overlay show" id="startScreen">
  <div class="dialog">
   <h1>NEON RIFT</h1>
   <p>Пройди десять секторов неоновой системы. Разбивай ядра, собирай кристаллы и используй усиления. Каждый сектор меняет поле и правила игры.</p>
   <p><b>Управление:</b> веди пальцем по экрану или используй ← →. На компьютере также доступны A и D.</p>
   <div class="actions">
    <button class="primary" id="startBtn">НАЧАТЬ ИГРУ</button>
    <button class="secondary" id="howBtn">ПРАВИЛА</button>
   </div>
  </div>
 </section>

 <section class="overlay" id="messageScreen">
  <div class="dialog">
   <h1 id="messageTitle">Сектор очищен!</h1>
   <p id="messageText"></p>
   <div class="actions">
    <button class="primary" id="continueBtn">ДАЛЕЕ</button>
    <button class="secondary" id="menuBtn">В МЕНЮ</button>
   </div>
  </div>
 </section>

 <section class="overlay" id="shopScreen">
  <div class="dialog">
   <h2>МАСТЕРСКАЯ</h2>
   <p>Кристаллы можно заработать в каждом секторе. Реклама — необязательный способ получить небольшую награду.</p>
   <div class="shoprow">
    <div><b>Широкая платформа</b><small>Постоянно увеличивает платформу</small></div>
    <button id="wideBuy">90 ◆</button>
   </div>
   <div class="shoprow">
    <div><b>Бирюзовый корпус</b><small>Новый цвет платформы</small></div>
    <button id="skinCyan">КУПИТЬ · 45 ◆</button>
   </div>
   <div class="shoprow">
    <div><b>Розовый корпус</b><small>Новый цвет платформы</small></div>
    <button id="skinPink">КУПИТЬ · 45 ◆</button>
   </div>
   <div class="shoprow">
    <div><b>Золотой корпус</b><small>Новый цвет платформы</small></div>
    <button id="skinGold">КУПИТЬ · 80 ◆</button>
   </div>
   <div class="actions">
    <button class="secondary" id="adBtn">🎁 Бонус за рекламу +20 ◆</button>
    <button class="primary" id="shopClose">ЗАКРЫТЬ</button>
   </div>
  </div>
 </section>

 <section class="overlay" id="helpScreen">
  <div class="dialog">
   <h2>КАК ИГРАТЬ</h2>
   <p>Не дай шару упасть. Разрушай все блоки, чтобы открыть выход в следующий сектор. Прочные блоки требуют нескольких попаданий. Бонусные блоки могут дать кристаллы или временное усиление. С каждым сектором поле становится сложнее.</p>
   <div class="actions">
    <button class="primary" id="helpClose">ПОНЯТНО</button>
   </div>
  </div>
 </section>
</div>

<div id="bottom">
 <span>КОМБО: <strong id="combo">×1</strong></span>
 <span id="sectorName">СЕКТОР: ПРОБУЖДЕНИЕ</span>
 <span>ПАУЗА: <strong id="pauseHint">Ⅱ</strong></span>
</div>
</div>

<script>
(() => {
'use strict';

const canvas=document.getElementById('game');
const ctx=canvas.getContext('2d');
const W=480,H=720;
const $=id=>document.getElementById(id);

const screens=['startScreen','messageScreen','shopScreen','helpScreen'];

const show=id=>{
 screens.forEach(s=>$(s).classList.toggle('show',s===id));
};

const hide=()=>{
 screens.forEach(s=>$(s).classList.remove('show'));
};

const themes=[
 ['ПРОБУЖДЕНИЕ','#43eaff','#784cff'],
 ['РАЗЛОМ','#ff5ca8','#743dff'],
 ['СОЛНЕЧНЫЙ УЗЕЛ','#ffd45b','#ff784e'],
 ['ЛЕДЯНОЙ КОРИДОР','#8deaff','#527bff'],
 ['ИМПУЛЬС','#78ff9d','#18a88e'],
 ['ТЁМНАЯ МАТЕРИЯ','#c18aff','#5c3acb'],
 ['КРАСНАЯ ЗОНА','#ff806a','#c52e66'],
 ['ОРБИТА','#8da8ff','#4b56c9'],
 ['СИНГУЛЯРНОСТЬ','#f5a0ff','#a23fe0'],
 ['ФИНАЛЬНЫЙ РУБЕЖ','#fff08a','#ff4b8e']
];

const levelNames=[
 'Пробуждение','Разлом','Солнечный узел',
 'Ледяной коридор','Импульс','Тёмная материя',
 'Красная зона','Орбита','Сингулярность',
 'Финальный рубеж'
];

const layouts=[
 ['11111','12221','11111'],
 ['10101','22222','13131','11111'],
 ['11111','1.2.1','33333','1.2.1','11111'],
 ['1.1.1','22222','1.3.1','22222','1.1.1'],
 ['11111','21212','33333','21212','11111','1...1'],
 ['1.2.1','23232','1.3.1','23232','1.2.1','11111'],
 ['33333','2...2','11111','2.2.2','33333','11111'],
 ['1.1.1','22222','3.3.3','22222','1.1.1','33333'],
 ['33333','3.2.3','22222','3.1.3','33333','11111','22222'],
 ['33333','22222','3.3.3','22222','33333','2.2.2','11111']
];

let data={
 coins:0,
 level:1,
 best:1,
 wide:false,
 skins:['#43eaff'],
 skin:'#43eaff'
};

try{
 const saved=JSON.parse(localStorage.getItem('neonRiftSave')||'null');
 if(saved&&typeof saved==='object')data={...data,...saved};
}catch(e){}

data.level=Math.max(1,Math.min(10,Number(data.level)||1));
data.coins=Math.max(0,Number(data.coins)||0);

function save(){
 try{
  localStorage.setItem('neonRiftSave',JSON.stringify(data));
 }catch(e){}
 updateHud();
}

function updateHud(){
 $('level').textContent=data.level;
 $('coins').textContent=data.coins;
 $('combo').textContent='×'+combo;
 $('sectorName').textContent='СЕКТОР: '+levelNames[data.level-1].toUpperCase();
}

const rand=(a,b)=>a+Math.random()*(b-a);

let ball,paddle,bricks=[],particles=[],drops=[];
let combo=1,comboTimer=0,score=0;
let paused=true,started=false,raf=0,last=0;
let barrier=null;
let keys={left:false,right:false};
let pointerX=null;
let slowTimer=0,wideTimer=0,levelWon=false;

function makeLevel(){
 const n=data.level;
 const layout=layouts[n-1];
 const cols=5,bw=76,gap=8;
 const startX=(W-(cols*bw+(cols-1)*gap))/2;

 bricks=[];

 layout.forEach((row,r)=>{
  [...row].forEach((v,c)=>{
   if(v==='.')return;

   let hp=v==='3'?3:v==='2'?2:1;

   bricks.push({
    x:startX+c*(bw+gap),
    y:82+r*39,
    w:bw,
    h:22,
    hp,
    max:hp,
    type:(n>=2&&(r+c+n)%9===0)?'bonus':'normal',
    alive:true,
    phase:rand(0,6.28)
   });
  });
 });

 if(n>=4){
  barrier={
   x:40,y:315,w:400,h:8,
   phase:0,active:true
  };
 }else{
  barrier=null;
 }

 const speed=235+n*13;

 ball={
  x:W/2,
  y:H-145,
  vx:speed*.52,
  vy:-speed,
  r:8
 };

 paddle={
  x:W/2-48,
  y:H-78,
  w:data.wide?126:96,
  h:13,
  target:W/2
 };

 particles=[];
 drops=[];
 combo=1;
 comboTimer=0;
 score=0;
 slowTimer=0;
 wideTimer=0;
 levelWon=false;
 paused=false;
 last=0;
 updateHud();
}

function addParticles(x,y,color,count=10){
 for(let i=0;i<count;i++){
  let a=Math.random()*Math.PI*2;
  let s=rand(40,220);

  particles.push({
   x,y,
   vx:Math.cos(a)*s,
   vy:Math.sin(a)*s,
   life:rand(.25,.65),
   max:.65,
   size:rand(2,5),
   color
  });
 }
}

function dropItem(x,y,type){
 drops.push({x,y,type,vy:115,life:7});
}

function breakBrick(b){
 b.hp--;

 addParticles(
  b.x+b.w/2,
  b.y+b.h/2,
  b.type==='bonus'?'#ffe47a':themes[data.level-1][0],
  b.hp<=0?14:5
 );

 if(b.hp<=0){
  b.alive=false;
  comboTimer=2.1;
  combo=Math.min(8,combo+1);
  score+=10*combo;

  data.coins+=b.type==='bonus'?5:1;

  if(b.type==='bonus'){
   dropItem(b.x+b.w/2,b.y,'bonus');
  }else if(Math.random()<.075){
   dropItem(
    b.x+b.w/2,b.y,
    Math.random()<.5?'slow':'wide'
   );
  }
 }

 updateHud();
}

function rectHitCircle(rx,ry,rw,rh,cx,cy,cr){
 let px=Math.max(rx,Math.min(cx,rx+rw));
 let py=Math.max(ry,Math.min(cy,ry+rh));

 return (cx-px)**2+(cy-py)**2<cr*cr;
}

function update(dt){
 const speed=slowTimer>0?.65:1;

 if(slowTimer>0)slowTimer-=dt;
 if(wideTimer>0)wideTimer-=dt;

 const move=(keys.right?1:0)-(keys.left?1:0);

 if(pointerX!==null){
  paddle.target=pointerX;
 }else{
  paddle.target+=move*430*dt;
 }

 const desired=Math.max(
  paddle.w/2+8,
  Math.min(W-paddle.w/2-8,paddle.target)
 );

 paddle.x+=(desired-paddle.w/2-paddle.x)*Math.min(1,dt*16);

 if(comboTimer>0)comboTimer-=dt;
 else combo=1;

 if(barrier&&barrier.active){
  barrier.phase+=dt;
  barrier.x=40+Math.sin(barrier.phase*1.25)*38;
 }

 ball.x+=ball.vx*dt*speed;
 ball.y+=ball.vy*dt*speed;

 if(ball.x<ball.r){
  ball.x=ball.r;
  ball.vx=Math.abs(ball.vx);
 }

 if(ball.x>W-ball.r){
  ball.x=W-ball.r;
  ball.vx=-Math.abs(ball.vx);
 }

 if(ball.y<ball.r+10){
  ball.y=ball.r+10;
  ball.vy=Math.abs(ball.vy);
 }

 if(
  ball.vy>0&&
  ball.y+ball.r>=paddle.y&&
  ball.y<paddle.y+paddle.h+ball.r&&
  ball.x>=paddle.x&&
  ball.x<=paddle.x+paddle.w
 ){
  const hit=(ball.x-(paddle.x+paddle.w/2))/(paddle.w/2);

  ball.vx=hit*300;
  ball.vy=-Math.max(230,Math.abs(ball.vy));
  ball.y=paddle.y-ball.r-1;

  addParticles(ball.x,paddle.y,'#51f6ff',5);
 }

 if(
  barrier&&barrier.active&&ball.vy>0&&
  rectHitCircle(
   barrier.x,barrier.y,barrier.w,barrier.h,
   ball.x,ball.y,ball.r
  )
 ){
  ball.vy=-Math.abs(ball.vy);
  addParticles(ball.x,ball.y,'#a879ff',5);
 }

 for(const b of bricks){
  if(!b.alive)continue;

  if(rectHitCircle(
   b.x,b.y,b.w,b.h,
   ball.x,ball.y,ball.r
  )){
   const prevX=ball.x-ball.vx*dt*speed;
   const prevY=ball.y-ball.vy*dt*speed;

   if(prevY+ball.r<=b.y||prevY-ball.r>=b.y+b.h){
    ball.vy*=-1;
   }else{
    ball.vx*=-1;
   }

   breakBrick(b);
   break;
  }
 }

 for(let i=drops.length-1;i>=0;i--){
  let d=drops[i];

  d.y+=d.vy*dt;
  d.life-=dt;

  if(
   d.y>paddle.y-8&&
   d.y<paddle.y+paddle.h+10&&
   d.x>paddle.x-8&&
   d.x<paddle.x+paddle.w+8
  ){
   if(d.type==='slow')slowTimer=6;
   else if(d.type==='wide'){
    wideTimer=8;
    paddle.w=data.wide?150:125;
   }else{
    data.coins+=5;
   }

   toast(
    d.type==='slow'?'ЗАМЕДЛЕНИЕ!':
    d.type==='wide'?'ШИРОКАЯ ПЛАТФОРМА!':
    '+5 КРИСТАЛЛОВ'
   );

   addParticles(d.x,d.y,'#ffe47a',12);
   drops.splice(i,1);
   save();

  }else if(d.y>H||d.life<=0){
   drops.splice(i,1);
  }
 }

 for(let i=particles.length-1;i>=0;i--){
  let p=particles[i];

  p.x+=p.vx*dt;
  p.y+=p.vy*dt;
  p.vy+=180*dt;
  p.life-=dt;

  if(p.life<=0)particles.splice(i,1);
 }

 if(ball.y>H+25){
  paused=true;
  showMessage(
   'ШАР ПОТЕРЯН',
   'Сектор не пройден. Попробуй ещё раз — твои кристаллы и открытые улучшения сохранены.',
   'ПОВТОРИТЬ',
   true
  );
  return;
 }

 if(bricks.every(b=>!b.alive)&&!levelWon){
  levelWon=true;
  paused=true;

  const reward=15+data.level*4;
  data.coins+=reward;
  data.best=Math.max(data.best,data.level+1);

  if(data.level===10){
   data.level=1;
   save();

   showMessage(
    'ЯДРО СТАБИЛИЗИРОВАНО!',
    'Ты прошёл все десять секторов. Получено '+reward+' кристаллов. Можно начать новый цикл.',
    'НОВЫЙ ЦИКЛ',
    false
   );

  }else{
   data.level++;
   save();

   showMessage(
    'СЕКТОР ОЧИЩЕН',
    'Награда: '+reward+' ◆. Следующий сектор изменит расположение и прочность блоков.',
    'СЛЕДУЮЩИЙ СЕКТОР',
    false
   );
  }
 }
}

function showMessage(title,text,btn,retry){
 $('messageTitle').textContent=title;
 $('messageText').textContent=text;
 $('continueBtn').textContent=btn;
 $('continueBtn').dataset.retry=retry?'1':'0';
 show('messageScreen');
}

function draw(){
 const [accent,secondary]=themes[data.level-1];

 ctx.clearRect(0,0,W,H);

 const bg=ctx.createLinearGradient(0,0,0,H);
 bg.addColorStop(0,'#101d36');
 bg.addColorStop(1,'#080d1b');

 ctx.fillStyle=bg;
 ctx.fillRect(0,0,W,H);

 // Звёздный фон
 for(let i=0;i<38;i++){
  let x=(i*137+31)%W;
  let y=(i*193+17)%H;

  ctx.globalAlpha=.12+(i%4)*.07;
  ctx.fillStyle='#d9f7ff';
  ctx.fillRect(x,y,2,2);
 }

 ctx.globalAlpha=1;

 ctx.strokeStyle=accent+'22';
 ctx.lineWidth=1;

 for(let y=45;y<H;y+=45){
  ctx.beginPath();
  ctx.moveTo(0,y);
  ctx.lineTo(W,y);
  ctx.stroke();
 }

 // Блоки
 for(const b of bricks){
  if(!b.alive)continue;

  const color=b.type==='bonus'?'#ffe47a':
   b.hp===3?'#ff5b78':
   b.hp===2?secondary:accent;

  ctx.shadowBlur=13;
  ctx.shadowColor=color;
  ctx.fillStyle=color;

  roundRect(b.x,b.y,b.w,b.h,6);
  ctx.fill();

  ctx.shadowBlur=0;

  ctx.fillStyle='#071020';
  ctx.globalAlpha=.55;
  ctx.fillRect(b.x+5,b.y+4,b.w-10,3);
  ctx.globalAlpha=1;

  if(b.hp>1){
   ctx.fillStyle='#fff';
   ctx.font='bold 11px Arial';
   ctx.textAlign='center';
   ctx.fillText(b.hp,b.x+b.w/2,b.y+16);
  }

  if(b.type==='bonus'){
   ctx.fillStyle='#5b3500';
   ctx.font='bold 14px Arial';
   ctx.textAlign='center';
   ctx.fillText('✦',b.x+b.w/2,b.y+16);
  }
 }

 // Движущийся барьер
 if(barrier&&barrier.active){
  ctx.shadowBlur=15;
  ctx.shadowColor='#9b76ff';
  ctx.fillStyle='#9b76ff';

  roundRect(barrier.x,barrier.y,barrier.w,barrier.h,5);
  ctx.fill();

  ctx.shadowBlur=0;
 }

 // Бонусы
 for(const d of drops){
  ctx.fillStyle=
   d.type==='slow'?'#73a5ff':
   d.type==='wide'?'#65ffbc':'#ffe47a';

  ctx.shadowBlur=12;
  ctx.shadowColor=ctx.fillStyle;

  ctx.beginPath();
  ctx.arc(d.x,d.y,9,0,Math.PI*2);
  ctx.fill();

  ctx.shadowBlur=0;
  ctx.fillStyle='#10172b';
  ctx.font='bold 11px Arial';
  ctx.textAlign='center';

  ctx.fillText(
   d.type==='slow'?'S':
   d.type==='wide'?'W':'+',
   d.x,d.y+4
  );
 }

 // Платформа
 ctx.shadowBlur=20;
 ctx.shadowColor=data.skin;
 ctx.fillStyle=data.skin;

 roundRect(paddle.x,paddle.y,paddle.w,paddle.h,7);
 ctx.fill();

 ctx.shadowBlur=0;
 ctx.fillStyle='#fff';
 ctx.globalAlpha=.55;

 roundRect(paddle.x+8,paddle.y+2,paddle.w-16,2,2);
 ctx.fill();

 ctx.globalAlpha=1;

 // Шар
 ctx.shadowBlur=22;
 ctx.shadowColor='#ff5ca8';
 ctx.fillStyle='#fff';

 ctx.beginPath();
 ctx.arc(ball.x,ball.y,ball.r,0,Math.PI*2);
 ctx.fill();

 ctx.fillStyle='#ff5ca8';
 ctx.beginPath();
 ctx.arc(ball.x,ball.y,ball.r*.62,0,Math.PI*2);
 ctx.fill();

 ctx.shadowBlur=0;

 // Частицы
 for(const p of particles){
  ctx.globalAlpha=Math.max(0,p.life/p.max);
  ctx.fillStyle=p.color;
  ctx.fillRect(p.x,p.y,p.size,p.size);
 }

 ctx.globalAlpha=1;

 ctx.fillStyle='#a8bbd8';
 ctx.font='12px Arial';
 ctx.textAlign='left';
 ctx.fillText('ОЧИСТИ ПОЛЕ',18,28);

 ctx.textAlign='right';
 ctx.fillText('◆ '+data.coins,W-18,28);

 if(slowTimer>0||wideTimer>0){
  ctx.fillStyle='#fff';
  ctx.textAlign='center';
  ctx.font='bold 12px Arial';

  ctx.fillText(
   slowTimer>0?
    'ЗАМЕДЛЕНИЕ '+Math.ceil(slowTimer)+'с':
    'УСИЛЕННАЯ ПЛАТФОРМА '+Math.ceil(wideTimer)+'с',
   W/2,H-22
  );
 }
}

function roundRect(x,y,w,h,r){
 ctx.beginPath();
 ctx.roundRect(x,y,w,h,r);
}

function frame(t){
 if(!started)return;

 const dt=Math.min(.025,(t-(last||t))/1000);
 last=t;

 if(!paused)update(dt);

 draw();
 raf=requestAnimationFrame(frame);
}

function start(){
 started=true;
 hide();
 makeLevel();
 cancelAnimationFrame(raf);
 last=0;
 raf=requestAnimationFrame(frame);
}

function showShop(){
 paused=true;
 refreshShop();
 show('shopScreen');
}

function refreshShop(){
 $('wideBuy').textContent=data.wide?'КУПЛЕНО':'90 ◆';
 $('wideBuy').disabled=data.wide||data.coins<90;

 [
  ['skinCyan','#43eaff',45],
  ['skinPink','#ff4da6',45],
  ['skinGold','#ffd45b',80]
 ].forEach(([id,color,price])=>{
  $(id).textContent=
   data.skin===color?'ВЫБРАНО':
   data.skins.includes(color)?'ВЫБРАТЬ':
   'КУПИТЬ · '+price+' ◆';

  $(id).disabled=
   data.skin===color||
   (!data.skins.includes(color)&&data.coins<price);
 });
}

function buySkin(color,price){
 if(!data.skins.includes(color)){
  if(data.coins<price)return;
  data.coins-=price;
  data.skins.push(color);
 }

 data.skin=color;
 save();
 refreshShop();
}

function toast(t){
 const el=$('toast');
 el.textContent=t;
 el.classList.add('on');

 clearTimeout(toast.timer);

 toast.timer=setTimeout(()=>{
  el.classList.remove('on');
 },1500);
}

// Магазин
$('shopOpen').onclick=showShop;

$('shopClose').onclick=()=>{
 hide();
 paused=!started;
};

$('wideBuy').onclick=()=>{
 if(!data.wide&&data.coins>=90){
  data.coins-=90;
  data.wide=true;
  paddle.w=126;
  save();
  refreshShop();
  toast('ПЛАТФОРМА УЛУЧШЕНА');
 }
};

$('skinCyan').onclick=()=>buySkin('#43eaff',45);
$('skinPink').onclick=()=>buySkin('#ff4da6',45);
$('skinGold').onclick=()=>buySkin('#ffd45b',80);

// Adsgram
$('adBtn').onclick=async()=>{
 try{
  if(!window.Adsgram||
     typeof window.Adsgram.init!=='function'){
   throw Error('SDK недоступен');
  }

  const controller=window.Adsgram.init({
   blockId:'int-60148'
  });

  if(!controller||typeof controller.show!=='function'){
   throw Error('Реклама недоступна');
  }

  await controller.show();

  data.coins+=20;
  save();
  refreshShop();
  toast('ПОЛУЧЕНО +20 ◆');

 }catch(e){
  toast('Реклама сейчас недоступна');
  console.warn('Adsgram:',e);
 }
};

// Кнопки
$('startBtn').onclick=start;
$('howBtn').onclick=()=>show('helpScreen');
$('helpClose').onclick=()=>show('startScreen');

$('continueBtn').onclick=()=>{
 const retry=$('c
                                        

        

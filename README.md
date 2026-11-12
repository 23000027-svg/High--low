# High--low

<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>HIGH & LOW</title>
<style>
*{box-sizing:border-box}
body{
 margin:0;min-height:100vh;padding:24px 14px;
 background:radial-gradient(circle at top,#243b55,#101827 60%);
 color:#f5f7fb;font-family:system-ui,sans-serif;
 display:flex;justify-content:center
}
main{width:min(100%,560px)}
header{text-align:center;margin:10px 0 22px}
h1{font-size:clamp(2rem,8vw,3rem);color:#f4c95d;margin:0}
header p{color:#aab7c8}
.panel{
 background:#1c293b;border:1px solid #34445b;
 border-radius:20px;padding:18px;margin-bottom:14px
}
button{
 font:inherit;font-weight:bold;border:0;border-radius:12px;
 padding:13px 10px;cursor:pointer
}
button:disabled{opacity:.4}
.modes,.stats,.choices{display:grid;gap:8px}
.modes{grid-template-columns:repeat(3,1fr)}
.mode{background:#314157;color:white}
.mode.active{background:#f4c95d;color:#172033}
.stats{grid-template-columns:repeat(3,1fr);margin:14px 0}
.stat{background:#111d2d;border-radius:13px;text-align:center;padding:10px 4px}
.stat small{display:block;color:#aab7c8;font-size:.78rem}
.stat strong{font-size:1.3rem}
.progress{height:7px;background:#101827;border-radius:9px;overflow:hidden;margin:10px 0 18px}
.progress div{height:100%;background:#f4c95d;width:0;transition:width .2s}
.cards{display:flex;justify-content:center}
.card-wrap{text-align:center}
.card-label{color:#aab7c8;margin-bottom:7px}
.card{
 width:145px;height:190px;border-radius:15px;background:white;
 color:#172033;display:flex;flex-direction:column;
 justify-content:space-between;padding:12px;text-align:left;
 box-shadow:0 8px 18px #0005;font-family:Georgia,serif
}
.card .corner{font-size:1.3rem;line-height:1.1}
.card .center{font-size:3.8rem;text-align:center;align-self:center}
.card .bottom{transform:rotate(180deg);align-self:flex-end}
.red{color:#d83243}
.choices{grid-template-columns:repeat(3,1fr);margin-top:16px}
.choices button{color:white;font-size:1rem}
.high{background:#268b60}
.low{background:#c44d58}
.same{background:#52647d}
.message{
 min-height:52px;display:grid;place-items:center;text-align:center;
 background:#111d2d;border-radius:13px;padding:12px;margin-top:16px;
 font-weight:bold
}
.good{color:#75e0a8}
.bad{color:#ff9297}
.neutral{color:#f5d77b}
.actions{display:flex;gap:9px;margin-top:12px}
.actions button{flex:1;background:#314157;color:white}
.hidden{display:none}
.hint{text-align:center;color:#aab7c8;font-size:.82rem;line-height:1.6;margin-top:14px}
@media(max-width:380px){
 .card{height:165px}
 .choices button{padding:12px 4px;font-size:.9rem}
 .panel{padding:14px}
}
</style>
</head>
<body>
<main>
<header>
 <h1>♠ HIGH & LOW ♥</h1>
 <p>次のカードを予想しよう</p>
</header>

<section class="panel">
 <div class="modes">
  <button class="mode active" data-mode="streak">連続モード</button>
  <button class="mode" data-mode="points">ポイント</button>
  <button class="mode" data-mode="lives">ライフ制</button>
 </div>

 <div class="stats">
  <div class="stat"><small>ポイント</small><strong id="score">0</strong></div>
  <div class="stat"><small>連続正解</small><strong id="streak">0</strong></div>
  <div class="stat"><small id="thirdLabel">ラウンド</small><strong id="third">0</strong></div>
 </div>

 <div class="progress"><div id="bar"></div></div>

 <div class="cards">
  <div class="card-wrap">
   <div class="card-label">現在のカード</div>
   <div class="card" id="current"></div>
  </div>
 </div>

 <div class="message neutral" id="message" aria-live="polite">
  HIGH・LOW・SAMEから予想してね！
 </div>

 <div class="choices">
  <button class="high" data-guess="high">HIGH ↑</button>
  <button class="low" data-guess="low">LOW ↓</button>
  <button class="same" data-guess="same">SAME ＝</button>
 </div>

 <div class="actions">
  <button id="restart">最初から</button>
  <button id="rules">ルール</button>
 </div>

 <p class="hint">
  Aが最小、Kが最大。同じ数字はSAMEで予想！<br>
  52枚・ジョーカーなし・1ゲーム内で重複なし。
 </p>
</section>

<section class="panel hidden" id="result">
 <h2 style="text-align:center">ゲーム終了！</h2>
 <p id="resultText" style="text-align:center;font-size:1.15rem"></p>
 <button id="again" style="width:100%;background:#f4c95d;color:#172033">
  もう一度プレイ
 </button>
</section>
</main>

<script>
const suits=[
 {s:'♠',red:false},{s:'♥',red:true},
 {s:'♣',red:false},{s:'♦',red:true}
];
const names=['','A','2','3','4','5','6','7','8','9','10','J','Q','K'];

let deck=[],index=1,current=null,mode='streak';
let score=0,streak=0,round=0,lives=3,ended=false;

const $=id=>document.getElementById(id);

function shuffle(){
 const a=suits.flatMap(s=>
  Array.from({length:13},(_,i)=>({
   s:s.s,red:s.red,v:i+1,n:names[i+1]
  }))
 );
 for(let i=a.length-1;i>0;i--){
  let j=Math.floor(Math.random()*(i+1));
  [a[i],a[j]]=[a[j],a[i]];
 }
 return a;
}

function cardHTML(c){
 const color=c.red?'red':'';
 return `
  <div class="corner ${color}">${c.n}<br>${c.s}</div>
  <div class="center ${color}">${c.s}</div>
  <div class="corner bottom ${color}">${c.n}<br>${c.s}</div>
 `;
}

function setMessage(text,type='neutral'){
 $('message').textContent=text;
 $('message').className='message '+type;
}

function update(){
 $('score').textContent=score;
 $('streak').textContent=streak;

 $('thirdLabel').textContent=
  mode==='lives'?'ライフ':
  mode==='points'?'目標':'ラウンド';

 $('third').textContent=
  mode==='lives'?'♥'.repeat(lives)+'♡'.repeat(3-lives):
  mode==='points'?`${score}/100`:round;

 $('bar').style.width=(index/52*100)+'%';

 document.querySelectorAll('[data-guess]').forEach(b=>{
  b.disabled=ended;
 });
}

function start(newMode=mode){
 mode=newMode;
 deck=shuffle();
 index=1;
 current=deck[0];
 score=0;
 streak=0;
 round=0;
 lives=3;
 ended=false;

 $('current').innerHTML=cardHTML(current);
 $('result').classList.add('hidden');

 document.querySelectorAll('.mode').forEach(b=>{
  b.classList.toggle('active',b.dataset.mode===mode);
 });

 setMessage('HIGH・LOW・SAMEから予想してね！');
 update();
}

function finish(text){
 ended=true;
 $('result').classList.remove('hidden');
 $('resultText').textContent=text;
 update();
}

function guess(g){
 if(ended)return;

 const next=deck[index];

 const right=
  (g==='high'&&next.v>current.v)||
  (g==='low'&&next.v<current.v)||
  (g==='same'&&next.v===current.v);

 round++;

 if(right){
  streak++;
  score+=10;
  setMessage('正解！ 次のカードが出ました。','good');
 }else{
  streak=0;
  if(mode==='points')score=Math.max(0,score-5);
  if(mode==='lives')lives--;
  setMessage('不正解！ 次のカードが出ました。','bad');
 }

 current=next;
 index++;
 $('current').innerHTML=cardHTML(current);
 update();

 if(mode==='streak'&&!right){
  finish(`連続正解 ${round-1} 回！`);
 }else if(mode==='lives'&&lives===0){
  finish(`${round}ラウンド挑戦！ ポイント ${score}`);
 }else if(mode==='points'&&score>=100){
  finish(`100ポイント達成！ ${round}ラウンドでクリア！`);
 }else if(index>=52){
  finish(`52枚使い切り！ ポイント ${score}・連続正解 ${streak} 回！`);
 }
}

document.querySelectorAll('.mode').forEach(b=>{
 b.addEventListener('click',()=>start(b.dataset.mode));
});

document.querySelectorAll('[data-guess]').forEach(b=>{
 b.addEventListener('click',()=>guess(b.dataset.guess));
});

$('restart').addEventListener('click',()=>start());
$('again').addEventListener('click',()=>start());

$('rules').addEventListener('click',()=>{
 alert(
  '連続モード：1回でも間違えたら終了。\\n'+
  'ポイント：正解+10点、不正解-5点。100点でクリア。\\n'+
  'ライフ制：ライフ3つ。間違えると1つ減る。\\n'+
  'HIGH＝次が大きい、LOW＝次が小さい、SAME＝同じ数字。\\n'+
  'Aが最小、Kが最大です。'
 );
});

start();
</script>
</body>
</html>

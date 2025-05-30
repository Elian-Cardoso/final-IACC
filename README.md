{{-- resources/views/coup/index.blade.php --}}
@extends('layouts.app')

@section('content')
@php
    $cardImages = $cards->pluck('image', 'title');   // “duke” => “/storage/cards/duke.png”
@endphp

<style>
body{font:15px/1.4 sans-serif;margin:2rem}
#join{display:flex;gap:.5rem;flex-wrap:wrap}
#lobby,#game{display:none}
button{margin:.25rem .5rem .25rem 0}
#playersWrap{margin-top:1rem}
#players{display:flex;gap:1rem;flex-wrap:wrap}
#players div{border:1px solid #888;border-radius:4px;padding:.5rem;min-width:170px}
.revealed img{width:34px;margin-right:2px}
.card{cursor:pointer;margin:4px;border:1px solid #444;border-radius:4px; transition: transform 0.2s;}
.card.sel{background:#ffe}
.card:hover{transform: scale(1.05)}
#history{border:1px solid #bbb;height:220px;overflow:auto;padding:.5rem;font-size:12px;margin-top:1rem}
#info{color:#c00;margin:.5rem 0;font-weight:bold}
.high-contrast * { background-color: black !important; color: yellow !important; border-color: yellow !important; }
</style>

<!-- ───────── ENTRAR ───────── -->
<div id="join" class="mb-3">
  <input id="name"  class="form-control d-inline w-auto" placeholder="Seu nome">
  <input id="roomIn" class="form-control d-inline w-auto" placeholder="Código (vazio = nova)">
  <button id="btnJoin" class="btn btn-primary">Entrar / Reconectar</button>
</div>

<!-- ───────── LOBBY (apenas botões) ───────── -->
<div id="lobby">
  <h3>Sala: <span id="roomCode"></span></h3>
  <button id="btnReady" class="btn btn-success btn-sm">PRONTO</button>
  <button id="btnLeave" class="btn btn-outline-danger btn-sm">SAIR</button>
  <!-- NOVO: Botões adicionais -->
  <button id="btnStatus" class="btn btn-info btn-sm">Mostrar Tudo</button>
  <button id="btnAcess" class="btn btn-warning btn-sm">Acessibilidade</button>
</div>

<!-- ───────── LISTA DE JOGADORES (visível sempre) ───────── -->
<div id="playersWrap" style="display:none">
  <h4>Jogadores</h4>
  <section id="players"></section>
</div>

<!-- ───────── PARTIDA ───────── -->
<div id="game" class="mt-3">
  <h2 id="me"></h2>
  <div id="hand" class="mb-2"></div>
  <div id="info"></div>

  <button id="add"    class="btn btn-success btn-sm">+1 moeda</button>
  <button id="rem"    class="btn btn-danger  btn-sm">-1 moeda</button>
  <button id="lose"   class="btn btn-warning btn-sm">Perder influência</button>
  <button id="reveal" class="btn btn-warning btn-sm">Revelar &amp; trocar</button>
  <button id="amb"    class="btn btn-info    btn-sm">Troca (Embaixador)</button>
  <button id="btnLeaveGame" class="btn btn-outline-danger btn-sm">SAIR</button>
</div>

<!-- ───────── HISTÓRICO ───────── -->
<h4 id="histTitle" class="mt-3" style="display:none">Histórico</h4>
<div id="history"></div>

<script>
const CARD_IMG = @json($cardImages);
let ws=null, roomId='', pid='', myName='';
let myHand=[], myReady=false, mode=null, pending=[];
const revealedByPid = {};
const nameToPid     = {};
let   playersState  = {};

// NOVO: Mostrar status no console
document.getElementById('btnStatus').onclick = () => {
  console.log("🚨 Status completo do jogo:", { playersState, myHand, roomId, pid });
  alert("Status do jogo foi exibido no console.");
};

// NOVO: Alternar modo de acessibilidade
document.getElementById('btnAcess').onclick = () => {
  document.body.classList.toggle('high-contrast');
};

// Reconexão automática
const sRoom = localStorage.getItem('coup_room');
const sPid  = localStorage.getItem('coup_pid');
if (sRoom && sPid) connectWS({roomId:sRoom, pid:sPid});

document.getElementById('btnJoin').onclick = () => {
  if (ws) return;
  myName = document.getElementById('name').value.trim() || 'Você';
  const code = document.getElementById('roomIn').value.trim();
  connectWS({name:myName, roomId:code||undefined});
};

document.getElementById('btnLeave').onclick =
document.getElementById('btnLeaveGame').onclick = leaveRoom;

function leaveRoom(){
  if (ws && ws.readyState===1) ws.send(JSON.stringify({type:'leave'}));
  if (ws) { ws.close(); ws=null; }
  localStorage.removeItem('coup_room'); localStorage.removeItem('coup_pid');
  roomId=''; pid=''; myHand=[]; myReady=false; mode=null; pending=[];
  Object.keys(revealedByPid).forEach(k=>delete revealedByPid[k]);
  ['lobby','game','playersWrap'].forEach(id=>document.getElementById(id).style.display='none');
  document.getElementById('join').style.display='flex';
  document.getElementById('players').innerHTML='';
  document.getElementById('hand').innerHTML='';
  resetHistory(); setInfo('');
}

function connectWS(payload){
  ws = new WebSocket('ws://localhost:8080');
  ws.onopen    = () => ws.send(JSON.stringify({type:'join', ...payload}));
  ws.onmessage = e   => handle(JSON.parse(e.data));
  ws.onclose   = leaveRoom;
}

document.getElementById('btnReady').onclick = () => {
  myReady=!myReady;
  send({type:'ready', ready:myReady});
  toggleReadyBtn();
};
function toggleReadyBtn(){
  const b=document.getElementById('btnReady');
  b.classList.toggle('btn-success',!myReady);
  b.classList.toggle('btn-secondary',myReady);
  b.textContent=myReady?'Cancelado':'PRONTO';
}

document.getElementById('add').onclick =()=>send({type:'coinDelta',delta:1});
document.getElementById('rem').onclick =()=>send({type:'coinDelta',delta:-1});
document.getElementById('lose').onclick   =()=>beginSelect('lose','Escolha a carta a perder');
document.getElementById('reveal').onclick =()=>beginSelect('reveal','Clique na carta para revelar');
document.getElementById('amb').onclick    =()=>send({type:'ambassadorDraw'});

function handle(m){
  switch(m.type){
    case 'welcome':
      roomId=m.roomId; pid=m.pid; myHand=m.hand||[];
      localStorage.setItem('coup_room',roomId);
      localStorage.setItem('coup_pid', pid);
      resetHistory();
      playersState=m.players; if(m.history) m.history.forEach(addHistory);
      renderPlayers(playersState); showLobby();
      break;
    case 'players': playersState = m.players; renderPlayers(playersState); break;
    case 'start': myHand=m.hand; playersState=m.players; renderPlayers(playersState); renderHand(); showGame(); clearMode(); break;
    case 'hand': myHand=m.hand; renderHand(); clearMode(); break;
    case 'ambassador': myHand=m.hand; renderHand(); beginSelect('amb','Selecione 2 cartas para devolver'); break;
    case 'coins': updateCoins(m.pid,m.coins); break;
    case 'cards': updateCards(m.pid,m.cards,m.alive); break;
    case 'history': addHistory(m.entry); break;
  }
}

function beginSelect(newMode,msg){ mode=newMode; pending=[]; setInfo(msg); highlight(true); }
function cardClick(card,img){
  if(!mode) {
    // NOVO: tocar áudio ao clicar na carta
    const audio = new Audio(`/audios/${card}.mp3`);
    audio.play().catch(()=>{});
    return;
  }
  if(['lose','reveal'].includes(mode)){
    send({type:mode==='lose'?'loseCard':'revealCard',card}); clearMode(); return;
  }
  if(img.classList.contains('sel')){ img.classList.remove('sel'); pending.splice(pending.indexOf(card),1);}
  else if(pending.length<2){ img.classList.add('sel'); pending.push(card); }
  if(pending.length===2){ send({type:'ambassadorReturn',return:pending}); clearMode(); }
}
function clearMode(){ mode=null; pending=[]; setInfo(''); highlight(false);}
function highlight(on){document.querySelectorAll('#hand .card').forEach(el=>{el.classList.remove('sel');el.style.cursor=on?'pointer':'default';});}

function renderHand(){
  const box=document.getElementById('hand'); box.innerHTML='';
  myHand.forEach(c=>{
    const img=document.createElement('img');
    img.src=CARD_IMG[c]||''; img.alt=c; img.width=80; img.className='card';
    img.onclick=()=>cardClick(c,img);
    box.appendChild(img);
  });
}
function renderPlayers(players){
  const area=document.getElementById('players'); area.innerHTML='';
  Object.entries(players).forEach(([id,p])=>{
    nameToPid[p.name]=id;
    const revealed=revealedByPid[id]||[];
    area.insertAdjacentHTML('beforeend',`
      <div data-pid="${id}">
        <strong>${p.name}${id===pid?' (você)':''}</strong> ${p.ready?'✅':'⏳'}<br>
        moedas: <span class="coins">${p.coins}</span><br>
        cartas restantes: <span class="cards">${p.cards}</span><br>
        vivo: <span class="alive">${p.alive}</span><br>
        <div class="revealed">${revealed.map(c=>'<img src="'+CARD_IMG[c]+'" alt="'+c+'">').join('')}</div>
      </div>`);
    if(id===pid){
      myReady=p.ready; myName=p.name; toggleReadyBtn();
      document.getElementById('me').textContent=`${p.name} – moedas: ${p.coins}`;
    }
  });
  playersState = players;
}
function updateCoins(id,val){
  const span=document.querySelector(`[data-pid="${id}"] .coins`);
  if(span) span.textContent=val;
  if(playersState[id]) playersState[id].coins=val;
  if(id===pid) document.getElementById('me').textContent=`${myName} – moedas: ${val}`;
}
function updateCards(id,cards,alive){
  const wrap=document.querySelector(`[data-pid="${id}"]`);
  if(wrap){
    wrap.querySelector('.cards').textContent=cards;
    wrap.querySelector('.alive').textContent=alive;
  }
  if(playersState[id]){ playersState[id].cards=cards; playersState[id].alive=alive; }
}
function resetHistory(){ document.getElementById('history').innerHTML=''; document.getElementById('histTitle').style.display='none';}
function addHistory(e){
  const h = document.getElementById('history');
  h.insertAdjacentHTML('beforeend',
      `<div>[${new Date(e.t).toLocaleTimeString()}] ${e.text}</div>`);
  h.scrollTop = h.scrollHeight;
  document.getElementById('histTitle').style.display = 'block';
  const m = e.text.match(/(.+?) perdeu (\w+)/i);
  if(!m) return;
  const pidT = nameToPid[m[1].trim()];
  if(!pidT) return;
  const card = m[2].toLowerCase();
  const arr  = revealedByPid[pidT] = revealedByPid[pidT] || [];
  if(!arr.includes(card)) arr.push(card);
  renderPlayers(playersState);
}
function send(o){ ws&&ws.readyState===1&&ws.send(JSON.stringify(o)); }
function setInfo(t){ document.getElementById('info').textContent=t; }
function showLobby(){
  document.getElementById('join').style.display='none';
  document.getElementById('lobby').style.display='block';
  document.getElementById('playersWrap').style.display='block';
  document.getElementById('roomCode').textContent=roomId;
}
function showGame(){
  document.getElementById('game').style.display='block';
  document.getElementById('lobby').style.display='block';
  document.getElementById('playersWrap').style.display='block';
}
</script>
@endsection

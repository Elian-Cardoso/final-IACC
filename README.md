@extends('layouts.app')

@section('content')
@php
    $cardImages = $cards->pluck('image', 'title'); // Ex: “duke” => “/storage/cards/duke.png”
@endphp

<style>
body { font: 15px/1.4 sans-serif; margin: 2rem; }
#join { display: flex; gap: .5rem; flex-wrap: wrap; }
#lobby, #game { display: none; }
button { margin: .25rem .5rem .25rem 0; }
#playersWrap { margin-top: 1rem; }
#players { display: flex; gap: 1rem; flex-wrap: wrap; }
#players div { border: 1px solid #888; border-radius: 4px; padding: .5rem; min-width: 170px; }
.revealed img { width: 34px; margin-right: 2px; }
.card { cursor: pointer; margin: 4px; border: 1px solid #444; border-radius: 4px; transition: transform 0.2s; }
.card.sel { background: #ffe; }
.card:hover { transform: scale(1.05); }
#history { border: 1px solid #bbb; height: 220px; overflow: auto; padding: .5rem; font-size: 12px; margin-top: 1rem; }
#info { color: #c00; margin: .5rem 0; font-weight: bold; }
.high-contrast * { background-color: black !important; color: yellow !important; border-color: yellow !important; }
</style>

<!-- ───────── ENTRAR ───────── -->
<div id="join" class="mb-3">
  <input id="name" class="form-control d-inline w-auto" placeholder="Seu nome">
  <input id="roomIn" class="form-control d-inline w-auto" placeholder="Código (vazio = nova)">
  <button id="btnJoin" class="btn btn-primary">Entrar / Reconectar</button>
</div>

<!-- ───────── LOBBY ───────── -->
<div id="lobby">
  <h3>Sala: <span id="roomCode"></span></h3>
  <button id="btnReady" class="btn btn-success btn-sm">PRONTO</button>
  <button id="btnLeave" class="btn btn-outline-danger btn-sm">SAIR</button>
  <button id="btnStatus" class="btn btn-info btn-sm">Mostrar Tudo</button>
  <button id="btnAcess" class="btn btn-warning btn-sm">Acessibilidade</button>
</div>

<!-- ───────── LISTA DE JOGADORES ───────── -->
<div id="playersWrap" style="display:none">
  <h4>Jogadores</h4>
  <section id="players"></section>
</div>

<!-- ───────── PARTIDA ───────── -->
<div id="game" class="mt-3">
  <h2 id="me"></h2>
  <div id="hand" class="mb-2"></div>
  <div id="info"></div>

  <button id="add" class="btn btn-success btn-sm">+1 moeda</button>
  <button id="rem" class="btn btn-danger btn-sm">-1 moeda</button>
  <button id="lose" class="btn btn-warning btn-sm">Perder influência</button>
  <button id="reveal" class="btn btn-warning btn-sm">Revelar &amp; trocar</button>
  <button id="amb" class="btn btn-info btn-sm">Troca (Embaixador)</button>
  <button id="btnLeaveGame" class="btn btn-outline-danger btn-sm">SAIR</button>
</div>

<!-- ───────── HISTÓRICO ───────── -->
<h4 id="histTitle" class="mt-3" style="display:none">Histórico</h4>
<div id="history"></div>

<script>
const CARD_IMG = @json($cardImages);
let ws = null, roomId = '', pid = '', myName = '';
let myHand = [], myReady = false, mode = null, pending = [];
const revealedByPid = {}, nameToPid = {};
let playersState = {};

// Botão mostrar status
document.getElementById('btnStatus').onclick = () => {
  console.log("🚨 Status completo do jogo:", { playersState, myHand, roomId, pid });
  alert("Status do jogo foi exibido no console.");
};

// Botão acessibilidade
document.getElementById('btnAcess').onclick = () => {
  document.body.classList.toggle('high-contrast');
};

const sRoom = localStorage.getItem('coup_room');
const sPid = localStorage.getItem('coup_pid');
if (sRoom && sPid) connectWS({ roomId: sRoom, pid: sPid });

document.getElementById('btnJoin').onclick = () => {
  if (ws) return;
  myName = document.getElementById('name').value.trim() || 'Você';
  const code = document.getElementById('roomIn').value.trim();
  connectWS({ name: myName, roomId: code || undefined });
};

document.getElementById('btnLeave').onclick =
document.getElementById('btnLeaveGame').onclick = leaveRoom;

function leaveRoom() {
  if (ws && ws.readyState === 1) ws.send(JSON.stringify({ type: 'leave' }));
  if (ws) { ws.close(); ws = null; }
  localStorage.removeItem('coup_room');
  localStorage.removeItem('coup_pid');
  roomId = ''; pid = ''; myHand = []; myReady = false; mode = null; pending = [];
  Object.keys(revealedByPid).forEach(k => delete revealedByPid[k]);
  ['lobby', 'game', 'playersWrap'].forEach(id => document.getElementById(id).style.display = 'none');
  document.getElementById('join').style.display = 'flex';
  document.getElementById('players').innerHTML = '';
  document.getElementById('hand').innerHTML = '';
  resetHistory(); setInfo('');
}

function connectWS(payload) {
  ws = new WebSocket('ws://localhost:8080');
  ws.onopen = () => ws.send(JSON.stringify({ type: 'join', ...payload }));
  ws.onmessage = e => handle(JSON.parse(e.data));
  ws.onclose = leaveRoom;
}

document.getElementById('btnReady').onclick = () => {
  myReady = !myReady;
  send({ type: 'ready', ready: myReady });
  toggleReadyBtn();
};

function toggleReadyBtn() {
  const b = document.getElementById('btnReady');
  b.classList.toggle('btn-success', !myReady);
  b.classList.toggle('btn-secondary', myReady);
  b.textContent = myReady ? 'Cancelado' : 'PRONTO';
}

document.getElementById('add').onclick = () => { send({ type: 'coinDelta', delta: 1 }); playSound('coin.mp3'); };
document.getElementById('rem').onclick = () => send({ type: 'coinDelta', delta: -1 });
document.getElementById('lose').onclick = () => beginSelect('lose', 'Escolha a carta a perder');
document.getElementById('reveal').onclick = () => {
  send({ type: 'revealCard' });
  playSound('reveal.mp3');
  playSound('voice-perdeuduque.mp3');
};
document.getElementById('amb').onclick = () => send({ type: 'ambassadorDraw' });

function handle(m) {
  switch (m.type) {
    case 'welcome':
      roomId = m.roomId; pid = m.pid; myHand = m.hand || [];
      localStorage.setItem('coup_room', roomId);
      localStorage.setItem('coup_pid', pid);
      resetHistory(); playersState = m.players;
      if (m.history) m.history.forEach(addHistory);
      renderPlayers(playersState); showLobby();
      break;
    case 'players': playersState = m.players; renderPlayers(playersState); break;
    case 'start': myHand = m.hand; playersState = m.players; renderPlayers(playersState); renderHand(); showGame(); clearMode(); break;
    case 'hand': myHand = m.hand; renderHand(); clearMode(); break;
    case 'ambassador': myHand = m.hand; renderHand(); beginSelect('amb', 'Selecione 2 cartas para devolver'); break;
    case 'coins': updateCoins(m.pid, m.coins); break;
    case 'cards': updateCards(m.pid, m.cards, m.alive); break;
    case 'history': addHistory(m.entry); break;
  }
}

function beginSelect(newMode, msg) {
  mode = newMode; pending = []; setInfo(msg); highlight(true);
}

function cardClick(card, img) {
  if (!mode) {
    playRandomVoice(card);
    return;
  }
  if (['lose', 'reveal'].includes(mode)) {
    send({ type: mode === 'lose' ? 'loseCard' : 'revealCard', card });
    clearMode(); return;
  }
  if (img.classList.contains('sel')) {
    img.classList.remove('sel');
    pending.splice(pending.indexOf(card), 1);
  } else if (pending.length < 2) {
    img.classList.add('sel');
    pending.push(card);
  }
  if (pending.length === 2) {
    send({ type: 'ambassadorReturn', return: pending });
    clearMode();
  }
}

function renderHand() {
  const box = document.getElementById('hand'); box.innerHTML = '';
  myHand.forEach(c => {
    const img = document.createElement('img');
    img.src = CARD_IMG[c] || ''; img.alt = c; img.width = 80; img.className = 'card';
    img.onclick = () => cardClick(c, img);
    box.appendChild(img);
  });
}

function renderPlayers(players) {
  const area = document.getElementById('players'); area.innerHTML = '';
  Object.entries(players).forEach(([id, p]) => {
    nameToPid[p.name] = id;
    const revealed = revealedByPid[id] || [];
    area.insertAdjacentHTML('beforeend', `
      <div data-pid="${id}">
        <strong>${p.name}${id === pid ? ' (você)' : ''}</strong> ${p.ready ? '✅' : '⏳'}<br>
        moedas: <span class="coins">${p.coins}</span><br>
        cartas restantes: <span class="cards">${p.cards}</span><br>
        vivo: <span class="alive">${p.alive}</span><br>
        <div class="revealed">${revealed.map(c => '<img src="' + CARD_IMG[c] + '" alt="' + c + '">').join('')}</div>
      </div>`);
    if (id === pid) {
      myReady = p.ready; myName = p.name; toggleReadyBtn();
      document.getElementById('me').textContent = `${p.name} – moedas: ${p.coins}`;
    }
  });
  playersState = players;
}

function updateCoins(id, val) {
  const span = document.querySelector(`[data-pid="${id}"] .coins`);
  if (span) span.textContent = val;
  if (playersState[id]) playersState[id].coins = val;
  if (id === pid) document.getElementById('me').textContent = `${myName} – moedas: ${val}`;
}

function updateCards(id, cards, alive) {
  const wrap = document.querySelector(`[data-pid="${id}"]`);
  if (wrap) {
    wrap.querySelector('.cards').textContent = cards;
    wrap.querySelector('.alive').textContent = alive ? 'Sim' : 'Não';
  }
}

function playSound(file) {
  const audio = new Audio(`/audios/${file}`);
  audio.play().catch(e => console.warn("Falha ao reproduzir:", file, e));
}

function playRandomVoice(card) {
  const random = Math.floor(Math.random() * 5) + 1;
  const filename = `${card}_main_${random}.mp3`;
  playSound(filename);
}
</script>
@endsection

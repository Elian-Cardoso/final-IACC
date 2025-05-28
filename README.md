<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Você achou o segredo... 💖</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <div class="container">

    <!-- Abertura -->
    <section class="intro">
      <h1>Oi, Amor</h1>
      <p>Se eu fosse um código... você seria meu ponto e vírgula — porque sem você, eu travo 😵‍💫</p>
      <button id="startBtn">Clique aqui pra continuar essa maluquice 💌</button>
    </section>

    <!-- Carta escondida -->
    <section class="card hidden" id="loveCard">
      <h2>Uma carta só pra você 💗</h2>
      <div class="envelope" id="envelope">
        <p>Clique para abrir ✉</p>
      </div>
      <div class="message hidden" id="message">
        <p>
          Hoje é seu dia, mas é como se o universo tivesse me dado o maior presente: <strong>você</strong>.<br>
          Você é maluca, cheirosa, engraçada (quando tenta KKK) e é o motivo dos meus dias terem cor e bug.<br>
          Obrigado por existir. Feliz aniversário + Dia dos Namorados!<br><br>
          Com amor,<br>Seu programador preferido 💻❤
        </p>
      </div>
      <button id="startGameBtn" class="hidden">Começar jogo dos balões 🎈</button>
    </section>

    <!-- Jogo dos balões -->
    <section class="balloon-game hidden" id="balloonGame">
      <h2>🎈 Estoure os 22 balões pra liberar seu presente!</h2>
      <p id="balloonCounter">Balões estourados: 0/22</p>
      <p id="timerDisplay">Tempo restante: 60s</p>
      <div id="balloonContainer"></div>
    </section>

    <!-- Presente escondido -->
    <section class="gift hidden" id="giftSection">
      <h2>🎁 Tem presente sim, minha MORENA</h2>
      <p>Clique no botão e descubra!</p>
      <button id="revealGift">Mostrar presente 🎉</button>
      <div id="giftImage" class="hidden">
        <img src="vale-boticario.png" alt="Vale O Boticário">
        <p>Vale-perfume do Boticário, pra você sair mais cheirosa do que já é (o que eu achava impossível KKK)</p>
      </div>
    </section>

  </div>

  <script src="script.js"></script>
</body>
</html>



body {
  margin: 0;
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(135deg, #ffe4ec, #f8d7ff);
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

.container {
  max-width: 600px;
  padding: 20px;
  background-color: #fff;
  border-radius: 20px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.1);
  text-align: center;
}

h1, h2 {
  color: #ff4081;
}

p {
  font-size: 1.1em;
  color: #333;
}

button {
  background-color: #ff4081;
  color: white;
  border: none;
  padding: 12px 20px;
  margin-top: 15px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 1em;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #e91e63;
}

.hidden {
  display: none;
}

.envelope {
  background-color: #ffcdd2;
  padding: 20px;
  border-radius: 10px;
  cursor: pointer;
  margin: 20px 0;
  font-weight: bold;
  color: #880e4f;
}

.message {
  background-color: #fff0f5;
  padding: 15px;
  border: 1px dashed #ff80ab;
  border-radius: 10px;
  margin-top: 10px;
}

/* Balões */
#balloonContainer {
  position: relative;
  width: 100%;
  height: 350px;
  overflow: hidden;
  margin-top: 20px;
  background: #fef6fb;
  border-radius: 15px;
}

.balloon {
  width: 60px;
  height: 80px;
  background-color: red;
  border-radius: 50% 50% 45% 45%;
  position: absolute;
  cursor: pointer;
  animation: floatUp 8s linear forwards;
  transition: transform 0.2s;
  box-shadow: 0 3px 6px rgba(0, 0, 0, 0.15);
}

.balloon:hover {
  transform: scale(1.15);
}

.balloon::after {
  content: '';
  position: absolute;
  bottom: -10px;
  left: 50%;
  width: 2px;
  height: 10px;
  background: #555;
  transform: translateX(-50%);
  border-radius: 1px;
}

@keyframes floatUp {
  from {
    transform: translateY(0);
    opacity: 1;
  }
  to {
    transform: translateY(-100vh);
    opacity: 0;
  }
}

/* Confete */
.confetti {
  position: absolute;
  width: 6px;
  height: 6px;
  animation: fallConfetti 1s ease-out forwards;
  opacity: 0.9;
  z-index: 1000;
  border-radius: 50%;
}

@keyframes fallConfetti {
  0% {
    transform: translateY(0) rotate(0deg);
    opacity: 1;
  }
  100% {
    transform: translateY(100px) rotate(360deg);
    opacity: 0;
  }
}

/* Responsivo */
@media (max-width: 480px) {
  .balloon {
    width: 50px;
    height: 70px;
  }

  #balloonContainer {
    height: 300px;
  }
}

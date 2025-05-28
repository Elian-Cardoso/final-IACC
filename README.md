document.addEventListener("DOMContentLoaded", function () {
  const startBtn = document.getElementById("startBtn");
  const loveCard = document.getElementById("loveCard");
  const envelope = document.getElementById("envelope");
  const message = document.getElementById("message");
  const startGameBtn = document.getElementById("startGameBtn");
  const giftSection = document.getElementById("giftSection");
  const revealGift = document.getElementById("revealGift");
  const giftImage = document.getElementById("giftImage");

  const balloonGame = document.getElementById("balloonGame");
  const balloonCounter = document.getElementById("balloonCounter");
  const balloonContainer = document.getElementById("balloonContainer");

  const timerDisplay = document.createElement("p");
  timerDisplay.id = "timerDisplay";
  timerDisplay.style.fontWeight = "bold";
  balloonGame.insertBefore(timerDisplay, balloonContainer);

  let popped = 0;
  const goal = 22;
  let timeLeft = 60;
  let timer;
  let balloonInterval;

  startBtn.addEventListener("click", function () {
    document.querySelector(".intro").classList.add("hidden");
    loveCard.classList.remove("hidden");
  });

  envelope.addEventListener("click", function () {
    envelope.classList.add("hidden");
    message.classList.remove("hidden");
    startGameBtn.classList.remove("hidden");
  });

  startGameBtn.addEventListener("click", function () {
    startGameBtn.classList.add("hidden");
    loveCard.classList.add("hidden");
    balloonGame.classList.remove("hidden");
    startBalloonGame();
  });

  function startBalloonGame() {
    popped = 0;
    timeLeft = 60;
    balloonCounter.textContent = `Balões estourados: 0/${goal}`;
    timerDisplay.textContent = `Tempo restante: ${timeLeft}s`;
    balloonContainer.innerHTML = "";

    clearInterval(timer);
    clearInterval(balloonInterval);

    timer = setInterval(updateTimer, 1000);

    balloonInterval = setInterval(function () {
      if (popped < goal) {
        createBalloon();
      }
    }, 1000);
  }

  function updateTimer() {
    timeLeft--;
    timerDisplay.textContent = `Tempo restante: ${timeLeft}s`;
    if (timeLeft <= 0) {
      clearInterval(timer);
      clearInterval(balloonInterval);
      if (popped < goal) {
        alert("Oooown... acabou o tempo 🥺 Tenta de novo, vai que agora vai! 💖");
        startBalloonGame();
      }
    }
  }

  function createBalloon() {
    if (popped >= goal) return;

    const balloon = document.createElement("div");
    balloon.classList.add("balloon");

    const colors = ["#FF0000", "#0000FF", "#00FF00", "#FFFF00", "#FF7F00", "#800080"];
    const randomColor = colors[Math.floor(Math.random() * colors.length)];
    balloon.style.backgroundColor = randomColor;

    balloon.style.left = `${Math.random() * 90}%`;
    balloon.style.bottom = "-70px";
    const speed = Math.random() * 4 + 7;
    balloon.style.animation = `floatUp ${speed}s linear forwards`;

    balloon.addEventListener("click", function () {
      const balloonRect = balloon.getBoundingClientRect();
      createConfetti(balloonRect.left, balloonRect.top);
      balloon.remove();
      popped++;
      balloonCounter.textContent = `Balões estourados: ${popped}/${goal}`;

      if (popped >= goal) {
        clearInterval(timer);
        clearInterval(balloonInterval);
        setTimeout(function () {
          balloonGame.classList.add("hidden");
          giftSection.classList.remove("hidden");
        }, 800);
      }
    });

    balloonContainer.appendChild(balloon);
  }

  function createConfetti(x, y) {
    for (let i = 0; i < 15; i++) {
      const confetti = document.createElement("div");
      confetti.classList.add("confetti");
      const colors = ["#ff4081", "#00e676", "#448aff", "#ffeb3b"];
      confetti.style.backgroundColor = colors[i % colors.length];
      confetti.style.left = `${x + (Math.random() * 40 - 20)}px`;
      confetti.style.top = `${y}px`;
      confetti.style.position = "fixed";
      confetti.style.width = "8px";
      confetti.style.height = "8px";
      confetti.style.borderRadius = "50%";
      confetti.style.zIndex = 1000;
      confetti.style.animation = `fall 1s ease-out`;

      document.body.appendChild(confetti);

      setTimeout(() => confetti.remove(), 1000);
    }
  }

  revealGift.addEventListener("click", function () {
    giftImage.classList.remove("hidden");
    revealGift.classList.add("hidden");
  });

  const style = document.createElement("style");
  style.innerHTML = `
    @keyframes floatUp {
      0% { transform: translateY(0); opacity: 1; }
      100% { transform: translateY(-100vh); opacity: 0; }
    }

    @keyframes fall {
      0% { transform: translateY(0); opacity: 1; }
      100% { transform: translateY(100px); opacity: 0; }
    }
  `;
  document.head.appendChild(style);
});

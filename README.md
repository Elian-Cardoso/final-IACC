<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Coup 2 - Jogo</title>
    <style>
        .card {
            display: inline-block;
            width: 120px;
            height: 180px;
            border: 1px solid #ccc;
            margin: 10px;
            background-color: #f9f9f9;
            cursor: pointer;
            background-size: cover;
            background-position: center;
        }

        .hidden {
            display: none;
        }

        .showing {
            border: 2px solid #007bff;
        }
    </style>
</head>
<body>
    <h1>Coup 2 - Jogo</h1>

    <button onclick="toggleMode()">Modo: <span id="modeLabel">Som</span></button>
    <button onclick="toggleAll()">Mostrar Tudo</button>
    <button onclick="toggleAccessibility()">Acessibilidade</button>

    <div id="cards">
        @foreach (['duke', 'assassin', 'captain', 'ambassador', 'contessa'] as $card)
            <div class="card" onclick="cardClick('{{ $card }}', this)" id="card-{{ $card }}" style="background-image: url('/images/{{ $card }}.jpg')"></div>
        @endforeach
    </div>

    <audio id="audio" class="hidden"></audio>

    <script>
        let mode = false; // false = som, true = mostrar imagem
        let allVisible = false;
        let accessibility = false;

        function toggleMode() {
            mode = !mode;
            document.getElementById('modeLabel').innerText = mode ? 'Imagem' : 'Som';
        }

        function toggleAll() {
            allVisible = !allVisible;
            const cards = document.querySelectorAll('.card');
            cards.forEach(card => {
                if (allVisible) {
                    card.classList.add('showing');
                } else {
                    card.classList.remove('showing');
                }
            });
        }

        function toggleAccessibility() {
            accessibility = !accessibility;
            const cards = document.querySelectorAll('.card');
            cards.forEach(card => {
                card.style.filter = accessibility ? 'grayscale(100%) contrast(200%)' : 'none';
            });
        }

        function cardClick(card, img) {
            if (!mode) {
                playSound(`${card}.mp3`);
                playCharacterVoice(card); // ← Adicionado para vozes aleatórias
                return;
            }

            if (img.classList.contains('showing')) {
                img.classList.remove('showing');
            } else {
                img.classList.add('showing');
            }
        }

        function playSound(filename) {
            const audio = document.getElementById('audio');
            audio.src = `/audios/${filename}`;
            audio.play();
        }

        // ───── ÁUDIOS DE VOZ DINÂMICOS ─────
        function playCharacterVoice(character) {
            const total = 5;
            const voiceIndex = Math.floor(Math.random() * total) + 1;
            const filename = `${character}_main_${voiceIndex}.mp3`;
            playSound(filename);
        }
    </script>
</body>
</html>

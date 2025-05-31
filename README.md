// Toca uma das falas de voz do personagem, se existir
function playCharacterVoice(character) {
  const total = 5; // temos 5 arquivos por personagem
  const index = Math.floor(Math.random() * total) + 1;
  const filename = `${character}_main_${index}.mp3`;
  playSound(filename);
}

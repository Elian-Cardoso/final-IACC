function playRandomVoice(card) {
  const random = Math.floor(Math.random() * 5) + 1; // 1 a 5
  const filename = `${card}_main_${random}.mp3`;
  playSound(filename);
}

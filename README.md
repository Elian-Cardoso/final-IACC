function updateCards(id,cards,alive){
  const wrap=document.querySelector(`[data-pid="${id}"]`);
  if(wrap){
    wrap.querySelector('.cards').textContent = cards;
    wrap.querySelector('.alive').textContent = alive ? 'Sim' : 'Não';
  }
}

// ⬇️ COLA AQUI ABAIXO ⬇️

function playSound(file) {
  const audio = new Audio(`/audios/${file}`);
  audio.play().catch(e => console.warn("Falha ao reproduzir:", file, e));
}

function playRandomVoice(card) {
  const random = Math.floor(Math.random() * 5) + 1; // Gera número de 1 a 5
  const filename = `${card}_main_${random}.mp3`;
  playSound(filename);
}

// ⬆️ ANTES do </script> ⬆️
</script>

function cardClick(card, img){
  if(!mode) {
    playCharacterVoice(card); // toca fala aleatória do personagem
    playSound(`${card}.mp3`); // toca efeito sonoro padrão, se existir
    return;
  }
  if(['lose','reveal'].includes(mode)){
    send({type:mode==='lose'?'loseCard':'revealCard',card});
    clearMode();
    return;
  }
  if(img.classList.contains('sel')){
    img.classList.remove('sel');
    pending.splice(pending.indexOf(card), 1);
  } else if(pending.length < 2){
    img.classList.add('sel');
    pending.push(card);
  }
  if(pending.length === 2){
    send({type:'ambassadorReturn', return:pending});
    clearMode();
  }
}

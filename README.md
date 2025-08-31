<script>
const CLAVE = '09/09/2006';
const $ = (s)=>document.querySelector(s);

document.addEventListener('DOMContentLoaded', () => {
  // ---- refs
  const lockScreen = $('#lockScreen');
  const inputClave = $('#claveInput');
  const togglePwd = $('#togglePwd');
  const unlockBtn = $('#unlockBtn');
  const errorMsg = $('#errorMsg');
  const mainContent = $('#mainContent');
  const giftWrapper = $('#giftWrapper');
  const giftBox = $('#giftBox');
  const letter = $('#letter');
  const typedText = $('#typedText');
  const videoPlayer = $('#videoPlayer');
  const openSound = $('#openSound');
  const bgAudio = $('#bgAudio');
  const btnClose = $('#btnClose');
  const numpad = document.getElementById('numpad');

  // ---- teclado numérico robusto (móvil)
  inputClave.value = '';                 // comienza vacío
  numpad.style.touchAction = 'manipulation'; // evita zoom/doble tap en iOS
  let lastTs = 0;

  function handleKey(k){
    if(k==='BACK'){ inputClave.value = inputClave.value.slice(0,-1); return; }
    if(inputClave.value.length>=10) return;
    if(/[0-9]/.test(k)){
      let v = inputClave.value;
      if(v.length===2 || v.length===5) v += '/';
      inputClave.value = v + k;
    }else if(k==='/'){
      if(inputClave.value.length===2 || inputClave.value.length===5)
        inputClave.value += '/';
    }
  }

  function press(e){
    const key = e.target.closest('.np-key');
    if(!key) return;
    const now = e.timeStamp || Date.now();

    // evita doble disparo (touchstart/pointerdown + click fantasma)
    if(now - lastTs < 180){ e.preventDefault(); return; }
    lastTs = now;

    e.preventDefault();
    handleKey(key.dataset.k);
    key.classList.add('pressed');
    setTimeout(()=>key.classList.remove('pressed'),120);
  }

  // usa pointerdown y touchstart; click solo de respaldo
  numpad.addEventListener('pointerdown', press, {passive:false});
  numpad.addEventListener('touchstart',  press, {passive:false});
  numpad.addEventListener('click', (e)=>{
    if((e.timeStamp||Date.now()) - lastTs < 250) return;
    press(e);
  }, {passive:false});

  // mostrar/ocultar clave
  togglePwd.addEventListener('click',()=>{
    inputClave.type = (inputClave.type === 'text') ? 'password' : 'text';
  });

  // verificación
  function verificarClave(){
    if(inputClave.value.trim() === CLAVE){
      lockScreen.style.display = 'none';
      mainContent.style.display = 'block';
    }else{
      errorMsg.textContent = 'Código incorrecto. Inténtalo de nuevo.';
      lockScreen.querySelector('.panel').animate(
        [{transform:'translateX(0)'},{transform:'translateX(-6px)'},
         {transform:'translateX(6px)'},{transform:'translateX(0)'}],
        {duration:300,easing:'ease-out'}
      );
    }
  }
  unlockBtn.addEventListener('click', verificarClave);

  // reproducibles tras interacción (autoplay-friendly)
  function desbloquearMediosUnaVez(){
    try{ bgAudio.play().then(()=>bgAudio.pause()).catch(()=>{}); }catch{}
    try{ videoPlayer.play().then(()=>videoPlayer.pause()).catch(()=>{}); }catch{}
    document.removeEventListener('click', desbloquearMediosUnaVez, {capture:true});
    document.removeEventListener('touchstart', desbloquearMediosUnaVez, {capture:true});
  }
  document.addEventListener('click', desbloquearMediosUnaVez, {capture:true, once:true});
  document.addEventListener('touchstart', desbloquearMediosUnaVez, {capture:true, once:true});

  // abrir regalo
  function abrirRegalo(){
    giftBox.classList.add('open');
    openSound.play().catch(()=>{});
    setTimeout(()=>{
      giftWrapper.style.display = 'none';
      letter.classList.add('show');
      typeWriter();
      fireWorks();
      bgAudio.play().catch(()=>{});
      videoPlayer.muted = true; // mantenlo silenciado si quieres
      videoPlayer.play().catch(()=>{});
    }, 1100);
  }
  giftWrapper.addEventListener('click', abrirRegalo);

  // cerrar carta
  function cerrarCarta(){
    try{ videoPlayer.pause(); }catch{}
    try{ bgAudio.pause(); }catch{}
    letter.classList.remove('show');
  }
  btnClose.addEventListener('click', cerrarCarta);

  // ---- carta (typewriter + fuegos)
  const text = `Hoy, 9 de septiembre, celebramos tus 19 años, y con ellos también celebro a la persona tan especial que eres y todo lo que significas en mi vida. Cumplir años no es solo sumar días, es crecer, aprender y dejar huellas en los demás… y tú, Aracely, has dejado huellas imborrables en mí. 🌷✨

Cada momento contigo se ha convertido en un recuerdo único. Como aquella vez que aprendiste a manejar mi moto y te fuiste de frente; recuerdo el miedo que sentí pensando que te ibas a mandar al monte, y al mismo tiempo la risa que nos quedó de esa experiencia. O ese día en que, sabiendo que tenía sueño y debía exponer, me sorprendiste trayéndome una Monster para darme energía y apoyarme, un gesto que nunca olvidaré porque me mostró cuánto te importa.

También pienso en cuando me llevaste pastillas para la gastritis, justo en el momento en que lo necesitaba, o en esa salida a la playa donde casi quedamos atrapados en la arena movediza y terminamos riéndonos de la situación. Incluso algo tan simple como cuando me llevas en tu moto se transforma en especial, porque contigo hasta el camino más sencillo tiene otro sentido.

Y cómo olvidar ese detalle tan bonito de hace poco, cuando me mandaste la foto de una pulsera con mi nombre “Fer”… fue un gesto que me hizo sonreír y que guardo con mucho cariño, porque en esas pequeñas cosas se refleja lo especial que eres.

Aracely, hoy cumples 19 años y deseo que este nuevo ciclo esté lleno de alegrías, de metas alcanzadas, de abrazos que reconforten y de sueños que poco a poco se vayan haciendo realidad. 🌟 Porque eres alguien única, con una luz propia que inspira y que hace que todo sea más bonito.

Gracias por existir, por brillar con tu esencia y por regalarme momentos que se quedarán para siempre en mi corazón. 🌷✨

🌟 Que nunca falten motivos para sonreír, que la vida siempre te sorprenda con cosas hermosas y que cada día sea tan especial como tú, Aracely. 🌟

Con todo mi cariño,
💌`;

  let i = 0;
  function typeWriter(){
    if(i<text.length){
      const ch = text.charAt(i);
      typedText.innerHTML += ch === '\n' ? '<br>' : ch;
      i++;
      setTimeout(typeWriter, 36);
    }
  }

  function fireWorks(){
    for(let j=0;j<24;j++) setTimeout(spawnFirework, j*180);
  }
  function spawnFirework(){
    const colors=['#ff0','#f0f','#0ff','#0f0','#ff4081','#3f51b5'];
    const fw=document.createElement('div');
    fw.style.cssText='position:fixed;width:5px;height:5px;border-radius:50%;animation:boom 1.5s forwards;pointer-events:none';
    fw.style.left=Math.random()*100+'vw';
    fw.style.top=Math.random()*100+'vh';
    fw.style.background=colors[(Math.random()*colors.length)|0];
    document.body.appendChild(fw);
    setTimeout(()=>fw.remove(),1500);
  }
});
</script>

# escape-room
juego conjuntos numericos
<!DOCTYPE html>
  <html lang="es">
  <head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Escape Room: Conjuntos Numéricos - 10 Niveles</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js"></script>
  <style>
    *{box-sizing:border-box;margin:0;padding:0;font-family:'Orbitron',sans-serif}
    body{background:#4f52ee;color:#000000;overflow:hidden}
    #gameContainer{position:relative;width:100vw;height:100vh}
    .room{position:absolute;width:100%;height:100%;background-size:cover;background-position:center;
      display:none;justify-content:center;align-items:center;flex-direction:column;text-align:center;padding:20px}

    /* Intro */
    #introScreen{position:absolute;inset:0;display:flex;flex-direction:column;justify-content:center;align-items:center;
      gap:20px;background:linear-gradient(180deg,#000,#050018);z-index:40}
    #introScreen img{max-width:85%;max-height:68%;border-radius:12px;border:3px solid #00ffff;box-shadow:0 0 30px #00ffff}
    #introScreen .introText{font-size:1.2rem;color:#dfefff;max-width:90%;text-align:center}

    /* HUD */
    #hud{position:absolute;top:14px;left:18px;display:flex;gap:12px;align-items:center;z-index:35}
    #musicControl{background:rgba(0,0,0,0.6);border:1px solid #00ffff;padding:8px 12px;border-radius:10px;color:#fff;cursor:pointer}
    #timer{position:absolute;top:14px;right:18px;font-size:1.25rem;color:#00ffff;text-shadow:0 0 8px #00ffff;padding:6px 10px;border-radius:8px;border:1px solid rgba(0,255,255,0.08);z-index:35}
    #lives{position:absolute;top:14px;right:170px;display:flex;gap:8px;align-items:center;z-index:35}
    .heart{width:36px;height:36px;display:inline-block;background-size:contain;opacity:1;transition:opacity .3s,transform .2s}
    .heart.lost{opacity:0.25;transform:scale(.9);filter:grayscale(70%)}

    /* particle container */
    #particles { position: absolute; inset:0; pointer-events:none; z-index:36; }

    /* Puzzle box */
    #puzzleBox{background:rgba(0,0,0,0.78);border:2px solid #00ffff;border-radius:15px;padding:22px;width:min(820px,88%);text-align:center;backdrop-filter: blur(2px)}
    #puzzleBox h2{font-size:1.6rem;color:#00ffff;margin-bottom:8px}
    #puzzleBox p{font-size:1.35rem;line-height:1.6;color:#e6f0ff;margin-bottom:16px}
    input[type="text"], input[type="number"]{font-size:1.05rem;padding:12px;border-radius:10px;border:1px solid #8a2be2;background:#111;color:#fff;width:60%;max-width:320px}
    .btn{background:linear-gradient(90deg,#00ffff,#8a2be2);border:none;color:white;font-size:1.05rem;padding:10px 18px;border-radius:10px;cursor:pointer;margin-left:10px;box-shadow:0 6px 18px rgba(0,255,255,0.06)}
    .msg{margin-top:12px;font-size:1.05rem}

    /* Menu */
    #menu{display:flex;flex-direction:column;align-items:center;justify-content:center;gap:12px}
    h1{font-size:2.6rem;color:#00ffff;text-shadow:0 0 20px #00ffff;margin-bottom:6px}

    /* Game Over modal (retry screen) */
    #gameOverModal {
      position: absolute;
      inset: 0;
      display: none;
      justify-content: center;
      align-items: center;
      z-index: 50;
      background: linear-gradient(180deg, rgba(0,0,0,0.6), rgba(0,0,0,0.8));
    }
    #gameOverCard {
      background: #06061a;
      border: 2px solid #ff6b6b;
      padding: 26px;
      border-radius: 12px;
      text-align:center;
      width: min(520px, 92%);
      box-shadow: 0 10px 40px rgba(0,0,0,0.6);
    }
    #gameOverCard h3 { color:#ff6b6b; margin-bottom:8px; font-size:1.4rem }
    #gameOverCard p { color:#ffdede; margin-bottom:14px }
    #retryBtn { background:linear-gradient(90deg,#ff7b7b,#ff5f9f); padding:12px 18px; border-radius:10px; border:none; color:#fff; font-weight:600; cursor:pointer }
    #retryBtn[disabled]{opacity:0.5;cursor:default}

    /* Victory screen */
    #victoryScreen{position:absolute;inset:0;display:none;justify-content:center;align-items:center;z-index:60;background:linear-gradient(180deg,#001a00,#002b13);color:#e6ffe6}
    #victoryCard{background:rgba(0,0,0,0.5);padding:28px;border-radius:14px;border:2px solid #8cff9a;text-align:center}
    #victoryCard h2{font-size:2rem;margin-bottom:8px}

    /* Responsive: botones más grandes en móvil */
    @media (max-width:520px){
      #puzzleBox p{font-size:1.05rem}
      input[type="text"],input[type="number"]{width:82%}
      h1{font-size:1.9rem}
      #introScreen img{max-height:56%}
      .btn{padding:14px 20px;font-size:1.15rem}
      #musicControl{padding:10px 14px;font-size:1.05rem}
      #timer{font-size:1rem;padding:8px}
      .heart{width:40px;height:40px}
    }
  </style>
  </head>
  <body>
    <!-- Audios -->
    <audio id="bgMusic" loop>
      <source src="https://cdn.pixabay.com/download/audio/2023/06/06/audio_4f7b266f1e.mp3?filename=cyberpunk-sport-epic-music-149076.mp3" type="audio/mpeg">
    </audio>
    <audio id="correctSound"><source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_0f43b1dd6a.mp3?filename=success-1-6297.mp3" type="audio/mpeg"></audio>
    <audio id="errorSound"><source src="https://cdn.pixabay.com/download/audio/2022/03/15/audio_6ecfdb2e33.mp3?filename=error-1-8863.mp3" type="audio/mpeg"></audio>
    <audio id="teleportSound"><source src="https://cdn.pixabay.com/download/audio/2022/03/16/audio_9932eec6d7.mp3?filename=transition-fast-9100.mp3" type="audio/mpeg"></audio>
    <audio id="victoryMusic"><source src="https://cdn.pixabay.com/download/audio/2021/08/04/audio_3f1b9f4b2b.mp3?filename=celebration-brief-6168.mp3" type="audio/mpeg"></audio>

    <div id="gameContainer">
      <!-- Menu (inicial) -->
      <div id="menu" class="room" style="display:flex;background:radial-gradient(circle at center,#0b0033,#000)">
        <h1>Escape Room: Conjuntos Numéricos</h1>
        <div style="display:flex;gap:12px">
          <button class="btn" onclick="showIntro()">🎮 Iniciar Misión</button>
          <button class="btn" onclick="credits()">ℹ️ Créditos</button>
        </div>
      </div>

      <!-- Intro con la imagen grande -->
      <div id="introScreen" style="display:none;">
        <img id="introImg" src="conjuntos-introduccion.png" alt="Conjuntos numéricos"
            onerror="this.onerror=null;this.src='/mnt/data/1bbb1bf2-fbc1-4203-8c3d-1700f5e2b4c0.png'">
        <div class="introText">Antes de comenzar verás 10 retos — 2 por cada conjunto numérico. Responde usando números o texto; el juego aceptará respuestas equivalentes o aproximadas. ¡Buena suerte!</div>
        <div style="display:flex;gap:12px">
          <button class="btn" onclick="startGame()">Continuar y comenzar ➤</button>
          <button class="btn" onclick="backToMenu()">Volver</button>
        </div>
      </div>

      <!-- HUD: música, vidas, timer -->
      <div id="hud" style="display:none">
        <button id="musicControl" onclick="toggleMusic()">🔊 Música: ON</button>
      </div>
      <div id="lives" style="display:none">
        <img id="heart1" class="heart" src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='%23ff4d6d' d='M12 21s-7-4.438-9-7.2C-0.7 9.4 3 4 7 6.4 9 8 12 11 12 11s3-3 5-4.6C21 4 25.7 9.4 21 13.8 19 16.562 12 21 12 21z'/></svg>">
        <img id="heart2" class="heart" src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='%23ff4d6d' d='M12 21s-7-4.438-9-7.2C-0.7 9.4 3 4 7 6.4 9 8 12 11 12 11s3-3 5-4.6C21 4 25.7 9.4 21 13.8 19 16.562 12 21 12 21z'/></svg>">
        <img id="heart3" class="heart" src="data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 24 24'><path fill='%23ff4d6d' d='M12 21s-7-4.438-9-7.2C-0.7 9.4 3 4 7 6.4 9 8 12 11 12 11s3-3 5-4.6C21 4 25.7 9.4 21 13.8 19 16.562 12 21 12 21z'/></svg>">
      </div>
      <div id="timer" style="display:none">⏱️ 60s</div>

      <!-- particle layer -->
      <div id="particles"></div>

      <!-- 10 HABITACIONES / NIVELES (2 por conjunto) -->

      <!-- Números Naturales - N -->
      <div id="room1" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2017/03/29/12/32/numbers-2185435_1280.png')">
        <div id="puzzleBox">
          <h2>Habitación 1 - Naturales (N)</h2>
          <p>Compras 3 paquetes, cada uno con 5 galletas. ¿Cuántas galletas llevas en total?</p>
          <div>
            <input type="number" id="answer1" placeholder="Tu respuesta">
            <button class="btn" onclick="checkAnswer(1)">Comprobar</button>
          </div>
          <div id="message1" class="msg"></div>
        </div>
      </div>

      <div id="room2" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2016/11/22/19/05/apples-1851145_1280.jpg')">
        <div id="puzzleBox">
          <h2>Habitación 2 - Naturales (N)</h2>
          <p>Tienes 7 manzanas y compras 4 más. ¿Cuántas tienes ahora?</p>
          <div>
            <input type="number" id="answer2" placeholder="Tu respuesta">
            <button class="btn" onclick="checkAnswer(2)">Comprobar</button>
          </div>
          <div id="message2" class="msg"></div>
        </div>
      </div>

      <!-- Enteros - Z -->
      <div id="room3" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2014/04/02/10/05/math-304825_1280.png')">
        <div id="puzzleBox">
          <h2>Habitación 3 - Enteros (Z)</h2>
          <p>La temperatura era -3°C. Si sube 7°C, ¿cuál es ahora la temperatura?</p>
          <div>
            <input type="number" id="answer3" placeholder="Tu respuesta">
            <button class="btn" onclick="checkAnswer(3)">Comprobar</button>
          </div>
          <div id="message3" class="msg"></div>
        </div>
      </div>

      <div id="room4" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2017/08/30/01/05/mountains-2696956_1280.jpg')">
        <div id="puzzleBox">
          <h2>Habitación 4 - Enteros (Z)</h2>
          <p>Si subes 5 pisos desde el piso 2 y luego bajas 8, ¿en qué piso quedas?</p>
          <div>
            <input type="number" id="answer4" placeholder="Tu respuesta (puede ser negativo)">
            <button class="btn" onclick="checkAnswer(4)">Comprobar</button>
          </div>
          <div id="message4" class="msg"></div>
        </div>
      </div>

      <!-- Racionales - Q -->
      <div id="room5" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2016/04/14/20/17/mathematics-1327496_1280.png')">
        <div id="puzzleBox">
          <h2>Habitación 5 - Racionales (Q)</h2>
          <p>Usas 1/2 taza de leche y luego agregas 3/4 de taza más. ¿Cuántas tazas tienes en total? (acepta 5/4 o 1.25)</p>
          <div>
            <input type="text" id="answer5" placeholder="Ejemplo: 5/4 o 1.25">
            <button class="btn" onclick="checkAnswer(5)">Comprobar</button>
          </div>
          <div id="message5" class="msg"></div>
        </div>
      </div>

      <div id="room6" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2018/03/05/09/17/coffee-3198196_1280.jpg')">
        <div id="puzzleBox">
          <h2>Habitación 6 - Racionales (Q)</h2>
          <p>Compartes 2/3 de una pizza y tu amigo añade 1/6 más. ¿Qué fracción total tienen? (acepta fracción o decimal)</p>
          <div>
            <input type="text" id="answer6" placeholder="Ejemplo: 5/6 o 0.8333">
            <button class="btn" onclick="checkAnswer(6)">Comprobar</button>
          </div>
          <div id="message6" class="msg"></div>
        </div>
      </div>

      <!-- Irracionales - I -->
      <div id="room7" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2020/06/03/18/01/math-5250974_1280.jpg')">
        <div id="puzzleBox">
          <h2>Habitación 7 - Irracionales (I)</h2>
          <p>La diagonal de un cuadrado de lado 1 m es √2 m. ¿Qué tipo de número es √2?</p>
          <div>
            <input type="text" id="answer7" placeholder="Ejemplo: irracional">
            <button class="btn" onclick="checkAnswer(7)">Comprobar</button>
          </div>
          <div id="message7" class="msg"></div>
        </div>
      </div>

      <div id="room8" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2017/01/31/21/24/book-2023512_1280.jpg')">
        <div id="puzzleBox">
          <h2>Habitación 8 - Irracionales (I)</h2>
          <p>La relación entre la circunferencia y el diámetro es π. ¿Qué tipo de número es π?</p>
          <div>
            <input type="text" id="answer8" placeholder="Ejemplo: irracional">
            <button class="btn" onclick="checkAnswer(8)">Comprobar</button>
          </div>
          <div id="message8" class="msg"></div>
        </div>
      </div>

      <!-- Reales - R -->
      <div id="room9" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2021/07/12/09/40/math-6404543_1280.jpg')">
        <div id="puzzleBox">
          <h2>Habitación 9 - Reales (R)</h2>
          <p>¿Qué conjunto contiene a los naturales, enteros, racionales e irracionales?</p>
          <div>
            <input type="text" id="answer9" placeholder="Ejemplo: reales">
            <button class="btn" onclick="checkAnswer(9)">Comprobar</button>
          </div>
          <div id="message9" class="msg"></div>
        </div>
      </div>

      <div id="room10" class="room" style="background-image:url('https://cdn.pixabay.com/photo/2016/11/29/03/53/numbers-1869138_1280.jpg')">
        <div id="puzzleBox">
          <h2>Habitación 10 - Reales (R)</h2>
          <p>¿El número -3/2 pertenece a los racionales, irracionales o reales?</p>
          <div>
            <input type="text" id="answer10" placeholder="Ejemplo: racional o reales">
            <button class="btn" onclick="checkAnswer(10)">Comprobar</button>
          </div>
          <div id="message10" class="msg"></div>
        </div>
      </div>

      <!-- Game over / retry modal -->
      <div id="gameOverModal">
        <div id="gameOverCard">
          <h3 id="gameOverTitle">Has perdido</h3>
          <p id="gameOverReason">Motivo...</p>
          <p id="countdownText">Puedes volver a intentar en <span id="countdown">5</span>s</p>
          <div style="display:flex;gap:10px;justify-content:center">
            <button id="retryBtn" disabled onclick="retryNow()">Volver a intentar ahora</button>
            <button class="btn" onclick="backToMenuFromModal()">Volver al inicio</button>
          </div>
        </div>
      </div>

      <!-- Victory screen -->
      <div id="victoryScreen">
        <div id="victoryCard">
          <h2>¡Has escapado del laboratorio numérico! 🧠✨</h2>
          <p>Completaste los 10 desafíos.</p>
          <div style="display:flex;gap:12px;justify-content:center;margin-top:12px">
            <button class="btn" onclick="backToMenuFromVictory()">Volver al inicio</button>
            <button class="btn" onclick="retryFromVictory()">Jugar de nuevo</button>
          </div>
        </div>
      </div>

    </div> <!-- /gameContainer -->

  <script>
  /* Estado del juego */
  let currentRoom = 0;
  let timeLeft = 60;
  let timerInterval = null;
  let lives = 3;
  let musicPlaying = true;
  let totalRooms = 10;

  /* Elementos */
  const music = document.getElementById('bgMusic');
  const correctSound = document.getElementById('correctSound');
  const errorSound = document.getElementById('errorSound');
  const teleportSound = document.getElementById('teleportSound');
  const victoryMusic = document.getElementById('victoryMusic');
  const musicButton = document.getElementById('musicControl');
  const hud = document.getElementById('hud');
  const livesBox = document.getElementById('lives');
  const timerEl = document.getElementById('timer');
  const particlesLayer = document.getElementById('particles');

  /* Inicial */
  music.volume = 0.28;

  /* Validadores flexibles para cada nivel */
  const validators = {
    1: v => isNumberApprox(v,15),
    2: v => isNumberApprox(v,11),
    3: v => isNumberApprox(v,4),
    4: v => isNumberApprox(v,-1),
    5: v => isFractionOrDecimalEquals(v,5/4),
    6: v => isFractionOrDecimalEquals(v,5/6),
    7: v => isTextMatch(v,['irracional','irracionales','no racional','no racionales']),
    8: v => isTextMatch(v,['irracional','irracionales','pi es irracional','no racional']),
    9: v => isTextMatch(v,['reales','real','conjunto real','numeros reales','números reales']),
    10: v => isTextMatch(v,['racional','racionales','reales','real','reales'])
  };

  /* Helpers de validación */
  function normalizeText(s){return (s||'').toString().trim().toLowerCase();}

  function isNumberApprox(input, target){
    const n = parseFloat((input||'').toString().replace(',','.'));
    if(isNaN(n)) return false;
    // para enteros exigir exactitud, para decimales aceptar tolerancia pequeña
    if(Number.isInteger(target)) return Math.abs(n - target) < 0.0001;
    return Math.abs(n - target) <= 0.02; // tolerancia para decimales
  }

  function parseFractionString(s){
    if(!s) return null;
    s = s.toString().trim();
    const fracMatch = s.match(/^(-?\d+)\s*\/\s*(\d+)$/);
    if(fracMatch){
      const a = parseInt(fracMatch[1],10);
      const b = parseInt(fracMatch[2],10);
      if(b===0) return null;
      return a / b;
    }
    const n = parseFloat(s.replace(',','.'));
    if(!isNaN(n)) return n;
    return null;
  }

  function isFractionOrDecimalEquals(input, target){
    const val = parseFractionString(input);
    if(val===null) return false;
    return Math.abs(val - target) <= 0.02;
  }

  function isTextMatch(input, allowed){
    const t = normalizeText(input);
    for(const a of allowed){
      if(t === a) return true;
      if(t.includes(a)) return true; // permite frases como "es irracional"
    }
    return false;
  }

  /* Mostrar intro */
  function showIntro(){
    document.getElementById('menu').style.display = 'none';
    document.getElementById('introScreen').style.display = 'flex';
    gsap.fromTo('#introScreen',{opacity:0},{opacity:1,duration:0.7});
  }

  /* Volver al menu */
  function backToMenu(){
    document.getElementById('introScreen').style.display = 'none';
    document.getElementById('menu').style.display = 'flex';
  }

  /* Iniciar juego desde intro */
  function startGame(){
    document.getElementById('introScreen').style.display = 'none';
    hud.style.display = 'flex';
    livesBox.style.display = 'flex';
    timerEl.style.display = 'block';
    music.play();
    musicButton.style.display = 'inline-block';
    resetLives();
    currentRoom = 0;
    nextRoom();
  }

  /* Créditos */
  function credits(){ alert("Creado por Miguel con ayuda de ChatGPT 💡"); }

  /* Música */
  function toggleMusic(){
    if(musicPlaying){ music.pause(); musicButton.innerText = '🔇 Música: OFF'; }
    else { music.play(); musicButton.innerText = '🔊 Música: ON'; }
    musicPlaying = !musicPlaying;
  }

  /* Resetear vidas */
  function resetLives(){
    lives = 3;
    updateHearts();
  }

  /* Actualizar iconos de corazón */
  function updateHearts(){
    for(let i=1;i<=3;i++){
      const el = document.getElementById('heart'+i);
      if(i<=lives) el.classList.remove('lost');
      else el.classList.add('lost');
    }
  }

  /* Crear partículas (pequeños círculos) en posición x,y */
  function spawnParticles(x,y,color='#ff7b7b'){
    for(let i=0;i<10;i++){
      const el = document.createElement('div');
      el.style.position='absolute';
      el.style.left=(x)+'px';
      el.style.top=(y)+'px';
      el.style.width='8px';
      el.style.height='8px';
      el.style.borderRadius='50%';
      el.style.background = color;
      el.style.opacity = '0.95';
      el.style.pointerEvents='none';
      particlesLayer.appendChild(el);
      const dx = (Math.random()-0.5)*160;
      const dy = (Math.random()-1.5)*160;
      gsap.to(el,{x:dx,y:dy,opacity:0,duration:0.9+Math.random()*0.6,ease:'power1.out',onComplete:()=>el.remove()});
    }
  }

  /* Avanzar a la siguiente habitación */
  function nextRoom(){
    currentRoom++;
    if(currentRoom>totalRooms){
      clearInterval(timerInterval);
      // victoria
      showVictory();
      return;
    }

    try{ teleportSound.currentTime = 0; teleportSound.play(); }catch(e){}

    document.querySelectorAll('.room').forEach(r => r.style.display = 'none');
    const room = document.getElementById('room' + currentRoom);
    room.style.display = 'flex';
    gsap.fromTo(room,{opacity:0,scale:0.95},{opacity:1,scale:1,duration:0.9,ease:'power2.out'});

    // reiniciar cronómetro por nivel
    startTimerForLevel(60);
  }

  /* Cronómetro por nivel */
  function startTimerForLevel(seconds){
    clearInterval(timerInterval);
    timeLeft = seconds;
    timerEl.innerText = `⏱️ ${timeLeft}s`;
    timerInterval = setInterval(()=>{
      timeLeft--;
      timerEl.innerText = `⏱️ ${timeLeft}s`;
      if(timeLeft<=10) gsap.to('#timer',{color:'red',repeat:1,yoyo:true,duration:0.4});
      if(timeLeft<=0){
        clearInterval(timerInterval);
        setTimeout(()=> showGameOverModal('Has perdido','Tiempo agotado.','Volver al inicio',true),200);
      }
    },1000);
  }

  /* Manejo de respuestas */
  function checkAnswer(roomNumber){
    const inputEl = document.getElementById('answer' + roomNumber);
    const messageEl = document.getElementById('message' + roomNumber);
    const value = (inputEl.value || '').toString().trim();

    const validFn = validators[roomNumber];
    if(validFn && validFn(value)){
      try{ correctSound.currentTime = 0; correctSound.play(); }catch(e){}
      messageEl.style.color = '#00ffff';
      messageEl.innerHTML = '✅ Correcto, avanzas a la siguiente habitación.';
      gsap.fromTo(messageEl,{scale:0.9},{scale:1.08,duration:0.4,yoyo:true,repeat:1});
      setTimeout(()=> {
        messageEl.innerHTML = '';
        inputEl.value = '';
        nextRoom();
      }, 900);
    } else {
      // incorrecto: perder vida
      try{ errorSound.currentTime = 0; errorSound.play(); }catch(e){}
      lives = Math.max(0, lives - 1);
      updateHearts();
      messageEl.style.color = '#ff8a8a';
      messageEl.innerHTML = '❌ Incorrecto, pierdes 1 vida.';
      gsap.fromTo(messageEl,{x:-8},{x:8,duration:0.08,repeat:4,yoyo:true});

      // animar corazón y partículas en su posición
      const lostIndex = lives+1;
      const lostHeart = document.getElementById('heart' + lostIndex);
      if(lostHeart){
        gsap.fromTo(lostHeart,{scale:1.35,opacity:1},{scale:1,duration:0.35,ease:'elastic.out(1,0.6)'});
        const rect = lostHeart.getBoundingClientRect();
        spawnParticles(rect.left + rect.width/2, rect.top + rect.height/2, '#ff8a8a');
      }

      if(lives === 0){
        setTimeout(()=> showGameOverModal('Has perdido','Se acabaron las vidas.','Volver al inicio',true), 700);
      }
    }
  }

  /* Mostrar modal de Game Over / Retry */
  let countdownTimer = null;
  function showGameOverModal(title, reason, altBtnText='Volver al inicio', allowRetry=true){
    clearInterval(timerInterval);
    const modal = document.getElementById('gameOverModal');
    document.getElementById('gameOverTitle').innerText = title;
    document.getElementById('gameOverReason').innerText = reason;
    document.getElementById('countdownText').style.display = allowRetry ? 'block' : 'none';
    const countdownEl = document.getElementById('countdown');
    const retryBtn = document.getElementById('retryBtn');
    retryBtn.disabled = true;
    modal.style.display = 'flex';
    gsap.fromTo('#gameOverCard',{y:20,opacity:0},{y:0,opacity:1,duration:0.45,ease:'power2.out'});

    if(allowRetry){
      let seconds = 5;
      countdownEl.innerText = seconds;
      countdownTimer = setInterval(()=>{
        seconds--;
        countdownEl.innerText = seconds;
        if(seconds<=0){
          clearInterval(countdownTimer);
          retryBtn.disabled = false;
          document.getElementById('countdownText').innerHTML = 'Puedes volver a intentar ahora';
        }
      },1000);
    } else {
      retryBtn.style.display = 'none';
      document.getElementById('countdownText').innerHTML = altBtnText;
    }
  }

  /* Reiniciar juego desde modal inmediatamente */
  function retryNow(){
    clearInterval(countdownTimer);
    document.getElementById('gameOverModal').style.display = 'none';
    resetEverything();
    showIntro();
  }

  /* Volver al menu desde modal */
  function backToMenuFromModal(){
    clearInterval(countdownTimer);
    document.getElementById('gameOverModal').style.display = 'none';
    resetEverything();
    document.getElementById('menu').style.display = 'flex';
  }

  /* Reset completo de estado */
  function resetEverything(){
    clearInterval(timerInterval);
    timerInterval = null;
    timeLeft = 60;
    lives = 3;
    updateHearts();
    document.querySelectorAll('.room').forEach(r => r.style.display = 'none');
    hud.style.display = 'none';
    livesBox.style.display = 'none';
    timerEl.style.display = 'none';
    for(let i=1;i<=totalRooms;i++){
      const msg = document.getElementById('message'+i);
      const inpt = document.getElementById('answer'+i);
      if(msg) msg.innerHTML = '';
      if(inpt) inpt.value = '';
    }
    try{ music.pause(); music.currentTime = 0; victoryMusic.pause(); victoryMusic.currentTime = 0; }catch(e){}
  }

  /* Victory */
  function showVictory(){
    document.querySelectorAll('.room').forEach(r=>r.style.display='none');
    hud.style.display = 'none';
    livesBox.style.display = 'none';
    timerEl.style.display = 'none';
    document.getElementById('victoryScreen').style.display = 'flex';
    try{ music.pause(); music.currentTime=0; victoryMusic.currentTime=0; victoryMusic.play(); }catch(e){}
    gsap.fromTo('#victoryCard',{scale:0.8,opacity:0},{scale:1,opacity:1,duration:0.8,ease:'elastic.out(1,0.6)'});
  }

  function backToMenuFromVictory(){
    document.getElementById('victoryScreen').style.display='none';
    resetEverything();
    document.getElementById('menu').style.display='flex';
  }
  function retryFromVictory(){
    document.getElementById('victoryScreen').style.display='none';
    resetEverything();
    showIntro();
  }

  /* Previene que Enter envíe el formulario y simula click en boton comprobar para el input visible */
  document.addEventListener('keydown', (e)=>{
    if(e.key === 'Enter'){
      for(let i=1;i<=totalRooms;i++){
        const room = document.getElementById('room'+i);
        if(room && room.style.display === 'flex'){
          const btn = room.querySelector('button.btn');
          if(btn) btn.click();
          e.preventDefault();
          break;
        }
      }
    }
  });

  </script>
  </body>
  </html>

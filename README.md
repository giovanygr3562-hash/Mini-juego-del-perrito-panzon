# Mini-juego-del-perrito-panzon
Mueve el jabón y limpia al perro 
<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Mini-juego: Lava al perrito</title>
  <style>
    :root{font-family:system-ui,Segoe UI,Roboto,Arial;margin:0}
    body{display:flex;flex-direction:column;gap:12px;align-items:center;padding:18px;background:#111;color:#fff}
    .controls{display:flex;gap:8px;flex-wrap:wrap;justify-content:center}
    button,input{padding:8px 12px;border-radius:10px;border:0;background:#2b2b2b;color:#fff}
    .stage{position:relative;width:360px;height:360px;border-radius:12px;overflow:hidden;background:#222;box-shadow:0 6px 18px rgba(0,0,0,.6)}
    .bg{position:absolute;inset:0;background-position:center;background-size:contain;background-repeat:no-repeat;display:flex;align-items:center;justify-content:center;transition:filter 600ms ease}
    .bg.clean{filter:brightness(1.25) saturate(1.1) contrast(1.02) drop-shadow(0 4px 8px rgba(255,255,255,.06))}
    .soap{position:absolute;width:96px;height:96px;left:24px;bottom:24px;touch-action:none;cursor:grab;user-select:none;transition:transform 120ms ease}
    .soap:active{cursor:grabbing}
    .hint{position:absolute;left:8px;top:8px;background:rgba(0,0,0,.45);padding:6px 8px;border-radius:8px;font-size:13px}
    .sparkle{position:absolute;width:18px;height:18px;border-radius:50%;background:linear-gradient(135deg,#fff,#ffd);opacity:0;pointer-events:none}
    .hidden{display:none}
    .credits{font-size:13px;opacity:.8}
  </style>
</head>
<body>
  <h2>Mini-juego: Lava al perrito</h2>
  <div class="controls">
    <label style="display:flex;gap:8px;align-items:center"><input id="bgFile" type="file" accept="image/*"> Subir fondo</label>
    <label style="display:flex;gap:8px;align-items:center"><input id="soapFile" type="file" accept="image/*"> Subir jabón</label>
    <button id="startBtn">Iniciar</button>
    <button id="resetBtn">Reiniciar</button>
  </div>

  <div class="stage" id="stage">
    <div class="bg" id="bg"> 
      <div class="hint" id="hint">Arrastra el jabón sobre el perrito para limpiarlo</div>
    </div>
    <img id="soap" class="soap hidden" src="" alt="soap">
  </div>

  <div class="controls credits">Consejos: sube la imagen del perrito como fondo y el jabón como objeto. Funciona en móvil y en PC.</div>

  <script>
    const bgFile = document.getElementById('bgFile');
    const soapFile = document.getElementById('soapFile');
    const startBtn = document.getElementById('startBtn');
    const resetBtn = document.getElementById('resetBtn');
    const bg = document.getElementById('bg');
    const soap = document.getElementById('soap');
    const stage = document.getElementById('stage');
    const hint = document.getElementById('hint');

    let bgURL = '';
    let soapURL = '';
    let dragging = false;
    let pointerId = null;
    let offset = {x:0,y:0};
    let cleaned = false;

    bgFile.addEventListener('change', e => {
      const f = e.target.files[0];
      if(!f) return;
      bgURL = URL.createObjectURL(f);
      bg.style.backgroundImage = `url('${bgURL}')`;
    });

    soapFile.addEventListener('change', e => {
      const f = e.target.files[0];
      if(!f) return;
      soapURL = URL.createObjectURL(f);
      soap.src = soapURL;
      soap.classList.remove('hidden');
    });

    startBtn.addEventListener('click', () => {
      if(!bgURL){ alert('Sube primero la imagen del perrito como fondo'); return; }
      if(!soapURL){ alert('Sube primero la imagen del jabón'); return; }
      // reset visual states
      cleaned = false;
      bg.classList.remove('clean');
      clearSparks();
      hint.textContent = 'Arrastra el jabón sobre el perrito para limpiarlo';
      soap.style.transform = 'none';
      // place soap bottom-left by default
      soap.style.left = '24px';
      soap.style.top = (stage.clientHeight - soap.clientHeight - 24) + 'px';
    });

    resetBtn.addEventListener('click', () => {
      cleaned = false;
      bg.classList.remove('clean');
      clearSparks();
      hint.textContent = 'Arrastra el jabón sobre el perrito para limpiarlo';
      soap.style.left = '24px';
      soap.style.top = (stage.clientHeight - soap.clientHeight - 24) + 'px';
    });

    // Pointer-based dragging (works for mouse & touch)
    soap.addEventListener('pointerdown', (ev) => {
      ev.preventDefault();
      soap.setPointerCapture(ev.pointerId);
      pointerId = ev.pointerId;
      dragging = true;
      const rect = soap.getBoundingClientRect();
      offset.x = ev.clientX - rect.left;
      offset.y = ev.clientY - rect.top;
      soap.style.transition = 'none';
    });

    document.addEventListener('pointermove', (ev) => {
      if(!dragging || ev.pointerId !== pointerId) return;
      const stageRect = stage.getBoundingClientRect();
      let x = ev.clientX - stageRect.left - offset.x;
      let y = ev.clientY - stageRect.top - offset.y;
      // keep inside stage
      x = Math.max(0, Math.min(stage.clientWidth - soap.clientWidth, x));
      y = Math.max(0, Math.min(stage.clientHeight - soap.clientHeight, y));
      soap.style.left = x + 'px';
      soap.style.top = y + 'px';
      checkCollision();
    });

    document.addEventListener('pointerup', (ev) => {
      if(ev.pointerId !== pointerId) return;
      dragging = false;
      pointerId = null;
      soap.releasePointerCapture && soap.releasePointerCapture(ev.pointerId);
      soap.style.transition = 'transform 120ms ease';
    });

    function checkCollision(){
      if(cleaned) return;
      const soapRect = soap.getBoundingClientRect();
      const stageRect = stage.getBoundingClientRect();
      // define target area as the central 60% of the stage (where the perrito usually is)
      const target = {
        left: stageRect.left + stageRect.width*0.2,
        right: stageRect.left + stageRect.width*0.8,
        top: stageRect.top + stageRect.height*0.2,
        bottom: stageRect.top + stageRect.height*0.8
      };
      const soapCenter = {x: soapRect.left + soapRect.width/2, y: soapRect.top + soapRect.height/2};
      const inside = soapCenter.x >= target.left && soapCenter.x <= target.right && soapCenter.y >= target.top && soapCenter.y <= target.bottom;
      if(inside){ triggerClean(); }
    }

    function triggerClean(){
      cleaned = true;
      bg.classList.add('clean');
      hint.textContent = '¡Listo! El perrito está limpio 🧼✨';
      makeSparks(14);
      // small gentle scale animation for soap
      soap.animate([
        {transform: 'scale(1) rotate(0deg)'},
        {transform: 'scale(.9) rotate(-8deg)'},
        {transform: 'scale(1) rotate(0deg)'}
      ],{duration:600,iterations:1});
      // optional: add a little ripple wash
      const wash = document.createElement('div');
      Object.assign(wash.style,{position:'absolute',left:'0',top:'0',right:'0',bottom:'0',background:'radial-gradient(circle at 40% 40%, rgba(255,255,255,0.15), rgba(255,255,255,0) 40%)',pointerEvents:'none',opacity:0,transition:'opacity 550ms ease'});
      bg.appendChild(wash);
      requestAnimationFrame(()=>wash.style.opacity = '1');
      setTimeout(()=>{ wash.style.opacity='0'; setTimeout(()=>wash.remove(),600); },900);
    }

    function makeSparks(n){
      for(let i=0;i<n;i++){
        const s = document.createElement('div');
        s.className = 'sparkle';
        const x = Math.random()* (stage.clientWidth - 24);
        const y = Math.random()* (stage.clientHeight - 24);
        s.style.left = x + 'px';
        s.style.top = y + 'px';
        stage.appendChild(s);
        // animate
        const delay = Math.random()*200;
        setTimeout(()=>{
          s.animate([
            {opacity:0, transform:'scale(.4) translateY(6px)'},
            {opacity:1, transform:'scale(1.1) translateY(-4px)'},
            {opacity:0, transform:'scale(.6) translateY(-18px)'}
          ],{duration:900 + Math.random()*600, easing:'ease-out', iterations:1});
          s.style.opacity = '1';
        }, delay);
        setTimeout(()=>s.remove(),1600 + Math.random()*800);
      }
    }

    function clearSparks(){
      document.querySelectorAll('.sparkle').forEach(n=>n.remove());
    }

    // Helpful: allow dropping images by drag & drop onto stage
    stage.addEventListener('dragover', e=>e.preventDefault());
    stage.addEventListener('drop', e=>{
      e.preventDefault();
      const f = e.dataTransfer.files && e.dataTransfer.files[0];
      if(!f) return;
      // if user drops an image while holding Shift => set soap, else set background
      if(e.shiftKey){
        soapURL = URL.createObjectURL(f);
        soap.src = soapURL; soap.classList.remove('hidden');
      } else {
        bgURL = URL.createObjectURL(f);
        bg.style.backgroundImage = `url('${bgURL}')`;
      }
    });

    // Make the layout responsive: scale stage to fit width on small screens
    function adaptStage(){
      const maxWidth = Math.min(window.innerWidth - 36, 520);
      stage.style.width = maxWidth + 'px';
      stage.style.height = maxWidth + 'px';
    }
    window.addEventListener('resize', adaptStage);
    adaptStage();
  </script>
</body>
</html>


{"updates":[{"pattern":".","multiple":true,"replacement":"<!doctype html>\n<html lang="es">\n<head>\n  <meta charset="utf-8" />\n  <meta name="viewport" content="width=device-width,initial-scale=1" />\n  <title>Mini-juego: Lava al perrito</title>\n  <style>\n    :root{font-family:system-ui,Segoe UI,Roboto,Arial;margin:0}\n    body{display:flex;flex-direction:column;gap:12px;align-items:center;padding:18px;background:#111;color:#fff}\n    .stage{position:relative;width:360px;height:360px;border-radius:12px;overflow:hidden;background:#222;box-shadow:0 6px 18px rgba(0,0,0,.6)}\n    .bg{position:absolute;inset:0;background:url('data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA...PERRITO_BASE64...') center/contain no-repeat;transition:filter 600ms ease}\n    .bg.clean{filter:brightness(1.25) saturate(1.1) contrast(1.02) drop-shadow(0 4px 8px rgba(255,255,255,.06))}\n    .soap{position:absolute;width:96px;height:96px;left:24px;bottom:24px;touch-action:none;cursor:grab;user-select:none;transition:transform 120ms ease}\n    .soap:active{cursor:grabbing}\n    .hint{position:absolute;left:8px;top:8px;background:rgba(0,0,0,.45);padding:6px 8px;border-radius:8px;font-size:13px}\n    .sparkle{position:absolute;width:18px;height:18px;border-radius:50%;background:linear-gradient(135deg,#fff,#ffd);opacity:0;pointer-events:none}\n  </style>\n</head>\n<body>\n  <h2>Mini-juego: Lava al perrito</h2>\n  <div class="stage" id="stage">\n    <div class="bg" id="bg"> \n      <div class="hint" id="hint">Arrastra el jabón sobre el perrito para limpiarlo</div>\n    </div>\n    <img id="soap" class="soap" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA...JABON_BASE64..." alt="soap">\n  </div>\n  <script>\n    const bg = document.getElementById('bg');\n    const soap = document.getElementById('soap');\n    const stage = document.getElementById('stage');\n    const hint = document.getElementById('hint');\n\n    let dragging=false;let pointerId=null;let offset={x:0,y:0};let cleaned=false;\n\n    soap.addEventListener('pointerdown', (ev) => {\n      ev.preventDefault();soap.setPointerCapture(ev.pointerId);pointerId=ev.pointerId;dragging=true;\n      const rect=soap.getBoundingClientRect();offset.x=ev.clientX-rect.left;offset.y=ev.clientY-rect.top;soap.style.transition='none';\n    });\n\n    document.addEventListener('pointermove',(ev)=>{\n      if(!dragging||ev.pointerId!==pointerId)return;\n      const stageRect=stage.getBoundingClientRect();\n      let x=ev.clientX-stageRect.left-offset.x;let y=ev.clientY-stageRect.top-offset.y;\n      x=Math.max(0,Math.min(stage.clientWidth-soap.clientWidth,x));\n      y=Math.max(0,Math.min(stage.clientHeight-soap.clientHeight,y));\n      soap.style.left=x+'px';soap.style.top=y+'px';checkCollision();\n    });\n\n    document.addEventListener('pointerup',(ev)=>{\n      if(ev.pointerId!==pointerId)return;dragging=false;pointerId=null;\n      soap.releasePointerCapture&&soap.releasePointerCapture(ev.pointerId);\n      soap.style.transition='transform 120ms ease';\n    });\n\n    function checkCollision(){\n      if(cleaned)return;\n      const soapRect=soap.getBoundingClientRect();const stageRect=stage.getBoundingClientRect();\n      const target={left:stageRect.left+stageRect.width0.2,right:stageRect.left+stageRect.width0.8,top:stageRect.top+stageRect.height0.2,bottom:stageRect.top+stageRect.height0.8};\n      const soapCenter={x:soapRect.left+soapRect.width/2,y:soapRect.top+soapRect.height/2};\n      if(soapCenter.x>=target.left&&soapCenter.x<=target.right&&soapCenter.y>=target.top&&soapCenter.y<=target.bottom){triggerClean();}\n    }\n\n    function triggerClean(){\n      cleaned=true;bg.classList.add('clean');hint.textContent='¡Listo! El perrito está limpio 🧼✨';makeSparks(14);\n    }\n\n    function makeSparks(n){\n      for(let i=0;i<n;i++){\n        const s=document.createElement('div');s.className='sparkle';\n        const x=Math.random()(stage.clientWidth-24);const y=Math.random()*(stage.clientHeight-24);\n        s.style.left=x+'px';s.style.top=y+'px';stage.appendChild(s);\n        const delay=Math.random()*200;\n        setTimeout(()=>{\n          s.animate([{opacity:0,transform:'scale(.4) translateY(6px)'},{opacity:1,transform:'scale(1.1) translateY(-4px)'},{opacity:0,transform:'scale(.6) translateY(-18px)'}],{duration:900+Math.random()*600,easing:'ease-out



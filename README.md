<!DOCTYPE html>
<html lang="hu">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>LED Vezérlés</title>
<style>
  body {
    font-family: 'Poppins', sans-serif;
    margin: 0; padding: 0;
    background: #0a0a0a;
    color: white;
    display: flex; flex-direction: column;
    align-items: center;
  }

  h2 { margin-top: 20px; color: #00c3ff; }

  .tab-container {
    display: flex;
    width: 100%;
    justify-content: center;
    margin-top: 10px;
  }
  .tab {
    flex: 1;
    text-align: center;
    padding: 10px;
    cursor: pointer;
    background: #111;
    border-bottom: 2px solid transparent;
  }
  .tab.active {
    background: #222;
    border-bottom: 2px solid #00c3ff;
  }

  .content {
    width: 90%;
    max-width: 400px;
    margin-top: 20px;
    display: none;
  }
  .content.active { display: block; }

  .slider { width: 100%; }
  .color-mode { display: flex; justify-content: space-around; margin: 15px 0; }
  .color-mode button {
    background: #222;
    border: 1px solid #00c3ff;
    color: white;
    padding: 8px 12px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
  }
  .color-mode button.active {
    background: #00c3ff;
    color: black;
  }
  .preview {
    width: 100%;
    height: 40px;
    border-radius: 8px;
    margin-top: 10px;
  }
  .control-btn {
    background: #00c3ff;
    color: black;
    border: none;
    border-radius: 8px;
    padding: 10px 15px;
    margin-top: 10px;
    margin-bottom: 0;
    width: 100%;
    font-size: 16px;
    cursor: pointer;
  }

  /* --- KÖRKÖRÖS LED-RENDEZÉS --- */
  .led-circle-container {
    position: relative;
    width: 260px; height: 260px;
    margin: 0 auto;
  }
  .circle-led {
    position: absolute;
    width: 48px;
    height: 48px;
    border-radius: 50%;
    background: #222;
    border: 2px solid #00c3ff;
    cursor: pointer;
    transition: all 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    color: white;
    font-size: 14px;
    user-select: none;
    box-sizing: border-box;
  }
  .circle-led.selected {
    border-color: #ffcc00;
  }

</style>
</head>
<body>

<h2>LED Vezérlés</h2>

<!-- Fülek -->
<div class="tab-container">
  <div class="tab active" data-tab="status">ℹ️ Áttekintés</div>
  <div class="tab" data-tab="runlight">🏃 Futófény</div>
  <div class="tab" data-tab="flame">🔥 Láng effekt</div>
  <div class="tab" data-tab="ledcolor">🎨 LED színek</div>
</div>

<!-- Tartalmak -->
<div id="status" class="content active">
  <p>LED állapot: <strong>Bekapcsolva</strong></p>
  <p>Aktuális effekt: <strong>Futófény</strong></p>
  <button class="control-btn">Frissítés</button>
</div>

<!-- Futófény fül -->
<div id="runlight" class="content">
  <label>Sebesség: <span id="speedValue">50</span></label>
  <input type="range" min="1" max="100" value="50" class="slider" id="speedRange">

  <div class="color-mode">
    <button id="rainbowBtn" class="active">🌈 Szivárvány</button>
    <button id="singleBtn">🎨 Egy szín</button>
  </div>

  <div id="colorSelect" style="display:none; text-align:center;">
    <label>Szín kiválasztása:</label><br>
    <input type="color" id="singleColor" value="#00ffcc">
  </div>

  <div class="preview" id="colorPreview"></div>
  <button class="control-btn">Mentés</button>
</div>

<!-- Láng effekt fül -->
<div id="flame" class="content">
  <h3>Láng effekt beállítások</h3>

  <div class="color-mode">
    <button class="flameMode active" data-mode="classic">🔥 Klasszikus</button>
    <button class="flameMode" data-mode="gas">🔵 Gázláng</button>
    <button class="flameMode" data-mode="campfire">🪵 Tábortűz</button>
    <button class="flameMode" data-mode="candle">🕯️ Gyertya</button>
  </div>

  <label>Sebesség: <span id="flameSpeedValue">50</span></label>
  <input type="range" min="1" max="100" value="50" class="slider" id="flameSpeed">

  <label>Intenzitás: <span id="flameIntensityValue">80</span></label>
  <input type="range" min="1" max="100" value="80" class="slider" id="flameIntensity">

  <div style="margin-top:10px;">
    <label>Színárnyalat:</label>
    <input type="color" id="flameColor" value="#ff4500">
  </div>

  <div class="preview" id="flamePreview" style="height:60px; margin-top:15px;"></div>
  <button class="control-btn">Mentés</button>
</div>

<!-- LED színek fül - KÖRKÖRÖS ELRENDEZÉS, csak egy másolás és beillesztés gomb -->
<div id="ledcolor" class="content">
  <h3>LED kiválasztás</h3>
  <div class="led-circle-container" id="ledCircleContainer" style="position:relative; width:260px; height:260px; margin:0 auto;">
    <!-- LED-ek ide generálódnak JS-ből -->
  </div>
  <div style="text-align:center; margin-top:10px;">
    <button class="control-btn" id="copyBtn">Másolás</button>
    <button class="control-btn" id="pasteBtn">Beillesztés</button>
  </div>
  <div style="text-align:center; margin-top:15px;">
    <label>Szín kiválasztása:</label>
    <input type="color" id="ledColor" value="#00ffcc">
    <button class="control-btn" id="applyColor">Alkalmaz</button>
  </div>
</div>

<script>
  // Fülek váltása
  const tabs = document.querySelectorAll('.tab');
  const contents = document.querySelectorAll('.content');
  tabs.forEach(tab => {
    tab.addEventListener('click', () => {
      tabs.forEach(t => t.classList.remove('active'));
      contents.forEach(c => c.classList.remove('active'));
      tab.classList.add('active');
      document.getElementById(tab.dataset.tab).classList.add('active');
    });
  });

  // Futófény
  const speedRange = document.getElementById('speedRange');
  const speedValue = document.getElementById('speedValue');
  speedRange.oninput = () => speedValue.innerText = speedRange.value;

  const rainbowBtn = document.getElementById('rainbowBtn');
  const singleBtn = document.getElementById('singleBtn');
  const colorSelect = document.getElementById('colorSelect');
  const colorPreview = document.getElementById('colorPreview');

  rainbowBtn.onclick = () => { rainbowBtn.classList.add('active'); singleBtn.classList.remove('active'); colorSelect.style.display='none'; colorPreview.style.background='linear-gradient(90deg, red, orange, yellow, green, cyan, blue, violet)'; }
  singleBtn.onclick = () => { singleBtn.classList.add('active'); rainbowBtn.classList.remove('active'); colorSelect.style.display='block'; updateColorPreview(); }
  document.getElementById('singleColor').oninput = updateColorPreview;
  function updateColorPreview() { colorPreview.style.background = document.getElementById('singleColor').value; }
  updateColorPreview();

  // Láng effekt
  const flameModes = document.querySelectorAll('.flameMode');
  const flameSpeed = document.getElementById('flameSpeed');
  const flameSpeedValue = document.getElementById('flameSpeedValue');
  const flameIntensity = document.getElementById('flameIntensity');
  const flameIntensityValue = document.getElementById('flameIntensityValue');
  const flameColor = document.getElementById('flameColor');
  const flamePreview = document.getElementById('flamePreview');
  let selectedFlameMode = 'classic';

  flameModes.forEach(btn => {
    btn.addEventListener('click', () => {
      flameModes.forEach(b=>b.classList.remove('active'));
      btn.classList.add('active');
      selectedFlameMode = btn.dataset.mode;
      updateFlamePreview();
    });
  });
  flameSpeed.oninput = () => { flameSpeedValue.innerText = flameSpeed.value; updateFlamePreview(); };
  flameIntensity.oninput = () => { flameIntensityValue.innerText = flameIntensity.value; updateFlamePreview(); };
  flameColor.oninput = updateFlamePreview;

  function updateFlamePreview() {
    let color = flameColor.value;
    let gradient;
    switch(selectedFlameMode){
      case 'classic': gradient = `linear-gradient(90deg, ${color}, #ffae42, #ff0000)`; break;
      case 'gas': gradient = `linear-gradient(90deg, #0044ff, #00aaff, ${color})`; break;
      case 'campfire': gradient = `linear-gradient(90deg, #ff4500, #ffae42, #ff0000, ${color})`; break;
      case 'candle': gradient = `linear-gradient(90deg, #fffacd, ${color})`; break;
    }
    flamePreview.style.background = gradient;
  }
  updateFlamePreview();

  // --- KÖRKÖRÖS LED-RENDEZÉS CSAK 1 MÁSOLÁS ÉS 1 BEILLESZTÉS GOMB ---
  const ledsCount = 12;
  const ledContainer = document.getElementById('ledCircleContainer');
  const ledColorInput = document.getElementById('ledColor');
  let selectedLed = 0;
  let copiedColor = null;
  let leds = [];

  function renderCircleLeds() {
    ledContainer.innerHTML = "";
    const radius = 110;
    const center = 130;
    for(let i=0; i<ledsCount; i++) {
      const angle = (2*Math.PI/ledsCount)*i - Math.PI/2;
      const x = center + radius*Math.cos(angle) - 24;
      const y = center + radius*Math.sin(angle) - 24;

      const led = document.createElement("div");
      led.className = "circle-led";
      led.style.left = `${x}px`;
      led.style.top = `${y}px`;
      led.dataset.index = i;
      led.style.background = leds[i]?.color || "#222";
      if(i === selectedLed) led.classList.add("selected");
      led.innerHTML = `${i+1}`;
      ledContainer.appendChild(led);

      led.addEventListener('click', function(){
        selectedLed = i;
        leds.forEach(l=>l.el.classList.remove('selected'));
        led.classList.add('selected');
        ledColorInput.value = rgbToHex(led.style.background || '#222222');
      });

      leds[i] = {el: led, color: leds[i]?.color || "#222"};
    }
  }
  document.getElementById('applyColor').addEventListener('click', ()=>{
    leds[selectedLed].el.style.background = ledColorInput.value;
    leds[selectedLed].color = ledColorInput.value;
  });

  document.getElementById('copyBtn').addEventListener('click', ()=>{
    copiedColor = rgbToHex(leds[selectedLed].el.style.background || '#222222');
  });
  document.getElementById('pasteBtn').addEventListener('click', ()=>{
    if(copiedColor) {
      leds[selectedLed].el.style.background = copiedColor;
      leds[selectedLed].color = copiedColor;
    }
  });
  function rgbToHex(rgb){
    if(!rgb) return '#222222';
    if(rgb.startsWith("#")) return rgb;
    const result = rgb.match(/\d+/g);
    if(!result) return '#222222';
    return "#" + result.map(x=> (+x).toString(16).padStart(2,'0')).join('');
  }
  renderCircleLeds();

</script>
</body>
</html>

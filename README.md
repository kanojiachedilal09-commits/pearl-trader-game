# pearl-trader-game
Author : Sahil Manoj kumar Kanojia
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover" />
  <title>Raj — Island Trader (Pearl Edition)</title>
  <style>
    :root{--bg:#071226;--card:#07121a;--accent:#ffb84d;--muted:#b7c6cf}
    html,body{height:100%;margin:0;font-family:Inter,system-ui,Segoe UI,Roboto,'Helvetica Neue',Arial;background:linear-gradient(180deg,#021426 0%,#041827 100%);color:#eaf2f7;-webkit-tap-highlight-color:transparent}
    #app{display:flex;flex-direction:column;height:100vh}
    .topbar{display:flex;align-items:center;justify-content:space-between;padding:10px 14px}
    .logo{font-weight:700;font-size:16px}
    .hud{display:flex;gap:8px;align-items:center}
    .panel{background:rgba(255,255,255,0.03);padding:8px 12px;border-radius:10px;border:1px solid rgba(255,255,255,0.04)}
    #canvas-wrap{flex:1;position:relative;margin:10px;border-radius:12px;overflow:hidden}
    canvas{width:100%;height:100%;display:block}
    .controls{position:absolute;right:14px;bottom:14px;display:flex;flex-direction:column;gap:8px;z-index:20}
    .btn{padding:10px 14px;border-radius:12px;border:2px solid var(--accent);background:transparent;color:var(--accent);font-weight:700;cursor:pointer;outline:none;transition:transform .12s ease,box-shadow .12s ease}
    .btn:active{transform:translateY(2px) scale(.99)}
    .btn-primary{background:linear-gradient(90deg,var(--accent),#ffdca3);color:#2b1a00;border-color:transparent;box-shadow:0 8px 24px rgba(255,184,77,0.12)}
    .btn-outline{box-shadow:0 6px 18px rgba(2,6,23,0.6)}
    .mobile-pad{position:absolute;left:14px;bottom:14px;display:flex;flex-direction:column;gap:8px;z-index:20}
    .pad-row{display:flex;gap:8px}
    .pad-btn{width:56px;height:56px;border-radius:12px;border:2px solid rgba(255,255,255,0.06);background:rgba(255,255,255,0.02);display:flex;align-items:center;justify-content:center;font-weight:800;color:var(--muted);touch-action:manipulation}
    .pad-btn:active{transform:scale(.96)}
    .floating{position:absolute;left:50%;top:12px;transform:translateX(-50%);padding:8px 12px;border-radius:999px;background:rgba(0,0,0,0.45);font-weight:600;color:var(--muted);z-index:20}
    .market-badge{position:absolute;right:22%;top:8%;padding:6px 10px;border-radius:8px;background:rgba(0,0,0,0.45);font-weight:700;z-index:20}
    .shop-modal{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);min-width:280px;background:rgba(7,18,34,0.95);padding:12px;border-radius:12px;border:1px solid rgba(255,255,255,0.04);z-index:30;display:none}
    .shop-row{display:flex;justify-content:space-between;align-items:center;padding:8px 6px;border-bottom:1px solid rgba(255,255,255,0.02)}
    .coins{position:absolute;right:18px;top:56px;display:flex;flex-direction:column;gap:6px;align-items:flex-end;z-index:25}
    .coin{padding:6px 10px;border-radius:999px;background:linear-gradient(90deg,#fff2cc,#ffd06b);color:#5a3300;font-weight:800;transform:translateY(-6px);opacity:0}
    @media(min-width:900px){.mobile-pad{display:none}}
    @media(max-width:899px){.controls{right:unset;left:50%;transform:translateX(-50%);flex-direction:row}}
  </style>
</head>
<body>
  <div id="app">
    <div class="topbar">
      <div class="logo">Raj — Pearl Trader</div>
      <div class="hud">
        <div class="panel">Money<br><strong id="money">₹0</strong></div>
        <div class="panel">Inventory<br><strong id="inventory">0/5</strong></div>
        <div class="panel">Level<br><strong id="level">1</strong></div>
        <div class="panel">Time<br><strong id="time">00:00</strong></div>
      </div>
    </div>

    <div id="canvas-wrap">
      <div id="three-root"></div>
      <div class="floating panel">Controls: WASD • Hold RUN • Jump • SELL at market • Open Shop</div>
      <div id="marketTag" class="market-badge panel">Market — Sell here</div>

      <div class="controls">
        <button id="startBtn" class="btn btn-outline">Start</button>
        <button id="sellBtn" class="btn btn-primary">SELL ALL</button>
        <button id="shopBtn" class="btn">SHOP</button>
      </div>

      <div class="mobile-pad">
        <div class="pad-row"><div id="btnUp" class="pad-btn">▲</div></div>
        <div class="pad-row">
          <div id="btnLeft" class="pad-btn">◀</div>
          <div id="btnDown" class="pad-btn">▼</div>
          <div id="btnRight" class="pad-btn">▶</div>
        </div>
        <div style="height:8px"></div>
        <div class="pad-row"><div id="btnJump" class="pad-btn">⤒</div><div id="btnRun" class="pad-btn">RUN</div></div>
      </div>

      <div id="shop" class="shop-modal">
        <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:8px"><strong>Shop</strong><button id="closeShop" class="btn">Close</button></div>
        <div class="shop-row"><div><strong>Backpack</strong><div style="font-size:12px;color:var(--muted)">Increase inventory +5</div></div><div><button class="btn" id="buyBag">Buy (₹500)</button></div></div>
        <div class="shop-row"><div><strong>Magnet</strong><div style="font-size:12px;color:var(--muted)">Auto-attract nearby pearls</div></div><div><button class="btn" id="buyMagnet">Buy (₹800)</button></div></div>
        <div class="shop-row"><div><strong>Speed Boots</strong><div style="font-size:12px;color:var(--muted)">Increase run speed</div></div><div><button class="btn" id="buyBoots">Buy (₹700)</button></div></div>
      </div>

      <div class="coins" id="coinLayer"></div>
      <audio id="sfxPickup" src="" preload="auto"></audio>
      <audio id="sfxFoot" src="" preload="auto"></audio>
      <audio id="sfxSell" src="" preload="auto"></audio>
      <audio id="bgOcean" src="" loop preload="auto"></audio>
    </div>
  </div>

  <script src="https://unpkg.com/three@0.154.0/build/three.min.js"></script>
  <script>
    const root = document.getElementById('three-root');
    let scene, camera, renderer, clock;
    let player, whale, market;
    let pearls = [];
    let money=0, inventory=0, invLimit=5; 
    let running=false, spawnTimer=0;
    let timeOfDay = 12;
    let priceMultiplier = 1.0;
    const owned = {bag:false, magnet:false, boots:false};

    // Level system
    let currentLevel = 1;
    const maxLevel = 10000;
    let pearlsToCollect = 5;
    const levelMultiplier = 1.05;

    function startLevel(level){
      currentLevel = level;
      pearlsToCollect = Math.floor(5 * Math.pow(levelMultiplier, level-1));
      inventory = 0;
      player.userData.inventory = [];
      updateUI();
      flash(Level ${currentLevel});
    }

    function checkLevelComplete(){
      if(inventory >= pearlsToCollect){
        if(currentLevel < maxLevel) startLevel(currentLevel+1);
        else flash("🎉 Congratulations! All levels completed!");
      }
    }

    function collectPearl(p){
      if(inventory >= invLimit){ flash('Inventory full'); return; }
      inventory += 1;
      player.userData.inventory.push(p.userData.price);

      const pop = document.createElement('div'); 
      pop.className='coin'; 
      pop.textContent = '●'; 
      document.getElementById('coinLayer').appendChild(pop);
      setTimeout(()=>{ pop.style.opacity='1'; pop.style.transform='translateY(0)'; setTimeout(()=>{ pop.style.opacity='0'; pop.remove(); },700); },10);
      scene.remove(p);

      if(navigator.vibrate) navigator.vibrate(80);
      if(window.playPickup) window.playPickup();

      checkLevelComplete();
    }

    function updateUI(){
      document.getElementById('money').textContent = '₹'+money;
      document.getElementById('inventory').textContent = inventory+'/'+invLimit;
      document.getElementById('level').textContent = currentLevel;
    }

    // rest of your 3D setup, movement, whales, shop, spawn, etc.
    // Replace previous collectVomit() calls with collectPearl() and adjust whale interaction accordingly.

    console.log("Pearl game with 10k levels ready");
  </script>
</body>
</html>

<!DOCTYPE html>
<html lang="ja">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>BLOCKFALL — 落下型ブロックパズル</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  html, body {
    height: 100%;
    background: radial-gradient(ellipse at center, #1a1033 0%, #07040f 70%);
    font-family: 'Segoe UI', 'Hiragino Sans', 'Meiryo', system-ui, sans-serif;
    color: #e8e8ff;
    overflow: hidden;
    user-select: none;
  }
  body {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    padding: 20px;
  }

  .wrap {
    display: grid;
    grid-template-columns: 180px auto 200px;
    gap: 20px;
    align-items: start;
  }

  .title {
    position: absolute;
    top: 18px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 28px;
    letter-spacing: 10px;
    font-weight: 800;
    background: linear-gradient(90deg, #ff4fd8, #4fc3ff, #a0ff4f);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
    text-shadow: 0 0 40px rgba(255,255,255,0.1);
  }
  .subtitle {
    position: absolute;
    top: 52px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 10px;
    letter-spacing: 4px;
    color: #8080a0;
  }

  .panel {
    background: linear-gradient(135deg, rgba(30,20,60,0.8), rgba(15,10,30,0.9));
    border: 1px solid rgba(150,130,255,0.2);
    border-radius: 8px;
    padding: 14px;
    box-shadow: 0 8px 32px rgba(0,0,0,0.6), inset 0 0 20px rgba(100,80,200,0.05);
    backdrop-filter: blur(6px);
  }
  .panel h3 {
    font-size: 11px;
    letter-spacing: 3px;
    color: #a090d0;
    margin-bottom: 10px;
    font-weight: 600;
    text-transform: uppercase;
  }

  .side { display: flex; flex-direction: column; gap: 16px; }

  #game-wrap {
    position: relative;
    display: inline-block;
    border-radius: 8px;
    overflow: hidden;
  }
  #game {
    display: block;
    background: #0a0618;
    border: 1px solid rgba(150,130,255,0.3);
    border-radius: 8px;
    box-shadow: 0 0 60px rgba(80,60,200,0.2), inset 0 0 40px rgba(0,0,0,0.5);
  }

  canvas.mini {
    display: block;
    background: rgba(5,3,15,0.6);
    border-radius: 4px;
  }

  .stat {
    display: flex;
    justify-content: space-between;
    font-size: 13px;
    padding: 6px 0;
    border-bottom: 1px dashed rgba(150,130,255,0.15);
  }
  .stat:last-child { border-bottom: none; }
  .stat .label { color: #9080b0; letter-spacing: 1px; font-size: 11px; }
  .stat .value { color: #ffffff; font-weight: 700; font-family: 'Courier New', monospace; }
  .stat .value.big { font-size: 18px; color: #4fc3ff; }

  .controls {
    font-size: 11px;
    line-height: 1.9;
    color: #9080b0;
  }
  .controls .key {
    display: inline-block;
    min-width: 40px;
    text-align: center;
    padding: 2px 6px;
    background: rgba(100,80,200,0.15);
    border: 1px solid rgba(150,130,255,0.3);
    border-radius: 3px;
    color: #d0c0ff;
    font-family: 'Courier New', monospace;
    font-size: 10px;
    margin-right: 6px;
  }

  .overlay {
    position: absolute;
    inset: 0;
    display: none;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    background: rgba(5, 3, 15, 0.85);
    backdrop-filter: blur(4px);
    border-radius: 8px;
    text-align: center;
    padding: 20px;
  }
  .overlay.show { display: flex; }
  .overlay h2 {
    font-size: 32px;
    letter-spacing: 6px;
    margin-bottom: 12px;
    background: linear-gradient(90deg, #ff4fd8, #4fc3ff);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
  }
  .overlay p {
    font-size: 13px;
    color: #a090d0;
    margin-bottom: 20px;
    line-height: 1.8;
  }
  .btn {
    padding: 10px 28px;
    background: linear-gradient(90deg, #ff4fd8, #4fc3ff);
    color: #ffffff;
    border: none;
    border-radius: 4px;
    font-size: 12px;
    font-weight: 700;
    letter-spacing: 3px;
    cursor: pointer;
    transition: transform 0.1s, box-shadow 0.2s;
    font-family: inherit;
  }
  .btn:hover { transform: translateY(-1px); box-shadow: 0 4px 20px rgba(255,79,216,0.4); }
  .btn:active { transform: translateY(0); }

  .flash {
    position: absolute;
    left: 50%;
    top: 30%;
    transform: translateX(-50%);
    font-size: 28px;
    font-weight: 800;
    letter-spacing: 4px;
    pointer-events: none;
    opacity: 0;
    text-shadow: 0 0 20px currentColor;
    white-space: nowrap;
  }

  @keyframes pop {
    0% { opacity: 0; transform: translate(-50%, 10px) scale(0.8); }
    20% { opacity: 1; transform: translate(-50%, 0) scale(1.1); }
    100% { opacity: 0; transform: translate(-50%, -30px) scale(1); }
  }
  .flash.show { animation: pop 1.2s ease-out forwards; }

  @media (max-width: 780px) {
    .wrap { grid-template-columns: auto; grid-template-areas: "main" "left" "right"; }
    .side.left { grid-area: left; flex-direction: row; flex-wrap: wrap; }
    .side.right { grid-area: right; flex-direction: row; flex-wrap: wrap; }
    #game-wrap { grid-area: main; }
    .title { position: static; transform: none; text-align: center; }
    .subtitle { position: static; transform: none; text-align: center; margin-bottom: 12px; }
  }
</style>
</head>
<body>

<div class="title">BLOCKFALL</div>
<div class="subtitle">FALLING BLOCK PUZZLE</div>

<div class="wrap">
  <div class="side left">
    <div class="panel">
      <h3>HOLD</h3>
      <canvas id="hold" class="mini" width="140" height="100"></canvas>
    </div>
    <div class="panel">
      <h3>STATS</h3>
      <div class="stat"><span class="label">SCORE</span><span class="value big" id="score">0</span></div>
      <div class="stat"><span class="label">LEVEL</span><span class="value" id="level">1</span></div>
      <div class="stat"><span class="label">LINES</span><span class="value" id="lines">0</span></div>
      <div class="stat"><span class="label">COMBO</span><span class="value" id="combo">0</span></div>
      <div class="stat"><span class="label">B2B</span><span class="value" id="b2b">0</span></div>
    </div>
  </div>

  <div id="game-wrap">
    <canvas id="game" width="300" height="600"></canvas>
    <div class="flash" id="flash"></div>
    <div class="overlay show" id="overlay">
      <h2 id="ov-title">BLOCKFALL</h2>
      <p id="ov-body">矢印キーで移動、Zで左回転、X で右回転<br>スペースでハードドロップ、Cでホールド<br>Pでポーズ</p>
      <button class="btn" id="startBtn">START</button>
    </div>
  </div>

  <div class="side right">
    <div class="panel">
      <h3>NEXT</h3>
      <canvas id="next" class="mini" width="140" height="360"></canvas>
    </div>
    <div class="panel">
      <h3>CONTROLS</h3>
      <div class="controls">
        <div><span class="key">← →</span>移動</div>
        <div><span class="key">↓</span>ソフト</div>
        <div><span class="key">SPC</span>ハード</div>
        <div><span class="key">Z</span>左回転</div>
        <div><span class="key">X / ↑</span>右回転</div>
        <div><span class="key">C</span>ホールド</div>
        <div><span class="key">P</span>ポーズ</div>
        <div><span class="key">M</span>ミュート</div>
      </div>
    </div>
  </div>
</div>

<script>
(() => {
  // ===== Constants =====
  const COLS = 10, ROWS = 20, HIDDEN = 2;
  const BLOCK = 30;
  const LOCK_DELAY = 500;
  const DAS = 140, ARR = 35;

  // Piece definitions (SRS-style rotation states)
  const PIECES = {
    I: { color: '#4fc3ff', glow: '#4fc3ff', shapes: [
      [[0,0,0,0],[1,1,1,1],[0,0,0,0],[0,0,0,0]],
      [[0,0,1,0],[0,0,1,0],[0,0,1,0],[0,0,1,0]],
      [[0,0,0,0],[0,0,0,0],[1,1,1,1],[0,0,0,0]],
      [[0,1,0,0],[0,1,0,0],[0,1,0,0],[0,1,0,0]]
    ]},
    O: { color: '#ffd64f', glow: '#ffd64f', shapes: [
      [[1,1],[1,1]],[[1,1],[1,1]],[[1,1],[1,1]],[[1,1],[1,1]]
    ]},
    T: { color: '#c04fff', glow: '#c04fff', shapes: [
      [[0,1,0],[1,1,1],[0,0,0]],
      [[0,1,0],[0,1,1],[0,1,0]],
      [[0,0,0],[1,1,1],[0,1,0]],
      [[0,1,0],[1,1,0],[0,1,0]]
    ]},
    S: { color: '#4fff88', glow: '#4fff88', shapes: [
      [[0,1,1],[1,1,0],[0,0,0]],
      [[0,1,0],[0,1,1],[0,0,1]],
      [[0,0,0],[0,1,1],[1,1,0]],
      [[1,0,0],[1,1,0],[0,1,0]]
    ]},
    Z: { color: '#ff4f6d', glow: '#ff4f6d', shapes: [
      [[1,1,0],[0,1,1],[0,0,0]],
      [[0,0,1],[0,1,1],[0,1,0]],
      [[0,0,0],[1,1,0],[0,1,1]],
      [[0,1,0],[1,1,0],[1,0,0]]
    ]},
    J: { color: '#4f6dff', glow: '#4f6dff', shapes: [
      [[1,0,0],[1,1,1],[0,0,0]],
      [[0,1,1],[0,1,0],[0,1,0]],
      [[0,0,0],[1,1,1],[0,0,1]],
      [[0,1,0],[0,1,0],[1,1,0]]
    ]},
    L: { color: '#ff9a4f', glow: '#ff9a4f', shapes: [
      [[0,0,1],[1,1,1],[0,0,0]],
      [[0,1,0],[0,1,0],[0,1,1]],
      [[0,0,0],[1,1,1],[1,0,0]],
      [[1,1,0],[0,1,0],[0,1,0]]
    ]}
  };

  // SRS wall kick tables
  const KICKS_JLSTZ = {
    '0->1':[[0,0],[-1,0],[-1,1],[0,-2],[-1,-2]],
    '1->0':[[0,0],[1,0],[1,-1],[0,2],[1,2]],
    '1->2':[[0,0],[1,0],[1,-1],[0,2],[1,2]],
    '2->1':[[0,0],[-1,0],[-1,1],[0,-2],[-1,-2]],
    '2->3':[[0,0],[1,0],[1,1],[0,-2],[1,-2]],
    '3->2':[[0,0],[-1,0],[-1,-1],[0,2],[-1,2]],
    '3->0':[[0,0],[-1,0],[-1,-1],[0,2],[-1,2]],
    '0->3':[[0,0],[1,0],[1,1],[0,-2],[1,-2]]
  };
  const KICKS_I = {
    '0->1':[[0,0],[-2,0],[1,0],[-2,-1],[1,2]],
    '1->0':[[0,0],[2,0],[-1,0],[2,1],[-1,-2]],
    '1->2':[[0,0],[-1,0],[2,0],[-1,2],[2,-1]],
    '2->1':[[0,0],[1,0],[-2,0],[1,-2],[-2,1]],
    '2->3':[[0,0],[2,0],[-1,0],[2,1],[-1,-2]],
    '3->2':[[0,0],[-2,0],[1,0],[-2,-1],[1,2]],
    '3->0':[[0,0],[1,0],[-2,0],[1,-2],[-2,1]],
    '0->3':[[0,0],[-1,0],[2,0],[-1,2],[2,-1]]
  };

  // ===== Audio =====
  let audioCtx = null;
  let muted = false;
  function sfx(freq, duration, type='sine', gain=0.08) {
    if (muted) return;
    try {
      if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      const osc = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      osc.type = type;
      osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
      g.gain.setValueAtTime(gain, audioCtx.currentTime);
      g.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + duration);
      osc.connect(g); g.connect(audioCtx.destination);
      osc.start(); osc.stop(audioCtx.currentTime + duration);
    } catch(e) {}
  }
  function sfxSweep(f1, f2, duration, type='sawtooth', gain=0.07) {
    if (muted) return;
    try {
      if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      const osc = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      osc.type = type;
      osc.frequency.setValueAtTime(f1, audioCtx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(f2, audioCtx.currentTime + duration);
      g.gain.setValueAtTime(gain, audioCtx.currentTime);
      g.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + duration);
      osc.connect(g); g.connect(audioCtx.destination);
      osc.start(); osc.stop(audioCtx.currentTime + duration);
    } catch(e) {}
  }

  // ===== State =====
  const canvas = document.getElementById('game');
  const ctx = canvas.getContext('2d');
  const holdCanvas = document.getElementById('hold');
  const holdCtx = holdCanvas.getContext('2d');
  const nextCanvas = document.getElementById('next');
  const nextCtx = nextCanvas.getContext('2d');

  let board, current, nextQueue, bag, holdPiece, canHold;
  let score, lines, level, combo, b2b;
  let dropTimer, lockTimer, locking;
  let gameOver, paused, started;
  let lastTime;
  let particles = [];
  let flashRows = [];   // row indices being cleared
  let flashStart = 0;
  let shakeTime = 0, shakeMag = 0;
  let lastClearWasHard = false;

  // Input state
  const keys = {};
  let dasTimer = 0, arrTimer = 0, dasDir = 0;

  function newBoard() {
    const b = [];
    for (let y = 0; y < ROWS + HIDDEN; y++) {
      b.push(new Array(COLS).fill(null));
    }
    return b;
  }

  function refillBag() {
    const types = ['I','O','T','S','Z','J','L'];
    for (let i = types.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [types[i], types[j]] = [types[j], types[i]];
    }
    bag.push(...types);
  }

  function pullNext() {
    if (bag.length < 7) refillBag();
    return bag.shift();
  }

  function spawnPiece(type) {
    const def = PIECES[type];
    const shape = def.shapes[0].map(r => r.slice());
    return {
      type,
      rot: 0,
      x: type === 'O' ? 4 : 3,
      y: type === 'I' ? HIDDEN - 1 : HIDDEN,
      shape,
      color: def.color,
      glow: def.glow
    };
  }

  function collide(p, dx, dy, shape) {
    const s = shape || p.shape;
    for (let y = 0; y < s.length; y++) {
      for (let x = 0; x < s[y].length; x++) {
        if (!s[y][x]) continue;
        const nx = p.x + x + dx;
        const ny = p.y + y + dy;
        if (nx < 0 || nx >= COLS || ny >= ROWS + HIDDEN) return true;
        if (ny >= 0 && board[ny][nx]) return true;
      }
    }
    return false;
  }

  function merge(p) {
    for (let y = 0; y < p.shape.length; y++) {
      for (let x = 0; x < p.shape[y].length; x++) {
        if (p.shape[y][x]) {
          const by = p.y + y, bx = p.x + x;
          if (by >= 0 && by < ROWS + HIDDEN) {
            board[by][bx] = { color: p.color, glow: p.glow };
          }
        }
      }
    }
  }

  function rotate(p, dir) {
    if (p.type === 'O') return;
    const oldRot = p.rot;
    const newRot = (p.rot + dir + 4) % 4;
    const newShape = PIECES[p.type].shapes[newRot].map(r => r.slice());
    const kicks = (p.type === 'I' ? KICKS_I : KICKS_JLSTZ)[`${oldRot}->${newRot}`] || [[0,0]];
    for (const [kx, ky] of kicks) {
      if (!collide(p, kx, -ky, newShape)) {
        p.x += kx;
        p.y += -ky;
        p.rot = newRot;
        p.shape = newShape;
        sfx(320, 0.05, 'square', 0.04);
        lockTimer = 0;
        return true;
      }
    }
    return false;
  }

  function tryMove(dx, dy) {
    if (!collide(current, dx, dy)) {
      current.x += dx;
      current.y += dy;
      if (dx !== 0) {
        sfx(200, 0.03, 'square', 0.03);
        lockTimer = 0;
      }
      return true;
    }
    return false;
  }

  function ghostY() {
    let gy = 0;
    while (!collide(current, 0, gy + 1)) gy++;
    return current.y + gy;
  }

  function hardDrop() {
    const gy = ghostY();
    const dropped = gy - current.y;
    score += dropped * 2;
    current.y = gy;
    sfxSweep(400, 120, 0.1, 'square', 0.08);
    lastClearWasHard = true;
    lockPiece();
  }

  function softDropStep() {
    if (tryMove(0, 1)) {
      score += 1;
    }
  }

  function lockPiece() {
    merge(current);
    // Check spawn zone overflow (game over)
    let topOut = false;
    for (let x = 0; x < COLS; x++) {
      for (let y = 0; y < HIDDEN; y++) {
        if (board[y][x]) { topOut = true; break; }
      }
      if (topOut) break;
    }
    // Check line clears
    const cleared = [];
    for (let y = 0; y < ROWS + HIDDEN; y++) {
      if (board[y].every(c => c !== null)) cleared.push(y);
    }
    if (cleared.length > 0) {
      flashRows = cleared.slice();
      flashStart = performance.now();
      // Score
      const pts = [0, 100, 300, 500, 800][cleared.length] * level;
      score += pts;
      lines += cleared.length;
      combo++;
      if (cleared.length === 4) { b2b++; } else { b2b = 0; }
      const newLevel = Math.min(20, Math.floor(lines / 10) + 1);
      if (newLevel !== level) {
        level = newLevel;
        sfxSweep(400, 800, 0.4, 'sine', 0.1);
      }

      // Spawn particles
      for (const row of cleared) {
        for (let x = 0; x < COLS; x++) {
          const c = board[row][x];
          if (c) {
            for (let i = 0; i < 6; i++) {
              particles.push({
                x: x * BLOCK + BLOCK/2,
                y: (row - HIDDEN) * BLOCK + BLOCK/2,
                vx: (Math.random() - 0.5) * 6,
                vy: (Math.random() - 0.8) * 8,
                life: 1,
                color: c.color,
                size: 2 + Math.random() * 3
              });
            }
          }
        }
      }

      // Shake
      shakeMag = cleared.length === 4 ? 8 : 3;
      shakeTime = 200;

      // Flash text
      const msgs = { 1: 'SINGLE', 2: 'DOUBLE', 3: 'TRIPLE', 4: 'QUAD!' };
      const flashEl = document.getElementById('flash');
      flashEl.textContent = (b2b > 1 && cleared.length === 4 ? `B2B ${b2b} · ` : '') + msgs[cleared.length] + (combo > 1 ? ` ×${combo}` : '');
      flashEl.style.color = cleared.length === 4 ? '#4fc3ff' : '#ff4fd8';
      flashEl.classList.remove('show');
      void flashEl.offsetWidth;
      flashEl.classList.add('show');

      if (cleared.length === 4) sfxSweep(200, 900, 0.5, 'sawtooth', 0.1);
      else sfx(500 + cleared.length * 100, 0.15, 'triangle', 0.08);

      // Delay actual clear for animation
      setTimeout(() => {
        for (const row of cleared) {
          board.splice(row, 1);
          board.unshift(new Array(COLS).fill(null));
        }
        flashRows = [];
      }, 280);
    } else {
      combo = 0;
      sfx(180, 0.08, 'sine', 0.06);
    }

    if (topOut) {
      endGame();
      return;
    }

    // Spawn next piece
    canHold = true;
    current = spawnPiece(pullNext());
    lockTimer = 0;
    locking = false;
    // Check immediate collision = game over
    if (collide(current, 0, 0)) {
      endGame();
    }
  }

  function hold() {
    if (!canHold) return;
    canHold = false;
    const t = current.type;
    if (holdPiece) {
      current = spawnPiece(holdPiece);
    } else {
      current = spawnPiece(pullNext());
    }
    holdPiece = t;
    lockTimer = 0;
    sfx(440, 0.06, 'triangle', 0.06);
  }

  function dropSpeed() {
    // Frames per step (60fps)
    const speeds = [0, 48, 43, 38, 33, 28, 23, 18, 13, 8, 6, 5, 5, 5, 4, 4, 4, 3, 3, 3, 2];
    const f = speeds[Math.min(level, speeds.length - 1)] || 2;
    return f * (1000 / 60);
  }

  function endGame() {
    gameOver = true;
    started = false;
    sfxSweep(400, 80, 1.0, 'sawtooth', 0.1);
    const ov = document.getElementById('overlay');
    document.getElementById('ov-title').textContent = 'GAME OVER';
    document.getElementById('ov-body').innerHTML = `最終スコア: <b style="color:#4fc3ff">${score}</b><br>ライン: ${lines} · レベル: ${level}`;
    document.getElementById('startBtn').textContent = 'RETRY';
    ov.classList.add('show');
  }

  function resetGame() {
    board = newBoard();
    bag = [];
    refillBag();
    nextQueue = [];
    for (let i = 0; i < 5; i++) nextQueue.push(pullNext());
    current = spawnPiece(nextQueue.shift());
    nextQueue.push(pullNext());
    holdPiece = null;
    canHold = true;
    score = 0; lines = 0; level = 1; combo = 0; b2b = 0;
    dropTimer = 0; lockTimer = 0; locking = false;
    gameOver = false; paused = false; started = true;
    particles = []; flashRows = [];
    shakeTime = 0;
    updateUI();
  }

  function updateUI() {
    document.getElementById('score').textContent = score.toLocaleString();
    document.getElementById('level').textContent = level;
    document.getElementById('lines').textContent = lines;
    document.getElementById('combo').textContent = combo;
    document.getElementById('b2b').textContent = b2b;
  }

  // ===== Rendering =====
  function drawBlock(c, x, y, color, glow, alpha = 1) {
    const px = x * BLOCK, py = y * BLOCK;
    c.save();
    c.globalAlpha = alpha;
    // Base
    const grad = c.createLinearGradient(px, py, px + BLOCK, py + BLOCK);
    grad.addColorStop(0, color);
    grad.addColorStop(1, shade(color, -30));
    c.fillStyle = grad;
    c.fillRect(px + 1, py + 1, BLOCK - 2, BLOCK - 2);
    // Highlight
    c.fillStyle = 'rgba(255,255,255,0.25)';
    c.fillRect(px + 2, py + 2, BLOCK - 4, 3);
    c.fillRect(px + 2, py + 2, 3, BLOCK - 4);
    // Shadow edge
    c.fillStyle = 'rgba(0,0,0,0.3)';
    c.fillRect(px + 2, py + BLOCK - 5, BLOCK - 4, 3);
    c.fillRect(px + BLOCK - 5, py + 2, 3, BLOCK - 4);
    // Outer glow
    c.strokeStyle = glow;
    c.lineWidth = 1;
    c.strokeRect(px + 0.5, py + 0.5, BLOCK - 1, BLOCK - 1);
    c.restore();
  }

  function shade(hex, amt) {
    const h = hex.replace('#','');
    let r = parseInt(h.substr(0,2),16);
    let g = parseInt(h.substr(2,2),16);
    let b = parseInt(h.substr(4,2),16);
    r = Math.max(0, Math.min(255, r + amt));
    g = Math.max(0, Math.min(255, g + amt));
    b = Math.max(0, Math.min(255, b + amt));
    return '#' + [r,g,b].map(v => v.toString(16).padStart(2,'0')).join('');
  }

  function drawGhost(c, x, y, color) {
    const px = x * BLOCK, py = y * BLOCK;
    c.save();
    c.strokeStyle = color;
    c.globalAlpha = 0.35;
    c.lineWidth = 2;
    c.strokeRect(px + 2, py + 2, BLOCK - 4, BLOCK - 4);
    c.fillStyle = color;
    c.globalAlpha = 0.08;
    c.fillRect(px + 2, py + 2, BLOCK - 4, BLOCK - 4);
    c.restore();
  }

  function render() {
    // Shake
    let ox = 0, oy = 0;
    if (shakeTime > 0) {
      ox = (Math.random() - 0.5) * shakeMag;
      oy = (Math.random() - 0.5) * shakeMag;
    }

    ctx.save();
    ctx.translate(ox, oy);

    // Background grid
    ctx.fillStyle = '#0a0618';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    ctx.strokeStyle = 'rgba(100,80,200,0.08)';
    ctx.lineWidth = 1;
    for (let i = 1; i < COLS; i++) {
      ctx.beginPath();
      ctx.moveTo(i * BLOCK, 0);
      ctx.lineTo(i * BLOCK, canvas.height);
      ctx.stroke();
    }
    for (let i = 1; i < ROWS; i++) {
      ctx.beginPath();
      ctx.moveTo(0, i * BLOCK);
      ctx.lineTo(canvas.width, i * BLOCK);
      ctx.stroke();
    }

    // Board blocks
    const now = performance.now();
    const flashPhase = flashRows.length ? Math.min(1, (now - flashStart) / 280) : 0;
    for (let y = HIDDEN; y < ROWS + HIDDEN; y++) {
      for (let x = 0; x < COLS; x++) {
        const cell = board[y][x];
        if (cell) {
          const visY = y - HIDDEN;
          if (flashRows.includes(y)) {
            const alpha = 1 - flashPhase;
            ctx.save();
            ctx.globalAlpha = alpha;
            drawBlock(ctx, x, visY, '#ffffff', '#ffffff', 1);
            ctx.restore();
          } else {
            drawBlock(ctx, x, visY, cell.color, cell.glow);
          }
        }
      }
    }

    if (current && started && !gameOver) {
      // Ghost
      const gy = ghostY();
      for (let y = 0; y < current.shape.length; y++) {
        for (let x = 0; x < current.shape[y].length; x++) {
          if (current.shape[y][x]) {
            const visY = gy + y - HIDDEN;
            if (visY >= 0) drawGhost(ctx, current.x + x, visY, current.color);
          }
        }
      }
      // Current piece
      for (let y = 0; y < current.shape.length; y++) {
        for (let x = 0; x < current.shape[y].length; x++) {
          if (current.shape[y][x]) {
            const visY = current.y + y - HIDDEN;
            if (visY >= 0) drawBlock(ctx, current.x + x, visY, current.color, current.glow);
          }
        }
      }
    }

    // Particles
    for (const p of particles) {
      ctx.save();
      ctx.globalAlpha = p.life;
      ctx.fillStyle = p.color;
      ctx.shadowColor = p.color;
      ctx.shadowBlur = 8;
      ctx.fillRect(p.x - p.size/2, p.y - p.size/2, p.size, p.size);
      ctx.restore();
    }

    ctx.restore();

    // Hold panel
    holdCtx.fillStyle = 'rgba(5,3,15,0.6)';
    holdCtx.fillRect(0, 0, holdCanvas.width, holdCanvas.height);
    if (holdPiece) drawMiniPiece(holdCtx, holdPiece, holdCanvas.width / 2, holdCanvas.height / 2, !canHold);

    // Next panel
    nextCtx.fillStyle = 'rgba(5,3,15,0.6)';
    nextCtx.fillRect(0, 0, nextCanvas.width, nextCanvas.height);
    for (let i = 0; i < Math.min(5, nextQueue.length); i++) {
      drawMiniPiece(nextCtx, nextQueue[i], nextCanvas.width / 2, 40 + i * 68, false);
    }
  }

  function drawMiniPiece(c, type, cx, cy, greyed) {
    const def = PIECES[type];
    const shape = def.shapes[0];
    const w = shape[0].length, h = shape.length;
    const size = 18;
    // Find actual bounds to center
    let minX = w, maxX = -1, minY = h, maxY = -1;
    for (let y = 0; y < h; y++) for (let x = 0; x < w; x++) if (shape[y][x]) {
      if (x < minX) minX = x; if (x > maxX) maxX = x;
      if (y < minY) minY = y; if (y > maxY) maxY = y;
    }
    const pw = (maxX - minX + 1) * size;
    const ph = (maxY - minY + 1) * size;
    const startX = cx - pw / 2 - minX * size;
    const startY = cy - ph / 2 - minY * size;
    for (let y = 0; y < h; y++) {
      for (let x = 0; x < w; x++) {
        if (shape[y][x]) {
          const px = startX + x * size;
          const py = startY + y * size;
          c.save();
          if (greyed) c.globalAlpha = 0.35;
          const grad = c.createLinearGradient(px, py, px + size, py + size);
          grad.addColorStop(0, def.color);
          grad.addColorStop(1, shade(def.color, -30));
          c.fillStyle = grad;
          c.fillRect(px + 1, py + 1, size - 2, size - 2);
          c.fillStyle = 'rgba(255,255,255,0.2)';
          c.fillRect(px + 1, py + 1, size - 2, 2);
          c.restore();
        }
      }
    }
  }

  // ===== Main loop =====
  function tick(now) {
    if (!lastTime) lastTime = now;
    const dt = now - lastTime;
    lastTime = now;

    if (started && !paused && !gameOver) {
      // DAS/ARR horizontal
      if (dasDir !== 0) {
        dasTimer += dt;
        if (dasTimer >= DAS) {
          arrTimer += dt;
          while (arrTimer >= ARR) {
            arrTimer -= ARR;
            if (!tryMove(dasDir, 0)) break;
          }
        }
      }

      // Soft drop
      if (keys['ArrowDown']) {
        dropTimer += dt * 15;
      }

      // Gravity
      dropTimer += dt;
      const step = dropSpeed();
      while (dropTimer >= step) {
        dropTimer -= step;
        if (!tryMove(0, 1)) {
          locking = true;
          break;
        }
      }

      // Lock delay
      if (locking && collide(current, 0, 1)) {
        lockTimer += dt;
        if (lockTimer >= LOCK_DELAY) {
          lockPiece();
        }
      } else {
        lockTimer = 0;
        locking = false;
      }

      // Particles
      for (const p of particles) {
        p.x += p.vx;
        p.y += p.vy;
        p.vy += 0.3;
        p.life -= 0.02;
      }
      particles = particles.filter(p => p.life > 0);

      // Shake
      if (shakeTime > 0) shakeTime -= dt;

      updateUI();
    }

    render();
    requestAnimationFrame(tick);
  }

  // ===== Input =====
  window.addEventListener('keydown', e => {
    if (['ArrowLeft','ArrowRight','ArrowDown','ArrowUp',' '].includes(e.key)) e.preventDefault();
    if (keys[e.key]) return;
    keys[e.key] = true;

    if (e.key === 'p' || e.key === 'P') {
      if (started && !gameOver) {
        paused = !paused;
        const ov = document.getElementById('overlay');
        if (paused) {
          document.getElementById('ov-title').textContent = 'PAUSED';
          document.getElementById('ov-body').textContent = 'P で再開';
          document.getElementById('startBtn').textContent = 'RESUME';
          ov.classList.add('show');
        } else {
          ov.classList.remove('show');
        }
      }
      return;
    }
    if (e.key === 'm' || e.key === 'M') { muted = !muted; return; }

    if (!started || paused || gameOver) return;

    if (e.key === 'ArrowLeft') {
      dasDir = -1; dasTimer = 0; arrTimer = 0;
      tryMove(-1, 0);
    } else if (e.key === 'ArrowRight') {
      dasDir = 1; dasTimer = 0; arrTimer = 0;
      tryMove(1, 0);
    } else if (e.key === 'ArrowUp' || e.key === 'x' || e.key === 'X') {
      rotate(current, 1);
    } else if (e.key === 'z' || e.key === 'Z') {
      rotate(current, -1);
    } else if (e.key === ' ') {
      hardDrop();
    } else if (e.key === 'c' || e.key === 'C' || e.key === 'Shift') {
      hold();
    }
  });
  window.addEventListener('keyup', e => {
    keys[e.key] = false;
    if (e.key === 'ArrowLeft' && dasDir === -1) dasDir = 0;
    if (e.key === 'ArrowRight' && dasDir === 1) dasDir = 0;
  });

  document.getElementById('startBtn').addEventListener('click', () => {
    if (paused && !gameOver) {
      paused = false;
      document.getElementById('overlay').classList.remove('show');
      return;
    }
    resetGame();
    document.getElementById('overlay').classList.remove('show');
    // Resume audio on user gesture
    try { if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)(); audioCtx.resume(); } catch(e){}
  });

  // Prevent space/arrows from scrolling
  window.addEventListener('keydown', e => {
    if ([' ','ArrowLeft','ArrowRight','ArrowUp','ArrowDown'].includes(e.key)) e.preventDefault();
  }, { passive: false });

  // Start render loop immediately so overlay is drawn
  board = newBoard();
  bag = [];
  nextQueue = [];
  holdPiece = null;
  score = lines = combo = b2b = 0; level = 1;
  requestAnimationFrame(tick);
})();
</script>

</body>
</html>

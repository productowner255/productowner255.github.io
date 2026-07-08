<!DOCTYPE html>
<html lang="bg">
<head>
<script async src="https://www.googletagmanager.com/gtag/js?id=G-P3BYSS346P"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-P3BYSS346P');
</script>
<script type="text/javascript">
    (function(c,l,a,r,i,t,y){
        c[a]=c[a]||function(){(c[a].q=c[a].q||[]).push(arguments)};
        t=l.createElement(r);t.async=1;t.src="https://www.clarity.ms/tag/"+i;
        y=l.getElementsByTagName(r)[0];y.parentNode.insertBefore(t,y);
    })(window, document, "clarity", "script", "xj5ik2owc0");
</script>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Arcade World Master - GitHub Edition</title>
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #6366f1; --secondary: #ec4899; --accent: #f59e0b;
            --bg: #0f172a; --card: rgba(30, 41, 59, 0.9); --text: #f8fafc;
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
        body {
            font-family: 'Montserrat', sans-serif; margin: 0; background: var(--bg); color: var(--text);
            display: flex; justify-content: center; align-items: center; min-height: 100vh; overflow: hidden;
        }

        #app {
            width: 100%; max-width: 450px; height: 95vh; background: var(--card);
            backdrop-filter: blur(20px); border: 2px solid rgba(255,255,255,0.1);
            border-radius: 30px; display: flex; flex-direction: column; position: relative;
            transition: transform 0.1s ease;
        }

        /* АНИМАЦИИ */
        .shake { animation: shake 0.4s both; }
        @keyframes shake { 10%, 90% { transform: translate3d(-2px, 0, 0); } 20%, 80% { transform: translate3d(3px, 0, 0); } 30%, 50%, 70% { transform: translate3d(-5px, 0, 0); } 40%, 60% { transform: translate3d(5px, 0, 0); } }
        
        .flash-red { animation: flashRed 0.5s; }
        @keyframes flashRed { 0% { background: rgba(255,0,0,0); } 50% { background: rgba(255,0,0,0.3); } 100% { background: rgba(255,0,0,0); } }

        /* UI ЕЛЕМЕНТИ */
        .header { padding: 15px 20px; display: flex; justify-content: space-between; align-items: center; background: rgba(255,255,255,0.05); border-radius: 30px 30px 0 0; }
        .stat-group { display: flex; flex-direction: column; align-items: center; }
        .stat-val { font-weight: 900; font-size: 1.1rem; }
        .timer-text { font-size: 0.7rem; color: #94a3b8; }

        .screen { flex: 1; display: none; flex-direction: column; padding: 20px; overflow-y: auto; }
        .active { display: flex; }

        .menu-btn {
            background: linear-gradient(135deg, var(--primary), var(--secondary));
            color: white; border: none; padding: 18px; border-radius: 15px;
            margin-bottom: 12px; cursor: pointer; font-weight: bold; font-size: 1rem;
            display: flex; justify-content: space-between; align-items: center; width: 100%;
            transition: transform 0.2s;
        }
        .menu-btn:active { transform: scale(0.95); }

        .grid { display: grid; gap: 8px; background: #334155; padding: 10px; border-radius: 15px; margin: 10px 0; width: 100%; aspect-ratio: 1; }
        .cell { background: #1e293b; border-radius: 8px; display: flex; justify-content: center; align-items: center; font-weight: 900; font-size: 1.5rem; cursor: pointer; color: white; }
        .cell.fixed { color: #94a3b8; background: #0f172a; cursor: default; }
        .cell.selected { background: var(--primary); transform: scale(1.05); box-shadow: 0 0 15px var(--primary); }

        canvas { background: #000; border-radius: 15px; width: 100%; display: block; margin: 0 auto; border: 2px solid #334155; }

        .modal { position: absolute; inset: 0; background: rgba(0,0,0,0.95); z-index: 1000; display: none; flex-direction: column; align-items: center; justify-content: center; padding: 30px; text-align: center; border-radius: 30px; }

        .controls { display: grid; grid-template-columns: repeat(3, 1fr); gap: 8px; margin: 15px auto; width: 180px; }
        .btn-ctrl { background: #475569; border: none; padding: 15px; border-radius: 12px; color: white; font-size: 1.2rem; cursor: pointer; }
        
        .t-2 { background: #eee4da; color: #776e65; } .t-4 { background: #ede0c8; color: #776e65; }
        .t-8 { background: #f2b179; color: white; } .t-16 { background: #f59563; color: white; }
        .t-32 { background: #f67c5f; color: white; } .t-64 { background: #f65e3b; color: white; }
        .t-128 { background: #edcf72; color: white; } .t-256 { background: #edcc61; color: white; }
        .t-512 { background: #edc850; color: white; } .t-1024 { background: #edc53f; color: white; }
        .t-2048 { background: #edc22e; color: white; }
    </style>
</head>
<body>

<div id="app">
    <div id="flash-overlay" style="position:absolute; inset:0; pointer-events:none; border-radius:30px; z-index:99;"></div>
    
    <div class="header">
        <div class="stat-group">
            <div class="stat-val">❤️ <span id="lives">5</span></div>
            <div id="life-timer" class="timer-text">Full</div>
        </div>
        <div class="stat-group">
            <div class="stat-val" style="color: #fbbf24;">💰 <span id="coins">80</span></div>
            <div class="timer-text">Монети</div>
        </div>
    </div>

    <!-- MODAL -->
    <div id="game-modal" class="modal">
        <h1 id="m-icon" style="font-size: 4rem; margin:0;">🏆</h1>
        <h2 id="m-title" style="margin:10px 0;">Браво!</h2>
        <p id="m-desc">Ниво завършено!</p>
        <button class="menu-btn" style="justify-content: center;" id="m-restart">ИГРАЙ ПАК</button>
        <button class="menu-btn" style="justify-content: center; background: #334155; margin-top: 10px;" onclick="backToMenu()">МЕНЮ</button>
    </div>

    <!-- SCREENS -->
    <div id="menu-screen" class="screen active">
        <h1 style="text-align:center; margin-bottom: 20px;">Arcade World</h1>
        <button class="menu-btn" onclick="initSudoku()">🧩 СУДОКУ <span id="sdk-lvl-display">Ниво 1</span></button>
        <button class="menu-btn" onclick="initSnake()">🐍 ЗМИЯ <span>Record: <b id="rec-snake">0</b></span></button>
        <button class="menu-btn" onclick="init2048()">🔢 2048 <span>Record: <b id="rec-2048">0</b></span></button>
        <button class="menu-btn" onclick="initPong()">🏓 ТЕНИС <span>Record: <b id="rec-pong">0</b></span></button>
    </div>

    <div id="game-screen" class="screen">
        <div style="display:flex; justify-content:space-between; align-items: center; margin-bottom:10px;">
            <h2 id="game-title" style="margin:0;">Игра</h2>
            <button id="hint-btn" class="menu-btn" style="width: auto; padding: 8px 15px; background: var(--accent); display:none" onclick="buyHint()">💡 Hint (80)</button>
        </div>
        <div id="board-container"></div>
        <div id="game-controls"></div>
        <button class="menu-btn" style="background:#334155; margin-top:20px; justify-content: center;" onclick="backToMenu()">НАЗАД</button>
    </div>
</div>

<script>
    // --- STATE & DATA ---
    let state = JSON.parse(localStorage.getItem('arcade_pro_final')) || {
        coins: 80, lives: 5, sdkLvl: 1, lastLifeTime: Date.now(),
        records: { sdk: 0, snake: 0, t2048: 0, pong: 0 }
    };

    function save() {
        localStorage.setItem('arcade_pro_final', JSON.stringify(state));
        document.getElementById('lives').innerText = state.lives;
        document.getElementById('coins').innerText = state.coins;
        document.getElementById('rec-snake').innerText = state.records.snake;
        document.getElementById('rec-2048').innerText = state.records.t2048;
        document.getElementById('rec-pong').innerText = state.records.pong;
        document.getElementById('sdk-lvl-display').innerText = `Ниво ${state.sdkLvl}`;
    }

    // --- REGEN LIVES (3 MINS) ---
    setInterval(() => {
        if(state.lives < 5) {
            let diff = Date.now() - state.lastLifeTime;
            let cooldown = 180000; 
            if(diff >= cooldown) { state.lives++; state.lastLifeTime = Date.now(); save(); }
            let rem = Math.ceil((cooldown - diff) / 1000);
            document.getElementById('life-timer').innerText = `${Math.floor(rem/60)}:${(rem%60).toString().padStart(2,'0')}`;
        } else { document.getElementById('life-timer').innerText = "Full"; }
    }, 1000);

    let loop = null;
    function showScreen(id) {
        clearInterval(loop);
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        document.getElementById('game-modal').style.display = 'none';
    }

    function backToMenu() { showScreen('menu-screen'); }

    function triggerDamage() {
        document.getElementById('app').classList.add('shake');
        document.getElementById('flash-overlay').classList.add('flash-red');
        setTimeout(() => {
            document.getElementById('app').classList.remove('shake');
            document.getElementById('flash-overlay').classList.remove('flash-red');
        }, 500);
        state.lives--; 
        if(state.lives === 4) state.lastLifeTime = Date.now();
        save();
        if(state.lives <= 0) endGame('💀', 'ГЕЙМ ОУВЪР', 'Нямаш повече животи!', null);
    }

    function endGame(icon, title, desc, restartFn, btnText = "ИГРАЙ ПАК") {
        clearInterval(loop);
        document.getElementById('m-icon').innerText = icon;
        document.getElementById('m-title').innerText = title;
        document.getElementById('m-desc').innerText = desc;
        document.getElementById('game-modal').style.display = 'flex';
        const rBtn = document.getElementById('m-restart');
        rBtn.innerText = btnText;
        rBtn.onclick = restartFn;
        save();
    }

    // --- SUDOKU ENGINE ---
    let sdk = {};
    function initSudoku() {
        if(state.lives <= 0) return;
        showScreen('game-screen');
        document.getElementById('game-title').innerText = `Судоку - Ниво ${state.sdkLvl}`;
        document.getElementById('hint-btn').style.display = 'block';
        const container = document.getElementById('board-container');
        container.innerHTML = '<div id="sdk-grid" class="grid" style="grid-template-columns: repeat(4, 1fr)"></div>';
        const sol = [1,2,3,4, 3,4,1,2, 2,1,4,3, 4,3,2,1];
        let hideChance = Math.min(0.8, 0.4 + (state.sdkLvl * 0.05));
        let board = sol.map(v => Math.random() < hideChance ? 0 : v);
        if(!board.includes(0)) board[Math.floor(Math.random()*16)] = 0;
        sdk = { sol: sol, board: board, selected: null };
        const controls = document.getElementById('game-controls');
        controls.innerHTML = '<div id="sdk-keypad" style="display:grid; grid-template-columns:repeat(4,1fr); gap:10px"></div>';
        for(let i=1; i<=4; i++) {
            let b = document.createElement('button'); b.className = 'btn-ctrl'; b.innerText = i;
            b.onclick = () => {
                if(sdk.selected !== null && sdk.board[sdk.selected] === 0) {
                    if(sdk.sol[sdk.selected] === i) {
                        sdk.board[sdk.selected] = i; drawSudoku();
                        if(!sdk.board.includes(0)) {
                            state.coins += 30; state.sdkLvl++; save();
                            endGame('🏆', 'ПОБЕДА!', `Завърши ниво ${state.sdkLvl-1}! +30 монети`, initSudoku, "СЛЕДВАЩО НИВО");
                        }
                    } else { triggerDamage(); }
                }
            };
            document.getElementById('sdk-keypad').appendChild(b);
        }
        drawSudoku();
    }

    function drawSudoku() {
        const grid = document.getElementById('sdk-grid'); grid.innerHTML = '';
        sdk.board.forEach((v, i) => {
            let c = document.createElement('div');
            c.className = `cell ${v !== 0 ? 'fixed' : ''} ${sdk.selected === i ? 'selected' : ''}`;
            c.innerText = v || '';
            c.onclick = () => { sdk.selected = i; drawSudoku(); };
            grid.appendChild(c);
        });
    }

    function buyHint() {
        if(state.coins >= 80) {
            let empty = sdk.board.map((v,i) => v===0?i:null).filter(v=>v!==null);
            if(empty.length) { 
                let r = empty[Math.floor(Math.random()*empty.length)]; 
                sdk.board[r] = sdk.sol[r]; state.coins -= 80; drawSudoku(); save(); 
            }
        }
    }

    // --- SNAKE ENGINE ---
    let snk = { body: [], dir: 'R', food: {} };
    function initSnake() {
        showScreen('game-screen');
        document.getElementById('game-title').innerText = "Змия";
        document.getElementById('hint-btn').style.display = 'none';
        document.getElementById('board-container').innerHTML = '<canvas id="snk-canvas" width="300" height="300"></canvas>';
        document.getElementById('game-controls').innerHTML = `
            <div class="controls">
                <div></div><button class="btn-ctrl" onclick="snk.dir='U'">↑</button><div></div>
                <button class="btn-ctrl" onclick="snk.dir='L'">←</button>
                <button class="btn-ctrl" onclick="snk.dir='D'">↓</button>
                <button class="btn-ctrl" onclick="snk.dir='R'">→</button>
            </div>`;
        snk.body = [{x:10, y:10}, {x:9, y:10}]; snk.dir = 'R'; snk.food = {x:15, y:15}; let score = 0;
        const ctx = document.getElementById('snk-canvas').getContext('2d');
        loop = setInterval(() => {
            let h = {...snk.body[0]};
            if(snk.dir==='U') h.y--; if(snk.dir==='D') h.y++; if(snk.dir==='L') h.x--; if(snk.dir==='R') h.x++;
            if(h.x<0 || h.x>=20 || h.y<0 || h.y>=20 || snk.body.some(s=>s.x===h.x && s.y===h.y)) {
                state.coins += score; if(score > state.records.snake) state.records.snake = score;
                endGame('🐍', 'КРАЙ', `Спечели ${score} монети`, initSnake); return;
            }
            snk.body.unshift(h);
            if(h.x === snk.food.x && h.y === snk.food.y) { score++; snk.food = {x:Math.floor(Math.random()*20), y:Math.floor(Math.random()*20)}; } else snk.body.pop();
            ctx.fillStyle = '#000'; ctx.fillRect(0,0,300,300);
            ctx.fillStyle = 'red'; ctx.beginPath(); ctx.arc(snk.food.x*15+7.5, snk.food.y*15+7.5, 7, 0, Math.PI*2); ctx.fill();
            snk.body.forEach((s, i) => { ctx.fillStyle = i===0 ? '#4ade80' : '#166534'; ctx.fillRect(s.x*15, s.y*15, 14, 14); });
        }, 130);
    }

    // --- 2048 ENGINE ---
    let b2048 = [];
    let current2048Score = 0;
    function init2048() {
        showScreen('game-screen');
        current2048Score = 0;
        document.getElementById('game-title').innerText = "2048: 0";
        document.getElementById('hint-btn').style.display = 'none';
        document.getElementById('board-container').innerHTML = '<div id="grid-2048" class="grid" style="grid-template-columns:repeat(4,1fr)"></div>';
        document.getElementById('game-controls').innerHTML = `
            <div class="controls">
                <div></div><button class="btn-ctrl" onclick="move2048('U')">↑</button><div></div>
                <button class="btn-ctrl" onclick="move2048('L')">←</button>
                <button class="btn-ctrl" onclick="move2048('D')">↓</button>
                <button class="btn-ctrl" onclick="move2048('R')">→</button>
            </div>`;
        b2048 = Array(16).fill(0); add2048(); add2048(); draw2048();
    }
    function add2048() { let empty = b2048.map((v,i)=>v===0?i:null).filter(v=>v!==null); if(empty.length) b2048[empty[Math.floor(Math.random()*empty.length)]] = 2; }
    function draw2048() {
        const g = document.getElementById('grid-2048'); g.innerHTML = '';
        b2048.forEach(v => { let c = document.createElement('div'); c.className = `cell t-${v}`; c.innerText = v || ''; g.appendChild(c); });
    }
    function move2048(d) {
        let moved = false;
        const combine = (row) => {
            let res = row.filter(v => v !== 0);
            for(let i=0; i<res.length-1; i++) { 
                if(res[i] === res[i+1]) { 
                    res[i]*=2; 
                    current2048Score += res[i]; 
                    res.splice(i+1, 1); 
                    moved = true; 
                } 
            }
            while(res.length < 4) res.push(0); return res;
        };
        for(let i=0; i<4; i++) {
            let row = [];
            for(let j=0; j<4; j++) { if(d==='L' || d==='R') row.push(b2048[i*4 + j]); else row.push(b2048[j*4 + i]); }
            if(d==='R' || d==='D') row.reverse();
            let newRow = combine(row);
            if(d==='R' || d==='D') newRow.reverse();
            if(JSON.stringify(row) !== JSON.stringify(newRow)) moved = true;
            for(let j=0; j<4; j++) { if(d==='L' || d==='R') b2048[i*4 + j] = newRow[j]; else b2048[j*4 + i] = newRow[j]; }
        }
        if(moved) { 
            add2048(); 
            draw2048(); 
            document.getElementById('game-title').innerText = `2048: ${current2048Score}`;
            save(); 
        }
        
        if(!b2048.includes(0)) {
            let canMove = false;
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 4; j++) {
                    let val = b2048[i * 4 + j];
                    if (j < 3 && val === b2048[i * 4 + (j + 1)]) canMove = true;
                    if (i < 3 && val === b2048[(i + 1) * 4 + j]) canMove = true;
                }
            }
            
            if(!canMove) {
                let maxTile = Math.max(...b2048); 
                state.coins += maxTile; 
                if(current2048Score > state.records.t2048) state.records.t2048 = current2048Score;
                endGame('🔢', 'КРАЙ', `Най-голямо число: ${maxTile} | Спечели ${maxTile} монети!`, init2048);
            }
        }
    }

    // --- PONG ENGINE ---
    function initPong() {
        showScreen('game-screen');
        let pongScore = 0;
        document.getElementById('game-title').innerText = "Тенис: 0";
        document.getElementById('hint-btn').style.display = 'none';
        document.getElementById('board-container').innerHTML = '<canvas id="pong-canvas" width="300" height="400"></canvas>';
        document.getElementById('game-controls').innerHTML = '<p style="text-align:center; opacity:0.5; margin:10px 0;">Движи с мишка или пръст</p>';
        const canvas = document.getElementById('pong-canvas');
        const ctx = canvas.getContext('2d');
        let ball = {x:150, y:200, dx:3, dy:3}, pad = 120;
        canvas.ontouchmove = (e) => { e.preventDefault(); pad = e.touches[0].clientX - canvas.getBoundingClientRect().left - 35; };
        canvas.onmousemove = (e) => pad = e.offsetX - 35;
        loop = setInterval(() => {
            ctx.fillStyle = '#000'; ctx.fillRect(0,0,300,400);
            ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(ball.x, ball.y, 8, 0, Math.PI*2); ctx.fill();
            ctx.fillStyle = '#6366f1'; ctx.fillRect(pad, 380, 70, 10);
            ball.x += ball.dx; ball.y += ball.dy;
            if(ball.x<8 || ball.x>292) ball.dx *= -1;
            if(ball.y<8) ball.dy *= -1;
            if(ball.y>370 && ball.x>pad && ball.x<pad+70) { ball.dy *= -1.1; pongScore++; document.getElementById('game-title').innerText = `Тенис: ${pongScore}`; }
            if(ball.y>400) { state.coins += pongScore; if(pongScore > state.records.pong) state.records.pong = pongScore; endGame('🏓', 'КРАЙ', `Резултат: ${pongScore} | Спечели ${pongScore} монети`, initPong); }
        }, 1000/60);
    }

    window.onkeydown = (e) => {
        if(e.key === 'ArrowUp') snk.dir = 'U';
        if(e.key === 'ArrowDown') snk.dir = 'D';
        if(e.key === 'ArrowLeft') snk.dir = 'L';
        if(e.key === 'ArrowRight') snk.dir = 'R';
    };

    save();
</script>
</body>
</html>

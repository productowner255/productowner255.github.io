<!DOCTYPE html>
<html lang="bg">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Arcade World Master - Ultimate Edition</title>
    <link href="https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #6366f1; --secondary: #ec4899; --accent: #f59e0b;
            --bg: #020617; --card: rgba(15, 23, 42, 0.98); --text: #f8fafc;
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; outline: none; }
        body {
            font-family: 'Montserrat', sans-serif; margin: 0; background: var(--bg); color: var(--text);
            display: flex; justify-content: center; align-items: center; min-height: 100vh; overflow: hidden;
        }

        #app {
            width: 100%; max-width: 450px; height: 95vh; background: var(--card);
            backdrop-filter: blur(25px); border: 1px solid rgba(255,255,255,0.1);
            border-radius: 40px; display: flex; flex-direction: column; position: relative;
            box-shadow: 0 0 50px rgba(0,0,0,0.8);
        }

        .screen { 
            flex: 1; display: none; flex-direction: column; padding: 20px; 
            overflow-y: auto; opacity: 0; transform: scale(0.98);
            transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
        }
        .screen.active { display: flex; opacity: 1; transform: scale(1); }

        .title-glow {
            font-size: 2.5rem; font-weight: 900; text-align: center;
            background: linear-gradient(135deg, #6366f1, #ec4899, #f59e0b);
            -webkit-background-clip: text; -webkit-text-fill-color: transparent;
            animation: float 3s ease-in-out infinite; margin: 20px 0;
        }
        @keyframes float { 0%, 100% { transform: translateY(0); filter: drop-shadow(0 0 10px rgba(99,102,241,0.4)); } 50% { transform: translateY(-10px); filter: drop-shadow(0 0 20px rgba(236,72,153,0.6)); } }

        .header { padding: 15px 25px; display: flex; justify-content: space-between; align-items: center; background: rgba(255,255,255,0.02); border-radius: 40px 40px 0 0; }
        .stat-val { font-weight: 900; font-size: 1.1rem; }
        .timer-text { font-size: 0.7rem; color: #94a3b8; }

        .menu-btn {
            background: linear-gradient(135deg, var(--primary), var(--secondary));
            color: white; border: none; padding: 16px; border-radius: 20px;
            margin-bottom: 10px; cursor: pointer; font-weight: bold; font-size: 0.9rem;
            display: flex; justify-content: space-between; align-items: center;
            transition: 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            box-shadow: 0 8px 15px rgba(0,0,0,0.2);
        }
        .menu-btn:hover { transform: scale(1.02); }

        .d-pad { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-top: 15px; justify-items: center; }
        .arrow-btn {
            width: 60px; height: 60px; background: rgba(255,255,255,0.08); border: 1px solid rgba(255,255,255,0.1);
            border-radius: 20px; display: flex; justify-content: center; align-items: center;
            color: white; font-size: 1.5rem; cursor: pointer; transition: 0.2s;
        }
        .arrow-btn:active { background: var(--primary); transform: scale(0.9); box-shadow: 0 0 20px var(--primary); }

        /* SUDOKU LEVEL MENU */
        .lvl-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px; margin-top: 10px; }
        .lvl-btn { 
            aspect-ratio: 1; background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.1); 
            border-radius: 15px; color: white; font-weight: 900; cursor: pointer; 
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            transition: all 0.3s ease; font-size: 1.1rem;
        }
        .lvl-btn.locked { opacity: 0.3; cursor: not-allowed; }
        .lvl-btn.locked::after { content: '🔒'; font-size: 0.7rem; margin-top: 4px; }
        .lvl-btn.current { border: 2px solid var(--accent); box-shadow: 0 0 10px var(--accent); animation: pulse 2s infinite; }
        @keyframes pulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.05); } }

        /* GRIDS */
        .grid { display: grid; gap: 2px; background: #334155; padding: 5px; border-radius: 15px; width: 100%; aspect-ratio: 1; border: 2px solid #334155; }
        .cell { background: #0f172a; display: flex; justify-content: center; align-items: center; font-weight: 900; color: white; transition: 0.2s; cursor: pointer; font-size: 1.3rem; }
        .cell.fixed { background: #020617; color: #64748b; cursor: default; }
        .cell.selected { background: #1e293b; box-shadow: inset 0 0 0 2px var(--primary); }
        .bb { border-bottom: 3px solid #334155 !important; }
        .br { border-right: 3px solid #334155 !important; }

        /* 2048 Tiles */
        .t-2 { background: #eee4da; color: #776e65; } .t-4 { background: #ede0c8; color: #776e65; }
        .t-8 { background: #f2b179; color: white; } .t-16 { background: #f59563; color: white; }
        .t-32 { background: #f67c5f; color: white; } .t-64 { background: #f65e3b; color: white; }
        .t-128 { background: #edcf72; color: white; } .t-256 { background: #edcc61; color: white; }
        .t-512 { background: #edc850; color: white; } .t-1024 { background: #edc53f; color: white; }
        .t-2048 { background: #edc22e; color: white; box-shadow: 0 0 10px #edc22e; }

        canvas { background: #000; border-radius: 25px; width: 100%; border: 2px solid #1e293b; }
        
        .shop-item { background: rgba(255,255,255,0.03); padding: 12px 18px; border-radius: 20px; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; border: 1px solid rgba(255,255,255,0.05); }
        .buy-btn { background: var(--primary); border: none; color: white; padding: 8px 15px; border-radius: 12px; font-weight: bold; cursor: pointer; }
        .buy-btn:disabled { background: #16a34a; cursor: default; }

        .modal { position: absolute; inset: 0; background: rgba(2, 6, 23, 0.98); z-index: 1000; display: none; flex-direction: column; align-items: center; justify-content: center; padding: 40px; border-radius: 40px; text-align: center; }
        #score-container { display: flex; gap: 10px; align-items: center; }
        .score-box { background: rgba(245, 158, 11, 0.15); padding: 5px 12px; border-radius: 10px; border: 1px solid var(--accent); color: var(--accent); font-weight: 900; font-size: 0.9rem; }
        
        .social-container { display: flex; gap: 10px; margin-top: auto; }
        .social-btn { flex: 1; padding: 12px; border-radius: 15px; text-align: center; text-decoration: none; color: white; font-weight: 900; font-size: 0.7rem; }
    </style>
</head>
<body>

<div id="app">
    <div id="flash-overlay" style="position:absolute; inset:0; pointer-events:none; border-radius:40px; z-index:99;"></div>
    
    <div class="header">
        <div class="stat-group"><div class="stat-val">❤️ <span id="lives">10</span>/10</div><div id="life-timer" class="timer-text">Full</div></div>
        <div class="stat-group"><div class="stat-val" style="color: #fbbf24;">💰 <span id="coins">80</span></div><div class="timer-text">Портфейл</div></div>
    </div>

    <div id="game-modal" class="modal">
        <h1 id="m-icon" style="font-size: 5rem; margin:0;">🏆</h1>
        <h2 id="m-title">КРАЙ!</h2>
        <p id="m-desc">Резултат: 0</p>
        <button class="menu-btn" style="justify-content: center; width: 100%;" id="m-restart">ИГРАЙ ПАК</button>
        <button class="menu-btn" style="justify-content: center; width: 100%; background: #1e293b; margin-top: 10px;" onclick="backToMenu()">МЕНЮ</button>
    </div>

    <div id="menu-screen" class="screen active">
        <div class="title-glow">ARCADE WORLD</div>
        <button class="menu-btn" onclick="showSudokuLevels()">🧩 СУДОКУ <span>Нива 1-100</span></button>
        <button class="menu-btn" onclick="initSnake()">🐍 ЗМИЯ <span>Rec: <b id="rec-snake">0</b></span></button>
        <button class="menu-btn" onclick="init2048()">🔢 2048 <span>Rec: <b id="rec-2048">0</b></span></button>
        <button class="menu-btn" onclick="initPong()">🏓 ТЕНИС <span>Rec: <b id="rec-pong">0</b></span></button>
        <button class="menu-btn" onclick="initSkyDash()">🚀 SKY DASH <span>Rec: <b id="rec-sky">0</b></span></button>
        <button class="menu-btn" onclick="initStarRaid()">🌠 STAR RAID <span>Rec: <b id="rec-star">0</b></span></button>
        <button class="menu-btn" style="background: var(--accent);" onclick="showShop()">🛒 МАГАЗИН (АКСЕСОАРИ)</button>
        
        <div class="social-container">
            <a href="https://www.tiktok.com/@arcade_verse" target="_blank" class="social-btn" style="background:#000; border:1px solid #ff0050;">TIKTOK</a>
            <a href="https://www.instagram.com/arcad_everse/" target="_blank" class="social-btn" style="background:linear-gradient(45deg,#f09433,#e6683c,#dc2743,#cc2366,#bc1888);">INSTAGRAM</a>
        </div>
    </div>

    <div id="sdk-levels-screen" class="screen">
        <h2 style="text-align:center;">Избери Ниво</h2>
        <div id="lvl-container" class="lvl-grid"></div>
        <button class="menu-btn" style="background:#1e293b; margin-top:30px; justify-content: center;" onclick="backToMenu()">НАЗАД</button>
    </div>

    <div id="shop-screen" class="screen">
        <h2 style="text-align:center;">Магазин</h2>
        <div id="shop-items-container"></div>
        <button class="menu-btn" style="background:#1e293b; margin-top:20px; justify-content: center;" onclick="backToMenu()">НАЗАД</button>
    </div>

    <div id="game-screen" class="screen">
        <div style="display:flex; justify-content:space-between; align-items: center; margin-bottom:15px;">
            <button onclick="backToMenu()" style="background:rgba(255,255,255,0.1); border:none; color:white; padding:8px 15px; border-radius:12px; font-weight:bold; cursor:pointer;">← НАЗАД</button>
            <div id="score-container">
                <div class="score-box" id="best-display">Best: 0</div>
                <div class="score-box" id="score-display">0</div>
            </div>
        </div>
        <div id="board-container"></div>
        <div class="game-controls" id="ctrl-area">
            <div class="d-pad">
                <div></div><button class="arrow-btn" id="btn-up">↑</button><div></div>
                <button class="arrow-btn" id="btn-left">←</button>
                <button class="arrow-btn" id="btn-down">↓</button>
                <button class="arrow-btn" id="btn-right">→</button>
            </div>
        </div>
    </div>
</div>

<script>
    let state = JSON.parse(localStorage.getItem('arcade_final_v14')) || {
        coins: 80, lives: 10, sdkLvl: 1, lastLifeTime: Date.now(),
        records: { snake: 0, t2048: 0, pong: 0, sky: 0, star: 0 },
        items: { diaSnake: false, goldPaddle: false, fireSky: false, neonSdk: false }
    };

    let loop = null, currentGame = "", raidPlayerX = 140, snkDir = {x:1, y:0}, b2048 = [];

    function save() {
        localStorage.setItem('arcade_final_v14', JSON.stringify(state));
        document.getElementById('lives').innerText = state.lives;
        document.getElementById('coins').innerText = state.coins;
        document.getElementById('rec-snake').innerText = state.records.snake;
        document.getElementById('rec-2048').innerText = state.records.t2048;
        document.getElementById('rec-pong').innerText = state.records.pong;
        document.getElementById('rec-sky').innerText = state.records.sky;
        document.getElementById('rec-star').innerText = state.records.star;
    }

    setInterval(() => {
        if(state.lives < 10) {
            let diff = Date.now() - state.lastLifeTime;
            if(diff >= 180000) { state.lives++; state.lastLifeTime = Date.now(); save(); }
            let rem = Math.ceil((180000 - diff) / 1000);
            document.getElementById('life-timer').innerText = `${Math.floor(rem/60)}:${(rem%60).toString().padStart(2,'0')}`;
        } else document.getElementById('life-timer').innerText = "Full";
    }, 1000);

    function showScreen(id) {
        clearInterval(loop); currentGame = "";
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById(id).classList.add('active');
        document.getElementById('game-modal').style.display = 'none';
        document.getElementById('ctrl-area').style.display = "none";
        document.getElementById('score-container').style.display = "flex";
    }

    function backToMenu() { showScreen('menu-screen'); }
    function setScore(v, gameKey) { 
        document.getElementById('score-display').innerText = v; 
        if(gameKey) document.getElementById('best-display').innerText = "Best: " + (state.records[gameKey] || 0);
    }

    function showShop() {
        showScreen('shop-screen');
        const container = document.getElementById('shop-items-container');
        container.innerHTML = `<div class="shop-item"><div><h3>❤️ Животи</h3><p>Допълни до 10</p></div><button class="buy-btn" onclick="buyLife()">1000 💰</button></div>`;
        const accs = [{id:'diaSnake',name:'💎 Диам. Змия',p:5000},{id:'goldPaddle',name:'👑 Златна Хилка',p:8000},{id:'fireSky',name:'🔥 Огнена Ракета',p:12000},{id:'neonSdk',name:'🟣 Neon Grid',p:10000}];
        accs.forEach(acc => {
            const owned = state.items[acc.id];
            container.innerHTML += `<div class="shop-item"><div><h3>${acc.name}</h3></div><button class="buy-btn" ${owned?'disabled':''} onclick="buyAccessory('${acc.id}', ${acc.p})">${owned?'КУПЕНО':acc.p+' 💰'}</button></div>`;
        });
    }

    function buyLife() { if(state.coins>=1000 && state.lives<10) { state.coins-=1000; state.lives=10; save(); showShop(); } }
    function buyAccessory(id, price) { if(state.coins >= price) { state.coins -= price; state.items[id] = true; save(); showShop(); } }

    function triggerLoss(icon, title, score, gameKey, restartFn) {
        if(gameKey && score > state.records[gameKey]) state.records[gameKey] = score;
        state.coins += score; // 1:1 Payout
        state.lives--; if(state.lives === 9) state.lastLifeTime = Date.now();
        save(); endGame(icon, title, `Спечели ${score} монети!`, restartFn);
    }

    function endGame(icon, title, desc, restartFn, btnText = "ИГРАЙ ПАК") {
        clearInterval(loop);
        document.getElementById('m-icon').innerText = icon;
        document.getElementById('m-title').innerText = title;
        document.getElementById('m-desc').innerText = desc;
        document.getElementById('m-restart').innerText = btnText;
        document.getElementById('game-modal').style.display = 'flex';
        document.getElementById('m-restart').onclick = restartFn;
    }

    // --- 2048 ---
    function init2048() {
        showScreen('game-screen'); currentGame = "2048"; setScore(0, "t2048");
        document.getElementById('ctrl-area').style.display = "flex";
        const container = document.getElementById('board-container');
        container.innerHTML = `<div id="grid-2048" class="grid" style="grid-template-columns:repeat(4,1fr)"></div>`;
        b2048 = Array(16).fill(0); let score = 0;
        const add = () => { let e = b2048.map((v,i)=>v===0?i:null).filter(v=>v!==null); if(e.length) b2048[e[Math.floor(Math.random()*e.length)]] = 2; };
        const draw = () => { const g = document.getElementById('grid-2048'); g.innerHTML = ''; b2048.forEach(v => { let c = document.createElement('div'); c.className = `cell t-${v}`; c.innerText = v || ''; g.appendChild(c); }); };
        window.m2048 = (d) => {
            let moved = false;
            for(let i=0; i<4; i++) {
                let row = [];
                for(let j=0; j<4; j++) { if(d==='L' || d==='R') row.push(b2048[i*4+j]); else row.push(b2048[j*4+i]); }
                if(d==='R' || d==='D') row.reverse();
                let filtered = row.filter(v => v !== 0);
                for(let j=0; j<filtered.length-1; j++) { 
                    if(filtered[j] === filtered[j+1]) { 
                        filtered[j]*=2; 
                        score += filtered[j]; // Normal score calc
                        filtered.splice(j+1, 1); moved = true; 
                    } 
                }
                while(filtered.length < 4) filtered.push(0);
                if(d==='R' || d==='D') filtered.reverse();
                if(JSON.stringify(row) !== JSON.stringify(filtered)) moved = true;
                for(let j=0; j<4; j++) { if(d==='L' || d==='R') b2048[i*4+j] = filtered[j]; else b2048[j*4+i] = filtered[j]; }
            }
            if(moved) { add(); setScore(score, "t2048"); draw(); if(!canMove2048()) triggerLoss('🔢', 'КРАЙ!', score, "t2048", init2048); }
        };
        add(); add(); draw();
    }
    function canMove2048() {
        if(b2048.includes(0)) return true;
        for(let i=0; i<4; i++) for(let j=0; j<4; j++) {
            let v = b2048[i*4+j];
            if(j<3 && v === b2048[i*4+j+1]) return true;
            if(i<3 && v === b2048[(i+1)*4+j]) return true;
        }
        return false;
    }

    // --- SKY DASH ---
    function initSkyDash() {
        if(state.lives <= 0) return;
        showScreen('game-screen'); currentGame = "sky"; setScore(0, "sky");
        const container = document.getElementById('board-container');
        container.innerHTML = '<canvas id="sky-canvas" width="300" height="400"></canvas>';
        const ctx = document.getElementById('sky-canvas').getContext('2d');
        let ship = {y:200, v:0}, pipes = [], score = 0, frame = 0;
        container.onclick = () => ship.v = -6.5;
        loop = setInterval(() => {
            ctx.fillStyle = '#050816'; ctx.fillRect(0,0,300,400); frame++;
            ship.v += 0.38; ship.y += ship.v;
            if(frame % 90 === 0) pipes.push({x:300, h:Math.random()*180+60, passed: false});
            pipes.forEach(p => {
                p.x -= 2.6; ctx.fillStyle = '#1e293b'; ctx.fillRect(p.x, 0, 45, p.h); ctx.fillRect(p.x, p.h+110, 45, 400);
                if(!p.passed && p.x < 100) { p.passed = true; score += 2; setScore(score, "sky"); }
                if(100 > p.x && 100 < p.x+45 && (ship.y < p.h || ship.y > p.h+110)) triggerLoss('🚀', 'BOOM!', score, "sky", initSkyDash);
            });
            ctx.fillStyle = state.items.fireSky ? '#f97316' : '#ec4899'; ctx.fillRect(100, ship.y, 22, 22);
            if(ship.y > 390 || ship.y < 0) triggerLoss('🚀', 'CRASH!', score, "sky", initSkyDash);
        }, 1000/60);
    }

    // --- STAR RAID ---
    function initStarRaid() {
        if(state.lives <= 0) return;
        showScreen('game-screen'); currentGame = "star"; setScore(0, "star");
        const container = document.getElementById('board-container');
        container.innerHTML = '<canvas id="star-canvas" width="300" height="400"></canvas>';
        const ctx = document.getElementById('star-canvas').getContext('2d');
        let enemies = [], bullets = [], score = 0, frame = 0; raidPlayerX = 140;
        container.onmousemove = (e) => { const r = container.getBoundingClientRect(); raidPlayerX = e.clientX - r.left - 15; };
        container.onclick = () => bullets.push({x: raidPlayerX+12, y: 370});
        loop = setInterval(() => {
            ctx.fillStyle = '#010413'; ctx.fillRect(0,0,300,400); frame++;
            if(Math.random() < 0.04) enemies.push({x: Math.random()*270, y: -30, s: 2.2 + (score/50)});
            bullets.forEach((b, bi) => { b.y -= 8; ctx.fillStyle = '#fbbf24'; ctx.fillRect(b.x, b.y, 6, 12); if(b.y < 0) bullets.splice(bi, 1); });
            enemies.forEach((e, ei) => {
                e.y += e.s; ctx.fillStyle = '#ef4444'; ctx.beginPath(); ctx.arc(e.x+15, e.y+15, 12, 0, 7); ctx.fill();
                bullets.forEach((b, bi) => { if(b.x > e.x && b.x < e.x+30 && b.y > e.y && b.y < e.y+30) { enemies.splice(ei, 1); bullets.splice(bi, 1); score++; setScore(score, "star"); } });
                if(e.y > 400) triggerLoss('🌠', 'ПРОПУСНАТИ!', score, "star", initStarRaid);
            });
            ctx.fillStyle = '#6366f1'; ctx.beginPath(); ctx.moveTo(raidPlayerX+15, 365); ctx.lineTo(raidPlayerX, 395); ctx.lineTo(raidPlayerX+30, 395); ctx.fill();
        }, 1000/60);
    }

    // --- SNAKE ---
    function initSnake() {
        showScreen('game-screen'); currentGame = "snake"; setScore(0, "snake");
        document.getElementById('ctrl-area').style.display = "flex";
        const container = document.getElementById('board-container');
        container.innerHTML = '<canvas id="snk-canvas" width="300" height="300"></canvas>';
        const ctx = document.getElementById('snk-canvas').getContext('2d');
        let snake = [{x:10, y:10}, {x:9, y:10}], food = {x:15, y:15}, score = 0; snkDir = {x:1, y:0};
        loop = setInterval(() => {
            let h = {x: snake[0].x + snkDir.x, y: snake[0].y + snkDir.y};
            if(h.x<0||h.x>=20||h.y<0||h.y>=20||snake.some(s=>s.x===h.x&&s.y===h.y)) { triggerLoss('🐍', 'КРАЙ!', score, "snake", initSnake); return; }
            snake.unshift(h);
            if(h.x===food.x && h.y===food.y) { score++; setScore(score, "snake"); food={x:Math.floor(Math.random()*20), y:Math.floor(Math.random()*20)}; } else snake.pop();
            ctx.fillStyle = '#020617'; ctx.fillRect(0,0,300,300);
            ctx.fillStyle = '#ef4444'; ctx.beginPath(); ctx.arc(food.x*15+7.5, food.y*15+7.5, 6, 0, 7); ctx.fill();
            snake.forEach((s, i) => { ctx.fillStyle = state.items.diaSnake ? '#22d3ee' : (i===0?'#6366f1':'#312e81'); ctx.beginPath(); ctx.roundRect(s.x*15+1, s.y*15+1, 13, 13, 5); ctx.fill(); });
        }, 115);
    }

    // --- PONG ---
    function initPong() {
        showScreen('game-screen'); currentGame = "pong"; setScore(0, "pong");
        const container = document.getElementById('board-container');
        container.innerHTML = '<canvas id="pong-canvas" width="300" height="400"></canvas>';
        const ctx = document.getElementById('pong-canvas').getContext('2d');
        let ball = {x:150, y:200, dx:3.5, dy:3.5}, p1 = 120, cpu = 120, score = 0;
        container.onmousemove = (e) => { const r = container.getBoundingClientRect(); p1 = e.clientX - r.left - 35; };
        loop = setInterval(() => {
            ctx.fillStyle = '#000'; ctx.fillRect(0,0,300,400);
            ctx.fillStyle = '#fff'; ctx.beginPath(); ctx.arc(ball.x, ball.y, 8, 0, 7); ctx.fill();
            ctx.fillStyle = state.items.goldPaddle ? '#fbbf24' : '#6366f1'; ctx.fillRect(p1, 385, 70, 10);
            ctx.fillStyle = '#ef4444'; ctx.fillRect(cpu, 5, 70, 10);
            ball.x += ball.dx; ball.y += ball.dy;
            if(ball.x<8 || ball.x>292) ball.dx *= -1;
            if(cpu+35 < ball.x) cpu += 3.3; else cpu -= 3.3;
            if(ball.y > 375 && ball.x > p1 && ball.x < p1+70) { ball.dy *= -1.05; score++; setScore(score, "pong"); }
            if(ball.y < 25 && ball.x > cpu && ball.x < cpu+70) ball.dy *= -1.05;
            if(ball.y > 400) triggerLoss('🏓', 'ЗАГУБА!', score, "pong", initPong);
            if(ball.y < 0) { ball = {x:150, y:200, dx:3.5, dy:3.5}; score += 10; setScore(score, "pong"); }
        }, 1000/60);
    }

    // --- SUDOKU ---
    function showSudokuLevels() {
        showScreen('sdk-levels-screen');
        const container = document.getElementById('lvl-container'); container.innerHTML = '';
        for(let i=1; i<=100; i++) {
            let b = document.createElement('button');
            b.className = `lvl-btn ${i > state.sdkLvl ? 'locked' : ''} ${i === state.sdkLvl ? 'current' : ''}`;
            b.innerHTML = `<span>${i}</span>`;
            b.onclick = () => { if(i <= state.sdkLvl) initSudoku(i); };
            container.appendChild(b);
        }
    }
    let sdkSelected = null, sdkSol = [];
    function initSudoku(lvl) {
        showScreen('game-screen'); currentGame = "sudoku";
        document.getElementById('score-container').style.display = "none";
        let size = lvl > 30 ? 9 : (lvl > 15 ? 6 : 4);
        const container = document.getElementById('board-container');
        container.innerHTML = `<div id="sdk-grid" class="grid" style="grid-template-columns: repeat(${size}, 1fr)"></div>`;
        let root = Math.floor(Math.sqrt(size));
        sdkSol = []; for(let i=0; i<size; i++) for(let j=0; j<size; j++) sdkSol.push(((i * root + Math.floor(i/root) + j) % size) + 1);
        let board = [...sdkSol], hide = Math.floor((size*size) * 0.5);
        let idx = Array.from({length:size*size}, (_,i)=>i).sort(()=>Math.random()-0.5);
        for(let i=0; i<hide; i++) board[idx[i]] = 0;
        sdkSelected = null;
        const draw = () => {
            const g = document.getElementById('sdk-grid'); g.innerHTML = '';
            board.forEach((v, i) => {
                let row = Math.floor(i/size), col = i % size;
                let c = document.createElement('div'); c.className = `cell ${v? 'fixed':''} ${sdkSelected===i?'selected':''}`;
                if(size===4){ if(col===1)c.classList.add('br'); if(row===1)c.classList.add('bb'); }
                if(size===6){ if(col===2)c.classList.add('br'); if(row===1||row===3)c.classList.add('bb'); }
                if(size===9){ if(col%3===2&&col<8)c.classList.add('br'); if(row%3===2&&row<8)c.classList.add('bb'); }
                c.innerText = v || ''; c.onclick = () => { if(board[i]===0) { sdkSelected = i; draw(); } }; g.appendChild(c);
            });
        };
        const ctrarea = document.getElementById('ctrl-area'); ctrarea.style.display = "flex";
        ctrarea.innerHTML = `<div id="sdk-ctrls" style="display:flex; gap:5px; flex-wrap:wrap; justify-content:center;"></div>`;
        for(let i=1; i<=size; i++) {
            let b = document.createElement('button'); b.className = "arrow-btn"; b.style.width="40px"; b.style.height="40px"; b.innerText = i;
            b.onclick = () => {
                if(sdkSelected !== null) {
                    if(sdkSol[sdkSelected] === i) { board[sdkSelected] = i; draw(); if(!board.includes(0)) { state.coins += 100; if(lvl === state.sdkLvl) state.sdkLvl++; save(); endGame('🏆', 'ПОБЕДА!', '+100 Монети', () => initSudoku(lvl+1), "СЛЕДВАЩО НИВО"); } }
                    else triggerLoss('🧩', 'ГРЕШКА!', 0, null, () => initSudoku(lvl));
                }
            };
            document.getElementById('sdk-ctrls').appendChild(b);
        }
        draw();
    }

    // --- CONTROLS ---
    window.addEventListener('keydown', (e) => {
        if (currentGame === "snake" || currentGame === "2048") {
            if (e.key === "ArrowUp" || e.key === "w") { if(currentGame === "snake" && snkDir.y === 0) snkDir = {x:0, y:-1}; if(currentGame === "2048") window.m2048('U'); }
            if (e.key === "ArrowDown" || e.key === "s") { if(currentGame === "snake" && snkDir.y === 0) snkDir = {x:0, y:1}; if(currentGame === "2048") window.m2048('D'); }
            if (e.key === "ArrowLeft" || e.key === "a") { if(currentGame === "snake" && snkDir.x === 0) snkDir = {x:-1, y:0}; if(currentGame === "2048") window.m2048('L'); }
            if (e.key === "ArrowRight" || e.key === "d") { if(currentGame === "snake" && snkDir.x === 0) snkDir = {x:1, y:0}; if(currentGame === "2048") window.m2048('R'); }
        }
        if ((currentGame === "sky" || currentGame === "star") && e.key === " ") document.getElementById('board-container').click();
        if (currentGame === "star") { if(e.key === "ArrowLeft" || e.key === "a") raidPlayerX = Math.max(0, raidPlayerX - 20); if(e.key === "ArrowRight" || e.key === "d") raidPlayerX = Math.min(270, raidPlayerX + 20); }
        if (currentGame === "sudoku" && e.key >= "1" && e.key <= "9") document.querySelectorAll('#sdk-ctrls button').forEach(b => { if(b.innerText === e.key) b.click(); });
        if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight"," "].includes(e.key)) e.preventDefault();
    });

    document.getElementById('btn-up').onclick = () => window.dispatchEvent(new KeyboardEvent('keydown', {key: 'ArrowUp'}));
    document.getElementById('btn-down').onclick = () => window.dispatchEvent(new KeyboardEvent('keydown', {key: 'ArrowDown'}));
    document.getElementById('btn-left').onclick = () => window.dispatchEvent(new KeyboardEvent('keydown', {key: 'ArrowLeft'}));
    document.getElementById('btn-right').onclick = () => window.dispatchEvent(new KeyboardEvent('keydown', {key: 'ArrowRight'}));

    save();
</script>
</body>
</html>

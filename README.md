<html lang="bg">
<head>
    <!-- Google tag (gtag.js) -->
    <script async src="https://www.googletagmanager.com/gtag/js?id=G-P3BYSS346P"></script>
    <script>
      window.dataLayer = window.dataLayer || [];
      function gtag(){dataLayer.push(arguments);}
      gtag('js', new Date());

      gtag('config', 'G-P3BYSS346P');
    </script>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Arcade World Pro - Logic Sudoku</title>
    <link href="https://fonts.googleapis.com/css2?family=Nunito:wght@400;700;900&display=swap" rel="stylesheet">
    <style>
        :root {
            --primary: #6366f1;
            --bg-gradient: linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab);
            --app-bg: rgba(255, 255, 255, 0.95);
            --text-main: #1e293b;
            --card-bg: #ffffff;
            --header-bg: #ffffff;
            --border: rgba(0,0,0,0.1);
            --correct: #22c55e;
            --error: #f43f5e;
            --coin: #fbbf24;
            --locked: #94a3b8;
        }

        [data-theme="pink"] {
            --primary: #ec4899;
            --bg-gradient: linear-gradient(-45deg, #fbcfe8, #f472b6, #ffffff, #fdf2f8);
            --app-bg: rgba(255, 255, 255, 0.96);
            --text-main: #831843;
            --card-bg: #fff1f2;
            --header-bg: #fff1f2;
        }

        [data-theme="neon"] {
            --primary: #818cf8;
            --bg-gradient: radial-gradient(circle at center, #1e1b4b 0%, #020617 100%);
            --app-bg: rgba(15, 23, 42, 0.9);
            --text-main: #ffffff;
            --card-bg: #1e293b;
            --header-bg: rgba(255, 255, 255, 0.05);
        }

        body {
            font-family: 'Nunito', sans-serif;
            margin: 0; display: flex; justify-content: center; align-items: center;
            min-height: 100vh; background: var(--bg-gradient); background-size: 400% 400%;
            animation: gradientBG 15s ease infinite; color: var(--text-main);
            overflow-x: hidden; overflow-y: auto; user-select: none;
        }

        @keyframes gradientBG { 0% { background-position: 0% 50%; } 50% { background-position: 100% 50%; } 100% { background-position: 0% 50%; } }

        #app {
            width: 95%; max-width: 800px; /* Увеличена ширина за компютър */
            background: var(--app-bg); backdrop-filter: blur(15px);
            min-height: 85vh; border-radius: 40px; box-shadow: 0 25px 50px rgba(0, 0, 0, 0.2);
            padding: 20px; display: flex; flex-direction: column; position: relative; transition: all 0.5s ease;
            margin: 20px 0;
        }

        .damage-shake { animation: damageAnim 0.4s ease-in-out; background: rgba(254, 226, 226, 1) !important; }
        @keyframes damageAnim { 0%, 100% { transform: translateX(0); } 20%, 60% { transform: translateX(-10px); } 40%, 80% { transform: translateX(10px); } }

        .header { display: flex; justify-content: space-between; align-items: center; background: var(--header-bg); padding: 15px 25px; border-radius: 20px; margin-bottom: 20px; box-shadow: 0 2px 10px rgba(0,0,0,0.05); }

        .screen { display: none; flex-direction: column; align-items: center; width: 100%; animation: fadeIn 0.3s ease; }
        .active { display: flex; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        .menu-btn { 
            width: 100%; max-width: 500px; padding: 25px; margin: 10px 0; border-radius: 25px; border: none; 
            cursor: pointer; background: var(--card-bg); color: var(--text-main); 
            font-family: inherit; font-weight: 900; font-size: 1.2rem; box-shadow: 0 4px 10px rgba(0,0,0,0.05); 
            transition: all 0.2s;
        }
        .menu-btn:hover { background-color: #cbd5e1 !important; transform: translateY(-2px); }

        /* SUDOKU BOARD RESPONSIVE */
        .board { display: grid; gap: 1px; background: #94a3b8; border: 4px solid #475569; border-radius: 8px; margin: 15px 0; overflow: hidden; width: 100%; }
        .grid-4 { grid-template-columns: repeat(4, 1fr); max-width: 350px; }
        .grid-6 { grid-template-columns: repeat(6, 1fr); max-width: 450px; }
        .grid-9 { grid-template-columns: repeat(9, 1fr); max-width: 600px; }
        
        .cell { background: white; aspect-ratio: 1; display: flex; justify-content: center; align-items: center; font-size: clamp(1rem, 4vw, 1.5rem); font-weight: 800; cursor: pointer; color: #1e293b; }
        .cell.fixed { color: #64748b; background: #f8fafc; }
        .cell.selected { background: #e0e7ff; box-shadow: inset 0 0 0 3px var(--primary); }

        /* Block Borders */
        .grid-4 .cell:nth-child(2n) { border-right: 2px solid #475569; }
        .grid-4 .cell:nth-child(n+5):nth-child(-n+8), .grid-4 .cell:nth-child(n+13) { border-top: 2px solid #475569; }
        .grid-6 .cell:nth-child(3n) { border-right: 2px solid #475569; }
        .grid-6 .cell:nth-child(n+7):nth-child(-n+12), .grid-6 .cell:nth-child(n+19):nth-child(-n+24), .grid-6 .cell:nth-child(n+31) { border-top: 2px solid #475569; }
        .grid-9 .cell:nth-child(3n) { border-right: 2px solid #475569; }
        .grid-9 .cell:nth-child(n+28):nth-child(-n+36), .grid-9 .cell:nth-child(n+55):nth-child(-n+63) { border-top: 3px solid #475569; }

        /* CANVAS SCALING */
        canvas { 
            background: #f8fafc; border: 5px solid #e2e8f0; border-radius: 20px; 
            max-width: 100%; height: auto; /* Позволява на платното да се свива за телефон */
        }
        #snakeCanvas { width: 400px; max-width: 100%; }
        #tetrisCanvas { width: 250px; max-width: 100%; background: #1e293b; border-color: #334155; }

        .btn { padding: 15px 30px; border-radius: 15px; border: none; font-weight: 800; cursor: pointer; font-family: inherit; transition: 0.2s; font-size: 1rem; }
        .btn-primary { background: var(--primary); color: white; }

        .result-screen { position: absolute; inset: 0; background: var(--app-bg); z-index: 500; display: none; flex-direction: column; align-items: center; justify-content: center; border-radius: 40px; text-align: center; padding: 20px; }
        
        .theme-selector { display: flex; gap: 15px; margin-top: 15px; }
        .theme-dot { width: 35px; height: 35px; border-radius: 50%; cursor: pointer; border: 3px solid white; box-shadow: 0 2px 5px rgba(0,0,0,0.2); }

        /* Mobile Adjustments */
        @media (max-width: 480px) {
            #app { padding: 15px; border-radius: 0; min-height: 100vh; margin: 0; width: 100%; }
            .menu-btn { padding: 20px; font-size: 1rem; }
            .header { padding: 10px 15px; }
        }
    </style>
</head>
<body>

<div id="app">
    <div class="header">
        <div class="life-container">
            <div id="header-lives" style="color:var(--error); font-weight:900; font-size: 1.2rem;">❤️❤️❤️❤️❤️</div>
            <div id="life-timer" style="font-size: 10px; font-weight: bold;">Arcade Ready</div>
        </div>
        <div style="color:var(--coin); font-weight:900; font-size: 1.2rem;">💰 <span id="coins-display">300</span></div>
    </div>

    <!-- MAIN MENU -->
    <div id="menu-screen" class="screen active">
        <h1 style="margin: 0 0 20px 0; font-size: 2.5rem;">Arcade World</h1>
        <button class="menu-btn" onclick="showScreen('sudoku-map')"><b>🧩 Sudoku</b><br><span id="stat-sudoku" style="font-size:0.8rem">Level 0/56</span></button>
        <button class="menu-btn" onclick="startSnakeGame()"><b>🐍 Snake</b><br><span id="stat-snake" style="font-size:0.8rem">Best Score: 0</span></button>
        <button class="menu-btn" onclick="startTetris()"><b>🧱 Tetris</b><br><span id="stat-tetris" style="font-size:0.8rem">Best Score: 0</span></button>
        
        <p style="margin-top:20px; font-weight:bold; font-size:1rem">THEMES:</p>
        <div class="theme-selector">
            <div class="theme-dot" style="background:#6366f1" onclick="setTheme('default')"></div>
            <div class="theme-dot" style="background:#ec4899" onclick="setTheme('pink')"></div>
            <div class="theme-dot" style="background:#1e1b4b" onclick="setTheme('neon')"></div>
        </div>
    </div>

    <!-- SUDOKU MAP -->
    <div id="sudoku-map" class="screen">
        <h2>Sudoku Progress</h2>
        <div id="level-list" style="display:grid; grid-template-columns: repeat(4, 1fr); gap:12px; width:100%; max-width: 600px; overflow-y:auto; max-height:50vh; padding:10px;"></div>
        <button class="btn btn-primary" style="margin-top:20px; width:100%; max-width:400px;" onclick="showScreen('menu-screen')">Back to Menu</button>
    </div>

    <!-- SUDOKU GAME -->
    <div id="sudoku-game" class="screen">
        <h3 id="sudoku-title">Level</h3>
        <div id="sudoku-board" class="board"></div>
        <div id="sudoku-keypad" style="display:grid; grid-template-columns: repeat(5, 1fr); gap:10px; width:100%; max-width: 500px;"></div>
        <button class="btn btn-primary" style="margin-top:20px; width:100%; max-width:400px;" onclick="buyHint()">💡 Hint (-50 💰)</button>
        <button class="btn" style="margin-top:10px; width:100%; max-width:400px; background:#f1f5f9; color:#334155" onclick="showScreen('sudoku-map')">Quit Game</button>
    </div>

    <!-- SNAKE GAME -->
    <div id="snake-screen" class="screen">
        <div style="width:100%; max-width:400px; display:flex; justify-content:space-between; margin-bottom:10px; font-weight:900">
            <span>Score: <span id="snake-score">0</span></span>
        </div>
        <canvas id="snakeCanvas" width="300" height="300"></canvas>
        <div style="display:grid; grid-template-columns: repeat(3, 1fr); gap:15px; margin-top:20px">
            <div></div><button class="btn" onclick="snakeDir('UP')">▲</button><div></div>
            <button class="btn" onclick="snakeDir('LEFT')">◀</button>
            <button class="btn" onclick="snakeDir('DOWN')">▼</button>
            <button class="btn" onclick="snakeDir('RIGHT')">▶</button>
        </div>
    </div>

    <!-- TETRIS GAME -->
    <div id="tetris-screen" class="screen">
        <div style="width:100%; max-width:250px; display:flex; justify-content:space-between; margin-bottom:10px; font-weight:900">
            <span>Score: <span id="tetris-score">0</span></span>
        </div>
        <canvas id="tetrisCanvas" width="200" height="400"></canvas>
        <div style="display:grid; grid-template-columns: repeat(4, 1fr); gap:8px; margin-top:15px; width:100%; max-width:400px;">
            <button class="btn" onclick="tetrisMove(-1)">◀</button>
            <button class="btn" onclick="tetrisRotate()">🔄</button>
            <button class="btn" onclick="tetrisMove(1)">▶</button>
            <button class="btn" onclick="tetrisDrop()">▼</button>
        </div>
    </div>

    <!-- RESULT MODAL -->
    <div id="result-modal" class="result-screen">
        <h1 id="res-icon" style="font-size:5rem; margin:0">🏆</h1>
        <h2 id="res-title">EXCELLENT!</h2>
        <p id="res-desc"></p>
        <div style="display:flex; flex-direction:column; gap:10px; width:100%; max-width:400px; margin-top:20px">
            <button id="btn-again" class="btn btn-primary" onclick="">TRY AGAIN</button>
            <button class="btn" onclick="showScreen('menu-screen')">BACK TO MENU</button>
        </div>
    </div>
</div>

<script>
    // Кодът в <script> остава абсолютно същият като в твоя файл, за да се запази логиката.
    const MAX_LIVES = 5;
    const REFILL_TIME = 120000;
    let activeGameType = null;

    let gameState = {
        coins: 300,
        unlockedCount: 1,
        completedLevels: [],
        snakeHighScore: 0,
        tetrisHighScore: 0,
        theme: 'default',
        lives: { sudoku: 5, snake: 5, tetris: 5 },
        lastRefill: { sudoku: Date.now(), snake: Date.now(), tetris: Date.now() }
    };

    function save() { 
        localStorage.setItem('ArcadeLogic_V6', JSON.stringify(gameState)); 
        updateMenuStats();
    }

    function load() {
        const data = localStorage.getItem('ArcadeLogic_V6');
        if (data) gameState = JSON.parse(data);
        setTheme(gameState.theme);
        updateRefills();
    }

    function updateRefills() {
        const now = Date.now();
        ['sudoku', 'snake', 'tetris'].forEach(type => {
            if (gameState.lives[type] < MAX_LIVES) {
                const diff = now - gameState.lastRefill[type];
                const refillCount = Math.floor(diff / REFILL_TIME);
                if (refillCount > 0) {
                    gameState.lives[type] = Math.min(MAX_LIVES, gameState.lives[type] + refillCount);
                    gameState.lastRefill[type] = now - (diff % REFILL_TIME);
                }
            } else { gameState.lastRefill[type] = now; }
        });
        renderLifeUI();
        save();
    }
    setInterval(updateRefills, 1000);

    function renderLifeUI() {
        const livesDisplay = document.getElementById('header-lives');
        const timerDisplay = document.getElementById('life-timer');
        if (!activeGameType) {
            livesDisplay.innerText = "❤️❤️❤️❤️❤️"; timerDisplay.innerText = "SELECT GAME";
        } else {
            const currentLives = gameState.lives[activeGameType];
            livesDisplay.innerText = "❤️".repeat(currentLives) + "🖤".repeat(MAX_LIVES - currentLives);
            if (currentLives < MAX_LIVES) {
                const rem = REFILL_TIME - (Date.now() - gameState.lastRefill[activeGameType]);
                timerDisplay.innerText = `REFILL: ${Math.floor(rem/60000)}:${Math.floor((rem%60000)/1000).toString().padStart(2, '0')}`;
            } else timerDisplay.innerText = "LIVES FULL";
        }
        document.getElementById('coins-display').innerText = gameState.coins;
    }

    function setTheme(t) { document.body.setAttribute('data-theme', t); gameState.theme = t; save(); }

    function showScreen(id) {
        document.querySelectorAll('.screen, .result-screen').forEach(s => s.style.display = 'none');
        document.getElementById(id).style.display = 'flex';
        if (id.includes('sudoku')) activeGameType = 'sudoku';
        else if (id === 'snake-screen') activeGameType = 'snake';
        else if (id === 'tetris-screen') activeGameType = 'tetris';
        else activeGameType = null;

        if (id === 'sudoku-map') renderSudokuMap();
        clearInterval(snakeInterval); clearInterval(tetrisInterval);
        renderLifeUI();
    }

    function triggerError() {
        const app = document.getElementById('app');
        app.classList.remove('damage-shake'); void app.offsetWidth;
        app.classList.add('damage-shake');
        setTimeout(() => app.classList.remove('damage-shake'), 400);
    }

    function showResult(icon, title, desc, action) {
        document.getElementById('res-icon').innerText = icon;
        document.getElementById('res-title').innerText = title;
        document.getElementById('res-desc').innerText = desc;
        document.getElementById('btn-again').onclick = action;
        showScreen('result-modal');
    }

    function updateMenuStats() {
        document.getElementById('stat-sudoku').innerText = `Level ${gameState.completedLevels.length}/56`;
        document.getElementById('stat-snake').innerText = `Best Score: ${gameState.snakeHighScore}`;
        document.getElementById('stat-tetris').innerText = `Best Score: ${gameState.tetrisHighScore}`;
        document.getElementById('coins-display').innerText = gameState.coins;
    }

    const seeds = {
        4: [1,2,3,4, 3,4,1,2, 2,1,4,3, 4,3,2,1],
        6: [1,2,3,4,5,6, 4,5,6,1,2,3, 2,3,1,5,6,4, 5,6,4,2,3,1, 3,1,2,6,4,5, 6,4,5,3,1,2],
        9: [5,3,4,6,7,8,9,1,2, 6,7,2,1,9,5,3,4,8, 1,9,8,3,4,2,5,6,7, 8,5,9,7,6,1,4,2,3, 4,2,6,8,5,3,7,9,1, 7,1,3,9,2,4,8,5,6, 9,6,1,5,3,7,2,8,4, 2,8,7,4,1,9,6,3,5, 3,4,5,2,8,6,1,7,9]
    };

    function shuffleSudoku(size) {
        let grid = [...seeds[size]];
        let nums = Array.from({length: size}, (_, i) => i + 1);
        for(let i = nums.length - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            [nums[i], nums[j]] = [nums[j], nums[i]];
        }
        return grid.map(n => nums[n-1]);
    }

    const levels = [];
    for (let i = 1; i <= 56; i++) {
        let size = i <= 15 ? 4 : (i <= 35 ? 6 : 9);
        let clues = size === 4 ? 6 : (size === 6 ? 14 : 32);
        let sol = shuffleSudoku(size);
        let grid = sol.map((v, idx) => Math.random() > (clues / (size*size)) ? 0 : v);
        levels.push({ id: i, size, reward: size * 10, solution: sol, grid: grid });
    }

    function renderSudokuMap() {
        const list = document.getElementById('level-list'); list.innerHTML = '';
        levels.forEach(l => {
            const isUnl = l.id <= (gameState.completedLevels.length + 1);
            const isComp = gameState.completedLevels.includes(l.id);
            const card = document.createElement('div');
            card.className = `level-card ${!isUnl ? 'level-locked' : ''} ${isComp ? 'level-completed' : ''}`;
            if (!isUnl) card.innerHTML = `<span>${l.id}</span>🔒`;
            else { card.innerHTML = l.id; card.onclick = () => startSudoku(l.id - 1); }
            list.appendChild(card);
        });
    }

    let curSudoku = null, userGrid = [], selectedIdx = null;
    function startSudoku(idx) {
        if(gameState.lives.sudoku <= 0) return;
        curSudoku = levels[idx]; userGrid = [...curSudoku.grid]; selectedIdx = null;
        showScreen('sudoku-game'); updateSudokuUI();
    }

    function updateSudokuUI() {
        document.getElementById('sudoku-title').innerText = `SUDOKU LVL ${curSudoku.id}`;
        const b = document.getElementById('sudoku-board');
        b.className = `board grid-${curSudoku.size}`; b.innerHTML = '';
        userGrid.forEach((v, i) => {
            const c = document.createElement('div');
            c.className = `cell ${curSudoku.grid[i] !== 0 ? 'fixed' : ''} ${selectedIdx === i ? 'selected' : ''}`;
            c.innerText = v || '';
            if(curSudoku.grid[i] === 0) c.onclick = () => { selectedIdx = i; updateSudokuUI(); };
            b.appendChild(c);
        });
        renderKeypad();
    }

    function renderKeypad() {
        const k = document.getElementById('sudoku-keypad'); k.innerHTML = '';
        for(let i=1; i<=curSudoku.size; i++) {
            const btn = document.createElement('button'); btn.className = 'btn'; btn.style.background='#1e293b'; btn.style.color='#fff'; btn.innerText = i;
            btn.onclick = () => sudokuInput(i);
            k.appendChild(btn);
        }
    }

    function sudokuInput(num) {
        if(selectedIdx === null) return;
        if(num === curSudoku.solution[selectedIdx]) {
            userGrid[selectedIdx] = num;
            if(userGrid.every((v,ix)=>v===curSudoku.solution[ix])) winSudoku();
            updateSudokuUI();
        } else {
            gameState.lives.sudoku--; if(gameState.lives.sudoku === MAX_LIVES-1) gameState.lastRefill.sudoku = Date.now();
            save(); triggerError(); renderLifeUI();
            if(gameState.lives.sudoku <= 0) showResult("💔", "GAME OVER", "No more lives!", () => showScreen('menu-screen'));
        }
    }

    function winSudoku() {
        if(!gameState.completedLevels.includes(curSudoku.id)) {
            gameState.completedLevels.push(curSudoku.id); gameState.coins += curSudoku.reward;
        }
        save(); showResult("✔️", "CLEARED!", `Level complete! +${curSudoku.reward} 💰`, () => showScreen('sudoku-map'));
    }

    let snake, food, snakeDirVal, snakeInterval, snakeScore;
    const sCtx = document.getElementById('snakeCanvas').getContext('2d');
    function startSnakeGame() {
        if(gameState.lives.snake <= 0) return;
        showScreen('snake-screen');
        snake = [{x:10, y:10}, {x:10, y:11}]; food = {x:5, y:5}; snakeDirVal = 'UP'; snakeScore = 0;
        snakeInterval = setInterval(updateSnake, 130);
    }
    function snakeDir(d) {
        if(d==='UP' && snakeDirVal==='DOWN' || d==='DOWN' && snakeDirVal==='UP' || d==='LEFT' && snakeDirVal==='RIGHT' || d==='RIGHT' && snakeDirVal==='LEFT') return;
        snakeDirVal = d;
    }
    function updateSnake() {
        let head = {...snake[0]};
        if(snakeDirVal==='UP') head.y--; if(snakeDirVal==='DOWN') head.y++; if(snakeDirVal==='LEFT') head.x--; if(snakeDirVal==='RIGHT') head.x++;
        if(head.x<0 || head.x>=20 || head.y<0 || head.y>=20 || snake.some(s=>s.x===head.x && s.y===head.y)) { endSnake(); return; }
        snake.unshift(head);
        if(head.x===food.x && head.y===food.y) {
            snakeScore += 10; food = {x:Math.floor(Math.random()*20), y:Math.floor(Math.random()*20)};
        } else snake.pop();
        sCtx.clearRect(0,0,300,300);
        sCtx.fillStyle='red'; sCtx.beginPath(); sCtx.arc(food.x*15+7.5, food.y*15+7.5, 6, 0, Math.PI*2); sCtx.fill();
        snake.forEach((s,i)=>{
            sCtx.fillStyle = i===0 ? '#059669':'#10b981';
            sCtx.beginPath(); sCtx.arc(s.x*15+7.5, s.y*15+7.5, i===0 ? 7.5:6, 0, Math.PI*2); sCtx.fill();
        });
        document.getElementById('snake-score').innerText = snakeScore;
    }
    function endSnake() {
        clearInterval(snakeInterval); gameState.lives.snake--; save();
        showResult("🐍", "SNAKE OVER", `Score: ${snakeScore}`, () => startSnakeGame());
    }

    let tCtx = document.getElementById('tetrisCanvas').getContext('2d');
    let tetrisArena, tetrisPlayer, tetrisInterval, tetrisScoreVal;
    const tColors = [null, '#06b6d4', '#3b82f6', '#f97316', '#eab308', '#22c55e', '#a855f7', '#ef4444'];
    function startTetris() {
        if(gameState.lives.tetris <= 0) return;
        showScreen('tetris-screen');
        tetrisArena = Array.from({length:20}, ()=>Array(10).fill(0));
        tetrisPlayer = {pos:{x:3,y:0}, matrix:[[0,1,0],[1,1,1],[0,0,0]]};
        tetrisScoreVal = 0; tetrisInterval = setInterval(tetrisUpdate, 800); drawTetris();
    }
    function tetrisUpdate() {
        tetrisPlayer.pos.y++; if(collide()) { tetrisPlayer.pos.y--; merge(); resetPlayer(); arenaSweep(); }
        drawTetris();
    }
    function collide() {
        const [m, o] = [tetrisPlayer.matrix, tetrisPlayer.pos];
        for(let y=0; y<m.length; y++) for(let x=0; x<m[y].length; x++) 
            if(m[y][x]!==0 && (tetrisArena[y+o.y] && tetrisArena[y+o.y][x+o.x])!==0) return true;
        return false;
    }
    function merge() {
        tetrisPlayer.matrix.forEach((row,y)=>row.forEach((v,x)=>{ if(v!==0) tetrisArena[y+tetrisPlayer.pos.y][x+tetrisPlayer.pos.x]=v; }));
    }
    function resetPlayer() {
        const p = [[[0,1,0,0],[0,1,0,0],[0,1,0,0],[0,1,0,0]],[[2,2],[2,2]],[[0,3,0],[3,3,3],[0,0,0]]];
        tetrisPlayer.matrix = p[Math.floor(Math.random()*p.length)]; tetrisPlayer.pos.y=0; tetrisPlayer.pos.x=3;
        if(collide()) endTetris();
    }
    function arenaSweep() {
        outer: for(let y=tetrisArena.length-1; y>0; y--) {
            for(let x=0; x<tetrisArena[y].length; x++) if(tetrisArena[y][x]===0) continue outer;
            tetrisArena.splice(y,1); tetrisArena.unshift(new Array(10).fill(0)); tetrisScoreVal+=100; y++;
        }
        document.getElementById('tetris-score').innerText = tetrisScoreVal;
    }
    function drawTetris() {
        tCtx.fillStyle='#1e293b'; tCtx.fillRect(0,0,200,400);
        tetrisArena.forEach((row,y)=>row.forEach((v,x)=>{ if(v) { tCtx.fillStyle=tColors[v]; tCtx.fillRect(x*20,y*20,19,19); }}));
        tetrisPlayer.matrix.forEach((row,y)=>row.forEach((v,x)=>{ if(v) { tCtx.fillStyle=tColors[v]; tCtx.fillRect((x+tetrisPlayer.pos.x)*20,(y+tetrisPlayer.pos.y)*20,19,19); }}));
    }
    function tetrisMove(d) { tetrisPlayer.pos.x+=d; if(collide()) tetrisPlayer.pos.x-=d; drawTetris(); }
    function tetrisRotate() {
        const m = tetrisPlayer.matrix; for(let y=0; y<m.length; y++) for(let x=0; x<y; x++) [m[x][y], m[y][x]]=[m[y][x], m[x][y]];
        m.forEach(r=>r.reverse()); if(collide()) m.forEach(r=>r.reverse()); drawTetris();
    }
    function tetrisDrop() { tetrisUpdate(); }
    function endTetris() {
        clearInterval(tetrisInterval); gameState.lives.tetris--; save();
        showResult("🧱", "GAME OVER", `Score: ${tetrisScoreVal}`, () => startTetris());
    }

    function buyHint() {
        if(gameState.coins < 50) return;
        const empty = userGrid.map((v,i)=>v===0?i:null).filter(v=>v!==null);
        if(empty.length) {
            gameState.coins -= 50; const r = empty[Math.floor(Math.random()*empty.length)];
            userGrid[r] = curSudoku.solution[r]; save(); updateSudokuUI();
        }
    }

    window.addEventListener('keydown', (e) => {
        if (activeGameType === 'snake') {
            if (e.key.includes('Arrow')) snakeDir(e.key.replace('Arrow', '').toUpperCase());
        }
        if (activeGameType === 'tetris') {
            if (e.key === 'ArrowLeft') tetrisMove(-1);
            if (e.key === 'ArrowRight') tetrisMove(1);
            if (e.key === 'ArrowUp') tetrisRotate();
            if (e.key === 'ArrowDown') tetrisDrop();
        }
    });

    load();
</script>
</body>
</html>

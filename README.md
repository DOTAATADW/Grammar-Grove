<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>森林语法棋</title>
    <style>
        :root {
            --forest-bg: #E9F4E3;
            --cell-base: #D0E0C5;
            --pause: #FBE699;
            --verb: #B0D7D0;
            --read: #DAB5F0;
            --meaning: #FFAEB4;
            --sentence: #9ABCFF;
            --reward: #BDE5B5;
            --player1: #E74C3C;
            --player2: #3498DB;
        }

        body {
            margin: 0;
            padding: 15px;
            background: var(--forest-bg);
            font-family: 'Segoe UI', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        .game-header {
            width: 100%;
            max-width: 500px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .game-title {
            color: #446633;
            margin: 0;
            font-size: 1.5rem;
        }

        .board {
            display: grid;
            grid-template-columns: repeat(10, 1fr);
            gap: 2px;
            width: 100%;
            max-width: 500px;
            aspect-ratio: 1;
            background: #C0D0B0;
            padding: 5px;
            border-radius: 8px;
            position: relative;
        }

        .cell {
            background: var(--cell-base);
            aspect-ratio: 1;
            border-radius: 3px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 12px;
            position: relative;
        }

        /* 特殊格子标识 */
        .cell::after {
            content: '';
            position: absolute;
            bottom: 2px;
            right: 2px;
            width: 8px;
            height: 8px;
            border-radius: 50%;
        }
        .cell.pause { background: var(--pause); }
        .cell.pause::after { background: #D4A413; }
        .cell.verb { background: var(--verb); }
        .cell.verb::after { background: #5BACA3; }
        .cell.read { background: var(--read); }
        .cell.read::after { background: #B87DDF; }
        .cell.meaning { background: var(--meaning); }
        .cell.meaning::after { background: #FF6B7A; }
        .cell.sentence { background: var(--sentence); }
        .cell.sentence::after { background: #5B8DEF; }
        .cell.reward { background: var(--reward); }
        .cell.reward::after { background: #80B875; }

        /* 玩家棋子 */
        .player {
            width: 16px;
            height: 16px;
            border: 2px solid white;
            border-radius: 4px;
            position: absolute;
            transition: all 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
            transform: translate(-50%, -50%);
            z-index: 2;
        }
        #player1 { background: var(--player1); }
        #player2 { background: var(--player2); }

        /* 骰子系统 */
        .dice-panel {
            width: 100%;
            max-width: 500px;
            margin: 20px 0;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 10px;
        }

        .turn-info {
            font-size: 14px;
            font-weight: bold;
            color: #446633;
            padding: 8px 15px;
            background: rgba(255,255,255,0.9);
            border-radius: 20px;
        }

        .dice-box {
            width: 60px;
            height: 60px;
            background: #A4BA8A;
            border-radius: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 24px;
            color: white;
            cursor: pointer;
            box-shadow: 0 3px 6px rgba(0,0,0,0.1);
        }

        /* 计分系统 */
        .score-panel {
            width: 100%;
            max-width: 500px;
            padding: 15px;
            background: rgba(255,255,255,0.3);
            border-radius: 8px;
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 15px;
        }

        .score-item {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .score-btn {
            width: 32px;
            height: 32px;
            border: none;
            border-radius: 6px;
            background: #A4BA8A;
            color: white;
            font-size: 18px;
            cursor: pointer;
        }

        @keyframes dice-animation {
            0% { transform: rotate(0deg) scale(1); }
            25% { transform: rotate(-15deg) scale(1.2); }
            50% { transform: rotate(15deg) scale(0.8); }
            75% { transform: rotate(-10deg); }
            100% { transform: rotate(0deg) scale(1); }
        }

        @media (max-width: 480px) {
            .cell { font-size: 10px; }
            .dice-box { width: 50px; height: 50px; }
            .score-btn { width: 28px; height: 28px; }
        }
    </style>
</head>
<body>
    <div class="game-header">
        <h1 class="game-title">🌳 森林语法棋</h1>
        <button onclick="resetGame()" style="background:#A4BA8A; border:none; padding:6px 12px; border-radius:4px; color:white">重置</button>
    </div>

    <div class="board" id="gameBoard"></div>

    <div class="dice-panel">
        <div class="turn-info" id="turnDisplay">玩家①的回合</div>
        <div class="dice-box" id="dice" onclick="rollDice()">🎲</div>
    </div>

    <div class="score-panel">
        <div class="score-item">
            <button class="score-btn" onclick="changeScore(1, -1)">-</button>
            <span style="font-weight:bold; min-width:30px" id="score1">0</span>
            <button class="score-btn" onclick="changeScore(1, 1)">+</button>
            <span style="color:#446633">玩家①</span>
        </div>
        <div class="score-item">
            <button class="score-btn" onclick="changeScore(2, -1)">-</button>
            <span style="font-weight:bold; min-width:30px" id="score2">0</span>
            <button class="score-btn" onclick="changeScore(2, 1)">+</button>
            <span style="color:#446633">玩家②</span>
        </div>
    </div>

<script>
let currentPlayer = 1;
let isMoving = false;
const gameState = {
    cells: generateCells(),
    players: {
        1: { pos: 0, score: 0 },
        2: { pos: 0, score: 0 }
    }
};

// 生成特殊格子
function generateCells() {
    const cells = new Array(100).fill(null);
    const cellTypes = [
        { type: 'pause', count: 4 },
        { type: 'verb', count: 5 },
        { type: 'read', count: 5 },
        { type: 'meaning', count: 5 },
        { type: 'sentence', count: 5 },
        { type: 'reward', count: 6 }
    ];

    cellTypes.forEach(({ type, count }) => {
        let placed = 0;
        while (placed < count) {
            const pos = Math.floor(Math.random() * 95) + 5;
            if (!cells[pos]) {
                cells[pos] = type;
                placed++;
            }
        }
    });
    return cells;
}

// 初始化棋盘
function initBoard() {
    const board = document.getElementById('gameBoard');
    board.innerHTML = '';

    // 创建单元格
    gameState.cells.forEach((type, index) => {
        const cell = document.createElement('div');
        cell.className = `cell${type ? ` ${type}` : ''}`;
        cell.textContent = index + 1;
        board.appendChild(cell);
    });

    // 创建玩家棋子
    [1, 2].forEach(id => {
        const marker = document.createElement('div');
        marker.id = `player${id}`;
        marker.className = 'player';
        board.appendChild(marker);
    });

    updateMarkers();
}

// 更新棋子位置
function updateMarkers() {
    const cells = document.querySelectorAll('.cell');
    [1, 2].forEach(id => {
        const player = document.getElementById(`player${id}`);
        const cell = cells[gameState.players[id].pos];
        if (cell) {
            const rect = cell.getBoundingClientRect();
            const boardRect = document.getElementById('gameBoard').getBoundingClientRect();
            const x = rect.left - boardRect.left + rect.width / (id === 1 ? 3 : 1.8);
            const y = rect.top - boardRect.top + rect.height / (id === 1 ? 3 : 1.8);
            player.style.left = `${x}px`;
            player.style.top = `${y}px`;
        }
    });
}

// 掷骰子逻辑
async function rollDice() {
    if (isMoving) return;
    isMoving = true;

    const dice = document.getElementById('dice');
    // 骰子动画
    dice.style.animation = 'dice-animation 1s ease';
    const steps = await diceRollEffect(dice);

    // 逐格移动
    for (let i = 0; i < steps; i++) {
        gameState.players[currentPlayer].pos++;
        if (gameState.players[currentPlayer].pos >= 99) break;
        updateMarkers();
        await new Promise(r => setTimeout(r, 600));
    }

    handleCellEffect();
    switchTurn();
    dice.style.animation = '';
    isMoving = false;
}

// 骰子动画效果
async function diceRollEffect(dice) {
    return new Promise(resolve => {
        let count = 0;
        const animate = () => {
            if (count++ < 10) {
                dice.textContent = (count % 6) + 1;
                setTimeout(animate, 120);
            } else {
                const result = Math.ceil(Math.random() * 6);
                dice.textContent = result;
                resolve(result);
            }
        }
        animate();
    });
}

// 处理格子效果
function handleCellEffect() {
    const pos = gameState.players[currentPlayer].pos;
    const cellType = gameState.cells[pos];
    if (!cellType) return;

    switch(cellType) {
        case 'pause':
            setTimeout(() => alert('暂停一回合！'), 300);
            switchTurn();
            break;
        case 'reward':
            changeScore(currentPlayer, 3);
            break;
        default:
            showChallenge(cellType);
    }
}

// 显示挑战模态
function showChallenge(type) {
    const challengeTypes = {
        verb: '动词填空',
        read: '朗读单词',
        meaning: '词义解释',
        sentence: '重组句子'
    };
    const result = confirm(`${challengeTypes[type]}挑战！正确选✅ 错误选❌`);
    if (result) {
        changeScore(currentPlayer, 2);
    } else {
        changeScore(currentPlayer, -1);
    }
}

// 切换回合
function switchTurn() {
    currentPlayer = currentPlayer === 1 ? 2 : 1;
    document.getElementById('turnDisplay').textContent = `玩家${currentPlayer === 1 ? '①' : '②'}的回合`;
}

// 修改分数
function changeScore(player, delta) {
    const currentScore = gameState.players[player].score + delta;
    gameState.players[player].score = Math.max(0, currentScore);
    document.getElementById(`score${player}`).textContent = gameState.players[player].score;
}

// 重置游戏
function resetGame() {
    gameState.cells = generateCells();
    gameState.players[1] = { pos: 0, score: 0 };
    gameState.players[2] = { pos: 0, score: 0 };
    currentPlayer = 1;
    document.getElementById('score1').textContent = '0';
    document.getElementById('score2').textContent = '0';
    document.getElementById('turnDisplay').textContent = '玩家①的回合';
    initBoard();
}

// 初始加载
initBoard();
window.addEventListener('resize', updateMarkers);
</script>
</body>
</html>

[01_dice.html](https://github.com/user-attachments/files/32440008/01_dice.html)
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2048 퍼즐</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Pretendard', sans-serif; user-select: none; }
        body { background: #faf8ef; color: #776e65; display: flex; flex-direction: column; align-items: center; justify-content: center; min-height: 100vh; }

        .header { display: flex; justify-content: space-between; align-items: center; width: 340px; margin-bottom: 15px; }
        .title { font-size: 2.5rem; font-weight: bold; color: #776e65; }
        .scores { display: flex; gap: 10px; }
        .score-box { background: #bbada0; padding: 8px 15px; border-radius: 6px; text-align: center; color: #fff; }
        .score-box .label { font-size: 0.7rem; text-transform: uppercase; color: #eee; }
        .score-box .value { font-size: 1.2rem; font-weight: bold; }

        .btn-newgame { background: #8f7a66; color: #f9f6f2; border: none; padding: 8px 16px; border-radius: 4px; font-weight: bold; cursor: pointer; transition: 0.2s; }
        .btn-newgame:hover { background: #9f8a76; }

        .game-container { position: relative; width: 340px; height: 340px; background: #bbada0; border-radius: 8px; padding: 10px; }
        .grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px; width: 100%; height: 100%; }
        
        .cell { background: rgba(238, 228, 218, 0.35); border-radius: 6px; display: flex; align-items: center; justify-content: center; font-size: 1.8rem; font-weight: bold; transition: all 0.15s ease-in-out; }

        /* 타일 색상 지정 */
        .tile-2 { background: #eee4da; color: #776e65; }
        .tile-4 { background: #ede0c8; color: #776e65; }
        .tile-8 { background: #f2b179; color: #f9f6f2; }
        .tile-16 { background: #f59563; color: #f9f6f2; }
        .tile-32 { background: #f67c5f; color: #f9f6f2; }
        .tile-64 { background: #f65e3b; color: #f9f6f2; }
        .tile-128 { background: #edcf72; color: #f9f6f2; font-size: 1.5rem; }
        .tile-256 { background: #edcc61; color: #f9f6f2; font-size: 1.5rem; }
        .tile-512 { background: #edc850; color: #f9f6f2; font-size: 1.5rem; }
        .tile-1024 { background: #edc53f; color: #f9f6f2; font-size: 1.2rem; }
        .tile-2048 { background: #edc22e; color: #f9f6f2; font-size: 1.2rem; box-shadow: 0 0 10px rgba(237,194,46,0.8); }

        /* 오버레이 (게임 오버 / 승리) */
        .overlay { display: none; position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(238, 228, 218, 0.73); border-radius: 8px; flex-direction: column; align-items: center; justify-content: center; z-index: 10; }
        .overlay h2 { font-size: 2.2rem; margin-bottom: 15px; color: #776e65; }

        .info { margin-top: 15px; font-size: 0.85rem; color: #776e65; }
    </style>
</head>
<body>

    <div class="header">
        <div class="title">2048</div>
        <div class="scores">
            <div class="score-box">
                <div class="label">SCORE</div>
                <div class="value" id="score">0</div>
            </div>
            <div class="score-box">
                <div class="label">BEST</div>
                <div class="value" id="bestScore">0</div>
            </div>
        </div>
    </div>

    <div style="width:340px; display:flex; justify-content:space-between; align-items:center; margin-bottom:15px;">
        <span class="info">동일한 숫자를 합쳐 <b>2048</b>을 만드세요!</span>
        <button class="btn-newgame" onclick="restartGame()">새 게임</button>
    </div>

    <div class="game-container">
        <div class="grid" id="grid"></div>
        <div class="overlay" id="overlay">
            <h2 id="overlayText">Game Over!</h2>
            <button class="btn-newgame" onclick="restartGame()">다시 도전</button>
        </div>
    </div>

    <script>
        const gridEl = document.getElementById('grid');
        const scoreEl = document.getElementById('score');
        const bestScoreEl = document.getElementById('bestScore');
        const overlay = document.getElementById('overlay');
        const overlayText = document.getElementById('overlayText');

        let board = Array(16).fill(0);
        let score = 0;
        let bestScore = localStorage.getItem('2048_best') || 0;
        bestScoreEl.innerText = bestScore;

        function init() {
            board = Array(16).fill(0);
            score = 0;
            scoreEl.innerText = score;
            overlay.style.display = 'none';
            addTile();
            addTile();
            render();
        }

        function render() {
            gridEl.innerHTML = '';
            board.forEach((val) => {
                const cell = document.createElement('div');
                cell.className = 'cell';
                if (val > 0) {
                    cell.innerText = val;
                    cell.classList.add(`tile-${val}`);
                }
                gridEl.appendChild(cell);
            });
        }

        function addTile() {
            const emptyIndices = board.map((val, idx) => val === 0 ? idx : null).filter(val => val !== null);
            if (emptyIndices.length > 0) {
                const randIndex = emptyIndices[Math.floor(Math.random() * emptyIndices.length)];
                board[randIndex] = Math.random() < 0.9 ? 2 : 4;
            }
        }

        function slide(row) {
            let arr = row.filter(val => val !== 0);
            for (let i = 0; i < arr.length - 1; i++) {
                if (arr[i] === arr[i + 1]) {
                    arr[i] *= 2;
                    score += arr[i];
                    arr[i + 1] = 0;
                    if (arr[i] === 2048) showWin();
                }
            }
            arr = arr.filter(val => val !== 0);
            while (arr.length < 4) arr.push(0);
            return arr;
        }

        function moveLeft() {
            let moved = false;
            for (let i = 0; i < 4; i++) {
                let row = board.slice(i * 4, i * 4 + 4);
                let newRow = slide(row);
                for (let j = 0; j < 4; j++) {
                    if (board[i * 4 + j] !== newRow[j]) moved = true;
                    board[i * 4 + j] = newRow[j];
                }
            }
            return moved;
        }

        function moveRight() {
            let moved = false;
            for (let i = 0; i < 4; i++) {
                let row = board.slice(i * 4, i * 4 + 4).reverse();
                let newRow = slide(row).reverse();
                for (let j = 0; j < 4; j++) {
                    if (board[i * 4 + j] !== newRow[j]) moved = true;
                    board[i * 4 + j] = newRow[j];
                }
            }
            return moved;
        }

        function moveUp() {
            let moved = false;
            for (let i = 0; i < 4; i++) {
                let col = [board[i], board[i + 4], board[i + 8], board[i + 12]];
                let newCol = slide(col);
                for (let j = 0; j < 4; j++) {
                    if (board[i + j * 4] !== newCol[j]) moved = true;
                    board[i + j * 4] = newCol[j];
                }
            }
            return moved;
        }

        function moveDown() {
            let moved = false;
            for (let i = 0; i < 4; i++) {
                let col = [board[i], board[i + 4], board[i + 8], board[i + 12]].reverse();
                let newCol = slide(col).reverse();
                for (let j = 0; j < 4; j++) {
                    if (board[i + j * 4] !== newCol[j]) moved = true;
                    board[i + j * 4] = newCol[j];
                }
            }
            return moved;
        }

        function handleInput(dir) {
            let moved = false;
            if (dir === 'left') moved = moveLeft();
            if (dir === 'right') moved = moveRight();
            if (dir === 'up') moved = moveUp();
            if (dir === 'down') moved = moveDown();

            if (moved) {
                addTile();
                scoreEl.innerText = score;
                if (score > bestScore) {
                    bestScore = score;
                    bestScoreEl.innerText = bestScore;
                    localStorage.setItem('2048_best', bestScore);
                }
                render();
                checkGameOver();
            }
        }

        function checkGameOver() {
            if (board.includes(0)) return;
            for (let i = 0; i < 4; i++) {
                for (let j = 0; j < 4; j++) {
                    let curr = board[i * 4 + j];
                    if (j < 3 && curr === board[i * 4 + j + 1]) return;
                    if (i < 3 && curr === board[(i + 1) * 4 + j]) return;
                }
            }
            overlayText.innerText = 'Game Over!';
            overlay.style.display = 'flex';
        }

        function showWin() {
            overlayText.innerText = '2048 달성 승리!';
            overlay.style.display = 'flex';
        }

        function restartGame() {
            init();
        }

        // 키보드 조작
        window.addEventListener('keydown', (e) => {
            if (e.key === 'ArrowLeft') handleInput('left');
            if (e.key === 'ArrowRight') handleInput('right');
            if (e.key === 'ArrowUp') handleInput('up');
            if (e.key === 'ArrowDown') handleInput('down');
        });

        // 모바일 터치/스와이프 지원
        let startX, startY;
        window.addEventListener('touchstart', (e) => {
            startX = e.touches[0].clientX;
            startY = e.touches[0].clientY;
        });

        window.addEventListener('touchend', (e) => {
            if (!startX || !startY) return;
            let diffX = e.changedTouches[0].clientX - startX;
            let diffY = e.changedTouches[0].clientY - startY;

            if (Math.abs(diffX) > Math.abs(diffY)) {
                if (diffX > 30) handleInput('right');
                else if (diffX < -30) handleInput('left');
            } else {
                if (diffY > 30) handleInput('down');
                else if (diffY < -30) handleInput('up');
            }
            startX = null; startY = null;
        });

        init();
    </script>
</body>
</html>
[02_dice.html](https://github.com/user-attachments/files/32440010/02_dice.html)
[03_dice.html](https://github.com/user-attachments/files/32440018/03_dice.html)
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>스파이더 카드 솔리테어</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Pretendard', sans-serif; user-select: none; }
        body { background: #0b6623; color: #fff; display: flex; flex-direction: column; align-items: center; min-height: 100vh; padding: 20px; }

        .header { display: flex; justify-content: space-between; align-items: center; width: 100%; max-width: 800px; margin-bottom: 20px; }
        .title { font-size: 1.8rem; font-weight: bold; }
        .status { display: flex; gap: 20px; font-size: 1.1rem; }
        .btn { background: #2e8b57; color: #fff; border: 1px solid #3cb371; padding: 8px 16px; border-radius: 6px; cursor: pointer; font-weight: bold; }
        .btn:hover { background: #3cb371; }

        /* 게임 필드 */
        .board { display: flex; gap: 10px; justify-content: center; width: 100%; max-width: 850px; height: 500px; }
        .column { flex: 1; position: relative; background: rgba(0,0,0,0.15); border-radius: 6px; border: 1px dashed rgba(255,255,255,0.2); height: 100%; }

        /* 카드 스타일 */
        .card { position: absolute; width: 100%; height: 90px; background: #fff; border: 1px solid #ccc; border-radius: 6px; display: flex; flex-direction: column; justify-content: space-between; padding: 5px; color: #000; font-weight: bold; cursor: pointer; box-shadow: 0 2px 5px rgba(0,0,0,0.3); transition: transform 0.1s, box-shadow 0.1s; }
        .card.back { background: linear-gradient(135deg, #1e3c72, #2a5298); border: 2px solid #fff; color: transparent; }
        .card.selected { transform: translateY(-10px); box-shadow: 0 5px 15px rgba(255,255,0,0.8); border: 2px solid #ffd700; }
        .card.red { color: #d63031; }

        /* 하단 컨트롤 (예비 덱 & 완성 영역) */
        .bottom-bar { display: flex; justify-content: space-between; align-items: center; width: 100%; max-width: 800px; margin-top: 20px; }
        .stock-deck { width: 70px; height: 90px; background: linear-gradient(135deg, #1e3c72, #2a5298); border: 2px solid #fff; border-radius: 6px; cursor: pointer; display: flex; align-items: center; justify-content: center; font-weight: bold; box-shadow: 0 4px 8px rgba(0,0,0,0.3); }
        .completed-sets { display: flex; gap: 5px; }
        .completed-slot { width: 50px; height: 70px; border: 1px dashed rgba(255,255,255,0.4); border-radius: 4px; display: flex; align-items: center; justify-content: center; font-size: 0.8rem; color: #ccc; }

        /* 승리 메시지 */
        .overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); z-index: 100; flex-direction: column; align-items: center; justify-content: center; }
        .overlay h1 { font-size: 3rem; color: #ffd700; margin-bottom: 20px; }
    </style>
</head>
<body>

    <div class="header">
        <div class="title">♠️ 스파이더 솔리테어</div>
        <div class="status">
            <div>점수: <span id="score">500</span></div>
            <div>이동: <span id="moves">0</span></div>
        </div>
        <button class="btn" onclick="initGame()">새 게임</button>
    </div>

    <div class="board" id="board">
        <!-- 10개의 카드가 들어갈 열 -->
        <div class="column" onclick="handleColumnClick(0)"></div>
        <div class="column" onclick="handleColumnClick(1)"></div>
        <div class="column" onclick="handleColumnClick(2)"></div>
        <div class="column" onclick="handleColumnClick(3)"></div>
        <div class="column" onclick="handleColumnClick(4)"></div>
        <div class="column" onclick="handleColumnClick(5)"></div>
        <div class="column" onclick="handleColumnClick(6)"></div>
        <div class="column" onclick="handleColumnClick(7)"></div>
        <div class="column" onclick="handleColumnClick(8)"></div>
        <div class="column" onclick="handleColumnClick(9)"></div>
    </div>

    <div class="bottom-bar">
        <div class="completed-sets" id="completedSets"></div>
        <div class="stock-deck" id="stockDeck" onclick="drawStockCards()">카드 배포</div>
    </div>

    <div class="overlay" id="winOverlay">
        <h1>축하합니다! 승리하셨습니다! 🎉</h1>
        <button class="btn" onclick="initGame()">다시 하기</button>
    </div>

    <script>
        const suits = ['♠']; // 1-Suit (초급 모드)
        const values = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13];
        const valNames = { 1:'A', 11:'J', 12:'Q', 13:'K' };

        let deck = [];
        let columns = [[],[],[],[],[],[],[],[],[],[]];
        let stock = [];
        let completedCount = 0;
        let selected = null; // { colIndex, cardIndex }
        let score = 500;
        let moves = 0;

        function createDeck() {
            deck = [];
            // 총 8세트 (104장)
            for (let i = 0; i < 8; i++) {
                values.forEach(v => {
                    deck.push({ suit: '♠', value: v, faceUp: false });
                });
            }
            deck.sort(() => Math.random() - 0.5);
        }

        function initGame() {
            createDeck();
            columns = [[],[],[],[],[],[],[],[],[],[]];
            stock = [];
            completedCount = 0;
            selected = null;
            score = 500;
            moves = 0;
            document.getElementById('winOverlay').style.display = 'none';

            // 54장을 10개 열에 배포 (처음 4열은 6장, 나머지 4장은 5장)
            for (let i = 0; i < 54; i++) {
                const colIdx = i % 10;
                columns[colIdx].push(deck.pop());
            }

            // 각 열의 가장 위 카드 오픈
            columns.forEach(col => {
                if (col.length > 0) col[col.length - 1].faceUp = true;
            });

            // 남은 50장은 예비 덱으로
            stock = deck;

            updateUI();
        }

        function updateUI() {
            document.getElementById('score').innerText = score;
            document.getElementById('moves').innerText = moves;

            // 보드 채우기
            const boardColEls = document.querySelectorAll('.column');
            boardColEls.forEach((colEl, cIdx) => {
                colEl.innerHTML = '';
                columns[cIdx].forEach((card, rIdx) => {
                    const cardEl = document.createElement('div');
                    cardEl.className = `card ${card.faceUp ? '' : 'back'}`;
                    
                    if (selected && selected.colIndex === cIdx && rIdx >= selected.cardIndex) {
                        cardEl.classList.add('selected');
                    }

                    if (card.faceUp) {
                        const valStr = valNames[card.value] || card.value;
                        cardEl.innerHTML = `<div>${valStr}</div><div style="align-self:flex-end;">${card.suit}</div>`;
                    }

                    cardEl.style.top = `${rIdx * 24}px`;
                    cardEl.onclick = (e) => {
                        e.stopPropagation();
                        handleCardClick(cIdx, rIdx);
                    };

                    colEl.appendChild(cardEl);
                });
            });

            // 예비 덱 갱신
            const stockEl = document.getElementById('stockDeck');
            stockEl.innerText = stock.length > 0 ? `카드 배포 (${stock.length / 10})` : '비어있음';
            stockEl.style.opacity = stock.length > 0 ? '1' : '0.5';

            // 완성 영역
            const completedEl = document.getElementById('completedSets');
            completedEl.innerHTML = '';
            for (let i = 0; i < completedCount; i++) {
                const slot = document.createElement('div');
                slot.className = 'completed-slot';
                slot.innerText = '♠ K-A';
                completedEl.appendChild(slot);
            }
        }

        function handleCardClick(colIndex, cardIndex) {
            const card = columns[colIndex][cardIndex];
            if (!card.faceUp) return;

            // 이미 선택된 상태라면 이동 시도
            if (selected) {
                if (selected.colIndex === colIndex) {
                    selected = null; // 같은 열 클릭 시 해제
                } else {
                    moveCards(selected.colIndex, selected.cardIndex, colIndex);
                }
            } else {
                // 새로운 카드 선택 (선택한 카드 아래로 연속 내림차순인지 확인)
                if (isValidSequence(colIndex, cardIndex)) {
                    selected = { colIndex, cardIndex };
                }
            }
            updateUI();
        }

        function handleColumnClick(colIndex) {
            if (selected) {
                moveCards(selected.colIndex, selected.cardIndex, colIndex);
                updateUI();
            }
        }

        function isValidSequence(colIndex, cardIndex) {
            const col = columns[colIndex];
            for (let i = cardIndex; i < col.length - 1; i++) {
                if (col[i].value !== col[i + 1].value + 1) {
                    return false; // 연속된 내림차순이 아니면 묶어서 이동 불가능
                }
            }
            return true;
        }

        function moveCards(fromCol, fromIdx, toCol) {
            const targetCol = columns[toCol];
            const movingCards = columns[fromCol].slice(fromIdx);

            // 타겟 열이 비어있거나, 타겟의 맨 위 카드보다 이동할 카드의 시작값이 1 작아야 함
            if (targetCol.length === 0 || targetCol[targetCol.length - 1].value === movingCards[0].value + 1) {
                columns[fromCol].splice(fromIdx);
                columns[toCol].push(...movingCards);

                // 이전 열의 맨 위 카드 뒤집기
                if (columns[fromCol].length > 0) {
                    columns[fromCol][columns[fromCol].length - 1].faceUp = true;
                }

                moves++;
                score = Math.max(0, score - 1);
                selected = null;

                checkCompletedSet(toCol);
            } else {
                selected = null; // 조건 불일치 시 해제
            }
        }

        function drawStockCards() {
            if (stock.length === 0) return;

            // 모든 열에 최소 1장의 카드가 있어야 배포 가능
            if (columns.some(col => col.length === 0)) {
                alert('빈 열이 없어야 새 카드를 배포할 수 있습니다!');
                return;
            }

            for (let i = 0; i < 10; i++) {
                const card = stock.pop();
                card.faceUp = true;
                columns[i].push(card);
                checkCompletedSet(i);
            }

            selected = null;
            updateUI();
        }

        function checkCompletedSet(colIndex) {
            const col = columns[colIndex];
            if (col.length < 13) return;

            // K(13)부터 A(1)까지 완성되어 있는지 확인
            const last13 = col.slice(col.length - 13);
            let isComplete = true;

            for (let i = 0; i < 13; i++) {
                if (!last13[i].faceUp || last13[i].value !== 13 - i) {
                    isComplete = false;
                    break;
                }
            }

            if (isComplete) {
                columns[colIndex].splice(col.length - 13);
                if (columns[colIndex].length > 0) {
                    columns[colIndex][columns[colIndex].length - 1].faceUp = true;
                }
                completedCount++;
                score += 100;

                if (completedCount === 8) {
                    document.getElementById('winOverlay').style.display = 'flex';
                }
            }
        }

        initGame();
    </script>
</body>
</html>
[04_dice.html](https://github.com/user-attachments/files/32440019/04_dice.html)
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>버블 슈터 게임</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Pretendard', sans-serif; user-select: none; }
        body { background: #0f172a; color: #fff; display: flex; flex-direction: column; align-items: center; justify-content: center; min-height: 100vh; }

        .header { display: flex; justify-content: space-between; align-items: center; width: 360px; margin-bottom: 12px; }
        .title { font-size: 1.5rem; font-weight: bold; color: #38bdf8; }
        .score-box { font-size: 1.2rem; font-weight: bold; color: #f1f5f9; }

        .game-container { position: relative; width: 360px; height: 520px; background: #1e293b; border-radius: 12px; overflow: hidden; box-shadow: 0 10px 25px rgba(0,0,0,0.5); border: 2px solid #334155; }
        canvas { background: #0f172a; display: block; }

        /* 오버레이 */
        .overlay { display: none; position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(15, 23, 42, 0.85); flex-direction: column; align-items: center; justify-content: center; z-index: 10; }
        .overlay h2 { font-size: 2.2rem; color: #ef4444; margin-bottom: 10px; }
        .overlay p { font-size: 1.2rem; color: #cbd5e1; margin-bottom: 20px; }
        
        .btn-restart { background: #0ea5e9; color: #fff; border: none; padding: 10px 20px; font-size: 1rem; font-weight: bold; border-radius: 20px; cursor: pointer; transition: 0.2s; }
        .btn-restart:hover { background: #38bdf8; transform: scale(1.05); }

        .info { margin-top: 12px; font-size: 0.85rem; color: #94a3b8; }
    </style>
</head>
<body>

    <div class="header">
        <div class="title">🫧 버블 슈터</div>
        <div class="score-box">점수: <span id="score">0</span></div>
    </div>

    <div class="game-container">
        <canvas id="gameCanvas" width="360" height="520"></canvas>
        <div class="overlay" id="overlay">
            <h2 id="overlayTitle">GAME OVER</h2>
            <p id="finalScore">최종 점수: 0</p>
            <button class="btn-restart" onclick="initGame()">다시 시작</button>
        </div>
    </div>

    <div class="info">마우스로 조준하고 클릭하여 같은 색상 버블을 3개 이상 맞추세요!</div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreEl = document.getElementById('score');
        const overlay = document.getElementById('overlay');
        const overlayTitle = document.getElementById('overlayTitle');
        const finalScore = document.getElementById('finalScore');

        const BUBBLE_RADIUS = 20;
        const GRID_ROWS = 10;
        const GRID_COLS = 9;
        const COLORS = ['#ef4444', '#3b82f6', '#10b981', '#f59e0b', '#a855f7'];

        let grid = [];
        let score = 0;
        let isGameOver = false;
        let currentBubble = null;
        let nextBubbleColor = '';
        let aimAngle = Math.PI / 2;
        let shotsCount = 0;

        // 버블 클래스
        class Bubble {
            constructor(x, y, color) {
                this.x = x;
                this.y = y;
                this.color = color;
                this.vx = 0;
                this.vy = 0;
                this.isMoving = false;
            }

            draw() {
                ctx.beginPath();
                ctx.arc(this.x, this.y, BUBBLE_RADIUS - 1, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                
                // 입체감 광택 효과
                ctx.beginPath();
                ctx.arc(this.x - BUBBLE_RADIUS / 3, this.y - BUBBLE_RADIUS / 3, BUBBLE_RADIUS / 4, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
                ctx.fill();
            }

            update() {
                if (!this.isMoving) return;

                this.x += this.vx;
                this.y += this.vy;

                // 좌우 벽면 반사
                if (this.x - BUBBLE_RADIUS <= 0 || this.x + BUBBLE_RADIUS >= canvas.width) {
                    this.vx *= -1;
                    this.x = Math.max(BUBBLE_RADIUS, Math.min(canvas.width - BUBBLE_RADIUS, this.x));
                }

                // 천장 또는 기존 버블 충돌 체크
                if (this.y - BUBBLE_RADIUS <= 0 || checkCollision(this)) {
                    this.isMoving = false;
                    snapToGrid(this);
                }
            }
        }

        function initGame() {
            score = 0;
            shotsCount = 0;
            isGameOver = false;
            scoreEl.innerText = score;
            overlay.style.display = 'none';

            // 그리드 초기화 (상단 4줄 버블 채우기)
            grid = Array.from({ length: GRID_ROWS }, () => Array(GRID_COLS).fill(null));
            for (let r = 0; r < 4; r++) {
                for (let c = 0; c < (r % 2 === 1 ? GRID_COLS - 1 : GRID_COLS); c++) {
                    const pos = getCanvasPos(r, c);
                    grid[r][c] = new Bubble(pos.x, pos.y, getRandomColor());
                }
            }

            nextBubbleColor = getRandomColor();
            spawnShooterBubble();
            requestAnimationFrame(gameLoop);
        }

        function getRandomColor() {
            return COLORS[Math.floor(Math.random() * COLORS.length)];
        }

        function getCanvasPos(row, col) {
            const isOffset = row % 2 === 1;
            const startX = isOffset ? BUBBLE_RADIUS * 2 : BUBBLE_RADIUS;
            const x = startX + col * (BUBBLE_RADIUS * 2);
            const y = BUBBLE_RADIUS + row * (BUBBLE_RADIUS * 1.732);
            return { x, y };
        }

        function spawnShooterBubble() {
            currentBubble = new Bubble(canvas.width / 2, canvas.height - 40, nextBubbleColor);
            nextBubbleColor = getRandomColor();
        }

        function checkCollision(bubble) {
            for (let r = 0; r < GRID_ROWS; r++) {
                for (let c = 0; c < GRID_COLS; c++) {
                    const target = grid[r][c];
                    if (target) {
                        const dist = Math.hypot(bubble.x - target.x, bubble.y - target.y);
                        if (dist < BUBBLE_RADIUS * 2 - 2) return true;
                    }
                }
            }
            return false;
        }

        function snapToGrid(bubble) {
            let closestDist = Infinity;
            let targetR = 0, targetC = 0;

            for (let r = 0; r < GRID_ROWS; r++) {
                const colsInRow = r % 2 === 1 ? GRID_COLS - 1 : GRID_COLS;
                for (let c = 0; c < colsInRow; c++) {
                    if (!grid[r][c]) {
                        const pos = getCanvasPos(r, c);
                        const dist = Math.hypot(bubble.x - pos.x, bubble.y - pos.y);
                        if (dist < closestDist) {
                            closestDist = dist;
                            targetR = r;
                            targetC = c;
                        }
                    }
                }
            }

            const pos = getCanvasPos(targetR, targetC);
            grid[targetR][targetC] = new Bubble(pos.x, pos.y, bubble.color);

            // 매칭 검사
            const matches = getConnectedMatches(targetR, targetC, bubble.color);
            if (matches.length >= 3) {
                popBubbles(matches);
                dropFloatingBubbles();
            }

            shotsCount++;
            if (shotsCount % 6 === 0) {
                shiftGridDown();
            }

            checkGameOverCondition();
            if (!isGameOver) {
                spawnShooterBubble();
            }
        }

        function getConnectedMatches(startR, startC, color) {
            const visited = new Set();
            const matches = [];

            function dfs(r, c) {
                const key = `${r},${c}`;
                if (visited.has(key)) return;
                visited.add(key);

                if (r < 0 || r >= GRID_ROWS || !grid[r][c] || grid[r][c].color !== color) return;

                matches.push({ r, c });

                getNeighbors(r, c).forEach(n => dfs(n.r, n.c));
            }

            dfs(startR, startC);
            return matches;
        }

        function getNeighbors(r, c) {
            const isOffset = r % 2 === 1;
            const directions = isOffset ? [
                { r: r, c: c - 1 }, { r: r, c: c + 1 },
                { r: r - 1, c: c }, { r: r - 1, c: c + 1 },
                { r: r + 1, c: c }, { r: r + 1, c: c + 1 }
            ] : [
                { r: r, c: c - 1 }, { r: r, c: c + 1 },
                { r: r - 1, c: c - 1 }, { r: r - 1, c: c },
                { r: r + 1, c: c - 1 }, { r: r + 1, c: c }
            ];

            return directions.filter(d => d.r >= 0 && d.r < GRID_ROWS && d.c >= 0 && d.c < (d.r % 2 === 1 ? GRID_COLS - 1 : GRID_COLS));
        }

        function popBubbles(matches) {
            matches.forEach(m => {
                grid[m.r][m.c] = null;
            });
            score += matches.length * 10;
            scoreEl.innerText = score;
        }

        function dropFloatingBubbles() {
            const connectedToTop = new Set();

            function dfs(r, c) {
                const key = `${r},${c}`;
                if (connectedToTop.has(key) || !grid[r][c]) return;
                connectedToTop.add(key);

                getNeighbors(r, c).forEach(n => dfs(n.r, n.c));
            }

            // 첫째 줄 버블에서 시작하는 모든 연결 탐색
            for (let c = 0; c < GRID_COLS; c++) {
                if (grid[0][c]) dfs(0, c);
            }

            // 연결되지 않은 공중 버블 제거
            for (let r = 0; r < GRID_ROWS; r++) {
                for (let c = 0; c < GRID_COLS; c++) {
                    if (grid[r][c] && !connectedToTop.has(`${r},${c}`)) {
                        grid[r][c] = null;
                        score += 20;
                    }
                }
            }
            scoreEl.innerText = score;
        }

        function shiftGridDown() {
            for (let r = GRID_ROWS - 1; r > 0; r--) {
                grid[r] = [...grid[r - 1]];
            }
            grid[0] = Array.from({ length: GRID_COLS }, () => new Bubble(0, 0, getRandomColor()));

            for (let r = 0; r < GRID_ROWS; r++) {
                for (let c = 0; c < GRID_COLS; c++) {
                    if (grid[r][c]) {
                        const pos = getCanvasPos(r, c);
                        grid[r][c].x = pos.x;
                        grid[r][c].y = pos.y;
                    }
                }
            }
        }

        function checkGameOverCondition() {
            for (let c = 0; c < GRID_COLS; c++) {
                if (grid[GRID_ROWS - 1][c]) {
                    isGameOver = true;
                    overlayTitle.innerText = "GAME OVER";
                    finalScore.innerText = `최종 점수: ${score}`;
                    overlay.style.display = 'flex';
                    return;
                }
            }
        }

        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 데드라인 선
            ctx.strokeStyle = '#ef4444';
            ctx.setLineDash([5, 5]);
            ctx.beginPath();
            ctx.moveTo(0, canvas.height - 80);
            ctx.lineTo(canvas.width, canvas.height - 80);
            ctx.stroke();
            ctx.setLineDash([]);

            // 그리드 버블 그리기
            for (let r = 0; r < GRID_ROWS; r++) {
                for (let c = 0; c < GRID_COLS; c++) {
                    if (grid[r][c]) grid[r][c].draw();
                }
            }

            // 발사 조준선 그리기
            if (currentBubble && !currentBubble.isMoving) {
                ctx.strokeStyle = 'rgba(255, 255, 255, 0.4)';
                ctx.lineWidth = 2;
                ctx.beginPath();
                ctx.moveTo(currentBubble.x, currentBubble.y);
                ctx.lineTo(currentBubble.x + Math.cos(aimAngle) * 80, currentBubble.y - Math.sin(aimAngle) * 80);
                ctx.stroke();
            }

            // 현재 버블 업데이트 및 그리기
            if (currentBubble) {
                currentBubble.update();
                currentBubble.draw();
            }

            // 다음 대기 버블 예시
            ctx.beginPath();
            ctx.arc(35, canvas.height - 35, BUBBLE_RADIUS - 4, 0, Math.PI * 2);
            ctx.fillStyle = nextBubbleColor;
            ctx.fill();

            if (!isGameOver) requestAnimationFrame(gameLoop);
        }

        // 마우스 조준 및 발사
        canvas.addEventListener('mousemove', (e) => {
            const rect = canvas.getBoundingClientRect();
            const mouseX = e.clientX - rect.left;
            const mouseY = e.clientY - rect.top;

            const dx = mouseX - canvas.width / 2;
            const dy = (canvas.height - 40) - mouseY;
            if (dy > 10) aimAngle = Math.atan2(dy, dx);
        });

        canvas.addEventListener('click', () => {
            if (currentBubble && !currentBubble.isMoving && !isGameOver) {
                const speed = 12;
                currentBubble.vx = Math.cos(aimAngle) * speed;
                currentBubble.vy = -Math.sin(aimAngle) * speed;
                currentBubble.isMoving = true;
            }
        });

        initGame();
    </script>
</body>
</html>
[05_dice.html](https://github.com/user-attachments/files/32440022/05_dice.html)
[06_dice.html](https://github.com/user-attachments/files/32440027/06_dice.html)

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>체스 vs AI</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Pretendard', sans-serif; user-select: none; }
        body { background: #0f172a; color: #fff; display: flex; flex-direction: column; align-items: center; justify-content: center; min-height: 100vh; padding: 10px; }

        .header { display: flex; justify-content: space-between; align-items: center; width: 400px; margin-bottom: 12px; }
        .title { font-size: 1.5rem; font-weight: 800; color: #38bdf8; display: flex; align-items: center; gap: 8px; }
        .status { font-size: 0.9rem; color: #cbd5e1; background: #1e293b; padding: 6px 12px; border-radius: 6px; border: 1px solid #334155; }

        /* 체스보드 */
        .board-container { position: relative; width: 400px; height: 400px; border-radius: 8px; overflow: hidden; border: 4px solid #334155; box-shadow: 0 10px 25px rgba(0,0,0,0.5); }
        .board { display: grid; grid-template-columns: repeat(8, 1fr); width: 100%; height: 100%; }

        .square { display: flex; justify-content: center; align-items: center; font-size: 2.3rem; cursor: pointer; position: relative; transition: background 0.1s ease; }
        .square.light { background: #e2e8f0; color: #0f172a; }
        .square.dark { background: #475569; color: #0f172a; }

        /* 하이라이트 효과 */
        .square.selected { background: #fde047 !important; }
        .square.valid-move::after { content: ''; width: 16px; height: 16px; background: rgba(34, 197, 94, 0.7); border-radius: 50%; position: absolute; }
        .square.valid-capture { background: #f87171 !important; }

        /* 컨트롤 영역 */
        .controls { width: 400px; margin-top: 15px; display: flex; justify-content: space-between; align-items: center; }
        .btn-restart { background: #0ea5e9; color: #fff; border: none; padding: 8px 18px; font-size: 0.95rem; font-weight: bold; border-radius: 6px; cursor: pointer; transition: 0.2s; }
        .btn-restart:hover { background: #38bdf8; }

        .captured-container { display: flex; gap: 5px; font-size: 1.1rem; }

        /* 오버레이 */
        .overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(15, 23, 42, 0.85); flex-direction: column; align-items: center; justify-content: center; z-index: 100; }
        .overlay h2 { font-size: 2.5rem; color: #38bdf8; margin-bottom: 10px; }
        .overlay p { font-size: 1.2rem; color: #cbd5e1; margin-bottom: 20px; }
    </style>
</head>
<body>

    <div class="header">
        <div class="title">♟️ 체스 (vs AI)</div>
        <div class="status" id="gameStatus">당신의 차례 (White)</div>
    </div>

    <div class="board-container">
        <div class="board" id="board"></div>
    </div>

    <div class="controls">
        <div class="captured-container" id="capturedPieces"></div>
        <button class="btn-restart" onclick="initGame()">새 게임</button>
    </div>

    <div class="overlay" id="overlay">
        <h2 id="winnerText">승리!</h2>
        <p id="winnerSubText">체크메이트로 승리했습니다.</p>
        <button class="btn-restart" onclick="initGame()">다시 플레이</button>
    </div>

    <script>
        // 기물 유니코드 문자
        const PIECES = {
            'wP': '♙', 'wR': '♖', 'wN': '♘', 'wB': '♗', 'wQ': '♕', 'wK': '♔',
            'bP': '♟', 'bR': '♜', 'bN': '♞', 'bB': '♝', 'bQ': '♛', 'bK': '♚'
        };

        const PIECE_VALUES = { 'P': 10, 'N': 30, 'B': 30, 'R': 50, 'Q': 90, 'K': 900 };

        let boardState = [];
        let selectedSquare = null;
        let validMoves = [];
        let isWhiteTurn = true;
        let gameOver = false;

        const boardEl = document.getElementById('board');
        const statusEl = document.getElementById('gameStatus');
        const overlayEl = document.getElementById('overlay');
        const winnerTextEl = document.getElementById('winnerText');

        function initGame() {
            boardState = [
                ['bR', 'bN', 'bB', 'bQ', 'bK', 'bB', 'bN', 'bR'],
                ['bP', 'bP', 'bP', 'bP', 'bP', 'bP', 'bP', 'bP'],
                ['', '', '', '', '', '', '', ''],
                ['', '', '', '', '', '', '', ''],
                ['', '', '', '', '', '', '', ''],
                ['', '', '', '', '', '', '', ''],
                ['wP', 'wP', 'wP', 'wP', 'wP', 'wP', 'wP', 'wP'],
                ['wR', 'wN', 'wB', 'wQ', 'wK', 'wB', 'wN', 'wR']
            ];
            selectedSquare = null;
            validMoves = [];
            isWhiteTurn = true;
            gameOver = false;
            overlayEl.style.display = 'none';
            statusEl.innerText = "당신의 차례 (White)";
            renderBoard();
        }

        function renderBoard() {
            boardEl.innerHTML = '';
            for (let r = 0; r < 8; r++) {
                for (let c = 0; c < 8; c++) {
                    const square = document.createElement('div');
                    const isLight = (r + c) % 2 === 0;
                    square.className = `square ${isLight ? 'light' : 'dark'}`;
                    square.dataset.row = r;
                    square.dataset.col = c;

                    const piece = boardState[r][c];
                    if (piece) {
                        square.innerText = PIECES[piece];
                        square.style.color = piece.startsWith('w') ? '#ffffff' : '#000000';
                        if (piece.startsWith('w')) square.style.textShadow = '0 0 3px #000';
                    }

                    if (selectedSquare && selectedSquare.r === r && selectedSquare.c === c) {
                        square.classList.add('selected');
                    }

                    const isMove = validMoves.some(m => m.r === r && m.c === c);
                    if (isMove) {
                        if (boardState[r][c]) {
                            square.classList.add('valid-capture');
                        } else {
                            square.classList.add('valid-move');
                        }
                    }

                    square.addEventListener('click', () => onSquareClick(r, c));
                    boardEl.appendChild(square);
                }
            }
        }

        function onSquareClick(r, c) {
            if (gameOver || !isWhiteTurn) return;

            const piece = boardState[r][c];

            // 자신의 기물 선택
            if (piece && piece.startsWith('w')) {
                selectedSquare = { r, c };
                validMoves = getPseudoLegalMoves(r, c, boardState);
                renderBoard();
                return;
            }

            // 이동 가능한 위치 클릭
            if (selectedSquare) {
                const move = validMoves.find(m => m.r === r && m.c === c);
                if (move) {
                    makeMove(selectedSquare.r, selectedSquare.c, r, c);
                    selectedSquare = null;
                    validMoves = [];
                    renderBoard();

                    if (!checkGameOver('b')) {
                        isWhiteTurn = false;
                        statusEl.innerText = "AI 생각 중...";
                        setTimeout(aiMove, 400);
                    }
                } else {
                    selectedSquare = null;
                    validMoves = [];
                    renderBoard();
                }
            }
        }

        function makeMove(fromR, fromC, toR, toC) {
            const piece = boardState[fromR][fromC];
            boardState[toR][toC] = piece;
            boardState[fromR][fromC] = '';

            // 폰 승급 (퀸으로 자동 변경)
            if (piece === 'wP' && toR === 0) boardState[toR][toC] = 'wQ';
            if (piece === 'bP' && toR === 7) boardState[toR][toC] = 'bQ';
        }

        // 단순화된 규칙 기물 이동 경로 계산
        function getPseudoLegalMoves(r, c, board) {
            const piece = board[r][c];
            if (!piece) return [];
            const color = piece[0];
            const type = piece[1];
            const moves = [];

            const addMove = (tr, tc) => {
                if (tr < 0 || tr >= 8 || tc < 0 || tc >= 8) return false;
                const target = board[tr][tc];
                if (!target) {
                    moves.push({ r: tr, c: tc });
                    return true;
                }
                if (target[0] !== color) {
                    moves.push({ r: tr, c: tc });
                }
                return false;
            };

            if (type === 'P') {
                const dir = color === 'w' ? -1 : 1;
                // 전진
                if (r + dir >= 0 && r + dir < 8 && !board[r + dir][c]) {
                    moves.push({ r: r + dir, c });
                    // 시작점 2칸 전진
                    if ((color === 'w' && r === 6) || (color === 'b' && r === 1)) {
                        if (!board[r + 2 * dir][c]) moves.push({ r: r + 2 * dir, c });
                    }
                }
                // 대각선 공격
                [-1, 1].forEach(dc => {
                    const tr = r + dir, tc = c + dc;
                    if (tr >= 0 && tr < 8 && tc >= 0 && tc < 8) {
                        const target = board[tr][tc];
                        if (target && target[0] !== color) moves.push({ r: tr, c: tc });
                    }
                });
            } else if (type === 'N') {
                const offsets = [[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]];
                offsets.forEach(([dr, dc]) => addMove(r + dr, c + dc));
            } else if (type === 'B' || type === 'R' || type === 'Q') {
                const dirs = [];
                if (type === 'B' || type === 'Q') dirs.push([-1,-1],[-1,1],[1,-1],[1,1]);
                if (type === 'R' || type === 'Q') dirs.push([-1,0],[1,0],[0,-1],[0,1]);
                dirs.forEach(([dr, dc]) => {
                    let tr = r + dr, tc = c + dc;
                    while (addMove(tr, tc)) {
                        tr += dr;
                        tc += dc;
                    }
                });
            } else if (type === 'K') {
                for (let dr = -1; dr <= 1; dr++) {
                    for (let dc = -1; dc <= 1; dc++) {
                        if (dr !== 0 || dc !== 0) addMove(r + dr, c + dc);
                    }
                }
            }

            return moves;
        }

        // AI 미니맥스 알고리즘 (간이)
        function aiMove() {
            let bestMove = null;
            let bestValue = -Infinity;

            for (let r = 0; r < 8; r++) {
                for (let c = 0; c < 8; c++) {
                    if (boardState[r][c].startsWith('b')) {
                        const moves = getPseudoLegalMoves(r, c, boardState);
                        for (const m of moves) {
                            const temp = boardState[m.r][m.c];
                            boardState[m.r][m.c] = boardState[r][c];
                            boardState[r][c] = '';

                            const val = evaluateBoard(boardState);

                            boardState[r][c] = boardState[m.r][m.c];
                            boardState[m.r][m.c] = temp;

                            if (val > bestValue) {
                                bestValue = val;
                                bestMove = { fromR: r, fromC: c, toR: m.r, toC: m.c };
                            }
                        }
                    }
                }
            }

            if (bestMove) {
                makeMove(bestMove.fromR, bestMove.fromC, bestMove.toR, bestMove.toC);
            }

            isWhiteTurn = true;
            statusEl.innerText = "당신의 차례 (White)";
            renderBoard();
            checkGameOver('w');
        }

        function evaluateBoard(board) {
            let score = 0;
            for (let r = 0; r < 8; r++) {
                for (let c = 0; c < 8; c++) {
                    const p = board[r][c];
                    if (p) {
                        const val = PIECE_VALUES[p[1]] || 0;
                        if (p.startsWith('b')) score += val;
                        else score -= val;
                    }
                }
            }
            return score;
        }

        function checkGameOver(nextColor) {
            let kingFound = false;
            for (let r = 0; r < 8; r++) {
                for (let c = 0; c < 8; c++) {
                    if (boardState[r][c] === (nextColor + 'K')) {
                        kingFound = true;
                        break;
                    }
                }
            }

            if (!kingFound) {
                gameOver = true;
                winnerTextEl.innerText = nextColor === 'w' ? "AI 승리!" : "플레이어 승리!";
                overlayEl.style.display = 'flex';
                return true;
            }
            return false;
        }

        initGame();
    </script>
</body>
</html>

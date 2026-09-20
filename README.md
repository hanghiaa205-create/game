[01_dice.html](https://github.com/user-attachments/files/32439914/01_dice.html)
[02_dice.html](https://github.com/user-attachments/files/32439915/02_dice.html)[03_dice.html](https://github.com/user-attachments/files/32439916/03_dice.html)[04_dice.html](https://github.com/user-attachments/files/32439917/04_dice.html)[05_dice.html](https://github.com/user-attachments/files/32439922/05_dice.html)[06_dice.html](https://github.com/user-attachments/files/32439923/06_dice.html)<!DOCTYPE html>
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

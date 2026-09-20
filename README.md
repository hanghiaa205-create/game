[hanghia_game_dice.html](https://github.com/user-attachments/files/32440180/hanghia_game_dice.html)
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Arcade Hub - All-in-One Mini Games Portal</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Pretendard', 'sans-serif'],
                    },
                    colors: {
                        darkBg: '#0f172a',
                        cardBg: '#1e293b',
                        accent: '#6366f1'
                    }
                }
            }
        }
    </script>
    <style>
        body {
            background-color: #0b0f19;
            color: #f8fafc;
            font-family: 'Pretendard', sans-serif;
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }
        .glass {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }
        .game-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 12px 30px -10px rgba(99, 102, 241, 0.3);
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: #0b0f19; }
        ::-webkit-scrollbar-thumb { background: #334155; border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: #475569; }

        /* Solitaire CSS */
        .solitaire-card {
            width: 58px;
            height: 80px;
            border-radius: 6px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.5);
            cursor: pointer;
            transition: transform 0.1s, top 0.1s;
        }

        /* Canvas fit */
        canvas {
            touch-action: none;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between overflow-x-hidden">

    <header class="sticky top-0 z-30 glass border-b border-slate-800 px-4 lg:px-8 py-3.5">
        <div class="max-w-7xl mx-auto flex items-center justify-between gap-4">
            <div class="flex items-center gap-3 cursor-pointer" onclick="closeGameOverlay()">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-indigo-500 to-purple-600 flex items-center justify-center text-white shadow-lg shadow-indigo-500/30">
                    <i class="fa-solid fa-gamepad text-xl"></i>
                </div>
                <div>
                    <span class="font-extrabold text-xl tracking-tight bg-clip-text text-transparent bg-gradient-to-r from-white via-slate-200 to-indigo-300">
                        ARCADE HUB
                    </span>
                    <span class="text-xs text-indigo-400 block font-medium -mt-1">Natively Integrated Mini Games</span>
                </div>
            </div>

            <!-- Search Bar -->
            <div class="hidden md:flex flex-1 max-w-md relative">
                <i class="fa-solid fa-magnifying-glass absolute left-3.5 top-1/2 -translate-y-1/2 text-slate-400 text-sm"></i>
                <input type="text" id="searchInput" placeholder="Search game, puzzle, chess..." 
                    class="w-full pl-10 pr-4 py-2 bg-slate-900/80 border border-slate-700/60 rounded-xl text-sm focus:outline-none focus:border-indigo-500 text-slate-200 placeholder-slate-500">
            </div>

            <!-- Header Right Badges -->
            <div class="flex items-center gap-2">
                <button id="favFilterBtn" onclick="toggleFavoritesFilter()" class="flex items-center gap-2 px-3.5 py-2 rounded-xl bg-slate-800/80 border border-slate-700 text-slate-300 hover:text-amber-400 text-xs sm:text-sm font-semibold transition">
                    <i class="fa-solid fa-star text-amber-400"></i>
                    <span class="hidden sm:inline">Favorites</span>
                    <span id="favCount" class="bg-amber-400/20 text-amber-300 px-1.5 py-0.5 rounded-full text-xs font-bold">0</span>
                </button>
            </div>
        </div>
    </header>

    <main class="flex-1 max-w-7xl w-full mx-auto px-4 lg:px-8 py-6 flex flex-col gap-6">

        <!-- Search Bar (Mobile) -->
        <div class="md:hidden relative">
            <i class="fa-solid fa-magnifying-glass absolute left-3.5 top-1/2 -translate-y-1/2 text-slate-400 text-sm"></i>
            <input type="text" id="searchInputMobile" placeholder="Search games..." 
                class="w-full pl-10 pr-4 py-2 bg-slate-900 border border-slate-700/60 rounded-xl text-sm focus:outline-none focus:border-indigo-500 text-slate-200">
        </div>

        <!-- Categories Nav -->
        <div class="flex items-center gap-2 overflow-x-auto pb-2" id="categoryNav">
            <!-- Categories injected by JS -->
        </div>

        <!-- Games Grid -->
        <div id="gamesGrid" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-6">
            <!-- Game Cards injected by JS -->
        </div>

        <!-- Empty State -->
        <div id="emptyState" class="hidden flex-col items-center justify-center py-20 gap-3 text-center">
            <i class="fa-solid fa-ghost text-4xl text-slate-600"></i>
            <p class="text-slate-400 font-medium">No games matched your filter.</p>
            <button onclick="resetFilters()" class="mt-2 px-4 py-2 bg-indigo-600 text-white rounded-xl text-xs font-bold hover:bg-indigo-500">Reset Filters</button>
        </div>
    </main>

    <div id="gameOverlay" class="fixed inset-0 z-50 bg-slate-950/95 backdrop-blur-xl hidden flex-col transition-all duration-300">
        <!-- Overlay Bar -->
        <div class="flex items-center justify-between px-4 sm:px-8 py-3 bg-slate-900 border-b border-slate-800">
            <div class="flex items-center gap-3">
                <button onclick="closeGameOverlay()" class="px-3 py-1.5 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 hover:text-white text-xs font-semibold flex items-center gap-2">
                    <i class="fa-solid fa-arrow-left"></i> Menu
                </button>
                <div class="h-5 w-px bg-slate-700"></div>
                <div class="flex items-center gap-2">
                    <i id="overlayGameIcon" class="fa-solid fa-gamepad text-indigo-400 text-lg"></i>
                    <h2 id="overlayGameTitle" class="font-bold text-base sm:text-lg text-white">Game Title</h2>
                </div>
            </div>

            <!-- Stats & Quick Actions -->
            <div class="flex items-center gap-3">
                <div class="hidden sm:flex items-center gap-2 px-3 py-1 rounded-lg bg-slate-800/80 border border-slate-700/50 text-xs">
                    <span class="text-slate-400">High Score:</span>
                    <span id="overlayHighScore" class="font-bold text-amber-400">0</span>
                </div>
                <button id="restartBtn" onclick="restartCurrentGame()" class="p-2 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 hover:text-white" title="Restart">
                    <i class="fa-solid fa-rotate-right text-sm"></i>
                </button>
                <button onclick="closeGameOverlay()" class="p-2 rounded-xl bg-rose-500/20 hover:bg-rose-500 text-rose-300 hover:text-white border border-rose-500/30" title="Close">
                    <i class="fa-solid fa-xmark text-sm"></i>
                </button>
            </div>
        </div>

        <!-- Dynamic Game Stage -->
        <div id="gameStage" class="flex-1 overflow-auto flex items-center justify-center p-2 sm:p-6 relative">
            <!-- Native Game Containers will be dynamically mounted here -->
        </div>
    </div>

    <footer class="border-t border-slate-800 py-4 px-6 text-center text-xs text-slate-500 glass">
        <p>© 2026 Arcade Hub. Built with pure JavaScript without external iFrames.</p>
    </footer>

    <script>
        // --- GAME CATALOG DEFINITION ---
        const GAMES = [
            {
                id: 'game2048',
                title: '2048 Puzzle',
                category: 'puzzle',
                categoryName: 'Puzzle',
                icon: 'fa-cubes',
                badge: 'Classic',
                color: 'from-amber-500/20 to-orange-600/20',
                border: 'border-amber-500/40',
                desc: 'Merge matching numbers to reach the legendary 2048 tile!'
            },
            {
                id: 'gameRhythm',
                title: 'Rhythm Piano',
                category: 'rhythm',
                categoryName: 'Rhythm',
                icon: 'fa-music',
                badge: 'Arcade',
                color: 'from-cyan-500/20 to-blue-600/20',
                border: 'border-cyan-500/40',
                desc: 'Tap key notes [A, S, D, F] in sync with falling tiles for huge combos!'
            },
            {
                id: 'gameSolitaire',
                title: 'Spider Solitaire',
                category: 'card',
                categoryName: 'Card',
                icon: 'fa-spade',
                badge: 'Popular',
                color: 'from-emerald-500/20 to-teal-600/20',
                border: 'border-emerald-500/40',
                desc: 'Arrange cards from King down to Ace to clear the board.'
            },
            {
                id: 'gameBubble',
                title: 'Bubble Shooter',
                category: 'casual',
                categoryName: 'Casual',
                icon: 'fa-circle-dot',
                badge: 'Addictive',
                color: 'from-pink-500/20 to-rose-600/20',
                border: 'border-pink-500/40',
                desc: 'Aim and pop groups of 3 or more matching bubbles!'
            },
            {
                id: 'gameBlock',
                title: 'Block Blast',
                category: 'puzzle',
                categoryName: 'Puzzle',
                icon: 'fa-shapes',
                badge: 'Trending',
                color: 'from-indigo-500/20 to-violet-600/20',
                border: 'border-indigo-500/40',
                desc: 'Place shapes on the 8x8 grid and blast full rows or columns.'
            },
            {
                id: 'gameChess',
                title: 'Chess vs AI',
                category: 'board',
                categoryName: 'Board',
                icon: 'fa-chess-king',
                badge: 'Strategy',
                color: 'from-purple-500/20 to-slate-700/20',
                border: 'border-purple-500/40',
                desc: 'Outsmart the AI opponent in classic tactical chess.'
            }
        ];

        const CATEGORIES = [
            { id: 'all', name: 'All Games', icon: 'fa-border-all' },
            { id: 'puzzle', name: 'Puzzle', icon: 'fa-cubes' },
            { id: 'rhythm', name: 'Rhythm', icon: 'fa-music' },
            { id: 'card', name: 'Card', icon: 'fa-spade' },
            { id: 'casual', name: 'Casual', icon: 'fa-gamepad' },
            { id: 'board', name: 'Board', icon: 'fa-chess' }
        ];

        // --- GLOBAL STATE ---
        let currentCategory = 'all';
        let searchQuery = '';
        let favoritesOnly = false;
        let favorites = JSON.parse(localStorage.getItem('hub_favs') || '[]');
        let highScores = JSON.parse(localStorage.getItem('hub_scores') || '{}');
        let activeGameId = null;
        let activeCleanup = null; // Cleanup function when switching games

        // Initialize App
        window.addEventListener('DOMContentLoaded', () => {
            renderCategories();
            renderGrid();
            setupSearchListeners();
            updateFavCount();
        });

        function setupSearchListeners() {
            const handleSearch = (e) => {
                searchQuery = e.target.value.toLowerCase().trim();
                renderGrid();
            };
            document.getElementById('searchInput').addEventListener('input', handleSearch);
            document.getElementById('searchInputMobile').addEventListener('input', handleSearch);
        }

        function renderCategories() {
            const container = document.getElementById('categoryNav');
            container.innerHTML = CATEGORIES.map(cat => {
                const isActive = currentCategory === cat.id && !favoritesOnly;
                return `
                    <button onclick="setCategory('${cat.id}')" class="px-3.5 py-2 rounded-xl text-xs sm:text-sm font-semibold flex items-center gap-2 whitespace-nowrap transition-all ${
                        isActive 
                        ? 'bg-indigo-600 text-white shadow-lg shadow-indigo-600/30' 
                        : 'bg-slate-800/80 text-slate-400 hover:bg-slate-700 hover:text-white border border-slate-700/50'
                    }">
                        <i class="fa-solid ${cat.icon}"></i> ${cat.name}
                    </button>
                `;
            }).join('');
        }

        function setCategory(id) {
            currentCategory = id;
            favoritesOnly = false;
            updateFavFilterBtn();
            renderCategories();
            renderGrid();
        }

        function toggleFavoritesFilter() {
            favoritesOnly = !favoritesOnly;
            if (favoritesOnly) currentCategory = 'all';
            updateFavFilterBtn();
            renderCategories();
            renderGrid();
        }

        function updateFavFilterBtn() {
            const btn = document.getElementById('favFilterBtn');
            if (favoritesOnly) {
                btn.classList.add('bg-amber-500/20', 'border-amber-500/50', 'text-amber-300');
            } else {
                btn.classList.remove('bg-amber-500/20', 'border-amber-500/50', 'text-amber-300');
            }
        }

        function toggleFavorite(e, id) {
            e.stopPropagation();
            if (favorites.includes(id)) {
                favorites = favorites.filter(f => f !== id);
            } else {
                favorites.push(id);
            }
            localStorage.setItem('hub_favs', JSON.stringify(favorites));
            updateFavCount();
            renderGrid();
        }

        function updateFavCount() {
            document.getElementById('favCount').innerText = favorites.length;
        }

        function resetFilters() {
            currentCategory = 'all';
            searchQuery = '';
            favoritesOnly = false;
            document.getElementById('searchInput').value = '';
            document.getElementById('searchInputMobile').value = '';
            updateFavFilterBtn();
            renderCategories();
            renderGrid();
        }

        function renderGrid() {
            const grid = document.getElementById('gamesGrid');
            const empty = document.getElementById('emptyState');

            const filtered = GAMES.filter(game => {
                const matchCat = currentCategory === 'all' || game.category === currentCategory;
                const matchSearch = game.title.toLowerCase().includes(searchQuery) || game.desc.toLowerCase().includes(searchQuery);
                const matchFav = !favoritesOnly || favorites.includes(game.id);
                return matchCat && matchSearch && matchFav;
            });

            if (filtered.length === 0) {
                grid.innerHTML = '';
                empty.classList.remove('hidden');
                empty.classList.add('flex');
                return;
            }

            empty.classList.add('hidden');
            empty.classList.remove('flex');

            grid.innerHTML = filtered.map(game => {
                const isFav = favorites.includes(game.id);
                const topScore = highScores[game.id] || 0;

                return `
                    <div onclick="launchGame('${game.id}')" class="game-card glass rounded-2xl p-5 cursor-pointer relative flex flex-col justify-between border border-slate-800 transition-all duration-200">
                        <div>
                            <div class="flex items-start justify-between mb-4">
                                <div class="w-12 h-12 rounded-xl bg-gradient-to-br ${game.color} border ${game.border} flex items-center justify-center text-indigo-300 text-xl">
                                    <i class="fa-solid ${game.icon}"></i>
                                </div>
                                <div class="flex items-center gap-2">
                                    <span class="text-[11px] font-bold px-2.5 py-0.5 rounded-full border bg-slate-800 text-slate-300 border-slate-700">
                                        ${game.badge}
                                    </span>
                                    <button onclick="toggleFavorite(event, '${game.id}')" class="p-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-400">
                                        <i class="fa-${isFav ? 'solid' : 'regular'} fa-star ${isFav ? 'text-amber-400' : ''}"></i>
                                    </button>
                                </div>
                            </div>
                            <h3 class="font-bold text-lg text-white mb-1">${game.title}</h3>
                            <p class="text-xs text-slate-400 leading-relaxed mb-4">${game.desc}</p>
                        </div>
                        <div class="pt-3 border-t border-slate-800/80 flex items-center justify-between text-xs">
                            <span class="text-slate-500 font-medium">Top Score: <strong class="text-amber-400">${topScore}</strong></span>
                            <span class="text-indigo-400 font-bold flex items-center gap-1">PLAY <i class="fa-solid fa-play text-[10px]"></i></span>
                        </div>
                    </div>
                `;
            }).join('');
        }

        // --- OVERLAY & GAME LAUNCHER ---
        function launchGame(id) {
            const game = GAMES.find(g => g.id === id);
            if (!game) return;

            if (activeCleanup) {
                activeCleanup();
                activeCleanup = null;
            }

            activeGameId = id;
            document.getElementById('overlayGameTitle').innerText = game.title;
            document.getElementById('overlayGameIcon').className = `fa-solid ${game.icon} text-indigo-400 text-lg`;
            document.getElementById('overlayHighScore').innerText = highScores[id] || 0;

            const stage = document.getElementById('gameStage');
            stage.innerHTML = ''; // Clear prior game DOM

            const overlay = document.getElementById('gameOverlay');
            overlay.classList.remove('hidden');
            overlay.classList.add('flex');

            // Route to game initializer
            switch(id) {
                case 'game2048': activeCleanup = init2048(stage); break;
                case 'gameRhythm': activeCleanup = initRhythm(stage); break;
                case 'gameSolitaire': activeCleanup = initSolitaire(stage); break;
                case 'gameBubble': activeCleanup = initBubbleShooter(stage); break;
                case 'gameBlock': activeCleanup = initBlockBlast(stage); break;
                case 'gameChess': activeCleanup = initChess(stage); break;
            }
        }

        function saveHighScore(gameId, score) {
            if (!highScores[gameId] || score > highScores[gameId]) {
                highScores[gameId] = score;
                localStorage.setItem('hub_scores', JSON.stringify(highScores));
                document.getElementById('overlayHighScore').innerText = score;
                renderGrid();
            }
        }

        function closeGameOverlay() {
            if (activeCleanup) {
                activeCleanup();
                activeCleanup = null;
            }
            activeGameId = null;
            const overlay = document.getElementById('gameOverlay');
            overlay.classList.add('hidden');
            overlay.classList.remove('flex');
            document.getElementById('gameStage').innerHTML = '';
        }

        function restartCurrentGame() {
            if (activeGameId) launchGame(activeGameId);
        }

        function init2048(container) {
            let board = Array(16).fill(0);
            let score = 0;

            container.innerHTML = `
                <div class="flex flex-col items-center gap-4 max-w-sm w-full">
                    <div class="flex items-center justify-between w-full px-2">
                        <div class="text-sm font-semibold text-slate-400">Score: <span id="s2048" class="text-xl font-bold text-amber-400">0</span></div>
                        <div class="text-xs text-slate-500">Use Arrow Keys or Swipe</div>
                    </div>
                    <div id="grid2048" class="grid grid-cols-4 gap-3 bg-slate-900 p-3 rounded-2xl border border-slate-800 w-full aspect-square relative">
                        ${Array(16).fill(0).map((_, i) => `<div class="bg-slate-800/60 rounded-xl flex items-center justify-center font-extrabold text-xl sm:text-2xl transition-all duration-100" id="tile2048-${i}"></div>`).join('')}
                    </div>
                </div>
            `;

            function spawn() {
                let empty = board.map((v, i) => v === 0 ? i : null).filter(v => v !== null);
                if (empty.length > 0) {
                    let idx = empty[Math.floor(Math.random() * empty.length)];
                    board[idx] = Math.random() < 0.9 ? 2 : 4;
                }
            }

            function updateUI() {
                const colors = {
                    0: 'bg-slate-800/50 text-transparent',
                    2: 'bg-amber-100 text-slate-800',
                    4: 'bg-amber-200 text-slate-800',
                    8: 'bg-orange-400 text-white',
                    16: 'bg-orange-500 text-white',
                    32: 'bg-rose-500 text-white',
                    64: 'bg-rose-600 text-white',
                    128: 'bg-yellow-400 text-slate-900',
                    256: 'bg-yellow-500 text-slate-900',
                    512: 'bg-amber-400 text-slate-900',
                    1024: 'bg-indigo-500 text-white',
                    2048: 'bg-purple-600 text-white'
                };

                for(let i=0; i<16; i++) {
                    const el = document.getElementById(`tile2048-${i}`);
                    const val = board[i];
                    el.innerText = val > 0 ? val : '';
                    el.className = `rounded-xl flex items-center justify-center font-extrabold text-xl sm:text-2xl transition-all duration-100 ${colors[val] || 'bg-indigo-600 text-white'}`;
                }
                document.getElementById('s2048').innerText = score;
                saveHighScore('game2048', score);
            }

            function slide(row) {
                let arr = row.filter(val => val);
                let missing = 4 - arr.length;
                let zeros = Array(missing).fill(0);
                return arr.concat(zeros);
            }

            function combine(row) {
                for (let i = 0; i < 3; i++) {
                    if (row[i] !== 0 && row[i] === row[i + 1]) {
                        row[i] *= 2;
                        score += row[i];
                        row[i + 1] = 0;
                    }
                }
                return row;
            }

            function moveLeft() {
                let changed = false;
                for (let i = 0; i < 4; i++) {
                    let row = [board[i*4], board[i*4+1], board[i*4+2], board[i*4+3]];
                    let newRow = slide(combine(slide(row)));
                    for (let j = 0; j < 4; j++) {
                        if (board[i*4+j] !== newRow[j]) changed = true;
                        board[i*4+j] = newRow[j];
                    }
                }
                return changed;
            }

            function rotate() {
                let newBoard = Array(16).fill(0);
                for (let r = 0; r < 4; r++) {
                    for (let c = 0; c < 4; c++) {
                        newBoard[c * 4 + (3 - r)] = board[r * 4 + c];
                    }
                }
                board = newBoard;
            }

            function handleKey(e) {
                let moved = false;
                if (e.key === 'ArrowLeft') { moved = moveLeft(); }
                else if (e.key === 'ArrowRight') { rotate(); rotate(); moved = moveLeft(); rotate(); rotate(); }
                else if (e.key === 'ArrowUp') { rotate(); rotate(); rotate(); moved = moveLeft(); rotate(); }
                else if (e.key === 'ArrowDown') { rotate(); moved = moveLeft(); rotate(); rotate(); rotate(); }

                if (moved) {
                    spawn();
                    updateUI();
                }
            }

            // Mobile Touch Support
            let startX, startY;
            const touchStart = (e) => {
                startX = e.touches[0].clientX;
                startY = e.touches[0].clientY;
            };
            const touchEnd = (e) => {
                if(!startX || !startY) return;
                let diffX = e.changedTouches[0].clientX - startX;
                let diffY = e.changedTouches[0].clientY - startY;
                let moved = false;

                if (Math.abs(diffX) > Math.abs(diffY)) {
                    if (diffX > 30) { rotate(); rotate(); moved = moveLeft(); rotate(); rotate(); }
                    else if (diffX < -30) { moved = moveLeft(); }
                } else {
                    if (diffY > 30) { rotate(); moved = moveLeft(); rotate(); rotate(); rotate(); }
                    else if (diffY < -30) { rotate(); rotate(); rotate(); moved = moveLeft(); rotate(); }
                }
                if (moved) { spawn(); updateUI(); }
            };

            window.addEventListener('keydown', handleKey);
            const gridEl = document.getElementById('grid2048');
            gridEl.addEventListener('touchstart', touchStart);
            gridEl.addEventListener('touchend', touchEnd);

            spawn(); spawn(); updateUI();

            return () => {
                window.removeEventListener('keydown', handleKey);
                if(gridEl) {
                    gridEl.removeEventListener('touchstart', touchStart);
                    gridEl.removeEventListener('touchend', touchEnd);
                }
            };
        }

        function initRhythm(container) {
            let score = 0, combo = 0;
            let animId = null;
            let notes = [];
            const lanes = ['A', 'S', 'D', 'F'];

            container.innerHTML = `
                <div class="flex flex-col items-center gap-3 w-full max-w-sm">
                    <div class="flex justify-between w-full px-4 text-sm font-semibold">
                        <div>Score: <span id="rhythmScore" class="text-cyan-400 text-lg font-bold">0</span></div>
                        <div>Combo: <span id="rhythmCombo" class="text-amber-400 text-lg font-bold">0</span></div>
                    </div>
                    <div id="rhythmStage" class="w-full h-[400px] bg-slate-900 rounded-2xl border border-slate-800 relative overflow-hidden flex">
                        <div class="absolute bottom-12 w-full h-10 border-t-2 border-b-2 border-cyan-400/80 bg-cyan-500/10 pointer-events-none"></div>
                        ${lanes.map((key, i) => `
                            <div class="flex-1 border-r border-slate-800/60 relative flex flex-col justify-end items-center pb-2">
                                <button onclick="triggerHit(${i})" class="w-12 h-10 rounded-lg bg-slate-800 active:bg-cyan-500 text-slate-300 font-bold text-sm border border-slate-700">${key}</button>
                            </div>
                        `).join('')}
                    </div>
                </div>
            `;

            const stage = document.getElementById('rhythmStage');

            function spawnNote() {
                if (Math.random() < 0.05) {
                    const lane = Math.floor(Math.random() * 4);
                    const noteEl = document.createElement('div');
                    noteEl.className = 'absolute w-10 h-6 bg-gradient-to-r from-cyan-400 to-blue-500 rounded-md shadow-md shadow-cyan-500/50';
                    noteEl.style.left = `${lane * 25 + 3}%`;
                    noteEl.style.top = '0px';
                    stage.appendChild(noteEl);

                    notes.push({ lane, y: 0, el: noteEl });
                }
            }

            function gameLoop() {
                spawnNote();
                for (let i = notes.length - 1; i >= 0; i--) {
                    let n = notes[i];
                    n.y += 4;
                    n.el.style.top = n.y + 'px';

                    if (n.y > 380) {
                        n.el.remove();
                        notes.splice(i, 1);
                        combo = 0;
                        document.getElementById('rhythmCombo').innerText = combo;
                    }
                }
                animId = requestAnimationFrame(gameLoop);
            }

            window.triggerHit = function(laneIdx) {
                let hit = false;
                for (let i = 0; i < notes.length; i++) {
                    let n = notes[i];
                    if (n.lane === laneIdx && n.y >= 300 && n.y <= 360) {
                        hit = true;
                        n.el.remove();
                        notes.splice(i, 1);
                        score += 10 + combo * 2;
                        combo += 1;
                        document.getElementById('rhythmScore').innerText = score;
                        document.getElementById('rhythmCombo').innerText = combo;
                        saveHighScore('gameRhythm', score);
                        break;
                    }
                }
                if (!hit && combo > 0) {
                    combo = 0;
                    document.getElementById('rhythmCombo').innerText = combo;
                }
            };

            function handleKey(e) {
                const k = e.key.toUpperCase();
                const idx = lanes.indexOf(k);
                if (idx !== -1) window.triggerHit(idx);
            }

            window.addEventListener('keydown', handleKey);
            animId = requestAnimationFrame(gameLoop);

            return () => {
                cancelAnimationFrame(animId);
                window.removeEventListener('keydown', handleKey);
                delete window.triggerHit;
            };
        }

        function initSolitaire(container) {
            let tableau = Array(10).fill(0).map(() => []);
            let stock = [];
            let score = 0;

            container.innerHTML = `
                <div class="flex flex-col items-center gap-3 w-full max-w-2xl">
                    <div class="flex justify-between w-full px-2 text-sm">
                        <div class="text-slate-400">Completed Sets: <span id="solScore" class="text-emerald-400 font-bold">0</span></div>
                        <button id="drawStockBtn" class="px-3 py-1 bg-emerald-600 hover:bg-emerald-500 rounded-lg text-xs font-bold text-white">Draw Deal</button>
                    </div>
                    <div id="solBoard" class="w-full bg-slate-900 border border-slate-800 rounded-2xl p-3 min-h-[420px] grid grid-cols-10 gap-1"></div>
                </div>
            `;

            // Build 1-suit deck (Spades: 1 to 13)
            let deck = [];
            for (let copy = 0; copy < 8; copy++) {
                for (let rank = 1; rank <= 13; rank++) deck.push(rank);
            }
            deck.sort(() => Math.random() - 0.5);

            // Deal initial tableau (54 cards)
            for (let i = 0; i < 54; i++) {
                let col = i % 10;
                let card = { rank: deck.pop(), faceUp: false };
                tableau[col].push(card);
            }
            // Flip top cards
            tableau.forEach(col => { if(col.length) col[col.length - 1].faceUp = true; });
            stock = deck; // Remaining 50 cards

            function render() {
                const boardEl = document.getElementById('solBoard');
                boardEl.innerHTML = '';

                tableau.forEach((col, colIdx) => {
                    const colEl = document.createElement('div');
                    colEl.className = 'relative flex flex-col items-center min-h-[300px]';
                    colEl.onclick = () => onColClick(colIdx);

                    col.forEach((card, cardIdx) => {
                        const cardEl = document.createElement('div');
                        cardEl.className = `solitaire-card absolute flex flex-col items-center justify-between p-1 font-bold text-xs ${card.faceUp ? 'bg-slate-100 text-slate-900 border border-slate-300' : 'bg-indigo-950 border border-indigo-700 text-transparent'}`;
                        cardEl.style.top = `${cardIdx * 18}px`;

                        if (card.faceUp) {
                            const valStr = card.rank === 1 ? 'A' : card.rank === 11 ? 'J' : card.rank === 12 ? 'Q' : card.rank === 13 ? 'K' : card.rank;
                            cardEl.innerHTML = `<span>${valStr}</span><span class="text-base">♠</span>`;
                        }

                        colEl.appendChild(cardEl);
                    });
                    boardEl.appendChild(colEl);
                });

                document.getElementById('solScore').innerText = score;
                saveHighScore('gameSolitaire', score * 100);
            }

            let selectedCol = null;

            function onColClick(colIdx) {
                if (selectedCol === null) {
                    if (tableau[colIdx].length > 0) selectedCol = colIdx;
                } else {
                    if (selectedCol !== colIdx) {
                        let src = tableau[selectedCol];
                        let dest = tableau[colIdx];

                        if (src.length > 0) {
                            let cardToMove = src[src.length - 1];
                            if (dest.length === 0 || dest[dest.length - 1].rank === cardToMove.rank + 1) {
                                dest.push(src.pop());
                                if (src.length > 0) src[src.length - 1].faceUp = true;
                                checkSequences(colIdx);
                            }
                        }
                    }
                    selectedCol = null;
                }
                render();
            }

            function checkSequences(colIdx) {
                let col = tableau[colIdx];
                if (col.length < 13) return;

                let isSeq = true;
                for (let i = 0; i < 13; i++) {
                    let card = col[col.length - 1 - i];
                    if (!card || !card.faceUp || card.rank !== i + 1) {
                        isSeq = false;
                        break;
                    }
                }

                if (isSeq) {
                    col.splice(col.length - 13, 13);
                    if (col.length > 0) col[col.length - 1].faceUp = true;
                    score++;
                }
            }

            document.getElementById('drawStockBtn').onclick = () => {
                if (stock.length >= 10) {
                    for (let i = 0; i < 10; i++) {
                        tableau[i].push({ rank: stock.pop(), faceUp: true });
                        checkSequences(i);
                    }
                    render();
                }
            };

            render();
            return () => {};
        }

        function initBubbleShooter(container) {
            container.innerHTML = `
                <div class="flex flex-col items-center gap-3">
                    <div class="text-sm font-semibold text-slate-400">Score: <span id="bubbleScore" class="text-pink-400 font-bold text-lg">0</span></div>
                    <canvas id="bubbleCanvas" width="360" height="480" class="bg-slate-900 border border-slate-800 rounded-2xl shadow-xl cursor-crosshair"></canvas>
                </div>
            `;

            const canvas = document.getElementById('bubbleCanvas');
            const ctx = canvas.getContext('2d');
            const colors = ['#f43f5e', '#ec4899', '#a855f7', '#3b82f6', '#10b981'];

            let score = 0;
            let rows = 6, cols = 8, radius = 22;
            let grid = [];

            for (let r = 0; r < rows; r++) {
                grid[r] = [];
                for (let c = 0; c < cols; c++) {
                    grid[r][c] = Math.floor(Math.random() * colors.length);
                }
            }

            let shooter = {
                x: canvas.width / 2,
                y: canvas.height - 30,
                colorIdx: Math.floor(Math.random() * colors.length),
                angle: -Math.PI / 2,
                bullet: null
            };

            function draw() {
                ctx.clearRect(0, 0, canvas.width, canvas.height);

                // Draw Grid
                for (let r = 0; r < grid.length; r++) {
                    for (let c = 0; c < cols; c++) {
                        if (grid[r][c] !== null) {
                            let x = c * radius * 2 + radius + (r % 2 === 1 ? radius : 0);
                            let y = r * radius * 2 + radius;
                            ctx.beginPath();
                            ctx.arc(x, y, radius - 2, 0, Math.PI * 2);
                            ctx.fillStyle = colors[grid[r][c]];
                            ctx.fill();
                        }
                    }
                }

                // Draw Shooter
                ctx.beginPath();
                ctx.arc(shooter.x, shooter.y, radius - 2, 0, Math.PI * 2);
                ctx.fillStyle = colors[shooter.colorIdx];
                ctx.fill();

                // Draw Bullet
                if (shooter.bullet) {
                    ctx.beginPath();
                    ctx.arc(shooter.bullet.x, shooter.bullet.y, radius - 2, 0, Math.PI * 2);
                    ctx.fillStyle = colors[shooter.bullet.colorIdx];
                    ctx.fill();

                    shooter.bullet.x += Math.cos(shooter.bullet.angle) * 10;
                    shooter.bullet.y += Math.sin(shooter.bullet.angle) * 10;

                    // Bounce Wall
                    if (shooter.bullet.x <= radius || shooter.bullet.x >= canvas.width - radius) {
                        shooter.bullet.angle = Math.PI - shooter.bullet.angle;
                    }

                    // Collision Top or Bubble
                    if (shooter.bullet.y <= radius || checkCollision()) {
                        snapBubble();
                    }
                }
                requestAnimationFrame(draw);
            }

            function checkCollision() {
                if (!shooter.bullet) return false;
                for (let r = 0; r < grid.length; r++) {
                    for (let c = 0; c < cols; c++) {
                        if (grid[r][c] !== null) {
                            let x = c * radius * 2 + radius + (r % 2 === 1 ? radius : 0);
                            let y = r * radius * 2 + radius;
                            let dist = Math.hypot(shooter.bullet.x - x, shooter.bullet.y - y);
                            if (dist < radius * 1.8) return true;
                        }
                    }
                }
                return false;
            }

            function snapBubble() {
                let r = Math.floor(shooter.bullet.y / (radius * 2));
                let c = Math.floor(shooter.bullet.x / (radius * 2));
                r = Math.max(0, Math.min(r, 10));
                c = Math.max(0, Math.min(c, cols - 1));

                if (!grid[r]) grid[r] = Array(cols).fill(null);
                grid[r][c] = shooter.bullet.colorIdx;

                score += 20;
                document.getElementById('bubbleScore').innerText = score;
                saveHighScore('gameBubble', score);

                shooter.bullet = null;
                shooter.colorIdx = Math.floor(Math.random() * colors.length);
            }

            function handlePointer(e) {
                const rect = canvas.getBoundingClientRect();
                const x = e.clientX - rect.left;
                const y = e.clientY - rect.top;
                shooter.angle = Math.atan2(y - shooter.y, x - shooter.x);

                if (!shooter.bullet) {
                    shooter.bullet = { x: shooter.x, y: shooter.y, angle: shooter.angle, colorIdx: shooter.colorIdx };
                }
            }

            canvas.addEventListener('click', handlePointer);
            draw();

            return () => {
                canvas.removeEventListener('click', handlePointer);
            };
        }

        function initBlockBlast(container) {
            let grid = Array(8).fill(0).map(() => Array(8).fill(0));
            let score = 0;

            container.innerHTML = `
                <div class="flex flex-col items-center gap-4 w-full max-w-sm">
                    <div class="text-sm font-semibold text-slate-400">Score: <span id="blockScore" class="text-indigo-400 font-bold text-lg">0</span></div>
                    <div id="blockBoard" class="grid grid-cols-8 gap-1.5 bg-slate-900 border border-slate-800 p-3 rounded-2xl w-full aspect-square">
                        ${Array(64).fill(0).map((_, i) => `<div class="bg-slate-800/80 rounded-lg cursor-pointer hover:bg-slate-700 transition" id="bcell-${i}"></div>`).join('')}
                    </div>
                    <div class="text-xs text-slate-500">Click empty grid tiles to place blocks & clear rows</div>
                </div>
            `;

            function updateUI() {
                for (let r = 0; r < 8; r++) {
                    for (let c = 0; c < 8; c++) {
                        const cell = document.getElementById(`bcell-${r * 8 + c}`);
                        if (grid[r][c] === 1) {
                            cell.className = 'bg-gradient-to-tr from-indigo-500 to-purple-500 rounded-lg shadow-md';
                        } else {
                            cell.className = 'bg-slate-800/80 rounded-lg cursor-pointer hover:bg-slate-700 transition';
                        }
                    }
                }
                document.getElementById('blockScore').innerText = score;
                saveHighScore('gameBlock', score);
            }

            function clearLines() {
                let fullRows = [];
                let fullCols = [];

                for (let r = 0; r < 8; r++) {
                    if (grid[r].every(v => v === 1)) fullRows.push(r);
                }
                for (let c = 0; c < 8; c++) {
                    let full = true;
                    for (let r = 0; r < 8; r++) {
                        if (grid[r][c] === 0) full = false;
                    }
                    if (full) fullCols.push(c);
                }

                fullRows.forEach(r => grid[r].fill(0));
                fullCols.forEach(c => {
                    for (let r = 0; r < 8; r++) grid[r][c] = 0;
                });

                score += (fullRows.length + fullCols.length) * 100;
            }

            for (let i = 0; i < 64; i++) {
                document.getElementById(`bcell-${i}`).onclick = () => {
                    let r = Math.floor(i / 8);
                    let c = i % 8;
                    if (grid[r][c] === 0) {
                        grid[r][c] = 1;
                        score += 10;
                        clearLines();
                        updateUI();
                    }
                };
            }

            updateUI();
            return () => {};
        }

        function initChess(container) {
            let board = [
                ['r','n','b','q','k','b','n','r'],
                ['p','p','p','p','p','p','p','p'],
                ['','','','','','','',''],
                ['','','','','','','',''],
                ['','','','','','','',''],
                ['','','','','','','',''],
                ['P','P','P','P','P','P','P','P'],
                ['R','N','B','Q','K','B','N','R']
            ];

            const symbols = {
                'k':'♚','q':'♛','r':'♜','b':'♝','n':'♞','p':'♟',
                'K':'♔','Q':'♕','R':'♖','B':'♗','N':'♘','P':'♙'
            };

            let selected = null;
            let score = 0;

            container.innerHTML = `
                <div class="flex flex-col items-center gap-3 w-full max-w-sm">
                    <div class="text-sm font-semibold text-slate-400">Play as White vs AI</div>
                    <div id="chessBoard" class="grid grid-cols-8 gap-0 border-2 border-slate-700 rounded-xl overflow-hidden w-full aspect-square shadow-2xl"></div>
                </div>
            `;

            function render() {
                const el = document.getElementById('chessBoard');
                el.innerHTML = '';

                for (let r = 0; r < 8; r++) {
                    for (let c = 0; c < 8; c++) {
                        const isLight = (r + c) % 2 === 0;
                        const p = board[r][c];
                        const cell = document.createElement('div');
                        cell.className = `flex items-center justify-center font-bold text-2xl sm:text-3xl cursor-pointer ${isLight ? 'bg-amber-100 text-slate-900' : 'bg-amber-800 text-amber-50'} ${selected && selected.r === r && selected.c === c ? 'ring-4 ring-indigo-500' : ''}`;
                        cell.innerText = symbols[p] || '';
                        cell.onclick = () => handleClick(r, c);
                        el.appendChild(cell);
                    }
                }
            }

            function handleClick(r, c) {
                const p = board[r][c];
                if (selected) {
                    // Move piece
                    if (selected.r !== r || selected.c !== c) {
                        board[r][c] = board[selected.r][selected.c];
                        board[selected.r][selected.c] = '';
                        selected = null;
                        render();
                        setTimeout(aiMove, 300);
                        return;
                    }
                    selected = null;
                } else if (p && p === p.toUpperCase()) {
                    selected = { r, c };
                }
                render();
            }

            function aiMove() {
                let moves = [];
                for (let r = 0; r < 8; r++) {
                    for (let c = 0; c < 8; c++) {
                        let p = board[r][c];
                        if (p && p === p.toLowerCase()) {
                            // Find simple forward move for AI
                            if (r + 1 < 8 && board[r + 1][c] === '') {
                                moves.push({ from: { r, c }, to: { r: r + 1, c } });
                            }
                        }
                    }
                }
                if (moves.length > 0) {
                    let m = moves[Math.floor(Math.random() * moves.length)];
                    board[m.to.r][m.to.c] = board[m.from.r][m.from.c];
                    board[m.from.r][m.from.c] = '';
                    score += 50;
                    saveHighScore('gameChess', score);
                }
                render();
            }

            render();
            return () => {};
        }
    </script>
</body>
</html>

<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>警察抓小偷遊戲</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            touch-action: none;
            overflow: hidden;
            background-color: #1a202c;
        }
        canvas {
            border: 4px solid #4a5568;
            border-radius: 8px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
            max-width: 100%;
            height: auto;
            background-color: #2d3748;
        }
    </style>
</head>
<body class="flex flex-col items-center justify-center min-h-screen">

    <div class="text-white text-center mb-4">
        <h1 class="text-3xl font-bold mb-2">警察抓小偷</h1>
        <div class="flex gap-4 justify-center text-lg">
            <p>得分: <span id="score">0</span></p>
            <p>剩餘時間: <span id="timer">30</span>s</p>
        </div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <div class="mt-6 text-gray-400 text-sm text-center px-4">
        <p>電腦：使用方向鍵 (↑↓←→) 控制警察</p>
        <p>手機：直接點擊螢幕區域移動警察</p>
    </div>

    <!-- 遊戲結束彈窗 -->
    <div id="gameOverModal" class="hidden fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center p-4">
        <div class="bg-white rounded-lg p-8 max-w-sm w-full text-center">
            <h2 class="text-2xl font-bold mb-4 text-gray-800">遊戲結束！</h2>
            <p class="text-xl mb-6 text-gray-600">最終得分：<span id="finalScore">0</span></p>
            <button onclick="resetGame()" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-full transition duration-200">
                再玩一次
            </button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreEl = document.getElementById('score');
        const timerEl = document.getElementById('timer');
        const modal = document.getElementById('gameOverModal');
        const finalScoreEl = document.getElementById('finalScore');

        // 遊戲設定
        let score = 0;
        let timeLeft = 30;
        let gameActive = true;
        let animationId;

        // 畫布尺寸
        function resizeCanvas() {
            canvas.width = Math.min(window.innerWidth - 40, 600);
            canvas.height = Math.min(window.innerHeight - 250, 400);
        }
        resizeCanvas();

        // 角色設定
        const player = {
            x: 50,
            y: 50,
            size: 30,
            speed: 5,
            color: '#3182ce', // 警察藍
            dx: 0,
            dy: 0
        };

        const thief = {
            x: 0,
            y: 0,
            size: 25,
            speed: 3,
            color: '#e53e3e', // 小偷紅
            angle: Math.random() * Math.PI * 2
        };

        function spawnThief() {
            thief.x = Math.random() * (canvas.width - thief.size);
            thief.y = Math.random() * (canvas.height - thief.size);
        }
        spawnThief();

        // 控制邏輯
        const keys = {};
        window.addEventListener('keydown', e => keys[e.code] = true);
        window.addEventListener('keyup', e => keys[e.code] = false);

        // 觸控控制
        canvas.addEventListener('touchstart', handleTouch);
        canvas.addEventListener('touchmove', handleTouch);
        function handleTouch(e) {
            e.preventDefault();
            const rect = canvas.getBoundingClientRect();
            const touchX = e.touches[0].clientX - rect.left;
            const touchY = e.touches[0].clientY - rect.top;
            
            // 朝向點擊位置移動
            const angle = Math.atan2(touchY - player.y, touchX - player.x);
            player.dx = Math.cos(angle) * player.speed;
            player.dy = Math.sin(angle) * player.speed;
        }

        canvas.addEventListener('touchend', () => {
            player.dx = 0;
            player.dy = 0;
        });

        function update() {
            if (!gameActive) return;

            // 鍵盤移動
            if (keys['ArrowUp']) player.y -= player.speed;
            if (keys['ArrowDown']) player.y += player.speed;
            if (keys['ArrowLeft']) player.x -= player.speed;
            if (keys['ArrowRight']) player.x += player.speed;

            // 觸控慣性移動
            if (player.dx !== 0 || player.dy !== 0) {
                player.x += player.dx;
                player.y += player.dy;
            }

            // 邊界限制
            player.x = Math.max(0, Math.min(canvas.width - player.size, player.x));
            player.y = Math.max(0, Math.min(canvas.height - player.size, player.y));

            // 小偷 AI (簡單的隨機遊走與邊界碰撞)
            thief.x += Math.cos(thief.angle) * thief.speed;
            thief.y += Math.sin(thief.angle) * thief.speed;

            if (thief.x <= 0 || thief.x >= canvas.width - thief.size) {
                thief.angle = Math.PI - thief.angle;
            }
            if (thief.y <= 0 || thief.y >= canvas.height - thief.size) {
                thief.angle = -thief.angle;
            }

            // 碰撞偵測
            if (
                player.x < thief.x + thief.size &&
                player.x + player.size > thief.x &&
                player.y < thief.y + thief.size &&
                player.y + player.size > thief.y
            ) {
                score++;
                scoreEl.textContent = score;
                spawnThief();
                // 增加一點難度
                thief.speed += 0.2;
            }
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 繪製小偷 (加上一點裝飾)
            ctx.fillStyle = thief.color;
            ctx.beginPath();
            ctx.roundRect(thief.x, thief.y, thief.size, thief.size, 5);
            ctx.fill();
            // 小偷的眼睛
            ctx.fillStyle = 'white';
            ctx.fillRect(thief.x + 5, thief.y + 5, 5, 5);
            ctx.fillRect(thief.x + 15, thief.y + 5, 5, 5);

            // 繪製警察
            ctx.fillStyle = player.color;
            ctx.beginPath();
            ctx.roundRect(player.x, player.y, player.size, player.size, 5);
            ctx.fill();
            // 警徽裝飾
            ctx.fillStyle = 'gold';
            ctx.beginPath();
            ctx.arc(player.x + player.size/2, player.y + player.size/2, 5, 0, Math.PI*2);
            ctx.fill();

            if (gameActive) {
                animationId = requestAnimationFrame(() => {
                    update();
                    draw();
                });
            }
        }

        // 計時器
        const timerInterval = setInterval(() => {
            if (gameActive) {
                timeLeft--;
                timerEl.textContent = timeLeft;
                if (timeLeft <= 0) {
                    endGame();
                }
            }
        }, 1000);

        function endGame() {
            gameActive = false;
            finalScoreEl.textContent = score;
            modal.classList.remove('hidden');
            cancelAnimationFrame(animationId);
        }

        function resetGame() {
            score = 0;
            timeLeft = 30;
            gameActive = true;
            thief.speed = 3;
            player.x = 50;
            player.y = 50;
            player.dx = 0;
            player.dy = 0;
            scoreEl.textContent = score;
            timerEl.textContent = timeLeft;
            modal.classList.add('hidden');
            spawnThief();
            draw();
        }

        window.onload = draw;
        window.addEventListener('resize', resizeCanvas);
    </script>
</body>
</html>

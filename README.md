[小游戏1.html](https://github.com/user-attachments/files/22586783/1.html)
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>阿天的星际探险家</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);
            font-family: 'Arial Rounded MT Bold', 'Arial', sans-serif;
            overflow: hidden;
            color: white;
        }
        
        #gameContainer {
            position: relative;
            width: 800px;
            height: 600px;
            box-shadow: 0 0 20px rgba(0, 255, 255, 0.5);
            border-radius: 10px;
            overflow: hidden;
        }
        
        #gameCanvas {
            background: #000;
            display: block;
        }
        
        #ui {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 18px;
            z-index: 10;
        }
        
        #startScreen, #gameOverScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: rgba(0, 0, 0, 0.7);
            z-index: 20;
        }
        
        #gameOverScreen {
            display: none;
        }
        
        h1 {
            font-size: 48px;
            margin-bottom: 20px;
            color: #4fc3f7;
            text-shadow: 0 0 10px rgba(79, 195, 247, 0.7);
        }
        
        h2 {
            font-size: 36px;
            margin-bottom: 30px;
            color: #ff4081;
        }
        
        button {
            padding: 12px 30px;
            font-size: 20px;
            background: linear-gradient(45deg, #ff4081, #7c4dff);
            border: none;
            border-radius: 30px;
            color: white;
            cursor: pointer;
            transition: all 0.3s;
            box-shadow: 0 0 15px rgba(124, 77, 255, 0.5);
        }
        
        button:hover {
            transform: scale(1.05);
            box-shadow: 0 0 20px rgba(124, 77, 255, 0.8);
        }
        
        .instructions {
            margin-top: 20px;
            text-align: center;
            max-width: 80%;
            line-height: 1.5;
        }
        
        .highlight {
            color: #4fc3f7;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="gameCanvas" width="800" height="600"></canvas>
        
        <div id="ui">
            <div>得分: <span id="score">0</span></div>
            <div>生命: <span id="lives">3</span></div>
        </div>
        
        <div id="startScreen">
            <h1>阿天的星际探险家</h1>
            <button id="startButton">开始游戏</button>
            <div class="instructions">
                <p>使用 <span class="highlight">← →</span> 键控制飞船左右移动</p>
                <p>避开陨石，收集 <span class="highlight">蓝色能量晶体</span></p>
                <p>每收集一个晶体得10分，碰到陨石会损失生命值</p>
            </div>
        </div>
        
        <div id="gameOverScreen">
            <h2>游戏结束</h2>
            <p>最终得分: <span id="finalScore">0</span></p>
            <button id="restartButton">再玩一次</button>
        </div>
    </div>

    <script>
        // 获取游戏元素
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');
        const scoreElement = document.getElementById('score');
        const finalScoreElement = document.getElementById('finalScore');
        const livesElement = document.getElementById('lives');

        // 游戏变量
        let gameActive = false;
        let score = 0;
        let lives = 3;
        
        // 玩家飞船
        const player = {
            x: canvas.width / 2 - 25,
            y: canvas.height - 80,
            width: 50,
            height: 70,
            speed: 7,
            color: '#4fc3f7'
        };
        
        // 陨石数组
        let asteroids = [];
        
        // 能量晶体数组
        let energyCrystals = [];
        
        // 键盘控制
        const keys = {
            ArrowLeft: false,
            ArrowRight: false
        };
        
        // 初始化游戏
        function initGame() {
            score = 0;
            lives = 3;
            asteroids = [];
            energyCrystals = [];
            player.x = canvas.width / 2 - 25;
            updateUI();
        }
        
        // 更新UI
        function updateUI() {
            scoreElement.textContent = score;
            livesElement.textContent = lives;
        }
        
        // 绘制玩家飞船
        function drawPlayer() {
            ctx.fillStyle = player.color;
            
            // 绘制飞船主体
            ctx.beginPath();
            ctx.moveTo(player.x + player.width / 2, player.y);
            ctx.lineTo(player.x + player.width, player.y + player.height);
            ctx.lineTo(player.x, player.y + player.height);
            ctx.closePath();
            ctx.fill();
            
            // 绘制飞船尾焰
            ctx.fillStyle = '#ff9800';
            ctx.beginPath();
            ctx.moveTo(player.x + player.width / 2 - 10, player.y + player.height);
            ctx.lineTo(player.x + player.width / 2 + 10, player.y + player.height);
            ctx.lineTo(player.x + player.width / 2, player.y + player.height + 20);
            ctx.closePath();
            ctx.fill();
        }
        
        // 创建陨石
        function createAsteroid() {
            const size = Math.random() * 30 + 20;
            asteroids.push({
                x: Math.random() * (canvas.width - size),
                y: -size,
                width: size,
                height: size,
                speed: Math.random() * 3 + 2,
                color: '#795548'
            });
        }
        
        // 创建能量晶体
        function createEnergyCrystal() {
            energyCrystals.push({
                x: Math.random() * (canvas.width - 20),
                y: -30,
                width: 20,
                height: 30,
                speed: Math.random() * 2 + 1,
                color: '#4fc3f7'
            });
        }
        
        // 绘制陨石
        function drawAsteroids() {
            asteroids.forEach(asteroid => {
                ctx.fillStyle = asteroid.color;
                ctx.beginPath();
                ctx.arc(
                    asteroid.x + asteroid.width / 2,
                    asteroid.y + asteroid.height / 2,
                    asteroid.width / 2,
                    0,
                    Math.PI * 2
                );
                ctx.fill();
            });
        }
        
        // 绘制能量晶体
        function drawEnergyCrystals() {
            energyCrystals.forEach(crystal => {
                ctx.fillStyle = crystal.color;
                ctx.beginPath();
                ctx.moveTo(crystal.x + crystal.width / 2, crystal.y);
                ctx.lineTo(crystal.x + crystal.width, crystal.y + crystal.height);
                ctx.lineTo(crystal.x, crystal.y + crystal.height);
                ctx.closePath();
                ctx.fill();
                
                // 添加发光效果
                ctx.shadowColor = crystal.color;
                ctx.shadowBlur = 10;
                ctx.fill();
                ctx.shadowBlur = 0;
            });
        }
        
        // 更新陨石位置
        function updateAsteroids() {
            for (let i = asteroids.length - 1; i >= 0; i--) {
                asteroids[i].y += asteroids[i].speed;
                
                // 移除超出屏幕的陨石
                if (asteroids[i].y > canvas.height) {
                    asteroids.splice(i, 1);
                }
            }
        }
        
        // 更新能量晶体位置
        function updateEnergyCrystals() {
            for (let i = energyCrystals.length - 1; i >= 0; i--) {
                energyCrystals[i].y += energyCrystals[i].speed;
                
                // 移除超出屏幕的能量晶体
                if (energyCrystals[i].y > canvas.height) {
                    energyCrystals.splice(i, 1);
                }
            }
        }
        
        // 检测碰撞
        function checkCollisions() {
            // 检测陨石碰撞
            for (let i = asteroids.length - 1; i >= 0; i--) {
                if (
                    player.x < asteroids[i].x + asteroids[i].width &&
                    player.x + player.width > asteroids[i].x &&
                    player.y < asteroids[i].y + asteroids[i].height &&
                    player.y + player.height > asteroids[i].y
                ) {
                    // 碰撞发生
                    asteroids.splice(i, 1);
                    lives--;
                    updateUI();
                    
                    if (lives <= 0) {
                        gameOver();
                    }
                }
            }
            
            // 检测能量晶体碰撞
            for (let i = energyCrystals.length - 1; i >= 0; i--) {
                if (
                    player.x < energyCrystals[i].x + energyCrystals[i].width &&
                    player.x + player.width > energyCrystals[i].x &&
                    player.y < energyCrystals[i].y + energyCrystals[i].height &&
                    player.y + player.height > energyCrystals[i].y
                ) {
                    // 收集能量晶体
                    energyCrystals.splice(i, 1);
                    score += 10;
                    updateUI();
                }
            }
        }
        
        // 更新玩家位置
        function updatePlayer() {
            if (keys.ArrowLeft && player.x > 0) {
                player.x -= player.speed;
            }
            if (keys.ArrowRight && player.x + player.width < canvas.width) {
                player.x += player.speed;
            }
        }
        
        // 绘制星空背景
        function drawBackground() {
            // 绘制深空背景
            ctx.fillStyle = '#000033';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // 绘制星星
            ctx.fillStyle = 'white';
            for (let i = 0; i < 100; i++) {
                const x = Math.random() * canvas.width;
                const y = Math.random() * canvas.height;
                const radius = Math.random() * 1.5;
                ctx.beginPath();
                ctx.arc(x, y, radius, 0, Math.PI * 2);
                ctx.fill();
            }
        }
        
        // 游戏循环
        function gameLoop() {
            if (!gameActive) return;
            
            // 清空画布
            drawBackground();
            
            // 更新和绘制游戏对象
            updatePlayer();
            drawPlayer();
            
            updateAsteroids();
            drawAsteroids();
            
            updateEnergyCrystals();
            drawEnergyCrystals();
            
            // 检测碰撞
            checkCollisions();
            
            // 随机生成陨石和能量晶体
            if (Math.random() < 0.03) {
                createAsteroid();
            }
            if (Math.random() < 0.02) {
                createEnergyCrystal();
            }
            
            requestAnimationFrame(gameLoop);
        }
        
        // 游戏结束
        function gameOver() {
            gameActive = false;
            finalScoreElement.textContent = score;
            gameOverScreen.style.display = 'flex';
        }
        
        // 开始游戏
        function startGame() {
            initGame();
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            gameActive = true;
            gameLoop();
        }
        
        // 事件监听
        startButton.addEventListener('click', startGame);
        restartButton.addEventListener('click', startGame);
        
        document.addEventListener('keydown', (e) => {
            if (e.key in keys) {
                keys[e.key] = true;
            }
        });
        
        document.addEventListener('keyup', (e) => {
            if (e.key in keys) {
                keys[e.key] = false;
            }
        });
    </script>
</body>
</html>

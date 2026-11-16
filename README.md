<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Neon Breakout</title>
    
    <script src="https://cdn.jsdelivr.net/npm/eruda"></script>
    <script>eruda.init();</script>

    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; }
        body {
            background-color: #0b0f19;
            color: #fff;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
            font-family: sans-serif;
        }
        #hud {
            position: fixed;
            top: 15px;
            right: 15px;
            z-index: 10;
            display: flex;
            gap: 8px;
            align-items: center;
        }
        .badge {
            background: rgba(10, 10, 25, 0.85);
            border: 1px solid #00f3ff;
            color: #00f3ff;
            padding: 5px 10px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: bold;
        }
        .shop-btn {
            background: linear-gradient(45deg, #ff007f, #792dca);
            border: none;
            color: white;
            padding: 5px 14px;
            border-radius: 20px;
            font-size: 12px;
            font-weight: bold;
            cursor: pointer;
        }
        canvas {
            background: #111625;
            border: 2px solid #00f3ff;
            border-radius: 8px;
            touch-action: none;
        }
        #shop-modal {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(5, 5, 15, 0.95);
            z-index: 100;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 15px;
        }
        .shop-content {
            background: #111625;
            border: 2px solid #ff007f;
            border-radius: 12px;
            width: 100%;
            max-width: 360px;
            padding: 15px;
            max-height: 85vh;
            overflow-y: auto;
        }
        .shop-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 12px;
        }
        .shop-header h2 { color: #00f3ff; font-size: 16px; }
        .close-btn { background: none; border: 1px solid #ff007f; color: #ff007f; padding: 3px 8px; border-radius: 6px; }
        .section-title { color: #ff007f; font-size: 12px; text-transform: uppercase; margin: 10px 0 5px 0; }
        .item-row { display: flex; justify-content: space-between; align-items: center; background: #181f33; padding: 7px 10px; border-radius: 8px; margin-bottom: 5px; border: 1px solid #243050; }
        .item-info .name { font-weight: bold; color: #fff; font-size: 11px; }
        .buy-btn { background: #00f3ff; color: #0b0f19; border: none; padding: 5px 10px; border-radius: 6px; font-weight: bold; font-size: 11px; cursor: pointer; }
        .buy-btn:disabled { background: #243050; color: #61759b; }
        .ad-btn { background: linear-gradient(45deg, #00ff66, #00b341); color: #0b0f19; width: 100%; padding: 9px; border: none; border-radius: 8px; font-weight: bold; font-size: 12px; cursor: pointer; margin-bottom: 10px; }
        #overlay-screen { display: none; position: fixed; inset: 0; background: rgba(5, 5, 15, 0.9); z-index: 50; flex-direction: column; align-items: center; justify-content: center; text-align: center; padding: 20px; }
        #overlay-screen h1 { font-size: 24px; margin-bottom: 10px; color: #ff007f; }
        #overlay-screen p { font-size: 14px; margin-bottom: 20px; color: #00f3ff; }
        .action-btn { background: linear-gradient(45deg, #00f3ff, #0077ff); border: none; color: #0b0f19; padding: 10px 24px; border-radius: 25px; font-size: 14px; font-weight: bold; cursor: pointer; }
    </style>
</head>
<body>

    <div id="hud">
        <div class="badge">Ур. <span id="level-display">1</span>/10</div>
        <div class="badge">💎 <span id="coin-count">0</span></div>
        <button class="shop-btn" onclick="toggleShop(true)">Магазин</button>
    </div>

    <canvas id="gameCanvas" width="360" height="480"></canvas>

    <div id="shop-modal">
        <div class="shop-content">
            <div class="shop-header">
                <h2>Магазин & Награды</h2>
                <button class="close-btn" onclick="toggleShop(false)">✕</button>
            </div>
            <button class="ad-btn" onclick="watchAd()">🎁 Смотреть рекламу (+25 💎)</button>
            <div class="section-title">Цвета платформы</div>
            <div id="skins-list"></div>
            <div class="section-title">Бонусы</div>
            <div id="boosts-list"></div>
        </div>
    </div>

    <div id="overlay-screen">
        <h1 id="overlay-title">Игра окончена</h1>
        <p id="overlay-text">Счет: <span id="final-score">0</span></p>
        <button class="action-btn" id="overlay-action-btn" onclick="nextLevelOrRestart()">Далее</button>
    </div>

    <script>
        let playerData = {
            coins: parseInt(localStorage.getItem('nb_coins')) || 0,
            level: parseInt(localStorage.getItem('nb_level')) || 1,
            unlockedSkins: ['#00f3ff'],
            activeSkin: localStorage.getItem('nb_active_skin') || '#00f3ff',
            widePaddle: localStorage.getItem('nb_wide') === 'true'
        };

        try {
            let savedSkins = JSON.parse(localStorage.getItem('nb_skins'));
            if (Array.isArray(savedSkins)) playerData.unlockedSkins = savedSkins;
        } catch (e) {}

        function saveGame() {
            localStorage.setItem('nb_coins', playerData.coins);
            localStorage.setItem('nb_level', playerData.level);
            localStorage.setItem('nb_skins', JSON.stringify(playerData.unlockedSkins));
            localStorage.setItem('nb_active_skin', playerData.activeSkin);
            localStorage.setItem('nb_wide', playerData.widePaddle);
        }

        const skins = [
            { id: '#00f3ff', name: 'Неоновый Голубой', price: 0 },
            { id: '#ff007f', name: 'Киберпанк Розовый', price: 50 },
            { id: '#00ff66', name: 'Неоновый Зеленый', price: 100 },
            { id: '#bd00ff', name: 'Энергетический Фиолетовый', price: 150 },
            { id: '#ffd700', name: 'Золотой Неон', price: 200 },
            { id: '#ff2200', name: 'Плазменный Красный', price: 250 },
            { id: '#00ffea', name: 'Кибернетический Циан', price: 300 },
            { id: '#ff00ea', name: 'Неоновая Фуксия', price: 350 }
        ];

        const boosts = [
            { id: 'widePaddle', name: 'Широкая платформа', desc: '+30% к ширине', price: 80 }
        ];

        function watchAd() {
            playerData.coins += 25;
            saveGame();
            renderShop();
            alert("📺 Рекламный ролик завершен! Получено +25 💎");
        }

        function renderShop() {
            document.getElementById('coin-count').innerText = playerData.coins;
            document.getElementById('level-display').innerText = playerData.level;
            
            let skinsHtml = '';
            skins.forEach(skin => {
                let isOwned = playerData.unlockedSkins.includes(skin.id);
                let isActive = playerData.activeSkin === skin.id;
                let btnText = isActive ? 'Активно' : (isOwned ? 'Выбрать' : `${skin.price} 💎`);
                let btnDisabled = isActive ? 'disabled' : '';
                
                skinsHtml += `
                    <div class="item-row">
                        <div class="item-info"><span class="name" style="color: ${skin.id}">${skin.name}</span></div>
                        <button class="buy-btn" ${btnDisabled} onclick="buySkin('${skin.id}', ${skin.price})">${btnText}</button>
                    </div>
                `;
            });
            document.getElementById('skins-list').innerHTML = skinsHtml;

            let boostsHtml = '';
            boosts.forEach(boost => {
                let isOwned = playerData[boost.id];
                let btnText = isOwned ? 'Куплено' : `${boost.price} 💎`;
                let btnDisabled = isOwned ? 'disabled' : '';

                boostsHtml += `
                    <div class="item-row">
                        <div class="item-info">
                            <span class="name">${boost.name}</span>
                            <span class="desc">${boost.desc}</span>
                        </div>
                        <button class="buy-btn" ${btnDisabled} onclick="buyBoost('${boost.id}', ${boost.price})">${btnText}</button>
                    </div>
                `;
            });
            document.getElementById('boosts-list').innerHTML = boostsHtml;
        }

        function toggleShop(open) {
            document.getElementById('shop-modal').style.display = open ? 'flex' : 'none';
            if (open) renderShop();
            isPaused = open;
        }

        function buySkin(id, price) {
            if (playerData.unlockedSkins.includes(id)) {
                playerData.activeSkin = id;
            } else if (playerData.coins >= price) {
                playerData.coins -= price;
                playerData.unlockedSkins.push(id);
                playerData.activeSkin = id;
            }
            saveGame();
            renderShop();
        }

        function buyBoost(id, price) {
            if (!playerData[id] && playerData.coins >= price) {
                playerData.coins -= price;
                playerData[id] = true;
                if (id === 'widePaddle') paddleWidth = playerWideWidth;
                saveGame();
                renderShop();
            }
        }

        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        let score = 0;
        let isPaused = false;
        let x = canvas.width / 2;
        let y = canvas.height - 40;
        
        let speedMultiplier = 1 + (playerData.level - 1) * 0.08;
        let dx = 2.2 * speedMultiplier;
        let dy = -2.2 * speedMultiplier;
        const ballRadius = 8;

        const paddleHeight = 12;
        const playerNormalWidth = 75;
        const playerWideWidth = 100;
        let paddleWidth = playerData.widePaddle ? playerWideWidth : playerNormalWidth;
        let paddleX = (canvas.width - paddleWidth) / 2;

        let rightPressed = false;
        let leftPressed = false;

        document.addEventListener("keydown", (e) => {
            if (e.key === "Right" || e.key === "ArrowRight") rightPressed = true;
            if (e.key === "Left" || e.key === "ArrowLeft") leftPressed = true;
        });
        document.addEventListener("keyup", (e) => {
            if (e.key === "Right" || e.key === "ArrowRight") rightPressed = false;
            if (e.key === "Left" || e.key === "ArrowLeft") leftPressed = false;
        });

        canvas.addEventListener("touchmove", handleTouch, { passive: false });
        canvas.addEventListener("touchstart", handleTouch, { passive: false });

        function handleTouch(e) {
            e.preventDefault();
            const touch = e.touches[0];
            const rect = canvas.getBoundingClientRect();
            const relativeX = touch.clientX - rect.left;
            if (relativeX > 0 && relativeX < canvas.width) {
                paddleX = relativeX - paddleWidth / 2;
            }
        }

        const brickColumnCount = 5;
        let brickRowCount = 3;
        const brickWidth = 54;
        const brickHeight = 18;
        const brickPadding = 8;
        const brickOffsetTop = 35;
        const brickOffsetLeft = 29;

        let bricks = [];
        function initBricks() {
            brickRowCount = Math.min(3 + Math.floor((playerData.level - 1) / 2), 7);
            bricks = [];
            for (let c = 0; c < brickColumnCount; c++) {
                bricks[c] = [];
                for (let r = 0; r < brickRowCount; r++) {
                    bricks[c][r] = { x: 0, y: 0, status: 1 };
                }
            }
        }
        initBricks();

        function collisionDetection() {
            for (let c = 0; c < brickColumnCount; c++) {
                for (let r = 0; r < brickRowCount; r++) {
                    let b = bricks[c][r];
                    if (b.status === 1) {
                        if (x > b.x && x < b.x + brickWidth && y > b.y && y < b.y + brickHeight) {
                            dy = -dy;
                            b.status = 0;
                            score += 10 * playerData.level;
                            playerData.coins += 1;
                            saveGame();
                            document.getElementById('coin-count').innerText = playerData.coins;

                            if (checkWin()) {
                                let reward = playerData.level * 15;
                                playerData.coins += reward;
                                
                                if (playerData.level < 10) {
                                    playerData.level++;
                                    showOverlay("Уровень пройден! 🎉", `Награда: +${reward} 💎<br>Переход на уровень ${playerData.level}`, "Следующий уровень");
                                } else {
                                    showOverlay("Победа в игре! 🏆", `Вы прошли все 10 уровней!<br>Бонус: +${reward} 💎`, "Начать заново");
                                    playerData.level = 1;
                                }
                                saveGame();
                            }
                        }
                    }
                }
            }
        }

        function checkWin() {
            for (let c = 0; c < brickColumnCount; c++) {
                for (let r = 0; r < brickRowCount; r++) {
                    if (bricks[c][r].status === 1) return false;
                }
            }
            return true;
        }

        function showOverlay(title, text, btnText) {
            isPaused = true;
            document.getElementById('overlay-title').innerText = title;
            document.getElementById('overlay-text').innerHTML = `${text}<br>Счет: <b>${score}</b>`;
            document.getElementById('overlay-action-btn').innerText = btnText;
            document.getElementById('overlay-screen').style.display = 'flex';
        }

        function showGameOver() {
            isPaused = true;
            document.getElementById('overlay-title').innerText = "Игра окончена";
            document.getElementById('overlay-text').innerHTML = `Мяч упал в пропасть!<br>Счет: <b>${score}</b>`;
            document.getElementById('overlay-action-btn').innerText = "Попробовать снова";
            document.getElementById('overlay-screen').style.display = 'flex';
        }

        function nextLevelOrRestart() {
            document.getElementById('overlay-screen').style.display = 'none';
            score = 0;
            x = canvas.width / 2;
            y = canvas.height - 40;
            speedMultiplier = 1 + (playerData.level - 1) * 0.08;
            dx = 2.2 * speedMultiplier;
            dy = -2.2 * speedMultiplier;
            paddleWidth = playerData.widePaddle ? playerWideWidth : playerNormalWidth;
            paddleX = (canvas.width - paddleWidth) / 2;
            initBricks();
            isPaused = false;
        }

        function drawBall() {
            ctx.beginPath();
            ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
            ctx.fillStyle = "#ff007f";
            ctx.fill();
            ctx.closePath();
        }

        function drawPaddle() {
            ctx.beginPath();
            ctx.rect(paddleX, canvas.height - paddleHeight - 10, paddleWidth, paddleHeight);
            ctx.fillStyle = playerData.activeSkin;
            ctx.fill();
            ctx.closePath();
        }

        function drawBricks() {
            for (let c = 0; c < brickColumnCount; c++) {
                for (let r = 0; r < brickRowCount; r++) {
                    if (bricks[c][r].status === 1) {
                        let brickX = (c * (brickWidth + brickPadding)) + brickOffsetLeft;
                        let brickY = (r * (brickHeight + brickPadding)) + brickOffsetTop;
                        bricks[c][r].x = brickX;
                        bricks[c][r].y = brickY;
                        ctx.beginPath();
                        ctx.rect(brickX, brickY, brickWidth, brickHeight);
                        ctx.fillStyle = r % 2 === 0 ? "#bd00ff" : "#792dca";
                        ctx.fill();
                        ctx.closePath();
                    }
                }
            }
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            drawBricks();
            drawBall();
            drawPaddle();

            if (!isPaused) {
                collisionDetection();

                if (x + dx > canvas.width - ballRadius || x + dx < ballRadius) {
                    dx = -dx;
                }
                if (y + dy < ballRadius) {
                    dy = -dy;
                } else if (y + dy > canvas.height - ballRadius - 10) {
                    if (x > paddleX && x < paddleX + paddleWidth) {
                        dy = -dy;
                    } else {
                        showGameOver();
                    }
                }

                if (rightPressed && paddleX < canvas.width - paddleWidth) {
                    paddleX += 7;
                } else if (leftPressed && paddleX > 0) {
                    paddleX -= 7;
                }

                x += dx;
                y += dy;
            }

            requestAnimationFrame(draw);
        }

        document.getElementById('coin-count').innerText = playerData.coins;
        document.getElementById('level-display').innerText = playerData.level;
        draw();
    </script>
</body>
</html>

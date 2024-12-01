<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Escape del Cubo Azul</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
            background: #222;
            color: white;
        }
        canvas {
            display: block;
        }
        #menu {
            position: absolute;
            bottom: 10px;
            left: 10px;
            width: 80%;
            background: rgba(0, 0, 0, 0.7);
            color: white;
            padding: 10px;
            text-align: center;
            border-radius: 10px;
            z-index: 100;
            display: none;
        }
        #menu button {
            margin: 10px;
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
        }
        #stats {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 16px;
            z-index: 1000;
        }
        #stats p {
            margin: 5px;
        }
        .menu-btn {
            position: absolute;
            top: 10px;
            left: 10px;
            padding: 10px;
            background: #333;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            z-index: 10;
        }
    </style>
</head>
<body>
    <button class="menu-btn">Menú</button>
    <div id="menu">
        <h2>Menú</h2>
        <p>Monedas: <span id="coins">0</span></p>
        <div class="skins">
            <h3>Colores del Cubo</h3>
            <button data-skin="green" style="background: green;">Verde</button>
            <button data-skin="red" style="background: red;">Rojo</button>
            <button data-skin="cyan" style="background: cyan;">Celeste</button>
            <button data-skin="orange" style="background: orange;">Naranja</button>
        </div>
        <div class="weapons">
            <h3>Armas</h3>
            <button data-weapon="pistol">Pistola</button>
            <button data-weapon="rifle">Rifle</button>
        </div>
        <button id="closeMenu">Cerrar Menú</button>
    </div>
    <div id="stats">
        <p>Vida Cubo Azul: <span id="blueCubeLife">7</span></p>
        <p>Monedas: <span id="coinsDisplay">0</span></p>
        <p>Arma: <span id="currentWeapon">Pistola</span></p>
    </div>

    <script>
        const canvas = document.createElement('canvas');
        const ctx = canvas.getContext('2d');
        document.body.appendChild(canvas);

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        // Variables de juego
        let blueCube = { x: canvas.width / 2, y: canvas.height - 50, size: 30, color: 'blue', life: 7, speed: 5, weapon: 'pistol', bullets: [] };
        let enemies = [];
        let coins = 0;
        let gamePaused = false;
        let touchStart = { x: 0, y: 0 };
        let touchMove = { x: 0, y: 0 };

        // Armas y tipos de disparos
        const weapons = {
            pistol: { name: 'Pistola', damage: 1, speed: 10, bulletSize: 5 },
            rifle: { name: 'Rifle', damage: 2, speed: 20, bulletSize: 7 }
        };

        // Elementos de menú y botones
        const menu = document.getElementById('menu');
        const menuBtn = document.querySelector('.menu-btn');
        const closeMenuBtn = document.getElementById('closeMenu');
        const coinsDisplay = document.getElementById('coinsDisplay');
        const blueCubeLifeSpan = document.getElementById('blueCubeLife');
        const currentWeaponDisplay = document.getElementById('currentWeapon');

        // Funciones de dibujo
        function drawBlueCube() {
            ctx.fillStyle = blueCube.color;
            ctx.fillRect(blueCube.x, blueCube.y, blueCube.size, blueCube.size);
        }

        function drawEnemies() {
            enemies.forEach(enemy => {
                ctx.fillStyle = enemy.color;
                ctx.fillRect(enemy.x, enemy.y, enemy.size, enemy.size);
            });
        }

        function drawBullets() {
            blueCube.bullets.forEach(bullet => {
                ctx.fillStyle = 'white';
                ctx.fillRect(bullet.x, bullet.y, bullet.size, bullet.size);
            });
        }

        function drawCoins() {
            for (let i = 0; i < coins; i++) {
                ctx.fillStyle = 'yellow';
                ctx.beginPath();
                ctx.arc(Math.random() * canvas.width, Math.random() * canvas.height, 5, 0, Math.PI * 2);
                ctx.fill();
            }
        }

        // Movimiento de cubo azul
        function moveBlueCube() {
            const moveSpeed = blueCube.speed;
            const moveX = touchMove.x - touchStart.x;
            const moveY = touchMove.y - touchStart.y;

            if (Math.abs(moveX) > Math.abs(moveY)) {
                if (moveX > 10 && blueCube.x + blueCube.size < canvas.width) {
                    blueCube.x += moveSpeed;
                } else if (moveX < -10 && blueCube.x > 0) {
                    blueCube.x -= moveSpeed;
                }
            } else {
                if (moveY > 10 && blueCube.y + blueCube.size < canvas.height) {
                    blueCube.y += moveSpeed;
                } else if (moveY < -10 && blueCube.y > 0) {
                    blueCube.y -= moveSpeed;
                }
            }
        }

        // Disparos
        function shoot() {
            const weapon = weapons[blueCube.weapon];
            blueCube.bullets.push({ x: blueCube.x + blueCube.size / 2, y: blueCube.y, size: weapon.bulletSize, speed: weapon.speed });
        }

        // Movimiento de las balas
        function moveBullets() {
            blueCube.bullets.forEach(bullet => {
                bullet.y -= bullet.speed;
            });
            blueCube.bullets = blueCube.bullets.filter(bullet => bullet.y > 0);
        }

        // Enemigos
        function createEnemy() {
            const enemy = { x: Math.random() * canvas.width, y: Math.random() * canvas.height, size: 30, color: 'red', life: 7, speed: 3, weapon: 'pistol', bullets: [] };
            enemies.push(enemy);
        }

        // Movimiento de enemigos con colisiones con las paredes
        function moveEnemies() {
            enemies.forEach(enemy => {
                if (enemy.x < blueCube.x && enemy.x + enemy.size < canvas.width) enemy.x += enemy.speed;
                if (enemy.x > blueCube.x && enemy.x > 0) enemy.x -= enemy.speed;
                if (enemy.y < blueCube.y && enemy.y + enemy.size < canvas.height) enemy.y += enemy.speed;
                if (enemy.y > blueCube.y && enemy.y > 0) enemy.y -= enemy.speed;

                if (Math.random() < 0.01) {  // Los enemigos disparan aleatoriamente
                    enemy.bullets.push({ x: enemy.x + enemy.size / 2, y: enemy.y, size: weapons[enemy.weapon].bulletSize, speed: weapons[enemy.weapon].speed });
                }
            });
        }

        // Colisiones
        function checkCollisions() {
            blueCube.bullets.forEach(bullet => {
                enemies.forEach(enemy => {
                    if (bullet.x < enemy.x + enemy.size &&
                        bullet.x + bullet.size > enemy.x &&
                        bullet.y < enemy.y + enemy.size &&
                        bullet.y + bullet.size > enemy.y) {
                        enemy.life -= weapons[blueCube.weapon].damage;
                        blueCube.bullets = blueCube.bullets.filter(b => b !== bullet);
                    }
                });
            });

            enemies.forEach(enemy => {
                if (blueCube.x < enemy.x + enemy.size &&
                    blueCube.x + blueCube.size > enemy.x &&
                    blueCube.y < enemy.y + enemy.size &&
                    blueCube.y + blueCube.size > enemy.y) {
                    blueCube.life--;
                    enemy.life = 0;  // El enemigo muere si colisiona con el cubo azul
                }
            });
        }

        // Menú de selección
        menuBtn.onclick = () => {
            menu.style.display = 'block';
        }

        closeMenuBtn.onclick = () => {
            menu.style.display = 'none';
        }

        // Evento táctil
        canvas.addEventListener('touchstart', e => {
            touchStart.x = e.touches[0].clientX;
            touchStart.y = e.touches[0].clientY;
        });

        canvas.addEventListener('touchmove', e => {
            touchMove.x = e.touches[0].clientX;
            touchMove.y = e.touches[0].clientY;
            moveBlueCube();
        });

        canvas.addEventListener('touchend', e => {
            if (!gamePaused) {
                shoot();
            }
        });

        // Renders y lógica del juego
        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            if (!gamePaused) {
                moveEnemies();
                moveBullets();
                checkCollisions();
                drawBlueCube();
                drawEnemies();
                drawBullets();
                drawCoins();
            }

            blueCubeLifeSpan.textContent = blueCube.life;
            coinsDisplay.textContent = coins;
            requestAnimationFrame(gameLoop);
        }

        // Iniciar juego
        gameLoop();

        setInterval(createEnemy, 2000);  // Crear enemigos cada 2 segundos
    </script>
</body>
</html>

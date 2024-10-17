<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Space Invaders</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #000;
            font-family: Arial, sans-serif;
        }
        #gameCanvas {
            border: 2px solid #fff;
        }
        #startScreen {
            position: absolute;
            color: #fff;
            font-size: 24px;
            text-align: center;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas" width="800" height="600"></canvas>
    <div id="startScreen">
        <h1>Space Invaders</h1>
        <p>Press SPACE to start/restart</p>
        <p>Use WASD to move, J to shoot</p>
        <p>Back two rows require 3 hits!</p>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startScreen = document.getElementById('startScreen');

        let player, aliens, bullets;
        let score = 0;
        let gameOver = true;
        let playerWon = false;
        let lastShootTime = 0;
        const shootDelay = 300; // 0.3 seconds in milliseconds

        class GameObject {
            constructor(x, y, width, height, color) {
                this.x = x;
                this.y = y;
                this.width = width;
                this.height = height;
                this.color = color;
            }

            draw() {
                ctx.fillStyle = this.color;
                ctx.fillRect(this.x, this.y, this.width, this.height);
            }
        }

        class Player extends GameObject {
            constructor() {
                super(400, 550, 50, 30, '#00ff00');
                this.speed = 5;
            }

            move(direction) {
                if (direction === 'left' && this.x > 0) {
                    this.x -= this.speed;
                } else if (direction === 'right' && this.x < canvas.width - this.width) {
                    this.x += this.speed;
                } else if (direction === 'up' && this.y > canvas.height / 2) {
                    this.y -= this.speed;
                } else if (direction === 'down' && this.y < canvas.height - this.height) {
                    this.y += this.speed;
                }
            }

            shoot() {
                const currentTime = Date.now();
                if (currentTime - lastShootTime >= shootDelay) {
                    bullets.push(new Bullet(this.x + this.width / 2, this.y));
                    lastShootTime = currentTime;
                }
            }
        }

        class Alien extends GameObject {
            constructor(x, y, health) {
                super(x, y, 40, 40, '#ff0000');
                this.baseSpeed = 1;
                this.direction = 1;
                this.health = health;
            }

            move() {
                const speedMultiplier = 1 + (50 - aliens.length) / 25; // Increased speed multiplier
                this.x += this.baseSpeed * this.direction * speedMultiplier;
                if (this.x <= 0 || this.x >= canvas.width - this.width) {
                    this.direction *= -1;
                    this.y += 20;
                }
            }

            draw() {
                ctx.fillStyle = this.health === 3 ? '#ff0000' : (this.health === 2 ? '#ff8000' : '#ffff00');
                ctx.fillRect(this.x, this.y, this.width, this.height);
            }

            hit() {
                this.health--;
                return this.health <= 0;
            }
        }

        class Bullet extends GameObject {
            constructor(x, y) {
                super(x, y, 5, 10, '#ffffff');
                this.speed = 7;
            }

            move() {
                this.y -= this.speed;
            }
        }

        function init() {
            player = new Player();
            aliens = [];
            bullets = [];
            score = 0;
            gameOver = false;
            playerWon = false;

            for (let i = 0; i < 5; i++) {
                for (let j = 0; j < 10; j++) {
                    const health = i < 2 ? 3 : 1; // First two rows have 3 health, others have 1
                    aliens.push(new Alien(j * 60 + 100, i * 60 + 50, health));
                }
            }
        }

        function update() {
            bullets = bullets.filter(bullet => bullet.y > 0);

            bullets.forEach(bullet => bullet.move());
            aliens.forEach(alien => alien.move());

            aliens.forEach((alien, alienIndex) => {
                bullets.forEach((bullet, bulletIndex) => {
                    if (collision(alien, bullet)) {
                        if (alien.hit()) {
                            aliens.splice(alienIndex, 1);
                            score += 10;
                        }
                        bullets.splice(bulletIndex, 1);
                    }
                });
            });

            if (aliens.length === 0) {
                gameOver = true;
                playerWon = true;
            } else if (aliens.some(alien => alien.y + alien.height >= player.y)) {
                gameOver = true;
                playerWon = false;
            }
        }

        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            player.draw();
            aliens.forEach(alien => alien.draw());
            bullets.forEach(bullet => bullet.draw());

            ctx.fillStyle = '#ffffff';
            ctx.font = '20px Arial';
            ctx.fillText(`Score: ${score}`, 10, 30);

            if (gameOver) {
                ctx.fillStyle = 'rgba(0, 0, 0, 0.5)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                ctx.fillStyle = '#ffffff';
                ctx.font = '40px Arial';
                if (playerWon) {
                    ctx.fillText('YOU WIN!', canvas.width / 2 - 80, canvas.height / 2);
                } else {
                    ctx.fillText('Game Over', canvas.width / 2 - 100, canvas.height / 2);
                }
                ctx.font = '20px Arial';
                ctx.fillText('Press SPACE to restart', canvas.width / 2 - 100, canvas.height / 2 + 40);
            }
        }

        function collision(obj1, obj2) {
            return obj1.x < obj2.x + obj2.width &&
                   obj1.x + obj1.width > obj2.x &&
                   obj1.y < obj2.y + obj2.height &&
                   obj1.y + obj1.height > obj2.y;
        }

        const keys = {};

        document.addEventListener('keydown', (event) => {
            keys[event.key.toLowerCase()] = true;

            if (gameOver && event.code === 'Space') {
                init();
                startScreen.style.display = 'none';
                gameLoop();
            }
        });

        document.addEventListener('keyup', (event) => {
            keys[event.key.toLowerCase()] = false;
        });

        function handleInput() {
            if (keys['a']) player.move('left');
            if (keys['d']) player.move('right');
            if (keys['w']) player.move('up');
            if (keys['s']) player.move('down');
            if (keys['j']) player.shoot();
        }

        function gameLoop() {
            if (!gameOver) {
                handleInput();
                update();
                draw();
                requestAnimationFrame(gameLoop);
            }
        }

        startScreen.style.display = 'block';
    </script>
</body>
</html>

Berikut adalah game perang "Lilim's Vengeance" lengkap dengan HTML, CSS, dan JavaScript. Pemain mengendalikan karakter Lilim, menembak musuh yang datang. Simpan sebagai file `.html` dan buka di browser.
```html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Lilim's Vengeance - Game Perang Melawan Lawan</title>
    <style>
        * {
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            background: linear-gradient(135deg, #0a0f1e, #0c1a2a);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Courier New', 'Segoe UI', monospace;
            margin: 0;
            padding: 20px;
        }

        .game-wrapper {
            background: #010a12;
            border-radius: 32px;
            padding: 20px;
            box-shadow: 0 20px 35px rgba(0,0,0,0.6), inset 0 1px 2px rgba(255,255,255,0.1);
            border: 1px solid #3f6e6e;
        }

        canvas {
            display: block;
            margin: 0 auto;
            border-radius: 20px;
            box-shadow: 0 0 0 3px #5a3f2c, 0 10px 25px black;
            cursor: crosshair;
            background-color: #1a2e2b;
        }

        .info-panel {
            margin-top: 18px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            gap: 15px;
            flex-wrap: wrap;
            background: #07161bd9;
            backdrop-filter: blur(6px);
            padding: 10px 22px;
            border-radius: 60px;
            border: 1px solid #ccaa77;
        }

        .hero-name {
            background: #aa3e4c;
            color: #f9e0a0;
            padding: 5px 24px;
            border-radius: 40px;
            font-weight: bold;
            font-size: 1.4rem;
            letter-spacing: 2px;
            font-family: 'Courier New', monospace;
            box-shadow: inset 0 -2px 0 #5a1e2a;
        }

        .stats {
            display: flex;
            gap: 30px;
            background: #0a191e;
            padding: 5px 22px;
            border-radius: 50px;
        }

        .stat {
            color: #d4eaea;
            font-weight: bold;
            font-size: 1rem;
        }

        .stat span {
            color: #f7b05e;
            font-size: 1.3rem;
            margin-left: 8px;
            font-weight: 800;
        }

        button {
            background: #2f6b47;
            border: none;
            font-family: monospace;
            font-weight: bold;
            font-size: 1rem;
            padding: 7px 25px;
            border-radius: 60px;
            color: white;
            cursor: pointer;
            transition: 0.1s linear;
            box-shadow: 0 4px 0 #1a3a28;
        }

        button:active {
            transform: translateY(2px);
            box-shadow: 0 1px 0 #1a3a28;
        }

        .controls {
            font-size: 0.7rem;
            background: #102226;
            padding: 5px 15px;
            border-radius: 28px;
            color: #bbd4ce;
        }
    </style>
</head>
<body>
<div>
    <div class="game-wrapper">
        <canvas id="warCanvas" width="900" height="600"></canvas>
        <div class="info-panel">
            <div class="hero-name">⚔️ LILIM ⚔️</div>
            <div class="stats">
                <div class="stat">❤️ NYAWA <span id="healthValue">5</span></div>
                <div class="stat">💀 MUSUH DIHANCURKAN <span id="scoreValue">0</span></div>
                <div class="stat">🔫 AMMO <span id="ammoValue">∞</span></div>
            </div>
            <button id="resetWarBtn">RESTART PERANG</button>
        </div>
        <div class="controls" style="display: flex; justify-content: space-between; margin-top: 12px;">
            <span>🎮 WASD / panah ←↑↓→ : GERAK</span>
            <span>💥 SPASI / KLIK KIRI : TEMBAK</span>
            <span>⚠️ Musuh datang & menembak! ⚠️</span>
        </div>
    </div>
</div>

<script>
    (function(){
        // ---------- CANVAS ----------
        const canvas = document.getElementById('warCanvas');
        const ctx = canvas.getContext('2d');

        // ---------- GAME DIMENSI ----------
        const W = 900, H = 600;
        
        // ---------- PEMAIN (LILIM) ----------
        let player = {
            x: W/2,
            y: H - 70,
            radius: 20,
            health: 5,
            maxHealth: 5,
            invincibleFrames: 0,   // after hit
            speed: 5.6
        };
        
        // ---------- PROYEKTIL PEMAIN ----------
        let bullets = [];
        const BULLET_RADIUS = 6;
        const BULLET_SPEED = -9;
        let shootCooldown = 0;
        const SHOOT_DELAY_FRAMES = 12; // tembakan per detik ~ 5-6
        
        // ---------- MUSUH (LAWAN) ----------
        let enemies = [];
        let enemyBullets = [];
        let score = 0;
        let gameOver = false;
        let gameWin = false;    // jika score mencapai target? tapi kita buat endless sampai mati
        
        // variabel spawn musuh
        let enemySpawnTimer = 0;
        let enemySpawnDelay = 35;  // frame
        
        // Variabel efek visual
        let screenShake = 0;
        let flashAlpha = 0;
        
        // ---------- DOM elements ----------
        const healthSpan = document.getElementById('healthValue');
        const scoreSpan = document.getElementById('scoreValue');
        const ammoSpan = document.getElementById('ammoValue');
        const resetBtn = document.getElementById('resetWarBtn');
        
        // ---------- Helper ----------
        function updateUI() {
            healthSpan.innerText = player.health;
            scoreSpan.innerText = score;
            ammoSpan.innerText = "∞";
        }
        
        // reset game total
        function resetGame() {
            gameOver = false;
            gameWin = false;
            player = {
                x: W/2,
                y: H - 70,
                radius: 20,
                health: 5,
                maxHealth: 5,
                invincibleFrames: 0,
                speed: 5.6
            };
            bullets = [];
            enemies = [];
            enemyBullets = [];
            score = 0;
            shootCooldown = 0;
            enemySpawnTimer = 5;
            screenShake = 0;
            flashAlpha = 0;
            updateUI();
        }
        
        // tembak dari player
        function shootFromPlayer() {
            if(gameOver) return false;
            bullets.push({
                x: player.x,
                y: player.y - 10,
                radius: BULLET_RADIUS,
                vx: 0,
                vy: BULLET_SPEED
            });
            return true;
        }
        
        // spawn musuh (lawan)
        function spawnEnemy() {
            let side = Math.floor(Math.random() * 3); // 0: kiri, 1: tengah, 2: kanan
            let xPos;
            if(side === 0) xPos = 40 + Math.random() * 100;
            else if(side === 2) xPos = W - 140 + Math.random() * 100;
            else xPos = W/2 - 40 + Math.random() * 80;
            
            xPos = Math.min(W - 35, Math.max(35, xPos));
            
            // tipe musuh: infanteri biasa
            let enemy = {
                x: xPos,
                y: 40,
                radius: 22,
                health: 2,      // butuh 2 tembakan untuk mati
                speedY: 1.8,
                shootCooldown: 0,
                color: "#7a2e3a"
            };
            enemies.push(enemy);
        }
        
        // update logika game
        function updateGame() {
            if(gameOver) return;
            
            // ---------- invincibility frames ----------
            if(player.invincibleFrames > 0) {
                player.invincibleFrames--;
            }
            
            // ---------- cooldown tembak player ----------
            if(shootCooldown > 0) shootCooldown--;
            
            // ---------- GERAK PLAYER (WASD / panah) ----------
            // kita handle di event listener, update posisi dari flag. Tapi lebih mudah pakai flag object
            // akan kita buat di global key state
            if(leftPressed && player.x - player.radius > 10) player.x -= player.speed;
            if(rightPressed && player.x + player.radius < W - 10) player.x += player.speed;
            if(upPressed && player.y - player.radius > 50) player.y -= player.speed;
            if(downPressed && player.y + player.radius < H - 20) player.y += player.speed;
            
            // batas aman
            player.x = Math.min(Math.max(player.x, 20 + player.radius), W - player.radius - 10);
            player.y = Math.min(Math.max(player.y, 50 + player.radius), H - 20);
            
            // ---------- spawn musuh otomatis ----------
            if(enemySpawnTimer <= 0) {
                if(enemies.length < 7) { // batas maks musuh di arena
                    spawnEnemy();
                }
                enemySpawnTimer = enemySpawnDelay;
                // makin banyak skor makin cepat spawn (susah)
                let reduction = Math.min(20, Math.floor(score / 18));
                enemySpawnDelay = Math.max(22, 38 - reduction);
            } else {
                enemySpawnTimer--;
            }
            
            // ---------- update peluru player ----------
            for(let i=0; i<bullets.length; i++) {
                bullets[i].x += bullets[i].vx;
                bullets[i].y += bullets[i].vy;
                if(bullets[i].y + bullets[i].radius < 0 || bullets[i].y - bullets[i].radius > H) {
                    bullets.splice(i,1);
                    i--;
                }
            }
            
            // ---------- update musuh & interaksi peluru ----------
            for(let i=0; i<enemies.length; i++) {
                let e = enemies[i];
                // gerak kebawah
                e.y += e.speedY;
                // batas bawah jika musuh lolos ke bawah, kurangi nyawa player (penalty)
                if(e.y + e.radius > H - 10) {
                    if(player.invincibleFrames <= 0 && !gameOver) {
                        player.health--;
                        player.invincibleFrames = 28;
                        screenShake = 12;
                        flashAlpha = 0.7;
                        updateUI();
                        if(player.health <= 0) {
                            gameOver = true;
                            player.health = 0;
                            updateUI();
                        }
                    }
                    enemies.splice(i,1);
                    i--;
                    continue;
                }
                
                // cooldown tembak musuh
                if(e.shootCooldown > 0) e.shootCooldown--;
                else {
                    // musuh menembak ke arah player dengan prediksi sederhana
                    let dx = player.x - e.x;
                    let dy = player.y - e.y;
                    let len = Math.hypot(dx, dy);
                    if(len > 0.1) {
                        let vx = (dx / len) * 4.2;
                        let vy = (dy / len) * 4.2;
                        enemyBullets.push({
                            x: e.x,
                            y: e.y + 8,
                            radius: 6,
                            vx: vx,
                            vy: vy,
                            damage: 1
                        });
                    }
                    e.shootCooldown = 45 + Math.random() * 25; // jeda tembak musuh
                }
                
                // tabrakan peluru player dengan musuh
                for(let j=0; j<bullets.length; j++) {
                    let b = bullets[j];
                    let dist = Math.hypot(b.x - e.x, b.y - e.y);
                    if(dist < b.radius + e.radius) {
                        // hit musuh
                        e.health -= 1;
                        bullets.splice(j,1);
                        if(e.health <= 0) {
                            // musuh mati
                            enemies.splice(i,1);
                            score++;
                            updateUI();
                            i--; // karena sudah hapus enemy
                            // efek gemuruh kecil
                            screenShake = 5;
                        }
                        break; // peluru hanya mengenai satu musuh
                    }
                }
            }
            
            // ---------- update peluru musuh & collision dengan player ----------
            for(let i=0; i<enemyBullets.length; i++) {
                let eb = enemyBullets[i];
                eb.x += eb.vx;
                eb.y += eb.vy;
                if(eb.x+eb.radius < 0 || eb.x-eb.radius > W || eb.y+eb.radius < 0 || eb.y-eb.radius > H) {
                    enemyBullets.splice(i,1);
                    i--;
                    continue;
                }
                let distToPlayer = Math.hypot(eb.x - player.x, eb.y - player.y);
                if(distToPlayer < eb.radius + player.radius) {
                    if(player.invincibleFrames <= 0 && !gameOver) {
                        player.health--;
                        player.invincibleFrames = 28;
                        screenShake = 12;
                        flashAlpha = 0.7;
                        updateUI();
                        if(player.health <= 0) {
                            gameOver = true;
                            player.health = 0;
                            updateUI();
                        }
                    }
                    enemyBullets.splice(i,1);
                    i--;
                }
            }
            
            // jika health 0, game over
            if(player.health <= 0) {
                gameOver = true;
            }
            
            // redam efek screen shake
            if(screenShake > 0) screenShake--;
            if(flashAlpha > 0) flashAlpha -= 0.03;
        }
        
        // ---------- GAMBAR SEMUA ELEMEN ----------
        function drawBackground() {
            // medan perang: tanah gelap, kawah
            const grad = ctx.createLinearGradient(0,0,0,H);
            grad.addColorStop(0,"#1f2b1c");
            grad.addColorStop(1,"#2b3d2a");
            ctx.fillStyle = grad;
            ctx.fillRect(0,0,W,H);
            
            // reruntuhan
            for(let i=0;i<40;i++) {
                ctx.fillStyle = "#5a4a3a";
                ctx.beginPath();
                ctx.ellipse(50+i*73, H-70+Math.sin(i)*15, 12, 6, 0, 0, Math.PI*2);
                ctx.fill();
            }
            // garis parit
            ctx.beginPath();
            ctx.strokeStyle = "#5f7044";
            ctx.lineWidth = 3;
            for(let i=0;i<3;i++) {
                ctx.beginPath();
                ctx.moveTo(0, H-130 + i*45);
                ctx.lineTo(W, H-120 + i*42);
                ctx.stroke();
            }
        }
        
        function drawPlayer() {
            // efek invincible berkedip
            let drawAlpha = 1;
            if(player.invincibleFrames > 0 && (Math.floor(Date.now()/50)%3 === 0)) {
                drawAlpha = 0.6;
            }
            ctx.save();
            ctx.globalAlpha = drawAlpha;
            // Lilim karakter prajurit wanita
            ctx.shadowBlur = 6;
            ctx.shadowColor = "#c97e5a";
            // body
            ctx.fillStyle = "#5d3e2e";
            ctx.beginPath();
            ctx.ellipse(player.x, player.y, player.radius, player.radius+2, 0, 0, Math.PI*2);
            ctx.fill();
            ctx.fillStyle = "#b8734a";
            ctx.beginPath();
            ctx.ellipse(player.x-5, player.y-5, 7, 9, 0, 0, Math.PI*2);
            ctx.fill();
            ctx.beginPath();
            ctx.ellipse(player.x+5, player.y-5, 7, 9, 0, 0, Math.PI*2);
            ctx.fill();
            // helm
            ctx.fillStyle = "#3e5a47";
            ctx.beginPath();
            ctx.ellipse(player.x, player.y-10, 13, 11, 0, 0, Math.PI*2);
            ctx.fill();
            // mata
            ctx.fillStyle = "#fef7cf";
            ctx.beginPath();
            ctx.arc(player.x-6, player.y-12, 3, 0, Math.PI*2);
            ctx.fill();
            ctx.beginPath();
            ctx.arc(player.x+6, player.y-12, 3, 0, Math.PI*2);
            ctx.fill();
            ctx.fillStyle = "#010101";
            ctx.beginPath();
            ctx.arc(player.x-6.5, player.y-13, 1.3, 0, Math.PI*2);
            ctx.fill();
            ctx.beginPath();
            ctx.arc(player.x+5.5, player.y-13, 1.3, 0, Math.PI*2);
            ctx.fill();
            
            // senjata
            ctx.fillStyle = "#3a2a1f";
            ctx.fillRect(player.x+12, player.y-4, 18, 6);
            ctx.fillRect(player.x+25, player.y-2, 8, 4);
            // nama Lilim
            ctx.font = "bold 14monospace";
            ctx.fillStyle = "#fbd26a";
            ctx.shadowBlur = 2;
            ctx.fillText("LILIM", player.x-15, player.y-17);
            ctx.restore();
        }
        
        function drawEnemies() {
            for(let e of enemies) {
                ctx.save();
                ctx.shadowBlur = 3;
                ctx.fillStyle = e.color;
                ctx.beginPath();
                ctx.ellipse(e.x, e.y, e.radius, e.radius-2, 0, 0, Math.PI*2);
                ctx.fill();
                ctx.fillStyle = "#cb5a3a";
                ctx.beginPath();
                ctx.ellipse(e.x-5, e.y-4, 5, 7, 0, 0, Math.PI*2);
                ctx.fill();
                ctx.beginPath();
                ctx.ellipse(e.x+5, e.y-4, 5, 7, 0, 0, Math.PI*2);
                ctx.fill();
                ctx.fillStyle = "#df1f2c";
                ctx.beginPath();
                ctx.ellipse(e.x, e.y+5, 9, 6, 0, 0, Math.PI*2);
                ctx.fill();
                // lambang musuh
                ctx.fillStyle = "#f2c94c";
                ctx.font = "bold 14monospace";
                ctx.fillText("⚔️", e.x-8, e.y-6);
                // health bar musuh (dua hit)
                let healthPercent = e.health / 2;
                ctx.fillStyle = "#aa3333";
                ctx.fillRect(e.x-15, e.y-18, 30, 5);
                ctx.fillStyle = "#56e04e";
                ctx.fillRect(e.x-15, e.y-18, 30 * healthPercent, 4);
                ctx.restore();
            }
        }
        
        function drawBullets() {
            for(let b of bullets) {
                ctx.beginPath();
                ctx.arc(b.x, b.y, b.radius-2, 0, Math.PI*2);
                ctx.fillStyle = "#ffdd77";
                ctx.shadowBlur = 6;
                ctx.fill();
                ctx.beginPath();
                ctx.arc(b.x, b.y, 3, 0, Math.PI*2);
                ctx.fillStyle = "orange";
                ctx.fill();
            }
            for(let eb of enemyBullets) {
                ctx.beginPath();
                ctx.arc(eb.x, eb.y, eb.radius-2, 0, Math.PI*2);
                ctx.fillStyle = "#ff5a3a";
                ctx.fill();
                ctx.beginPath();
                ctx.arc(eb.x, eb.y, 3, 0, Math.PI*2);
                ctx.fillStyle = "#aa2a1a";
                ctx.fill();
            }
        }
        
        function drawUIeffects() {
            if(flashAlpha > 0) {
                ctx.fillStyle = `rgba(230, 70, 40, ${flashAlpha})`;
                ctx.fillRect(0,0,W,H);
            }
            // teks game over
            if(gameOver) {
                ctx.font = "800 42monospace";
                ctx.fillStyle = "#d9b48b";
                ctx.shadowBlur = 10;
                ctx.fillText("GAME OVER", W/2-140, H/2-50);
                ctx.font = "bold 22monospace";
                ctx.fillStyle = "#ffbb77";
                ctx.fillText("Klik RESTART untuk bertempur lagi", W/2-190, H/2+40);
                ctx.font = "18monospace";
                ctx.fillStyle = "#bba88a";
                ctx.fillText("Lilim gugur dalam medan perang...", W/2-170, H/2+100);
            } else if(score >= 50) { // kemenangan epic
                ctx.font = "bold 36monospace";
                ctx.fillStyle = "#f5cb5c";
                ctx.fillText("✨ KEMENANGAN EPIK! ✨", W/2-200, H/2);
                ctx.font = "18monospace";
                ctx.fillStyle = "white";
                ctx.fillText("Lilim berhasil menghancurkan gerombolan!", W/2-210, H/2+50);
            }
        }
        
        // info tambahan medan perang
        function drawWarDetails() {
            ctx.font = "bold 14monospace";
            ctx.fillStyle = "#d8d09c";
            ctx.fillText("MUSUH DATANG DARI UTARA", 30, 35);
            ctx.fillStyle = "#cea776";
            ctx.fillText("SKOR: "+score, W-110, 45);
            if(shootCooldown === 0 && !gameOver) {
                ctx.fillStyle = "#7effb3";
                ctx.fillText("🔫 SIAP TEMBAK", W-150, H-20);
            }
        }
        
        // ---------- KEYBOARD HANDLING ----------
        let leftPressed = false, rightPressed = false, upPressed = false, downPressed = false;
        let spacePressed = false;
        
        function handleKeyDown(e) {
            const key = e.key;
            if(gameOver) return;
            if(key === 'ArrowLeft' || key === 'a' || key === 'A') {
                leftPressed = true;
                e.preventDefault();
            }
            else if(key === 'ArrowRight' || key === 'd' || key === 'D') {
                rightPressed = true;
                e.preventDefault();
            }
            else if(key === 'ArrowUp' || key === 'w' || key === 'W') {
                upPressed = true;
                e.preventDefault();
            }
            else if(key === 'ArrowDown' || key === 's' || key === 'S') {
                downPressed = true;
                e.preventDefault();
            }
            else if(key === ' ' || key === 'Space') {
                if(!gameOver && shootCooldown === 0) {
                    shootFromPlayer();
                    shootCooldown = SHOOT_DELAY_FRAMES;
                }
                e.preventDefault();
            }
        }
        
        function handleKeyUp(e) {
            const key = e.key;
            if(key === 'ArrowLeft' || key === 'a' || key === 'A') leftPressed = false;
            else if(key === 'ArrowRight' || key === 'd' || key === 'D') rightPressed = false;
            else if(key === 'ArrowUp' || key === 'w' || key === 'W') upPressed = false;
            else if(key === 'ArrowDown' || key === 's' || key === 'S') downPressed = false;
            else if(key === ' ' || key === 'Space') spacePressed = false;
            e.preventDefault();
        }
        
        // mouse click tembak
        function handleCanvasClick(e) {
            if(gameOver) return;
            if(shootCooldown === 0) {
                shootFromPlayer();
                shootCooldown = SHOOT_DELAY_FRAMES;
            }
            e.preventDefault();
        }
        
        // fungsi animasi dan rendering
        function drawScreenShake() {
            let dx = 0, dy = 0;
            if(screenShake > 0) {
                dx = (Math.random() - 0.5) * screenShake * 1.6;
                dy = (Math.random() - 0.5) * screenShake * 1.2;
            }
            ctx.save();
            ctx.translate(dx, dy);
        }
        
        function render() {
            drawScreenShake();
            drawBackground();
            drawEnemies();
            drawBullets();
            drawPlayer();
            drawWarDetails();
            drawUIeffects();
            ctx.restore(); // restore setelah shake
            if(gameOver) {
                // tambahan pesan
                ctx.font = "bold 20monospace";
                ctx.fillStyle = "#ffc285";
                ctx.fillText("TEKAN RESTART", W/2-80, H-50);
            } else if(score >= 50) {
                ctx.font = "24monospace";
                ctx.fillStyle = "#fad460";
                ctx.fillText("✧ LILIM THE WARRIOR ✧", W/2-140, H-30);
            }
        }
        
        // Loop utama
        let lastTime = 0;
        function gameLoop() {
            if(!gameOver && player.health > 0) {
                updateGame();
            }
            render();
            requestAnimationFrame(gameLoop);
        }
        
        // initialize
        function init() {
            resetGame();
            gameLoop();
            window.addEventListener('keydown', handleKeyDown);
            window.addEventListener('keyup', handleKeyUp);
            canvas.addEventListener('click', handleCanvasClick);
            resetBtn.addEventListener('click', () => {
                resetGame();
                gameOver = false;
                enemies = [];
                bullets = [];
                enemyBullets = [];
                player.invincibleFrames = 0;
                screenShake = 0;
                flashAlpha = 0;
                shootCooldown = 0;
                updateUI();
            });
            // sentuhan touch untuk mobile (opsional)
            canvas.addEventListener('touchstart', (e) => {
                e.preventDefault();
                let rect = canvas.getBoundingClientRect();
                let touchX = (e.touches[0].clientX - rect.left) * (W/rect.width);
                let touchY = (e.touches[0].clientY - rect.top) * (H/rect.height);
                // klik tembak jika disentuh
                if(shootCooldown === 0 && !gameOver) {
                    shootFromPlayer();
                    shootCooldown = SHOOT_DELAY_FRAMES;
                }
                // juga bisa menggerakkan? tapi skip agar sederhana
            });
        }
        
        init();
    })();
</script>
</body>
</html>
```

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>RED CARPET TYCOON</title>
    <style>
        body { margin: 0; background: #111; color: white; font-family: 'Impact', sans-serif; overflow: hidden; }
        #ui { position: absolute; top: 10px; left: 10px; pointer-events: none; text-shadow: 2px 2px #000; z-index: 10; }
        #money-display { font-size: 45px; color: #0f0; }
        #shop { position: absolute; right: 10px; top: 10px; background: rgba(0,0,0,0.9); padding: 20px; border: 3px solid gold; border-radius: 15px; width: 200px; text-align: center; }
        .btn { background: gold; color: black; border: none; padding: 12px; cursor: pointer; font-weight: bold; width: 100%; margin-top: 10px; font-family: inherit; font-size: 16px; border-radius: 5px; }
        .btn:hover { transform: scale(1.05); background: #fff; }
        canvas { display: block; }
        #fever-bar { width: 0%; height: 10px; background: #f0f; transition: width 0.1s; margin-top: 5px; border-radius: 5px; }
    </style>
</head>
<body>

    <div id="ui">
        <div id="money-display">CASH: $0</div>
        <div>REBIRTHS: <span id="rebirth-count">0</span></div>
        <div style="color: cyan;">MULTI: x<span id="multi-display">1</span></div>
        <div style="font-size: 12px; color: #f0f; margin-top: 10px;">FEVER METER:</div>
        <div style="width: 200px; height: 10px; background: #333; border-radius: 5px;">
            <div id="fever-bar"></div>
        </div>
    </div>

    <div id="shop">
        <div style="color:gold; font-size: 24px;">SHOP</div>
        <button class="btn" onclick="buyUpgrade()">+ POWER ($<span id="cost">50</span>)</button>
        <button class="btn" onclick="doRebirth()" id="rebirth-btn" style="background: #f0f; color: white; margin-top: 20px;">REBIRTH ($1000)</button>
    </div>

    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");
        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        let player = { x: canvas.width/2, y: canvas.height - 120 };
        let money = 0;
        let rebirths = 0;
        let multiplier = 1;
        let upgradeCost = 50;
        let fever = 0;
        let isFever = false;
        let coins = [];

        let mouseX = player.x;
        window.addEventListener('mousemove', (e) => { mouseX = e.clientX; });
        window.addEventListener('touchmove', (e) => { mouseX = e.touches[0].clientX; });

        function spawnCoin() {
            let count = isFever ? 5 : 1;
            for(let i=0; i<count; i++) {
                coins.push({
                    x: Math.random() * (canvas.width - 40) + 20,
                    y: isFever ? Math.random() * -500 : -50,
                    speed: isFever ? 12 : 5
                });
            }
        }

        function update() {
            player.x += (mouseX - player.x) * 0.15;
            if (Math.random() < (isFever ? 0.2 : 0.05)) spawnCoin();

            for (let i = coins.length - 1; i >= 0; i--) {
                coins[i].y += coins[i].speed;
                if (Math.abs(coins[i].x - player.x) < 50 && Math.abs(coins[i].y - player.y) < 50) {
                    let gain = 10 * multiplier;
                    money += gain;
                    if (!isFever) {
                        fever += 2;
                        if (fever >= 100) startFever();
                    }
                    coins.splice(i, 1);
                    updateUI();
                } else if (coins[i].y > canvas.height) {
                    coins.splice(i, 1);
                }
            }
            if (!isFever && fever > 0) fever -= 0.05;
            document.getElementById('fever-bar').style.width = fever + "%";
        }

        function startFever() {
            isFever = true;
            fever = 100;
            let timer = setInterval(() => {
                fever -= 2;
                if (fever <= 0) {
                    clearInterval(timer);
                    isFever = false;
                    fever = 0;
                }
            }, 100);
        }

        function updateUI() {
            document.getElementById('money-display').innerText = `CASH: $${Math.floor(money)}`;
            document.getElementById('cost').innerText = Math.floor(upgradeCost);
        }

        function draw() {
            ctx.fillStyle = isFever ? `hsl(${Date.now() % 360}, 70%, 20%)` : "#222";
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.fillStyle = "#800";
            ctx.fillRect(canvas.width/2 - 150, 0, 300, canvas.height);

            ctx.font = isFever ? "80px Arial" : "60px Arial";
            ctx.fillText(isFever ? "🔥🗿🔥" : "🗿", player.x - 40, player.y);

            ctx.font = "30px Arial";
            coins.forEach(c => ctx.fillText(isFever ? "💎" : "💵", c.x, c.y));

            requestAnimationFrame(() => { update(); draw(); });
        }

        function buyUpgrade() {
            if (money >= upgradeCost) {
                money -= upgradeCost;
                multiplier += 0.5;
                upgradeCost *= 1.3;
                document.getElementById('multi-display').innerText = multiplier.toFixed(1);
                updateUI();
            }
        }

        function doRebirth() {
            if (money >= 1000) {
                rebirths++;
                money = 0;
                upgradeCost = 50;
                multiplier = 1 + (rebirths * 5);
                document.getElementById('rebirth-count').innerText = rebirths;
                document.getElementById('multi-display').innerText = multiplier;
                updateUI();
                alert("REBIRTHED! YOU ARE NOW MORE POWERFUL.");
            }
        }
        draw();
    </script>
</body>
</html>

# -2525
ホームボタンで弾を発射します
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JavaScript Invaders</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <h1>JavaScript Invaders</h1>
    <canvas id="gameCanvas" width="600" height="400"></canvas>
    <div class="info-panel">
        <p>スコア: <span id="score">0</span></p>
        <p>ライフ: <span id="lives">3</span></p>
        <button id="startButton">ゲーム開始</button>
    </div>
    <script src="script.js"></script>
</body>
</html>

body {
    display: flex;
    flex-direction: column;
    align-items: center;
    font-family: 'Press Start 2P', cursive; /* ドット絵風フォントがあれば */
    background-color: #222;
    color: #eee;
}

canvas {
    border: 3px solid #0f0; /* レトロなグリーンの枠線 */
    background-color: #000;
    display: block;
    margin-bottom: 20px;
}

.info-panel {
    text-align: center;
    font-size: 1.2em;
    display: flex;
    gap: 30px;
}

button {
    padding: 10px 20px;
    font-size: 1em;
    cursor: pointer;
    background-color: #007bff;
    color: white;
    border: none;
    border-radius: 5px;
    margin-top: 10px;
}

button:hover {
    background-color: #0056b3;
}

const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const scoreDisplay = document.getElementById('score');
const livesDisplay = document.getElementById('lives');
const startButton = document.getElementById('startButton');

// ゲーム設定
const PLAYER_SPEED = 5;
const PLAYER_WIDTH = 40;
const PLAYER_HEIGHT = 20;
const BULLET_SPEED = 7;
const BULLET_WIDTH = 4;
const BULLET_HEIGHT = 10;
const INVADER_SPEED_X = 2;
const INVADER_SPEED_Y = 20; // 落下速度
const INVADER_WIDTH = 30;
const INVADER_HEIGHT = 20;
const INVADER_ROWS = 5;
const INVADER_COLS = 10;
const INVADER_MARGIN_X = 10;
const INVADER_MARGIN_Y = 10;
const INVADER_SPACING_X = 40;
const INVADER_SPACING_Y = 30;
const INVADER_SHOOT_INTERVAL = 1000; // インベーダーの弾の発射間隔 (ms)
const INVADER_BULLET_SPEED = 3;

let gameRunning = false;
let score = 0;
let lives = 3;
let player = {
    x: canvas.width / 2 - PLAYER_WIDTH / 2,
    y: canvas.height - PLAYER_HEIGHT - 10,
    width: PLAYER_WIDTH,
    height: PLAYER_HEIGHT,
    dx: 0
};
let playerBullets = [];
let invaders = [];
let invaderBullets = [];
let keys = {}; // 押されているキーを管理
let lastInvaderShootTime = 0; // 最後のインベーダー発射時刻

// ゲーム初期化
function initGame() {
    score = 0;
    lives = 3;
    player.x = canvas.width / 2 - PLAYER_WIDTH / 2;
    playerBullets = [];
    invaders = [];
    invaderBullets = [];
    keys = {};
    lastInvaderShootTime = 0;
    scoreDisplay.textContent = score;
    livesDisplay.textContent = lives;
    createInvaders();
    drawAll();
}

// プレイヤーを描画
function drawPlayer() {
    ctx.fillStyle = 'green';
    ctx.fillRect(player.x, player.y, player.width, player.height);
}

// プレイヤーの弾を描画
function drawPlayerBullets() {
    ctx.fillStyle = 'white';
    playerBullets.forEach(bullet => {
        ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
    });
}

// インベーダーを描画
function drawInvaders() {
    invaders.forEach(invader => {
        if (!invader.destroyed) {
            ctx.fillStyle = 'red';
            ctx.fillRect(invader.x, invader.y, invader.width, invader.height);
        }
    });
}

// インベーダーの弾を描画
function drawInvaderBullets() {
    ctx.fillStyle = 'yellow';
    invaderBullets.forEach(bullet => {
        ctx.fillRect(bullet.x, bullet.y, bullet.width, bullet.height);
    });
}

// 全体をクリアして描画
function drawAll() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    drawPlayer();
    drawPlayerBullets();
    drawInvaders();
    drawInvaderBullets();
}

// プレイヤーの更新
function updatePlayer() {
    player.x += player.dx;

    // 画面端での制限
    if (player.x < 0) {
        player.x = 0;
    }
    if (player.x + player.width > canvas.width) {
        player.x = canvas.width - player.width;
    }
}

// プレイヤーの弾の更新
function updatePlayerBullets() {
    playerBullets.forEach((bullet, index) => {
        bullet.y -= BULLET_SPEED;
        if (bullet.y < 0) {
            playerBullets.splice(index, 1); // 画面外に出たら削除
        }
    });
}

// インベーダーの生成
function createInvaders() {
    for (let r = 0; r < INVADER_ROWS; r++) {
        for (let c = 0; c < INVADER_COLS; c++) {
            invaders.push({
                x: INVADER_MARGIN_X + c * INVADER_SPACING_X,
                y: INVADER_MARGIN_Y + r * INVADER_SPACING_Y,
                width: INVADER_WIDTH,
                height: INVADER_HEIGHT,
                dx: INVADER_SPEED_X,
                destroyed: false // 破壊されたかどうか
            });
        }
    }
}

// インベーダーの更新
function updateInvaders() {
    let hitWall = false;
    invaders.forEach(invader => {
        if (!invader.destroyed) {
            invader.x += invader.dx;

            if (invader.x + invader.width > canvas.width || invader.x < 0) {
                hitWall = true;
            }
        }
    });

    if (hitWall) {
        invaders.forEach(invader => {
            if (!invader.destroyed) {
                invader.dx *= -1; // 向きを反転
                invader.y += INVADER_SPEED_Y; // 落下
                // インベーダーがプレイヤーのY座標より下に来たらゲームオーバー
                if (invader.y + invader.height >= player.y) {
                    endGame(false); // 負け
                }
            }
        });
    }
    
    // インベーダーの弾発射
    const now = Date.now();
    if (now - lastInvaderShootTime > INVADER_SHOOT_INTERVAL) {
        let aliveInvaders = invaders.filter(inv => !inv.destroyed);
        if (aliveInvaders.length > 0) {
            const shooter = aliveInvaders[Math.floor(Math.random() * aliveInvaders.length)];
            invaderBullets.push({
                x: shooter.x + shooter.width / 2 - BULLET_WIDTH / 2,
                y: shooter.y + shooter.height,
                width: BULLET_WIDTH,
                height: BULLET_HEIGHT
            });
        }
        lastInvaderShootTime = now;
    }
}

// インベーダーの弾の更新
function updateInvaderBullets() {
    invaderBullets.forEach((bullet, index) => {
        bullet.y += INVADER_BULLET_SPEED;
        if (bullet.y > canvas.height) {
            invaderBullets.splice(index, 1); // 画面外に出たら削除
        }
    });
}

// 衝突判定 (AABB: Axis-Aligned Bounding Box)
function checkCollision(obj1, obj2) {
    return obj1.x < obj2.x + obj2.width &&
           obj1.x + obj1.width > obj2.x &&
           obj1.y < obj2.y + obj2.height &&
           obj1.y + obj1.height > obj2.y;
}

// 衝突のチェック
function checkCollisions() {
    // プレイヤーの弾とインベーダーの衝突
    playerBullets.forEach((bullet, bulletIndex) => {
        invaders.forEach((invader, invaderIndex) => {
            if (!invader.destroyed && checkCollision(bullet, invader)) {
                bullet.y = -100; // 弾を画面外に飛ばして非表示（後で削除される）
                invader.destroyed = true;
                score += 10;
                scoreDisplay.textContent = score;
            }
        });
    });

    // インベーダーの弾とプレイヤーの衝突
    invaderBullets.forEach((bullet, bulletIndex) => {
        if (checkCollision(bullet, player)) {
            bullet.y = canvas.height + 100; // 弾を画面外に飛ばして非表示
            lives--;
            livesDisplay.textContent = lives;
        }
    });

    // 破壊された弾とインベーダーを配列から削除
    playerBullets = playerBullets.filter(bullet => bullet.y > -50);
    invaderBullets = invaderBullets.filter(bullet => bullet.y < canvas.height + 50);
    invaders = invaders.filter(invader => !invader.destroyed || invader.y < canvas.height); // 画面外に出た破壊済みインベーダーも削除

    // 全てのインベーダーが破壊されたかチェック
    if (invaders.filter(inv => !inv.destroyed).length === 0 && gameRunning) {
        endGame(true); // 勝ち
    }

    // ライフが0になったらゲームオーバー
    if (lives <= 0 && gameRunning) {
        endGame(false); // 負け
    }
}

// ゲームループ
function gameLoop() {
    if (!gameRunning) return;

    updatePlayer();
    updatePlayerBullets();
    updateInvaders();
    updateInvaderBullets();
    checkCollisions();
    drawAll();

    requestAnimationFrame(gameLoop);
}

// ゲーム終了
function endGame(win) {
    gameRunning = false;
    if (win) {
        alert("ゲームクリア！おめでとう！スコア: " + score);
    } else {
        alert("ゲームオーバー！残念！スコア: " + score);
    }
    startButton.textContent = "再プレイ";
}

// キーボードイベントリスナー
document.addEventListener('keydown', (e) => {
    keys[e.code] = true;
    if (gameRunning) {
        if (e.code === 'ArrowLeft') {
            player.dx = -PLAYER_SPEED;
        } else if (e.code === 'ArrowRight') {
            player.dx = PLAYER_SPEED;
        } else if (e.code === 'Space') {
            // スペースキー長押しで連射しないように
            if (!keys['SpacePressed']) {
                playerBullets.push({
                    x: player.x + player.width / 2 - BULLET_WIDTH / 2,
                    y: player.y,
                    width: BULLET_WIDTH,
                    height: BULLET_HEIGHT
                });
                keys['SpacePressed'] = true;
            }
        }
    }
});

document.addEventListener('keyup', (e) => {
    keys[e.code] = false;
    if (e.code === 'ArrowLeft' || e.code === 'ArrowRight') {
        player.dx = 0;
    }
    if (e.code === 'Space') {
        keys['SpacePressed'] = false;
    }
});

// ゲーム開始ボタン
startButton.addEventListener('click', () => {
    if (!gameRunning) {
        initGame();
        gameRunning = true;
        startButton.textContent = "ゲーム中..."; // ゲーム中はボタンを無効化する方が良い
        startButton.disabled = true; // ボタンを無効化
        gameLoop();
    }
});

// 初期描画
drawAll();

<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>物理實驗室 - 牛頓運動定律</title>
    <style>
        :root {
            --primary-color: #2563eb;
            --secondary-color: #3b82f6;
            --accent-color: #f59e0b;
            --error-color: #ef4444;
            --success-color: #10b981;
            --bg-color: #f8fafc;
            --panel-bg: #ffffff;
            --text-color: #1e293b;
            --border-radius: 12px;
            --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', 'Microsoft JhengHei', sans-serif; }
        body { background-color: var(--bg-color); color: var(--text-color); height: 100vh; display: flex; flex-direction: column; overflow: hidden; }

        header {
            background: var(--panel-bg); padding: 12px 25px; box-shadow: var(--shadow);
            z-index: 10; flex-shrink: 0; display: flex; justify-content: space-between; align-items: center;
        }
        h1 { font-size: 1.3rem; color: var(--primary-color); }
        .subtitle { font-size: 0.85rem; color: #64748b; }

        .main-container { display: flex; flex: 1; overflow: hidden; padding: 15px; gap: 15px; height: 100%; }

        /* 左側控制面板 */
        .control-panel {
            background: var(--panel-bg); border-radius: var(--border-radius); padding: 20px;
            width: 320px; box-shadow: var(--shadow); display: flex; flex-direction: column;
            gap: 18px; overflow-y: auto; flex-shrink: 0;
        }
        .control-group { border-bottom: 1px solid #f1f5f9; padding-bottom: 15px; }
        label { display: block; margin-bottom: 8px; font-weight: 600; font-size: 0.9rem; color: #475569; }
        .value-display { font-size: 0.85rem; color: var(--primary-color); float: right; font-weight: bold; }
        input[type="range"] { width: 100%; accent-color: var(--primary-color); cursor: pointer; }

        .btn-group { display: flex; gap: 8px; margin-top: 5px; }
        button {
            flex: 1; padding: 10px; border: 1px solid #e2e8f0; background: #f8fafc;
            border-radius: 8px; cursor: pointer; transition: all 0.2s; font-size: 0.85rem; font-weight: 500;
        }
        button:hover { background: #f1f5f9; }
        button.active { background: var(--primary-color); color: white; border-color: var(--primary-color); }

        /* 中間畫布區域 (PhET 風格) */
        .visualization-area { flex: 1; display: flex; flex-direction: column; gap: 15px; min-width: 0; }
        .canvas-container {
            background: #000; border-radius: var(--border-radius); box-shadow: var(--shadow);
            flex: 1; position: relative; overflow: hidden;
        }
        canvas { width: 100%; height: 100%; display: block; }

        /* 數據監測欄 */
        .stats-panel {
            background: var(--panel-bg); border-radius: var(--border-radius); padding: 15px;
            box-shadow: var(--shadow); display: flex; justify-content: space-around;
            align-items: center; flex-shrink: 0; height: 90px;
        }
        .stat-item { text-align: center; }
        .stat-value { font-size: 1.2rem; font-weight: bold; color: var(--text-color); }
        .stat-label { font-size: 0.75rem; color: #64748b; margin-top: 4px; }
        .math-formula { font-family: 'Times New Roman', serif; font-style: italic; background: #eff6ff; padding: 6px 12px; border-radius: 6px; color: var(--primary-color); font-weight: bold; }

        /* 右側教學區 (獨立捲動) */
        .education-section {
            background: #fff; width: 380px; padding: 20px; border-radius: var(--border-radius);
            box-shadow: var(--shadow); overflow-y: auto; flex-shrink: 0; display: flex;
            flex-direction: column; gap: 15px;
        }
        .edu-card { background: #f8fafc; border-left: 4px solid var(--primary-color); padding: 15px; border-radius: 6px; }
        .edu-title { font-weight: bold; margin-bottom: 8px; color: #334155; }
        .edu-content { font-size: 0.88rem; line-height: 1.6; color: #475569; }

        /* 小考樣式 */
        .quiz-option {
            display: block; width: 100%; padding: 10px; margin: 8px 0; border: 1px solid #cbd5e1;
            background: white; border-radius: 6px; cursor: pointer; text-align: left; font-size: 0.85rem;
        }
        .quiz-option:hover { background: #f1f5f9; }
        .quiz-feedback { margin-top: 8px; font-weight: bold; font-size: 0.85rem; display: none; }

        @media (max-width: 1024px) {
            .main-container { flex-direction: column; overflow-y: auto; }
            .control-panel, .education-section { width: 100%; height: auto; flex: none; }
            .visualization-area { height: 400px; flex: none; }
            body { overflow: auto; height: auto; }
        }
    </style>
</head>
<body>

    <header>
        <div>
            <h1>牛頓運動定律</h1>
            <span class="subtitle">Newton's Laws of Motion</span>
        </div>
    </header>

    <div class="main-container">
        <div class="control-panel">
            <div class="control-group">
                <label>選擇觀測定律</label>
                <div class="btn-group">
                    <button onclick="setMode(1)" id="btn-m1" class="active">第一定律</button>
                    <button onclick="setMode(2)" id="btn-m2">第二定律</button>
                    <button onclick="setMode(3)" id="btn-m3">第三定律</button>
                </div>
            </div>

            <div class="control-group">
                <label>物體質量 (m) <span id="val-mass" class="value-display">2.0 kg</span></label>
                <input type="range" id="slider-mass" min="0.5" max="10" step="0.5" value="2.0">
            </div>

            <div class="control-group">
                <label>外加推力 (F) <span id="val-force" class="value-display">0 N</span></label>
                <input type="range" id="slider-force" min="-50" max="50" step="1" value="0">
            </div>

            <div class="control-group">
                <label>環境摩擦力</label>
                <div class="btn-group">
                    <button onclick="setFriction(0)" id="f-none" class="active">無</button>
                    <button onclick="setFriction(0.2)" id="f-low">冰面</button>
                    <button onclick="setFriction(0.5)" id="f-high">草地</button>
                </div>
            </div>

            <button style="background: var(--accent-color); color: white; border: none; margin-top: 10px; font-weight: bold; padding: 12px;" onclick="resetSim()">重置模擬系統</button>
        </div>

        <div class="visualization-area">
            <div class="canvas-container">
                <canvas id="simCanvas"></canvas>
            </div>
            
            <div class="stats-panel">
                <div class="stat-item">
                    <div class="stat-label">合力sum F</div>
                    <div class="stat-value" id="disp-force">0.0 N</div>
                </div>
                <div class="stat-item">
                    <div class="math-formula" id="formula-display">a = F / m</div>
                </div>
                <div class="stat-item">
                    <div class="stat-label">加速度 a</div>
                    <div class="stat-value" id="disp-accel">0.00 m/s²</div>
                </div>
                <div class="stat-item">
                    <div class="stat-label">當前速度 v</div>
                    <div class="stat-value" id="disp-vel">0.00 m/s</div>
                </div>
            </div>
        </div>

        <div class="education-section">
            <div class="edu-card">
                <div class="edu-title">📖 定律內容</div>
                <div class="edu-content" id="edu-content">
                    <strong>第一定律（慣性定律）：</strong><br>
                    若物體不受外力或所受合力為零，則靜者恆靜，動者恆作等速度直線運動。此性質稱為「慣性」。
                </div>
            </div>
            <div class="edu-card">
                <div class="edu-title">🖼️ 圖像理解</div>
                <div class="edu-content" id="img-content">
                    觀察模擬器：<br>
                    1. <strong>紅色箭頭</strong>代表您施加的推力。<br>
                    2. <strong>黃色箭頭</strong>代表阻礙運動的摩擦力。<br>
                    3. 當合力為零時，觀察速度計是否保持不變。
                </div>
            </div>
            
            <div class="edu-card">
                <div class="edu-title">⚠️ 適用範圍</div>
                <div class="edu-content">
                    適用於<strong>慣性參考系</strong>（Inertial Reference Frame）。在非慣性參考系中（如加速中的車廂），需引入虛擬力（慣性力）進行修正。
                </div>
            </div>
            <div class="edu-card">
                <div class="edu-title">🚀 常見應用</div>
                <div class="edu-content" id="app-content">
                    1. 乘客在剎車時向前傾倒。<br>
                    2. 拍打衣服上的灰塵使之脫落。
                </div>
            </div>
            <div class="edu-card">
                <div class="edu-title">🧪 實驗探究任務</div>
                <div class="edu-content" id="task-content">
                    將摩擦力設為「無」，給予物體一個短暫推力後歸零。觀察物體是否會因慣性而永不停止？
                </div>
            </div>
            <div class="edu-card" style="background: #fffbeb; border-color: #f59e0b;">
                <div class="edu-title">📝 隨堂小考</div>
                <div class="edu-content" id="quiz-container">
                    <p id="quiz-q">當合力為零時，原本運動中的物體會？</p>
                    <button class="quiz-option" onclick="answer(1)">立即停止</button>
                    <button class="quiz-option" onclick="answer(2)">繼續作等速度直線運動</button>
                    <button class="quiz-option" onclick="answer(3)">作變加速度運動</button>
                    <div id="quiz-feedback" class="quiz-feedback"></div>
                </div>
            </div>
        </div>
    </div>

<script>
    let state = {
        mode: 1, mass: 2.0, force: 0, mu: 0,
        vel: 0, accel: 0, posX: 100, running: true
    };

    const canvas = document.getElementById('simCanvas');
    const ctx = canvas.getContext('2d');
    const ui = {
        mass: document.getElementById('slider-mass'),
        force: document.getElementById('slider-force'),
        edu: document.getElementById('edu-content'),
        task: document.getElementById('task-content'),
        app: document.getElementById('app-content'),
        imgText: document.getElementById('img-content'),
        quizQ: document.getElementById('quiz-q'),
        quizOpts: document.querySelectorAll('.quiz-option')
    };

    function init() {
        resize();
        window.addEventListener('resize', resize);
        ui.mass.oninput = (e) => {
            state.mass = parseFloat(e.target.value);
            document.getElementById('val-mass').textContent = state.mass.toFixed(1) + " kg";
        };
        ui.force.oninput = (e) => {
            state.force = parseFloat(e.target.value);
            document.getElementById('val-force').textContent = state.force + " N";
        };
        loop();
    }

    function resize() {
        canvas.width = canvas.parentElement.clientWidth;
        canvas.height = canvas.parentElement.clientHeight;
    }

    function setMode(m) {
        state.mode = m;
        document.querySelectorAll('.btn-group button[id^="btn-"]').forEach(b => b.classList.remove('active'));
        document.getElementById('btn-m' + m).classList.add('active');
        
        const contents = [
            { 
                text: "<strong>第一定律（慣性定律）：</strong><br>若物體合力為零，則靜者恆靜，動者恆作等速度直線運動。", 
                task: "摩擦力歸零，給予瞬間推力後放開，驗證物體是否能保持原速。",
                app: "安全帶防止慣性衝擊、洗手後甩乾水滴。",
                img: "合力向量為零時，速度計指標將固定不跳動。",
                q: "當物體受合力不為零時，物體會？", a: ["維持原速", "產生加速度", "保持靜止"], c: 2
            },
            { 
                text: "<strong>第二定律（加速度定律）：</strong><br>物體的加速度 a 與合力 F 成正比，與質量 m 成反比。公式：F = ma。",
                task: "固定推力，將質量從 1kg 變為 10kg，觀察加速度如何劇減。",
                app: "賽車輕量化提升加速性能、推動不同重量的購物車所需的力不同。",
                img: "觀察紅色合力箭頭變長時，物體啟動變快的過程。",
                q: "相同推力下，質量增加 2 倍，加速度變為？", a: ["2 倍", "1/2 倍", "不變"], c: 2
            },
            { 
                text: "<strong>第三定律（作用與反作用）：</strong><br>兩物體間的交互作用力大小相等、方向相反，且作用在不同物體上。",
                task: "觀察模擬中小人推牆時，牆壁同時產生回彈方向的力。",
                app: "游泳撥水前進、噴射機引擎向後噴氣、划船划槳。",
                img: "觀察藍色作用力與紅色反作用力如何同時出現且方向相反。",
                q: "作用力與反作用力是否可以抵消？", a: ["可以，合力為零", "不可以，因為作用在不同受力物上", "看情況"], c: 2
            }
        ];
        
        const c = contents[m-1];
        ui.edu.innerHTML = c.text;
        ui.task.textContent = c.task;
        ui.app.innerHTML = c.app;
        ui.imgText.innerHTML = c.img;
        ui.quizQ.textContent = c.q;
        ui.quizOpts.forEach((btn, i) => btn.textContent = c.a[i]);
        document.getElementById('formula-display').textContent = m === 2 ? "F = m · a" : m === 1 ? "ΣF = 0" : "F₁₂ = -F₂₁";
        document.getElementById('quiz-feedback').style.display = "none";
        resetSim();
    }

    function answer(ans) {
        const correct = [2, 2, 2];
        const fb = document.getElementById('quiz-feedback');
        fb.style.display = "block";
        if (ans === correct[state.mode-1]) {
            fb.textContent = "✅ 正確！觀念完全正確。";
            fb.style.color = "var(--success-color)";
        } else {
            fb.textContent = "❌ 錯誤，再思考一下定律的核心定義。";
            fb.style.color = "var(--error-color)";
        }
    }

    function setFriction(mu) {
        state.mu = mu;
        document.querySelectorAll('[id^="f-"]').forEach(b => b.classList.remove('active'));
        const ids = {0:'f-none', 0.2:'f-low', 0.5:'f-high'};
        document.getElementById(ids[mu]).classList.add('active');
    }

    function resetSim() {
        state.posX = 50; state.vel = 0; state.accel = 0; state.force = 0;
        ui.force.value = 0; document.getElementById('val-force').textContent = "0 N";
    }

    function loop() {
        let fric = (state.vel === 0) ? 0 : -Math.sign(state.vel) * (state.mu * state.mass * 9.8);
        if (state.mode === 3 && state.force !== 0) fric = 0; // 第三定律純受力模擬
        
        let netF = state.force + fric;
        state.accel = netF / state.mass;
        state.vel += state.accel * (1/60);
        state.posX += state.vel * 40 * (1/60);

        if (state.posX < 0 || state.posX > canvas.width - 60) state.vel = 0;

        document.getElementById('disp-force').textContent = netF.toFixed(1) + " N";
        document.getElementById('disp-accel').textContent = state.accel.toFixed(2) + " m/s²";
        document.getElementById('disp-vel').textContent = state.vel.toFixed(2) + " m/s";

        draw();
        requestAnimationFrame(loop);
    }

    function draw() {
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        const ground = canvas.height - 50;
        ctx.strokeStyle = "#666"; ctx.lineWidth = 2;
        ctx.beginPath(); ctx.moveTo(0, ground); ctx.lineTo(canvas.width, ground); ctx.stroke();

        const size = 40 + state.mass * 2;
        const y = ground - size;

        ctx.fillStyle = "#3b82f6"; ctx.fillRect(state.posX, y, size, size);
        ctx.strokeStyle = "#fff"; ctx.strokeRect(state.posX, y, size, size);

        if (state.force !== 0) {
            drawArrow(state.posX + size/2, y - 20, state.posX + size/2 + state.force * 2, y - 20, "#ef4444", "推力 F");
            if (state.mode === 3) drawArrow(state.posX + size/2, y + size/2, state.posX + size/2 - state.force * 2, y + size/2, "#f59e0b", "反作用力");
        }
    }

    function drawArrow(fx, fy, tx, ty, color, label) {
        if (Math.abs(tx - fx) < 5) return;
        ctx.strokeStyle = color; ctx.fillStyle = color; ctx.lineWidth = 3;
        ctx.beginPath(); ctx.moveTo(fx, fy); ctx.lineTo(tx, ty); ctx.stroke();
        const head = (tx > fx) ? 10 : -10;
        ctx.beginPath(); ctx.moveTo(tx, ty); ctx.lineTo(tx-head, ty-5); ctx.lineTo(tx-head, ty+5); ctx.fill();
        ctx.fillStyle = "#fff"; ctx.font = "12px Arial";
        ctx.fillText(label, tx > fx ? tx + 5 : tx - 45, ty + 4);
    }

    init();
</script>
</body>
</html>

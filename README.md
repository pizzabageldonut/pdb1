<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pizza Bagel Donut: The Daily Logic Challenge 🍕</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        :root { --crust: #f1d5a7; --sauce: #d32f2f; --cheese: #ffeb3b; --dough: #ffffff; }
        body { font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; background: var(--crust); display: flex; flex-direction: column; align-items: center; padding: 20px; color: #2d2d2d; margin: 0; }
        
        .game-container { background: var(--dough); padding: 30px; border-radius: 20px; border: 6px solid var(--sauce); box-shadow: 0 15px 35px rgba(0,0,0,0.15); max-width: 400px; width: 100%; text-align: center; }
        
        h1 { margin: 0 0 10px 0; font-size: 1.8rem; letter-spacing: -1px; color: var(--sauce); }
        .player-badge { font-size: 0.8rem; background: #eee; padding: 5px 12px; border-radius: 15px; display: inline-block; margin-bottom: 20px; color: #666; font-weight: bold; }
        
        .how-to-play { text-align: left; background: #fff9c4; padding: 15px; border-radius: 10px; margin-bottom: 20px; font-size: 0.85rem; line-height: 1.4; border-left: 5px solid var(--sauce); }
        .rules-grid { display: grid; grid-template-columns: 1fr; gap: 8px; margin-top: 10px; }
        .rule-item { display: flex; align-items: center; gap: 10px; }

        .scoreboard { background: #2d2d2d; color: #fff; padding: 12px; border-radius: 10px; margin-bottom: 20px; display: grid; grid-template-columns: 1fr 1fr; }
        .stat-label { font-size: 0.65rem; color: #aaa; text-transform: uppercase; display: block; }
        .stat-val { font-size: 1.3rem; font-weight: bold; }

        input { width: 150px; font-size: 2.5rem; text-align: center; border: 3px solid #ddd; border-radius: 12px; margin-bottom: 15px; padding: 5px; outline-color: var(--sauce); }
        button { width: 100%; padding: 16px; background: var(--sauce); color: white; border: none; border-radius: 10px; font-size: 1.1rem; cursor: pointer; font-weight: bold; transition: 0.2s; }
        button:hover { background: #b71c1c; transform: translateY(-2px); }
        
        .log { margin-top: 25px; text-align: left; background: #fafafa; padding: 15px; border-radius: 10px; height: 180px; overflow-y: auto; font-family: 'Courier New', Courier, monospace; font-size: 0.95rem; border: 1px solid #eee; }
        .p-result { color: #2e7d32; font-weight: bold; }
        .b-result { color: #f57c00; font-weight: bold; }
        .d-result { color: #9e9e9e; font-weight: bold; }
        .win-splash { color: #2e7d32; font-weight: 900; animation: bounce 0.5s infinite; }
        
        footer { margin-top: 30px; font-size: 0.7rem; color: #8d6e63; text-align: center; opacity: 0.8; }
        @keyframes bounce { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.05); } }
    </style>
</head>
<body>

<div class="game-container">
    <h1>🍕 PIZZA BAGEL DONUT</h1>
    <div id="player-tag" class="player-badge">Loading Profile...</div>

    <div class="how-to-play">
        <strong>HOW TO PLAY:</strong><br>
        Guess the daily 3-digit code in 9 tries.
        <div class="rules-grid">
            <div class="rule-item">🍕 <strong>PIZZA:</strong> Correct number, correct spot.</div>
            <div class="rule-item">🥯 <strong>BAGEL:</strong> Correct number, wrong spot.</div>
            <div class="rule-item">🍩 <strong>DONUT:</strong> Number is not in the code.</div>
        </div>
    </div>

    <div class="scoreboard">
        <div><span class="stat-label">Season AVG</span><span class="stat-val" id="stat-avg">.000</span></div>
        <div><span class="stat-label">Games Played</span><span class="stat-val" id="stat-gp">0</span></div>
    </div>

    <div id="input-area">
        <input type="text" id="user-guess" maxlength="3" placeholder="000" inputmode="numeric">
        <button onclick="checkOrder()">Submit Order</button>
    </div>

    <div class="log" id="game-log"><strong>DAILY LOG:</strong><br>Taking your order...</div>
    <button id="share-btn" style="display:none; margin-top:10px; background:#43a047;" onclick="copyResult()">Share Results 📋</button>
</div>

<footer>
    &copy; 2026 Kelly Choi. All Rights Reserved.<br>
    pizzabageldonut.com
</footer>

<script>
    /* --- GAME CONFIG --- */
    const PUBLIC_KEY = "5fa8af5feb371a09c4c51d17"; 
    const PRIVATE_KEY  = "cgpr101Ep0yMn0IZPhMAqwVghoK20BG06c_rPh-i1Npg";
    /* ------------------- */

    const today = new Date().toISOString().slice(0, 10);
    const CODE = generateDailyCode();
    let tries = 0;
    const MAX_TRIES = 9;
    
    let stats = JSON.parse(localStorage.getItem('pbd_v1_stats')) || { games: 0, hits: 0, lastDate: "", nick: "" };

    if (!stats.nick) {
        let n = prompt("What should we call you on the leaderboard?");
        stats.nick = n ? n.substring(0, 12) : "Foodie" + Math.floor(Math.random()*999);
        localStorage.setItem('pbd_v1_stats', JSON.stringify(stats));
    }
    document.getElementById('player-tag').innerText = "📍 " + stats.nick;

    function generateDailyCode() {
        const seed = today.split('-').reduce((a, b) => parseInt(a) + parseInt(b), 0);
        let pool = [0,1,2,3,4,5,6,7,8,9];
        let result = "";
        for(let i=0; i<3; i++) {
            let idx = (seed + i * 17) % pool.length; 
            result += pool.splice(idx, 1);
        }
        return result;
    }

    async function sendToBoard(score) {
        const pts = 10 - score; 
        const url = `https://www.dreamlo.com/lb/${PRIVATE_KEY}/add/${encodeURIComponent(stats.nick)}/${pts}`;
        try { await fetch(url); loadLeaderboard(); } catch(e) {}
    }

    async function loadLeaderboard() {
        const url = `https://www.dreamlo.com/lb/${PUBLIC_KEY}/json`;
        try {
            const resp = await fetch(url);
            const data = await resp.json();
            if(!data.dreamlo.leaderboard) return;
            let html = "<strong>TOP FOODIES TODAY:</strong><br>";
            const entries = data.dreamlo.leaderboard.entry;
            const top = Array.isArray(entries) ? entries.slice(0, 5) : (entries ? [entries] : []);
            top.forEach(e => { html += `${e.name}: ${10 - e.score} tries<br>`; });
            document.getElementById('game-log').innerHTML = html;
        } catch (e) { }
    }

    function refreshUI() {
        const avg = stats.games === 0 ? ".000" : (stats.hits / stats.games).toFixed(3).replace(/^0/, '');
        document.getElementById('stat-avg').innerText = avg;
        document.getElementById('stat-gp').innerText = stats.games;
        if (stats.lastDate === today) {
            document.getElementById('input-area').innerHTML = "<h3>Great job! Come back tomorrow for a new code.</h3>";
            document.getElementById('share-btn').style.display = 'block';
            loadLeaderboard();
        }
    }

    function checkOrder() {
        const input = document.getElementById('user-guess');
        const val = input.value;
        if (val.length !== 3 || isNaN(val)) return;

        tries++;
        let p = 0, b = 0, d = 0;
        let guessArr = val.split(''), codeArr = CODE.split('');
        const log = document.getElementById('game-log');
        if (tries === 1) log.innerHTML = "";
        const row = document.createElement('div');

        if (val === CODE) {
            row.innerHTML = `[${tries}] ${val} → <span class="win-splash">🍕 PIZZA PIZZA PIZZA!</span>`;
            log.prepend(row);
            confetti({ particleCount: 150, spread: 70, origin: { y: 0.6 }, colors: ['#d32f2f', '#ffeb3b', '#ffffff'] });
            finish(true);
        } else {
            guessArr.forEach((num, i) => {
                if (num === codeArr[i]) { p++; } 
                else if (codeArr.includes(num)) { b++; } 
                else { d++; }
            });
            row.innerHTML = `[${tries}] ${val} → <span class="p-result">${p}P</span> <span class="b-result">${b}B</span> <span class="d-result">${d}D</span>`;
            log.prepend(row);
            if (tries >= MAX_TRIES) finish(false);
        }
        input.value = "";
    }

    function finish(isWin) {
        if (stats.lastDate !== today) {
            stats.games++;
            if (isWin) { stats.hits++; sendToBoard(tries); } 
            else { alert("Order closed! The code was " + CODE); }
            stats.lastDate = today;
            localStorage.setItem('pbd_v1_stats', JSON.stringify(stats));
        }
        refreshUI();
    }

    function copyResult() {
        const text = `🍕 Pizza Bagel Donut\nDaily Challenge\nScore: ${tries}/${MAX_TRIES}\nResult: PIZZA PIZZA PIZZA!\n#PizzaBagelDonut`;
        navigator.clipboard.writeText(text).then(() => alert("Results Copied!"));
    }

    refreshUI();
    if (stats.lastDate !== today) loadLeaderboard();
</script>
</body>
</html>

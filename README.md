<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Triple Play: Pizza, Bagel, Donut 🍕</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        :root { --crust: #e2b07e; --sauce: #b22222; --dough: #f4f4f4; --box: #fff; }
        body { font-family: 'Arial Black', sans-serif; background: var(--crust); display: flex; flex-direction: column; align-items: center; padding: 20px; color: #333; }
        .stadium { background: var(--box); padding: 25px; border-radius: 15px; border: 5px solid var(--sauce); box-shadow: 0 10px 30px rgba(0,0,0,0.3); max-width: 400px; width: 100%; text-align: center; }
        .scoreboard { background: #333; color: #fff; padding: 10px; font-family: 'Courier New', monospace; border-radius: 8px; margin-bottom: 20px; display: grid; grid-template-columns: 1fr 1fr; border: 2px solid #000; }
        .stat-label { font-size: 0.7rem; color: #ccc; display: block; }
        .player-tag { font-size: 0.8rem; color: #fff; background: var(--sauce); padding: 4px 12px; border-radius: 20px; margin-bottom: 10px; display: inline-block; }
        input { width: 140px; font-size: 2.5rem; text-align: center; border: 3px solid #333; border-radius: 10px; margin-bottom: 15px; letter-spacing: 5px; font-weight: bold; }
        button { width: 100%; padding: 15px; background: var(--sauce); color: white; border: none; border-radius: 8px; font-size: 1.1rem; cursor: pointer; font-weight: bold; text-transform: uppercase; }
        .log { margin-top: 20px; text-align: left; background: #fffdf0; padding: 12px; border-radius: 8px; height: 180px; overflow-y: auto; font-family: 'Courier New', monospace; font-size: 0.9rem; border: 1px solid #ddd; }
        .pizza { color: #28a745; font-weight: bold; }
        .bagel { color: #d4a017; font-weight: bold; }
        .donut { color: #999; font-weight: bold; }
        .win-text { color: #28a745; font-weight: bold; animation: blink 0.5s infinite; }
        @keyframes blink { 50% { opacity: 0; } }
    </style>
</head>
<body>

<div class="stadium">
    <h1 style="margin: 0 0 5px 0;">🍕 TRIPLE PLAY</h1>
    <div id="player-display" class="player-tag">Guest Player</div>
    
    <div style="font-size: 0.75rem; color: #555; margin-bottom: 15px; line-height: 1.4;">
        <strong>Pizza:</strong> Correct spot 🍕<br>
        <strong>Bagel:</strong> Correct number, wrong spot 🥯<br>
        <strong>Donut:</strong> Not in the code 🍩
    </div>

    <div class="scoreboard">
        <div><span class="stat-label">DAILY AVG</span><span class="stat-val" id="stat-avg">.000</span></div>
        <div><span class="stat-label">GAMES</span><span class="stat-val" id="stat-gp">0</span></div>
    </div>

    <div id="game-ui">
        <input type="text" id="guess" maxlength="3" placeholder="000" inputmode="numeric">
        <button onclick="makeGuess()">Submit Order</button>
    </div>

    <div class="log" id="log-content"><strong>KITCHEN LOG:</strong><br>Ready for orders...</div>
    <button id="share-btn" style="display:none; margin-top:10px; background:#28a745;" onclick="shareResults()">Copy Results 📋</button>
</div>

<script>
    /* --- CONFIGURATION --- */
    const PUBLIC_KEY = "5fa8af5feb371a09c4c51d17"; 
    const PRIVATE_KEY  = "cgpr101Ep0yMn0IZPhMAqwVghoK20BG06c_rPh-i1Npg";
    /* --------------------- */

    const today = new Date().toISOString().slice(0, 10);
    const SECRET_CODE = generateDailyCode();
    let attempts = 0;
    const MAX_ATTEMPTS = 9;
    
    let stats = JSON.parse(localStorage.getItem('pbd_stats')) || { games: 0, hits: 0, lastPlayed: "", playerName: "" };

    if (!stats.playerName) {
        let name = prompt("Enter a Player Name (max 10 chars):");
        stats.playerName = name ? name.substring(0, 10) : "Player" + Math.floor(Math.random()*999);
        localStorage.setItem('pbd_stats', JSON.stringify(stats));
    }
    document.getElementById('player-display').innerText = "👤 " + stats.playerName;

    function generateDailyCode() {
        const seed = today.split('-').reduce((a, b) => parseInt(a) + parseInt(b), 0);
        let nums = [0,1,2,3,4,5,6,7,8,9];
        let code = "";
        for(let i=0; i<3; i++) {
            let idx = (seed + i * 17) % nums.length; 
            code += nums.splice(idx, 1);
        }
        return code;
    }

    async function uploadScore(score) {
        const points = 10 - score; 
        const url = `https://www.dreamlo.com/lb/${PRIVATE_KEY}/add/${encodeURIComponent(stats.playerName)}/${points}`;
        try { await fetch(url); showGlobalLeaderboard(); } catch(e) { }
    }

    async function showGlobalLeaderboard() {
        const url = `https://www.dreamlo.com/lb/${PUBLIC_KEY}/json`;
        try {
            const response = await fetch(url);
            const data = await response.json();
            if(!data.dreamlo.leaderboard) return;
            let boardHtml = "<strong>GLOBAL TOP SCORES:</strong><br>";
            const scores = data.dreamlo.leaderboard.entry;
            const list = Array.isArray(scores) ? scores.slice(0, 5) : (scores ? [scores] : []);
            list.forEach(entry => { boardHtml += `${entry.name}: ${10 - entry.score} tries<br>`; });
            document.getElementById('log-content').innerHTML = boardHtml;
        } catch (e) { document.getElementById('log-content').innerHTML = "No scores recorded yet."; }
    }

    function updateDisplay() {
        const avg = stats.games === 0 ? ".000" : (stats.hits / stats.games).toFixed(3).replace(/^0/, '');
        document.getElementById('stat-avg').innerText = avg;
        document.getElementById('stat-gp').innerText = stats.games;
        if (stats.lastPlayed === today) {
            document.getElementById('game-ui').innerHTML = "<h3>See you tomorrow!</h3>";
            document.getElementById('share-btn').style.display = 'block';
            showGlobalLeaderboard();
        }
    }

    function makeGuess() {
        const input = document.getElementById('guess');
        const val = input.value;
        if (val.length !== 3 || isNaN(val)) return;

        attempts++;
        let p = 0, b = 0, d = 0;
        let guessArr = val.split(''), codeArr = SECRET_CODE.split('');
        const log = document.getElementById('log-content');
        if (attempts === 1) log.innerHTML = "";
        const entry = document.createElement('div');

        if (val === SECRET_CODE) {
            entry.innerHTML = `[${attempts}] ${val} → <span class="win-text">🍕 PIZZA PIZZA PIZZA!</span>`;
            log.prepend(entry);
            confetti({ particleCount: 150, spread: 70, origin: { y: 0.6 } });
            endGame(true);
        } else {
            guessArr.forEach((num, i) => {
                if (num === codeArr[i]) { p++; } 
                else if (codeArr.includes(num)) { b++; } 
                else { d++; }
            });
            entry.innerHTML = `[${attempts}] ${val} → <span class="pizza">${p}P</span> <span class="bagel">${b}B</span> <span class="donut">${d}D</span>`;
            log.prepend(entry);
            if (attempts >= MAX_ATTEMPTS) endGame(false);
        }
        input.value = "";
    }

    function endGame(win) {
        if (stats.lastPlayed !== today) {
            stats.games++;
            if (win) { stats.hits++; uploadScore(attempts); } 
            else { alert("Better luck tomorrow! The code was " + SECRET_CODE); }
            stats.lastPlayed = today;
            localStorage.setItem('pbd_stats', JSON.stringify(stats));
        }
        updateDisplay();
    }

    function shareResults() {
        const text = `🍕 Triple Play: Pizza Bagel Donut\nScore: ${attempts}/${MAX_ATTEMPTS}\nResult: PIZZA PIZZA PIZZA! 🍕\n#TriplePlayGame`;
        navigator.clipboard.writeText(text).then(() => alert("Results Copied!"));
    }

    updateDisplay();
    if (stats.lastPlayed !== today) showGlobalLeaderboard();
</script>
</body>
</html>

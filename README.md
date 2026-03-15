<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pizza Bagel Donut</title>
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        :root { --crust: #f1d5a7; --sauce: #d32f2f; --cheese: #ffeb3b; --dough: #ffffff; }
        body { font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; background: var(--crust); display: flex; flex-direction: column; align-items: center; padding: 20px; color: #2d2d2d; margin: 0; }
        
        .game-container { background: var(--dough); padding: 30px; border-radius: 20px; border: 6px solid var(--sauce); box-shadow: 0 15px 35px rgba(0,0,0,0.15); max-width: 400px; width: 100%; text-align: center; }
        
        /* BIG EMOJIS */
        .icon-header { font-size: 4.5rem; line-height: 1; margin-bottom: 0px; padding-top: 10px; }
        
        /* SMALLER TITLE */
        h1 { margin: 5px 0 20px 0; font-size: 1.3rem; letter-spacing: 2px; color: var(--sauce); text-transform: uppercase; font-weight: 900; }
        
        .player-badge { font-size: 0.8rem; background: #f5f5f5; padding: 5px 12px; border-radius: 15px; display: inline-block; margin-bottom: 20px; color: #666; font-weight: bold; border: 1px solid #eee; }
        
        .scoreboard { background: #2d2d2d; color: #fff; padding: 12px; border-radius: 10px; margin-bottom: 20px; display: grid; grid-template-columns: 1fr 1fr; }
        .stat-label { font-size: 0.6rem; color: #aaa; text-transform: uppercase; display: block; margin-bottom: 2px; }
        .stat-val { font-size: 1.3rem; font-weight: bold; }

        input { width: 150px; font-size: 2.5rem; text-align: center; border: 3px solid #ddd; border-radius: 12px; margin-bottom: 15px; padding: 5px; outline-color: var(--sauce); font-weight: bold; }
        button { width: 100%; padding: 16px; background: var(--sauce); color: white; border: none; border-radius: 10px; font-size: 1.1rem; cursor: pointer; font-weight: bold; transition: 0.2s; text-transform: uppercase; }
        button:hover { background: #b71c1c; transform: translateY(-2px); }
        
        .log { margin-top: 25px; text-align: left; background: #fafafa; padding: 15px; border-radius: 10px; height: 180px; overflow-y: auto; font-family: 'Courier New', Courier, monospace; font-size: 1.1rem; border: 1px solid #eee; }
        
        .how-to-play { text-align: left; background: #fff9c4; padding: 15px; border-radius: 10px; margin-top: 2

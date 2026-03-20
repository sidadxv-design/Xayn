html
<!DOCTYPE html>
<html lang="ku" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Undercover - Mystery Edition</title>
    <style>
        * { -webkit-tap-highlight-color: transparent; box-sizing: border-box; }
        body {
            font-family: -apple-system, system-ui, sans-serif;
            background: #0a0a0a; color: white; margin: 0; padding: 15px;
            display: flex; justify-content: center; min-height: 100vh;
        }
        .container {
            width: 100%; max-width: 400px; background: #151515;
            padding: 25px; border-radius: 25px; border: 1px solid #222;
            box-shadow: 0 15px 40px rgba(0,0,0,0.6);
        }
        h1 { color: #2ecc71; text-align: center; font-size: 26px; margin-bottom: 20px; }
        input, button {
            width: 100%; height: 55px; border-radius: 16px; border: none;
            margin-bottom: 12px; font-size: 16px; font-weight: 700;
        }
        input { padding: 0 15px; background: #222; color: white; border: 1px solid #333; }
        input:focus { border-color: #2ecc71; outline: none; }
        .add-btn { background: #27ae60; color: white; }
        .start-btn { background: #e67e22; color: white; margin-top: 10px; }
        .next-btn { background: #3498db; color: white; }
        .reset-btn { background: #c0392b; color: white; margin-top: 20px; }

        .player-list { background: #1d1d1d; border-radius: 16px; margin-bottom: 15px; max-height: 180px; overflow-y: auto; }
        .player-item { display: flex; justify-content: space-between; padding: 12px 18px; border-bottom: 1px solid #252525; }
        
        .card {
            background: #f1c40f; color: #1a1a1a; padding: 25px;
            border-radius: 20px; text-align: center; margin: 25px 0;
            min-height: 160px; display: flex; align-items: center; justify-content: center;
            font-size: 22px; font-weight: 800; cursor: pointer;
            box-shadow: 0 10px 20px rgba(241, 196, 15, 0.2);
        }
        
        .hidden { display: none !important; }

        .result-item {
            background: #1d1d1d; padding: 15px; border-radius: 14px;
            margin-bottom: 10px; border-right: 5px solid #333;
        }
        .is-imp-border { border-right-color: #e74c3c !important; }
        .q-label { font-size: 13px; color: #888; display: block; margin-bottom: 4px; }
    </style>
</head>
<body>

<div class="container">
    <div id="setup-screen">
        <h1>🕵️‍♂️ یارییا ڤەشارتی</h1>
        <p style="text-align: center; color: #777; font-size: 14px; margin-top: -10px;">هۆست ژی وەک یاریزان ناڤێ خۆ بنڤیسە</p>
        <div style="display: flex; gap: 8px;">
            <input type="text" id="player-name" placeholder="ناڤێ تە چییە؟" style="flex:1">
            <button class="add-btn" id="add-btn" style="width: 70px;">+</button>
        </div>
        <div id="player-list-ui" class="player-list"></div>
        <button id="start-btn" class="start-btn hidden">دەستپێکرن ▶️</button>
    </div>

    <div id="game-screen" class="hidden">
        <h2 id="turn-title" style="text-align: center; color: #f1c40f;"></h2>
        <p id="instruction" style="text-align: center; color: #888; font-size: 14px;">ل کارتێ بدە دا پسیارا خۆ ببینی</p>
        
        <div id="card" class="card">نیشان بدە 👁️</div>
        
        <div id="answer-area" class="hidden">
            <input type="text" id="player-ans" placeholder="بەرسڤا تە چییە؟" enterkeyhint="done">
            <button class="next-btn" id="submit-btn">پاشکەفتن ⏭️</button>
        </div>
    </div>

    <div id="result-screen" class="hidden">
        <h2 style="text-align: center; color: #2ecc71;">ئەنجامێن یاریێ ✅</h2>
        
        <div style="background: #222; padding: 15px; border-radius: 15px; margin-bottom: 20px;">
            <p style="margin: 5px 0; text-align: center;"><b style="color:#2ecc71">پسیارا دروست ئەڤە بوو:</b><br> <span id="res-normal-q" style="font-size: 20px;"></span></p>
            </div>

        <div id="results-list"></div>
        
        <button class="reset-btn" id="restart-btn">گەڕەکەکا نوی 🔄</button>
    </div>
</div>

<script>
    const db = [
        { n: "خۆشترین کاتی ڕۆژێ چییە؟", i: "درێژترین کاتی ڕۆژێ چییە؟" },
        { n: "تشتەکێ بێزارکەر د ناڤ جاددێ دا؟", i: "تشتەکێ مەترسیدار د ناڤ جاددێ دا؟" },
        { n: "ئەگەر ببیە ئاژەڵ، دێ حەز کەی چ بی؟", i: "ئەگەر ببیە ئاژەڵەکێ درندە، دێ چ بی؟" },
        { n: "تشتەک چییە مرۆڤی بەختەوەر دکەت؟", i: "تشتەک چییە مرۆڤی دەوڵەمەند دکەت؟" },
        { n: "گرنگترین تشت د ناڤ مالێ دا؟", i: "گرانترین تشت د ناڤ مالێ دا؟" },
        { n: "ب دیتنا تە 'عەشق' چییە؟", i: "ب دیتنا تە 'غیرە' چییە؟" },
        { n: "باشترین جهـ بۆ گەشتێ؟", i: "خۆشترین جهـ بۆ ژیانێ؟" },
        { n: "کەسەکێ 'ئاقڵ' ب دیتنا تە کێیە؟", i: "کەسەکێ 'سەیر' ب دیتنا تە کێیە؟" },
        { n: "تشتەک کو مۆدێ تە خۆش دکەت؟", i: "تشتەک کو تە تووڕە دکەت؟" }
    ];

    let players = [];
    let state = { normalQ: "", impQ: "", impIndex: -1, current: 0, answers: [] };

    document.getElementById('add-btn').onclick = () => {
        const val = document.getElementById('player-name').value.trim();
        if(val) { players.push(val); document.getElementById('player-name').value = ""; render(); }
    };

    function render() {
        document.getElementById('player-list-ui').innerHTML = players.map((p, i) => `<div class="player-item"><b>${i+1}.</b> ${p}</div>`).join('');
        document.getElementById('start-btn').classList.toggle('hidden', players.length < 3);
    }

    document.getElementById('start-btn').onclick = () => {
        const pair = db[Math.floor(Math.random() * db.length)];
        state = { 
            normalQ: pair.n, impQ: pair.i, 
            impIndex: Math.floor(Math.random() * players.length), 
            current: 0, answers: [] 
        };
        document.getElementById('setup-screen').classList.add('hidden');
        document.getElementById('game-screen').classList.remove('hidden');
        showTurn();
    };

    function showTurn() {
        document.getElementById('turn-title').innerText = "نۆکا ( " + players[state.current] + " )";
        document.getElementById('card').innerText = "نیشان بدە 👁️";
        document.getElementById('answer-area').classList.add('hidden');
        document.getElementById('player-ans').value = "";
    }

    document.getElementById('card').onclick = function() {
        if(!document.getElementById('answer-area').classList.contains('hidden')) return;
        const isImp = state.current === state.impIndex;
        this.innerText = isImp ? state.impQ : state.normalQ;
        document.getElementById('answer-area').classList.remove('hidden');
    };

    document.getElementById('submit-btn').onclick = () => {
        const val = document.getElementById('player-ans').value.trim() || "بێ بەرسڤ";
        state.answers.push({ name: players[state.current], ans: val, isImp: state.current === state.impIndex });

        if(state.current < players.length - 1) {
            state.current++;
            showTurn();
        } else {
            showResults();
        }
    };

    function showResults() {
        document.getElementById('game-screen').classList.add('hidden');
        document.getElementById('result-screen').classList.remove('hidden');
        document.getElementById('res-normal-q').innerText = state.normalQ;

        document.getElementById('results-list').innerHTML = state.answers.map(a => `
            <div class="result-item ${a.isImp ? 'is-imp-border' : ''}">
                <span class="q-label">${a.name} گۆت:</span>
                <div style="font-size:18px; font-weight:700;">${a.ans} ${a.isImp ? '<span style="color:#e74c3c; font-size:12px;"> (خائین 🕵️‍♂️)</span>' : ''}</div>
            </div>
        `).join('');
    }

    document.getElementById('restart-btn').onclick = () => {
        document.getElementById('result-screen').classList.add('hidden');
        document.getElementById('setup-screen').classList.remove('hidden');
    };
</script>

</body>
</html>

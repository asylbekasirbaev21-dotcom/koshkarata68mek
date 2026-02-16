<!DOCTYPE html>
<html lang="kk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Koshkar Ata Health Analyzer</title>
    <style>
        :root {
            --primary: #00d2ff;
            --secondary: #3a7bd5;
            --bg: #0f172a;
            --card-bg: rgba(255, 255, 255, 0.05);
        }

        body {
            background: var(--bg);
            color: white;
            font-family: 'Segoe UI', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }

        .test-container {
            background: var(--card-bg);
            backdrop-filter: blur(15px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            padding: 40px;
            border-radius: 24px;
            width: 90%;
            max-width: 600px;
            box-shadow: 0 25px 50px rgba(0, 0, 0, 0.5);
        }

        .progress-container {
            width: 100%;
            height: 8px;
            background: rgba(255,255,255,0.1);
            border-radius: 10px;
            margin-bottom: 30px;
        }

        #progress-bar {
            width: 0%;
            height: 100%;
            background: linear-gradient(to right, var(--primary), var(--secondary));
            border-radius: 10px;
            transition: 0.4s;
        }

        h2 { color: var(--primary); margin-bottom: 20px; font-size: 22px; }
        
        .options-grid { display: grid; gap: 10px; }

        .btn {
            background: rgba(255,255,255,0.1);
            border: 1px solid rgba(255,255,255,0.2);
            padding: 15px;
            border-radius: 12px;
            color: white;
            cursor: pointer;
            transition: 0.3s;
            text-align: center;
            font-size: 16px;
        }

        .btn:hover { background: var(--primary); color: #000; font-weight: bold; }

        input {
            width: 100%;
            padding: 15px;
            background: rgba(255,255,255,0.1);
            border: 1px solid rgba(255,255,255,0.2);
            border-radius: 12px;
            color: white;
            margin-bottom: 20px;
            box-sizing: border-box;
        }

        #results { display: none; }
        .res-card { 
            background: rgba(0, 210, 255, 0.1); 
            padding: 20px; 
            border-radius: 15px; 
            border-left: 5px solid var(--primary);
            line-height: 1.6;
        }
    </style>
</head>
<body>

<div class="test-container">
    <div class="progress-container"><div id="progress-bar"></div></div>
    
    <div id="quiz-ui">
        <h2 id="question-text">Жүктелуде...</h2>
        <div id="input-area"></div>
        <div id="options-area" class="options-grid"></div>
    </div>

    <div id="results">
        <h2 id="res-title" style="color: #38ef7d;">📊 Нәтиже</h2>
        <div class="res-card" id="res-body"></div>
        <button class="btn" style="width:100%; margin-top:20px;" onclick="location.reload()">Қайта бастау</button>
    </div>
</div>

<script>
    // 11 сұрақтан тұратын массив
    const questions = [
        { q: "Сәлем! Жасыңыз нешеде?", type: "number", key: "age" },
        { q: "Созылмалы ауруларыңыз (қан қысымы, буын т.б.) бар ма?", type: "choice", key: "ill", options: ["Иә, бар", "Жоқ, денім сау"] },
        { q: "Қошқар атаға келудегі басты мақсатыңыз?", type: "choice", key: "goal", options: ["Емделу", "Шынығу", "Тек қыдыру"] },
        { q: "Суға шомылғаннан кейін қандай сезімде боласыз?", type: "choice", key: "feel", options: ["Жеңілдік", "Сергектік", "Шаршау"] },
        { q: "Судың температурасы сізге қатты батпай ма?", type: "choice", key: "temp", options: ["Жоқ, үйреніп кеттім", "Өте суық", "Денем мұздайды"] },
        { q: "Аптасына неше рет келесіз?", type: "choice", key: "freq", options: ["Күнде", "1-2 рет", "Өте сирек"] },
        { q: "Суға түскен соң денеңізде бөртпе немесе аллергия болды ма?", type: "choice", key: "allergy", options: ["Мүлдем болған емес", "Иә, болды"] },
        { q: "Бұлақ суын ішуге пайдаланасыз ба?", type: "choice", key: "drink", options: ["Иә, пайдалы", "Жоқ, тек шомыламын"] },
        { q: "Суға түсу ұйқыңызды реттеді ме?", type: "choice", key: "sleep", options: ["Иә, ұйқым жақсарды", "Әсері болмады"] },
        { q: "Қыс мезгілінде суға түсіп көрдіңіз бе?", type: "choice", key: "winter", options: ["Иә, моржбын", "Жоқ, қорқамын"] },
        { q: "Бұл жерді басқаларға ұсынар ма едіңіз?", type: "choice", key: "recom", options: ["Иә, әрине", "Жоқ, кеңес бермеймін"] }
    ];

    let currentStep = 0;
    let userAnswers = {};

    function updateUI() {
        const qData = questions[currentStep];
        const progress = (currentStep / questions.length) * 100;
        document.getElementById('progress-bar').style.width = progress + "%";
        document.getElementById('question-text').innerText = (currentStep + 1) + ". " + qData.q;

        const inputArea = document.getElementById('input-area');
        const optionsArea = document.getElementById('options-area');

        // Тазалау
        inputArea.innerHTML = "";
        optionsArea.innerHTML = "";

        if (qData.type === "number") {
            const input = document.createElement("input");
            input.type = "number";
            input.id = "val-input";
            input.placeholder = "Мысалы: 25";
            
            const btn = document.createElement("button");
            btn.className = "btn";
            btn.style.width = "100%";
            btn.innerText = "Келесі";
            btn.onclick = function() {
                const v = document.getElementById('val-input').value;
                if(v) saveNext(v); else alert("Жауапты жазыңыз");
            };
            
            inputArea.appendChild(input);
            inputArea.appendChild(btn);
        } else {
            qData.options.forEach(opt => {
                const btn = document.createElement("button");
                btn.className = "btn";
                btn.innerText = opt;
                btn.onclick = () => saveNext(opt);
                optionsArea.appendChild(btn);
            });
        }
    }

    function saveNext(val) {
        userAnswers[questions[currentStep].key] = val;
        currentStep++;
        if (currentStep < questions.length) updateUI(); else showResult();
    }

    function showResult() {
        document.getElementById('quiz-ui').style.display = "none";
        document.getElementById('progress-bar').style.width = "100%";
        document.getElementById('results').style.display = "block";
        
        const body = document.getElementById('res-body');
        let txt = "Пайдаланушы жасы: " + userAnswers.age + ". ";

        if(userAnswers.winter === "Иә, моржбын") {
            txt += "Сіз нағыз титансыз! Қыста суға түсу — мықты денсаулықтың белгісі.";
        } else if(userAnswers.goal === "Емделу") {
            txt += "Судың минералды құрамы сіздің ағзаңызды қалпына келтіруге көмектеседі. Үнемі келіп тұруды ұсынамыз.";
        } else {
            txt += "Қошқар ата суы — сіз үшін энергия мен сергектік көзі. Жақсы таңдау!";
        }
        
        body.innerText = txt;
    }

    updateUI();
</script>

</body>
</html>

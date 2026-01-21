<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>114義消尾牙 - 座位導覽圖</title>
    <style>
        /* --- 全局設定 --- */
        :root {
            --primary-red: #d32f2f;
            --table-main: #ff5252;   /* 主桌紅 */
            --table-vip: #ff9800;    /* 貴賓橘 */
            --table-blue: #2196f3;   /* 分隊藍 */
            --table-green: #4caf50;  /* 其他綠 */
            --bg-color: #f0f2f5;
        }

        body {
            font-family: "Microsoft JhengHei", "Heiti TC", sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            display: flex;
            flex-direction: column;
            height: 100vh;
            overflow: hidden;
        }

        /* --- 頂部搜尋列 --- */
        .header {
            background: white;
            padding: 15px 20px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
            z-index: 100;
            display: flex;
            flex-direction: column;
            align-items: center;
            flex-shrink: 0;
        }

        h1 { margin: 0 0 10px 0; color: #333; font-size: 1.5rem; }

        .search-container {
            position: relative;
            width: 100%;
            max-width: 500px;
            display: flex;
            gap: 10px;
        }

        input {
            width: 100%;
            padding: 12px 20px;
            border: 2px solid #ddd;
            border-radius: 50px;
            font-size: 16px;
            outline: none;
            transition: 0.3s;
        }
        input:focus { border-color: var(--primary-red); }

        /* --- 地圖區域 (核心佈局) --- */
        .map-wrapper {
            flex-grow: 1;
            overflow: auto; /* 允許地圖捲動 */
            padding: 20px;
            display: flex;
            justify-content: center;
            /* 背景網格裝飾 */
            background-image: radial-gradient(#cbd5e0 1px, transparent 1px);
            background-size: 20px 20px;
        }

        .map-content {
            position: relative;
            width: 1200px; /* 地圖固定寬度，確保不跑版 */
            min-height: 800px;
            background: white;
            border-radius: 20px;
            box-shadow: 0 0 20px rgba(0,0,0,0.05);
            padding: 20px;
            box-sizing: border-box;
        }

        /* 舞台 */
        .stage {
            width: 400px;
            height: 60px;
            background: #3f51b5;
            color: white;
            font-size: 24px;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            margin: 0 auto 30px auto;
            border-radius: 0 0 20px 20px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
        }

        /* 佈局容器：使用 Flexbox 模擬圖片的三大區塊 */
        .layout-row {
            display: flex;
            justify-content: center;
            gap: 40px;
        }

        /* 左側區塊 */
        .group-left {
            display: grid;
            grid-template-columns: 1fr 1fr; /* 兩排 */
            gap: 20px;
            width: 250px;
        }

        /* 中間星光大道區塊 */
        .group-center {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 160px;
        }

        /* 星光大道文字 */
        .aisle-text {
            writing-mode: vertical-rl;
            font-size: 30px;
            color: #ccc;
            letter-spacing: 20px;
            font-weight: bold;
            margin: 50px 0;
            border-left: 2px dashed #ddd;
            border-right: 2px dashed #ddd;
            padding: 0 20px;
            height: 400px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* 右側區塊 (含最右邊的複雜區) */
        .group-right {
            display: flex;
            gap: 30px;
        }
        
        .col-right-inner {
            display: grid;
            grid-template-columns: 1fr 1fr; /* 兩排 */
            gap: 20px;
            width: 250px;
        }

        .col-right-far {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr; /* 最右邊三排 */
            gap: 15px;
            width: 380px;
            align-content: start;
        }

        /* --- 桌子樣式 --- */
        .table {
            width: 90px;
            height: 90px;
            background: var(--table-green);
            color: white;
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
            font-size: 14px;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0,0,0,0.15);
            transition: transform 0.2s;
            position: relative;
            border: 3px solid white;
            line-height: 1.2;
            padding: 5px;
        }
        .table:hover { transform: scale(1.1); z-index: 10; }
        .table span { font-size: 12px; font-weight: normal; opacity: 0.9; }

        /* 特殊桌顏色 */
        .t-main { background: var(--table-main); width: 110px; height: 110px; font-size: 18px; z-index: 5; } /* 主桌大一點 */
        .t-vip { background: var(--table-vip); }
        .t-blue { background: var(--table-blue); }

        /* 搜尋亮起特效 */
        .highlight {
            background-color: var(--primary-red) !important;
            box-shadow: 0 0 0 5px #ffcdd2, 0 0 30px var(--primary-red);
            transform: scale(1.15);
            animation: blink 1s infinite alternate;
            z-index: 20;
        }
        @keyframes blink { from { opacity: 1; } to { opacity: 0.7; } }

        /* --- 彈出視窗 --- */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.5);
            z-index: 200;
            display: none;
            justify-content: center;
            align-items: center;
        }
        .modal {
            background: white;
            width: 90%;
            max-width: 400px;
            border-radius: 15px;
            padding: 20px;
            max-height: 80vh;
            overflow-y: auto;
            box-shadow: 0 10px 25px rgba(0,0,0,0.3);
        }
        .modal-header {
            display: flex; justify-content: space-between; align-items: center;
            border-bottom: 1px solid #eee; padding-bottom: 10px; margin-bottom: 10px;
        }
        .modal-title { font-size: 1.5rem; color: var(--primary-red); font-weight: bold; }
        .close-btn { font-size: 24px; cursor: pointer; background: none; border: none; color: #888; }
        .list-item { padding: 8px 0; border-bottom: 1px dashed #eee; font-size: 16px; }
        .list-item b { color: #333; }
        .match-name { color: white; background: var(--primary-red); padding: 2px 5px; border-radius: 4px; }

        /* 底部入口 */
        .entrance {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            border: 2px solid #333;
            padding: 10px 30px;
            font-weight: bold;
            color: #333;
            background: #fff;
        }
    </style>
</head>
<body>

    <div class="header">
        <h1>🧧 114義消尾牙座次圖</h1>
        <div class="search-container">
            <input type="text" id="searchInput" placeholder="請輸入姓名 (例如：林謙志)">
        </div>
    </div>

    <div class="map-wrapper">
        <div class="map-content">
            <div class="stage">舞 台 (STAGE)</div>

            <div class="layout-row">
                
                <div class="group-left">
                    <div class="table t-vip" data-label="親友桌 14">親友<br>14</div>
                    <div class="table t-vip" data-label="親友桌 15">親友<br>15</div>
                    <div class="table t-green" data-label="第四大隊 17-25">大肚<br>守望</div> <div class="table t-vip" data-label="四大港桌 12">貴賓<br>12</div>
                    <div class="table t-blue" data-label="中龍分隊 39">中龍<br>39</div> <div class="table t-blue" data-label="中龍分隊 40">中龍<br>40</div>

                    <div class="table t-vip" data-label="貴賓桌 3">貴賓<br>3</div>
                    <div class="table t-vip" data-label="貴賓桌 6">貴賓<br>6</div>
                    <div class="table t-vip" data-label="顧問桌 9">顧問<br>9</div>
                    <div class="table t-blue" data-label="第二中隊 28">二中隊<br>28</div>
                    <div class="table t-blue" data-label="南堤分隊 31">南堤<br>31</div>
                </div>

                <div class="group-center">
                    <div style="display:flex; gap:20px; margin-bottom: 20px;">
                        <div class="table t-main" data-label="主桌1">主桌<br>1</div>
                        <div class="table t-main" data-label="主桌2 (議員)">主桌<br>2</div>
                    </div>
                    <div class="aisle-text">星 光 大 道</div>
                </div>

                <div class="group-right">
                    
                    <div class="col-right-inner">
                        <div class="table t-vip" data-label="貴賓桌4 (風電)">貴賓<br>4</div>
                        <div class="table t-vip" data-label="顧問桌 7">顧問<br>7</div>
                        <div class="table t-vip" data-label="顧問桌 10">顧問<br>10</div>
                        <div class="table t-blue" data-label="第一中隊 26-27">一中隊<br>26</div>
                        <div class="table t-blue" data-label="第一分隊 29-30">第一<br>30</div>
                        <div class="table t-blue" data-label="第一分隊 29-30">第一<br>31</div> <div class="table t-vip" data-label="副團長桌5">副團長<br>5</div>
                        <div class="table t-vip" data-label="顧問桌 8">顧問<br>8</div>
                        <div class="table t-vip" data-label="貴賓桌11">港警<br>11</div>
                        <div class="table t-blue" data-label="第一中隊 26-27">一中隊<br>27</div>
                        <div class="table t-blue" data-label="防風林分隊 34">防風林<br>34</div>
                        <div class="table t-blue" data-label="合桌 32">合桌<br>33</div>
                    </div>

                    <div class="col-right-far">
                        <div class="table t-vip" data-label="四大港桌 13">四港<br>13</div>
                        <div class="table t-green" data-label="第四大隊 17-25">第四<br>大隊</div>
                        <div class="table t-green" data-label="第四大隊 17-25">清泉</div>
                        
                        <div class="table t-vip" data-label="四大港桌 14">四港<br>14</div>
                        <div class="table t-green" data-label="第四大隊 17-25">大肚</div>
                        <div class="table t-green" data-label="第四大隊 17-25">清水</div>

                        <div class="table t-blue" data-label="西碼頭分隊 36">西碼頭<br>36</div>
                        <div class="table t-green" data-label="第四大隊 17-25">龍井</div>
                        <div class="table t-green" data-label="第四大隊 17-25">犁份</div>

                        <div class="table t-blue" data-label="西碼頭分隊 37">西碼頭<br>37</div>
                        <div class="table t-green" data-label="第四大隊 17-25">沙鹿</div>
                        <div class="table t-green" data-label="第四大隊 17-25">梧棲</div>

                        <div class="table t-blue" data-label="西碼頭分隊 38">西碼頭<br>38</div>
                        <div class="table t-blue" data-label="中龍分隊 39">西碼頭<br>39</div> <div class="table t-green" data-label="第四大隊 17-25">海爆</div>

                        <div class="table t-blue" data-label="西碼頭分隊 35">防風林<br>35</div>
                        <div class="table t-blue" data-label="中龍分隊 39">童綜合</div> <div class="table t-blue" data-label="中龍分隊 39">光田</div>
                    </div>
                </div>
            </div>

            <div class="entrance">入 口</div>
        </div>
    </div>

    <div class="modal-overlay" id="modal">
        <div class="modal">
            <div class="modal-header">
                <div class="modal-title" id="modalTitle">桌名</div>
                <button class="close-btn" onclick="closeModal()">×</button>
            </div>
            <div id="modalContent"></div>
        </div>
    </div>

    <script>
        // ==========================================
        // 📋 114 義消尾牙完整資料 (已對應圖片邏輯)
        // ==========================================
        const rawData = [
            { "n": "林謙志", "s": "主桌1" }, { "n": "駱啟明", "s": "主桌1" }, { "n": "孫福佑", "s": "主桌1" }, { "n": "陳高尚", "s": "主桌1" }, { "n": "林志宏", "s": "主桌1" }, { "n": "吳瓊華", "s": "主桌1" }, { "n": "孫文山", "s": "主桌1" }, { "n": "王俊傑", "s": "主桌1" }, { "n": "魏福添", "s": "主桌1" }, { "n": "張家豪", "s": "主桌1" }, { "n": "李文義", "s": "主桌1" }, { "n": "游永中", "s": "主桌1" },
            { "n": "陳俊青", "s": "主桌2 (議員)" }, { "n": "趙彬然", "s": "主桌2 (議員)" }, { "n": "林瑞才", "s": "主桌2 (議員)" }, { "n": "賴俊男", "s": "主桌2 (議員)" }, { "n": "曾百溪", "s": "主桌2 (議員)" }, { "n": "林昊佑", "s": "主桌2 (議員)" }, { "n": "林茂發", "s": "主桌2 (議員)" }, { "n": "張東玄", "s": "主桌2 (議員)" }, { "n": "吳進宗", "s": "主桌2 (議員)" }, { "n": "陳義方", "s": "主桌2 (議員)" }, { "n": "陳木生", "s": "主桌2 (議員)" }, { "n": "張家銨", "s": "主桌2 (議員)" },
            { "n": "曾星明", "s": "貴賓桌3" }, { "n": "林暉智", "s": "貴賓桌3" }, { "n": "沈明賢", "s": "貴賓桌3" }, { "n": "李文彬", "s": "貴賓桌3" }, { "n": "蔡清松", "s": "貴賓桌3" }, { "n": "余凌冲", "s": "貴賓桌3" }, { "n": "張道銘", "s": "貴賓桌3" },
            { "n": "王志龍", "s": "貴賓桌4 (風電)" }, { "n": "洪瑞添", "s": "貴賓桌4 (風電)" }, { "n": "黃憲章", "s": "貴賓桌4 (風電)" }, { "n": "黃國誌", "s": "貴賓桌4 (風電)" }, { "n": "廖光政", "s": "貴賓桌4 (風電)" }, { "n": "林永晟", "s": "貴賓桌4 (風電)" }, { "n": "許鴻茗", "s": "貴賓桌4 (風電)" }, { "n": "喬永福", "s": "貴賓桌4 (風電)" }, { "n": "孫境堯", "s": "貴賓桌4 (風電)" },
            { "n": "林美秀", "s": "副團長桌5" }, { "n": "林曼莉", "s": "副團長桌5" }, { "n": "黃章一", "s": "副團長桌5" }, { "n": "陳德聰", "s": "副團長桌5" }, { "n": "王順元", "s": "副團長桌5" }, { "n": "高貴美", "s": "副團長桌5" },
            { "n": "洪偉欽", "s": "貴賓桌6" }, { "n": "張文烈", "s": "貴賓桌6" }, { "n": "許宥鈞", "s": "貴賓桌6" }, { "n": "許博任", "s": "貴賓桌6" }, { "n": "古崇序", "s": "貴賓桌6" }, { "n": "賴景民", "s": "貴賓桌6" }, { "n": "吳俊毅", "s": "貴賓桌6" }, { "n": "陳寓綸", "s": "貴賓桌6" }, { "n": "余家均", "s": "貴賓桌6" },
            { "n": "張介堂", "s": "顧問桌7" }, { "n": "楊朝凱", "s": "顧問桌7" }, { "n": "劉純娟", "s": "顧問桌7" }, { "n": "蔡青榕", "s": "顧問桌7" }, { "n": "廖世義", "s": "顧問桌7" }, { "n": "施隆昌", "s": "顧問桌7" },
            { "n": "陳世昌", "s": "顧問桌8" }, { "n": "張家華", "s": "顧問桌8" }, { "n": "陳文宗", "s": "顧問桌8" }, { "n": "游世雍", "s": "顧問桌8" }, { "n": "黃盛業", "s": "顧問桌8" }, { "n": "王詠建", "s": "顧問桌8" }, { "n": "陶明揚", "s": "顧問桌8" },
            { "n": "紀穎龍", "s": "顧問桌9" }, { "n": "陳厚諭", "s": "顧問桌9" }, { "n": "李彥鋒", "s": "顧問桌9" }, { "n": "郭明原", "s": "顧問桌9" }, { "n": "張耀潭", "s": "顧問桌9" }, { "n": "郭庭興", "s": "顧問桌9" },
            { "n": "張春洲", "s": "顧問桌10" }, { "n": "童經理", "s": "顧問桌10" }, { "n": "戶張龍一", "s": "顧問桌10" }, { "n": "蕭梓芮", "s": "顧問桌10" }, { "n": "張晉維", "s": "顧問桌10" }, { "n": "江東英", "s": "顧問桌10" },
            { "n": "王慶陸", "s": "貴賓桌11" }, { "n": "張熙坤", "s": "貴賓桌11" }, { "n": "洪崑峯", "s": "貴賓桌11" }, { "n": "張書凱", "s": "貴賓桌11" }, { "n": "廖秀娥", "s": "貴賓桌11" }, { "n": "王泰文", "s": "貴賓桌11" },
            { "n": "基隆港", "s": "四大港桌 12" }, { "n": "西碼頭分隊", "s": "四大港桌 12" }, { "n": "周朝祥", "s": "四大港桌 12" }, { "n": "蔡賢廸", "s": "四大港桌 12" },
            { "n": "基隆港", "s": "四大港桌 13" }, { "n": "高雄港", "s": "四大港桌 13" }, { "n": "黃敬雲", "s": "四大港桌 13" }, { "n": "朱榮聰", "s": "四大港桌 13" },
            { "n": "大隊長夫人", "s": "親友桌 14" }, { "n": "岳父岳母", "s": "親友桌 14" }, { "n": "孫媽媽", "s": "親友桌 14" }, { "n": "淑君老師", "s": "親友桌 14" }, { "n": "緻瑋", "s": "親友桌 14" },
            { "n": "謝東曉", "s": "親友桌 15" }, { "n": "孫倚文", "s": "親友桌 15" }, { "n": "孫倚琳", "s": "親友桌 15" }, { "n": "孫文川", "s": "親友桌 15" }, { "n": "懿慧", "s": "親友桌 15" }, { "n": "吳宏健", "s": "親友桌 15" },
            { "n": "第四大隊/海爆", "s": "第四大隊 17-25" }, { "n": "大肚/龍井/沙鹿", "s": "第四大隊 17-25" }, { "n": "梧棲/清水/清泉/犁份", "s": "第四大隊 17-25" },
            { "n": "第一中隊", "s": "第一中隊 26-27" }, { "n": "邦尼國際", "s": "第一中隊 26-27" },
            { "n": "陳思學", "s": "第二中隊 28" }, { "n": "郭丁湖", "s": "第二中隊 28" }, { "n": "陳科賓", "s": "第二中隊 28" }, { "n": "鄭紫妤", "s": "第二中隊 28" }, { "n": "吳慶章", "s": "第二中隊 28" }, { "n": "海巡隨行", "s": "第二中隊 28" },
            { "n": "第一分隊", "s": "第一分隊 29-30" }, { "n": "第一分隊", "s": "第一分隊 29-30" },
            { "n": "南堤分隊", "s": "南堤分隊 31" },
            { "n": "防風林分隊", "s": "合桌 32" }, { "n": "第一分隊", "s": "合桌 32" },
            { "n": "防風林分隊", "s": "防風林分隊 33" }, { "n": "防風林分隊", "s": "防風林分隊 34" },
            { "n": "西碼頭分隊", "s": "西碼頭分隊 35" }, { "n": "西碼頭分隊", "s": "西碼頭分隊 36" }, { "n": "西碼頭分隊", "s": "西碼頭分隊 37" }, { "n": "西碼頭分隊", "s": "西碼頭分隊 38" },
            { "n": "中龍分隊", "s": "中龍分隊 39" }, { "n": "中龍分隊", "s": "中龍分隊 40" }
        ];

        // 整理資料：將人名歸類到桌號
        const tableMap = {};
        rawData.forEach(p => {
            const key = p.s.split('(')[0].trim(); // 簡化Key，例如 "主桌1 (xx)" -> "主桌1"
            if(!tableMap[key]) tableMap[key] = [];
            tableMap[key].push(p.n);
        });

        // 綁定點擊事件
        document.querySelectorAll('.table').forEach(el => {
            el.addEventListener('click', function() {
                const label = this.getAttribute('data-label');
                // 模糊匹配：因為data-label和實際資料可能有些微差距
                let targetKey = Object.keys(tableMap).find(k => label.includes(k) || k.includes(label.split(' ')[0]));
                
                // 特殊處理：第四大隊全部顯示
                if (label.includes("第四大隊")) targetKey = "第四大隊 17-25";
                
                showModal(label, tableMap[targetKey] || ["目前無詳細名單"]);
            });
        });

        // 顯示彈窗
        const modal = document.getElementById('modal');
        const modalTitle = document.getElementById('modalTitle');
        const modalContent = document.getElementById('modalContent');

        function showModal(title, names) {
            modalTitle.innerText = title;
            modalContent.innerHTML = names.map(n => `<div class="list-item"><b>${n}</b></div>`).join('');
            modal.style.display = 'flex';
        }

        function closeModal() {
            modal.style.display = 'none';
        }

        // 點擊背景關閉
        modal.addEventListener('click', (e) => {
            if(e.target === modal) closeModal();
        });

        // 搜尋功能
        document.getElementById('searchInput').addEventListener('input', function(e) {
            const val = e.target.value.trim();
            const tables = document.querySelectorAll('.table');
            
            // 清除亮起
            tables.forEach(t => t.classList.remove('highlight'));

            if(!val) return;

            // 尋找符合的人名
            let foundTables = new Set();
            
            // 1. 搜人名
            rawData.forEach(p => {
                if(p.n.includes(val)) {
                   // 反查這個人在哪桌，然後找對應的HTML元素
                   const key = p.s.split('(')[0].trim();
                   tables.forEach(t => {
                       if(t.getAttribute('data-label').includes(key)) {
                           foundTables.add(t);
                       }
                   });
                }
            });

            // 2. 搜桌名 (例如搜"主桌")
            tables.forEach(t => {
                if(t.innerText.includes(val)) foundTables.add(t);
            });

            // 亮起
            if(foundTables.size > 0) {
                foundTables.forEach(t => t.classList.add('highlight'));
                // 捲動到第一個結果
                const first = [...foundTables][0];
                first.scrollIntoView({behavior: "smooth", block: "center"});
            }
        });
    </script>
</body>
</html>

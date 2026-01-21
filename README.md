<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>114義消尾牙 - 座位導覽圖</title>
    <script src="https://unpkg.com/@panzoom/panzoom@4.5.1/dist/panzoom.min.js"></script>
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
            touch-action: none; /* 禁止瀏覽器預設的縮放行為，交給腳本處理 */
        }

        /* --- 頂部搜尋列 --- */
        .header {
            background: white;
            padding: 10px 20px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
            z-index: 100;
            display: flex;
            flex-direction: column;
            align-items: center;
            flex-shrink: 0;
        }

        h1 { margin: 0 0 8px 0; color: #333; font-size: 1.2rem; }

        .search-container {
            position: relative;
            width: 100%;
            max-width: 500px;
            display: flex;
            gap: 10px;
        }

        input {
            width: 100%;
            padding: 10px 20px;
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
            position: relative;
            overflow: hidden; /* 隱藏溢出，讓 Panzoom 處理 */
            background-color: #cbd5e0;
            background-image: radial-gradient(#fff 1px, transparent 1px);
            background-size: 20px 20px;
            cursor: grab;
        }
        .map-wrapper:active { cursor: grabbing; }

        .map-content {
            /* 這裡不設固定寬高，由內容撐開，Panzoom 會操作這個元素 */
            display: inline-block;
            background: white;
            border-radius: 20px;
            box-shadow: 0 0 50px rgba(0,0,0,0.1);
            padding: 40px;
            margin: 100px; /* 預留邊界 */
            transform-origin: center center; /* 從中心縮放 */
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
            margin: 0 auto 50px auto;
            border-radius: 0 0 20px 20px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
        }

        /* 佈局容器 */
        .layout-row {
            display: flex;
            justify-content: center;
            gap: 60px;
        }

        /* 左側區塊 */
        .group-left {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            width: 260px;
        }

        /* 中間星光大道區塊 */
        .group-center {
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 200px;
        }

        /* 星光大道文字 */
        .aisle-text {
            writing-mode: vertical-rl;
            font-size: 40px;
            color: #ccc;
            letter-spacing: 30px;
            font-weight: bold;
            margin: 60px 0;
            border-left: 3px dashed #eee;
            border-right: 3px dashed #eee;
            padding: 0 30px;
            height: 500px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* 右側區塊 */
        .group-right {
            display: flex;
            gap: 40px;
        }
        
        .col-right-inner {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            width: 260px;
        }

        .col-right-far {
            display: grid;
            grid-template-columns: 1fr 1fr 1fr;
            gap: 20px;
            width: 400px;
            align-content: start;
        }

        /* --- 桌子樣式 --- */
        .table {
            width: 100px;
            height: 100px;
            background: var(--table-green);
            color: white;
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
            font-size: 15px;
            font-weight: bold;
            cursor: pointer; /* 在手機上即使是拖曳，點擊還是有效 */
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            border: 4px solid white;
            line-height: 1.2;
            padding: 5px;
            user-select: none; /* 防止拖曳時選取文字 */
        }
        
        .table span { font-size: 13px; font-weight: normal; opacity: 0.9; }

        /* 特殊桌顏色 */
        .t-main { background: var(--table-main); width: 130px; height: 130px; font-size: 20px; z-index: 5; }
        .t-vip { background: var(--table-vip); }
        .t-blue { background: var(--table-blue); }

        /* 搜尋亮起特效 */
        .highlight {
            background-color: var(--primary-red) !important;
            box-shadow: 0 0 0 8px #ffcdd2, 0 0 50px var(--primary-red);
            transform: scale(1.2);
            animation: blink 1s infinite alternate;
            z-index: 20;
        }
        @keyframes blink { from { opacity: 1; } to { opacity: 0.7; } }

        /* --- 彈出視窗 --- */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.6);
            z-index: 200;
            display: none;
            justify-content: center;
            align-items: center;
            backdrop-filter: blur(2px);
        }
        .modal {
            background: white;
            width: 90%;
            max-width: 380px;
            border-radius: 15px;
            padding: 20px;
            max-height: 70vh;
            overflow-y: auto;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
            animation: popUp 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }
        @keyframes popUp { from { transform: scale(0.8); opacity: 0; } to { transform: scale(1); opacity: 1; } }

        .modal-header {
            display: flex; justify-content: space-between; align-items: center;
            border-bottom: 2px solid #f0f0f0; padding-bottom: 15px; margin-bottom: 15px;
        }
        .modal-title { font-size: 1.6rem; color: var(--primary-red); font-weight: 800; }
        .close-btn { font-size: 28px; cursor: pointer; background: none; border: none; color: #999; padding: 0 10px;}
        .list-item { padding: 10px 0; border-bottom: 1px dashed #eee; font-size: 17px; display: flex; align-items: center; }
        .list-item:before { content: '👤'; margin-right: 10px; font-size: 14px;}
        
        /* 底部入口與控制項 */
        .entrance {
            position: absolute;
            bottom: -60px;
            left: 50%;
            transform: translateX(-50%);
            border: 3px solid #333;
            padding: 15px 40px;
            font-weight: 900;
            font-size: 20px;
            color: #333;
            background: #fff;
            letter-spacing: 5px;
        }

        /* 縮放控制按鈕 */
        .controls {
            position: fixed;
            bottom: 30px;
            right: 30px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            z-index: 90;
        }
        .control-btn {
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background: white;
            border: none;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2);
            font-size: 24px;
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            color: #555;
            transition: 0.2s;
        }
        .control-btn:active { transform: scale(0.9); background: #eee; }

    </style>
</head>
<body>

    <div class="header">
        <h1>🧧 114義消尾牙座次圖</h1>
        <div class="search-container">
            <input type="text" id="searchInput" placeholder="請輸入姓名 (例如：林謙志)">
        </div>
    </div>

    <div class="map-wrapper" id="mapWrapper">
        <div class="map-content" id="mapContent">
            
            <div class="stage">舞 台 (STAGE)</div>

            <div class="layout-row">
                <div class="group-left">
                    <div class="table t-vip" data-label="親友桌 14">親友<br>14</div>
                    <div class="table t-vip" data-label="親友桌 15">親友<br>15</div>
                    <div class="table t-green" data-label="第四大隊 17-25">大肚<br>守望</div>
                    <div class="table t-vip" data-label="四大港桌 12">貴賓<br>12</div>
                    <div class="table t-blue" data-label="中龍分隊 39">中龍<br>39</div>
                    <div class="table t-blue" data-label="中龍分隊 40">中龍<br>40</div>

                    <div class="table t-vip" data-label="貴賓桌 3">貴賓<br>3</div>
                    <div class="table t-vip" data-label="貴賓桌 6">貴賓<br>6</div>
                    <div class="table t-vip" data-label="顧問桌 9">顧問<br>9</div>
                    <div class="table t-blue" data-label="第二中隊 28">二中隊<br>28</div>
                    <div class="table t-blue" data-label="南堤分隊 31">南堤<br>31</div>
                </div>

                <div class="group-center">
                    <div style="display:flex; gap:30px; margin-bottom: 20px;">
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
                        <div class="table t-blue" data-label="第一分隊 29-30">第一<br>31</div>

                        <div class="table t-vip" data-label="副團長桌5">副團長<br>5</div>
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
                        <div class="table t-blue" data-label="中龍分隊 39">西碼頭<br>39</div>
                        <div class="table t-green" data-label="第四大隊 17-25">海爆</div>

                        <div class="table t-blue" data-label="西碼頭分隊 35">防風林<br>35</div>
                        <div class="table t-blue" data-label="中龍分隊 39">童綜合</div>
                        <div class="table t-blue" data-label="中龍分隊 39">光田</div>
                    </div>
                </div>
            </div>

            <div class="entrance">入 口</div>
        </div>
    </div>

    <div class="controls">
        <button class="control-btn" id="btnReset" title="回正">⟲</button>
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
        // 📋 114 義消尾牙完整資料
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

        // 整理資料庫
        const tableMap = {};
        rawData.forEach(p => {
            const key = p.s.split('(')[0].trim();
            if(!tableMap[key]) tableMap[key] = [];
            tableMap[key].push(p.n);
        });

        // 綁定點擊事件 (支援拖曳時不觸發點擊)
        let isDragging = false;
        
        document.querySelectorAll('.table').forEach(el => {
            el.addEventListener('mousedown', () => isDragging = false);
            el.addEventListener('mousemove', () => isDragging = true);
            el.addEventListener('touchstart', () => isDragging = false);
            el.addEventListener('touchmove', () => isDragging = true);

            el.addEventListener('click', function() {
                if (isDragging) return; // 如果在拖曳地圖，不觸發點擊
                
                const label = this.getAttribute('data-label');
                let targetKey = Object.keys(tableMap).find(k => label.includes(k) || k.includes(label.split(' ')[0]));
                if (label.includes("第四大隊")) targetKey = "第四大隊 17-25";
                
                showModal(label, tableMap[targetKey] || ["無詳細名單"]);
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

        function closeModal() { modal.style.display = 'none'; }
        modal.addEventListener('click', (e) => { if(e.target === modal) closeModal(); });

        // ==========================================
        // 🚀 Panzoom 縮放設定
        // ==========================================
        const mapElement = document.getElementById('mapContent');
        const panzoom = Panzoom(mapElement, {
            maxScale: 3,
            minScale: 0.3,
            contain: 'outside',
            startScale: 0.8
        });

        // 啟用滑鼠滾輪縮放
        mapElement.parentElement.addEventListener('wheel', panzoom.zoomWithWheel);

        // 重置按鈕
        document.getElementById('btnReset').addEventListener('click', () => {
            panzoom.reset();
        });

        // ==========================================
        // 🔍 搜尋與自動定位功能
        // ==========================================
        document.getElementById('searchInput').addEventListener('input', function(e) {
            const val = e.target.value.trim();
            const tables = document.querySelectorAll('.table');
            
            tables.forEach(t => t.classList.remove('highlight'));

            if(!val) return;

            let foundTable = null;
            
            // 搜尋邏輯
            rawData.some(p => {
                if(p.n.includes(val)) {
                   const key = p.s.split('(')[0].trim();
                   tables.forEach(t => {
                       if(t.getAttribute('data-label').includes(key) && !foundTable) {
                           foundTable = t;
                       }
                   });
                   return true; // 找到就停止
                }
            });

            // 如果找到，亮起並移動地圖
            if(foundTable) {
                foundTable.classList.add('highlight');
                
                // 計算位移量，將目標移到畫面中心
                const rect = foundTable.getBoundingClientRect();
                const containerRect = document.getElementById('mapWrapper').getBoundingClientRect();
                
                // 讓 Panzoom 對焦該元素 (簡化版邏輯：放大並移動)
                panzoom.zoom(1.5, { animate: true });
                setTimeout(() => {
                    panzoom.pan(
                        (containerRect.width / 2) - foundTable.offsetLeft - (foundTable.offsetWidth / 2) - 100, // 100是修正margin
                        (containerRect.height / 2) - foundTable.offsetTop - (foundTable.offsetHeight / 2) - 100
                    );
                }, 100);
            }
        });

    </script>
</body>
</html>

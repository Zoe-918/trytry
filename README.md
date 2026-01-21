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
            --table-main: #ff5252;
            --table-vip: #ff9800;
            --table-blue: #2196f3;
            --table-green: #4caf50;
            --bg-color: #f0f2f5;
        }

        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden; /* 禁止瀏覽器本身的捲動 */
            font-family: "Microsoft JhengHei", sans-serif;
            background-color: #cbd5e0;
        }

        /* --- 頂部搜尋列 --- */
        .header {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            background: rgba(255, 255, 255, 0.95);
            padding: 10px 0;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            z-index: 1000;
            display: flex;
            flex-direction: column;
            align-items: center;
            backdrop-filter: blur(5px);
        }

        h1 { margin: 0 0 5px 0; color: #333; font-size: 1.2rem; }

        .search-container {
            width: 90%;
            max-width: 400px;
        }

        input {
            width: 100%;
            padding: 10px 20px;
            border: 2px solid #ddd;
            border-radius: 50px;
            font-size: 16px;
            outline: none;
            box-sizing: border-box; /* 修正寬度計算 */
        }
        input:focus { border-color: var(--primary-red); }

        /* --- 地圖容器 --- */
        #mapScene {
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            background-image: radial-gradient(#aaa 1px, transparent 1px);
            background-size: 30px 30px;
            cursor: grab;
        }
        #mapScene:active { cursor: grabbing; }

        /* --- 地圖內容 (被縮放的對象) --- */
        .map-content {
            width: 1200px; /* 固定寬度，確保佈局不跑版 */
            padding: 50px;
            background: white;
            border-radius: 30px;
            box-shadow: 0 0 50px rgba(0,0,0,0.1);
            transform-origin: center center;
            /* 預設隱藏，等程式計算好縮放比例再顯示，避免閃爍 */
            visibility: hidden; 
        }

        /* 舞台 */
        .stage {
            width: 500px;
            height: 60px;
            background: #3f51b5;
            color: white;
            font-size: 24px;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            margin: 0 auto 40px auto;
            border-radius: 0 0 20px 20px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
        }

        /* 佈局排版 */
        .layout-row { display: flex; justify-content: center; gap: 50px; }
        .group-left { display: grid; grid-template-columns: 1fr 1fr; gap: 25px; width: 250px; }
        .group-center { display: flex; flex-direction: column; align-items: center; width: 200px; }
        .group-right { display: flex; gap: 30px; }
        .col-right-inner { display: grid; grid-template-columns: 1fr 1fr; gap: 25px; width: 250px; }
        .col-right-far { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 20px; width: 380px; align-content: start; }

        /* 星光大道 */
        .aisle-text {
            writing-mode: vertical-rl;
            font-size: 40px;
            color: #ddd;
            letter-spacing: 30px;
            font-weight: bold;
            margin: 50px 0;
            border-left: 3px dashed #eee;
            border-right: 3px dashed #eee;
            padding: 0 30px;
            height: 500px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        /* 桌子樣式 */
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
            box-shadow: 0 5px 10px rgba(0,0,0,0.2);
            border: 3px solid white;
            line-height: 1.2;
            user-select: none;
        }
        .table span { font-size: 12px; font-weight: normal; opacity: 0.9; }

        .t-main { background: var(--table-main); width: 120px; height: 120px; font-size: 18px; z-index: 5; }
        .t-vip { background: var(--table-vip); }
        .t-blue { background: var(--table-blue); }

        /* 亮起特效 */
        .highlight {
            background-color: var(--primary-red) !important;
            box-shadow: 0 0 0 6px #ffcdd2, 0 0 40px var(--primary-red);
            transform: scale(1.2);
            animation: pulse 1s infinite alternate;
            z-index: 20;
        }
        @keyframes pulse { from { opacity: 1; } to { opacity: 0.8; } }

        /* --- 控制按鈕 (移到右上角，確保可見) --- */
        .controls {
            position: fixed;
            top: 90px; /* 在搜尋列下方 */
            right: 20px;
            z-index: 1100; /* 保證在最上層 */
            display: flex;
            flex-direction: column;
            gap: 10px;
        }
        .control-btn {
            width: 45px;
            height: 45px;
            border-radius: 50%;
            background: white;
            border: 2px solid #ddd;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2);
            font-size: 20px;
            cursor: pointer;
            color: #333;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .control-btn:active { background: #eee; }

        /* --- 彈出視窗 --- */
        .modal-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.6);
            z-index: 2000;
            display: none;
            justify-content: center;
            align-items: center;
            backdrop-filter: blur(3px);
        }
        .modal {
            background: white;
            width: 85%;
            max-width: 350px;
            border-radius: 15px;
            padding: 20px;
            max-height: 70vh;
            overflow-y: auto;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
        }
        .modal-header { display: flex; justify-content: space-between; border-bottom: 2px solid #eee; padding-bottom: 10px; margin-bottom: 10px; }
        .modal-title { font-size: 1.4rem; color: var(--primary-red); font-weight: 800; }
        .close-btn { font-size: 24px; background: none; border: none; }
        .list-item { padding: 8px 0; border-bottom: 1px dashed #eee; font-size: 16px; display: flex; align-items: center; }
        .list-item:before { content: '👤'; margin-right: 8px; }
        
        /* 底部入口 */
        .entrance {
            position: absolute;
            bottom: -50px;
            left: 50%;
            transform: translateX(-50%);
            border: 3px solid #333;
            padding: 10px 40px;
            font-weight: 900;
            font-size: 20px;
            color: #333;
            background: #fff;
            letter-spacing: 5px;
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

    <div class="controls">
        <button class="control-btn" id="btnReset" title="回正/全覽">⟲</button>
    </div>

    <div id="mapScene">
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
        // 資料庫 (已內建)
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

        // 整理資料
        const tableMap = {};
        rawData.forEach(p => {
            const key = p.s.split('(')[0].trim();
            if(!tableMap[key]) tableMap[key] = [];
            tableMap[key].push(p.n);
        });

        // 啟動 Panzoom (平移與縮放)
        const elem = document.getElementById('mapContent');
        const panzoom = Panzoom(elem, {
            maxScale: 3,
            minScale: 0.1, // 允許縮很小
            contain: false, // 解除邊界限制 (關鍵！)
            startScale: 1
        });

        // 綁定滑鼠滾輪
        elem.parentElement.addEventListener('wheel', panzoom.zoomWithWheel);

        // 自動適應螢幕大小 (Fit to Screen)
        function fitToScreen() {
            const container = document.getElementById('mapScene');
            const content = document.getElementById('mapContent');
            const scale = Math.min(
                container.clientWidth / content.offsetWidth,
                container.clientHeight / content.offsetHeight
            ) * 0.9; // 留 10% 邊距
            
            panzoom.zoom(scale, { animate: true });
            setTimeout(() => panzoom.pan(0, 0), 100); // 置中
            content.style.visibility = 'visible'; // 計算完再顯示
        }

        // 頁面載入後自動適應
        window.onload = fitToScreen;
        
        // 重置按鈕
        document.getElementById('btnReset').addEventListener('click', fitToScreen);

        // 綁定桌子點擊 (處理拖曳衝突)
        let isDragging = false;
        document.querySelectorAll('.table').forEach(el => {
            el.addEventListener('pointerdown', () => isDragging = false);
            el.addEventListener('pointermove', () => isDragging = true);
            el.addEventListener('click', function() {
                if(isDragging) return;
                
                const label = this.getAttribute('data-label');
                let targetKey = Object.keys(tableMap).find(k => label.includes(k) || k.includes(label.split(' ')[0]));
                if (label.includes("第四大隊")) targetKey = "第四大隊 17-25";
                
                showModal(label, tableMap[targetKey] || ["無詳細名單"]);
            });
        });

        // 彈出視窗功能
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

        // 搜尋功能 (帶自動定位)
        document.getElementById('searchInput').addEventListener('input', function(e) {
            const val = e.target.value.trim();
            const tables = document.querySelectorAll('.table');
            tables.forEach(t => t.classList.remove('highlight'));

            if(!val) return;

            let foundTable = null;
            rawData.some(p => {
                if(p.n.includes(val)) {
                   const key = p.s.split('(')[0].trim();
                   tables.forEach(t => {
                       if(t.getAttribute('data-label').includes(key) && !foundTable) {
                           foundTable = t;
                       }
                   });
                   return true;
                }
            });

            if(foundTable) {
                foundTable.classList.add('highlight');
                
                // 自動放大並移動到該桌子
                // 計算位移量
                const rect = foundTable.getBoundingClientRect(); // 取得桌子目前在螢幕的位置
                
                // 讓 Panzoom 對焦該元素
                panzoom.zoom(1.2, { animate: true });
                
                // 這裡需要稍微複雜的計算來置中，但為了穩定性，我們先做簡單的置中重置
                // 或者讓使用者自己滑動，因為 highlight 已經很明顯了
            }
        });
    </script>
</body>
</html>

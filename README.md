<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>114義消尾牙 - 視覺化座位導覽</title>
    <style>
        /* --- 全局設定 --- */
        :root {
            --primary-color: #d32f2f;
            --secondary-color: #f8f9fa;
            --table-main-color: #ff5252; /* 主桌顏色 */
            --table-vip-color: #ff9800;  /* 貴賓/顧問顏色 */
            --table-normal-color: #4caf50; /* 一般桌顏色 */
            --highlight-color: #2979ff;  /* 搜尋選中顏色 */
        }
        body {
            font-family: "Microsoft JhengHei", sans-serif;
            background-color: #eceff1;
            margin: 0;
            display: flex;
            flex-direction: column;
            height: 100vh;
            overflow: hidden; /* 防止整個頁面捲動，只讓地圖區捲動 */
        }

        /* --- 上方搜尋列 --- */
        .header {
            background: white;
            padding: 15px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            z-index: 10;
            flex-shrink: 0;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 { margin: 0 0 10px 0; font-size: 1.5rem; color: #333; }
        .search-box {
            position: relative;
            width: 100%;
            max-width: 500px;
        }
        input {
            width: 100%;
            padding: 12px 20px;
            border: 2px solid #ddd;
            border-radius: 25px;
            font-size: 16px;
            outline: none;
            transition: 0.3s;
            box-sizing: border-box;
        }
        input:focus { border-color: var(--primary-color); }

        /* --- 地圖區域 (可縮放移動) --- */
        .map-container {
            flex-grow: 1;
            overflow: auto; /* 允許捲動 */
            position: relative;
            background-image: radial-gradient(#ddd 1px, transparent 1px);
            background-size: 20px 20px; /* 網格背景 */
            padding: 40px 20px;
            display: flex;
            justify-content: center;
        }
        
        .stage {
            width: 60%;
            height: 50px;
            background: #5c6bc0;
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 0 0 20px 20px;
            margin: 0 auto 40px auto;
            font-weight: bold;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
            position: absolute;
            top: 0;
            left: 20%;
        }

        /* 桌子佈局網格 */
        .tables-grid {
            margin-top: 80px; /* 留給舞台空間 */
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 20px;
            max-width: 1000px;
            padding-bottom: 100px; /* 底部留白 */
        }

        /* 單個桌子樣式 */
        .table {
            width: 80px;
            height: 80px;
            border-radius: 50%;
            background-color: var(--table-normal-color);
            color: white;
            display: flex;
            justify-content: center;
            align-items: center;
            text-align: center;
            font-size: 14px;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2);
            transition: transform 0.2s, box-shadow 0.2s;
            position: relative;
            padding: 5px;
            border: 4px solid white;
        }
        .table:hover { transform: scale(1.1); z-index: 5; }
        
        /* 不同類型的桌子顏色 */
        .type-main { background-color: var(--table-main-color); width: 100px; height: 100px; font-size: 16px; z-index: 2; }
        .type-vip { background-color: var(--table-vip-color); }
        
        /* 搜尋選中特效 */
        .highlight {
            background-color: var(--highlight-color) !important;
            animation: pulse 1.5s infinite;
            border-color: #ffeb3b;
            box-shadow: 0 0 20px var(--highlight-color);
            transform: scale(1.2);
            z-index: 10;
        }

        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(41, 121, 255, 0.7); }
            70% { box-shadow: 0 0 0 20px rgba(41, 121, 255, 0); }
            100% { box-shadow: 0 0 0 0 rgba(41, 121, 255, 0); }
        }

        /* --- 彈出視窗 (Modal) --- */
        .modal-overlay {
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.6);
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 100;
            backdrop-filter: blur(3px);
        }
        .modal {
            background: white;
            padding: 25px;
            border-radius: 15px;
            width: 90%;
            max-width: 400px;
            max-height: 80vh;
            overflow-y: auto;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
            animation: slideUp 0.3s ease;
        }
        @keyframes slideUp { from { transform: translateY(50px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
        
        .modal-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #eee; padding-bottom: 10px; margin-bottom: 15px; }
        .modal-title { font-size: 1.5rem; font-weight: bold; color: #333; }
        .close-btn { background: none; border: none; font-size: 24px; cursor: pointer; color: #999; }
        
        .guest-list { list-style: none; padding: 0; margin: 0; }
        .guest-item { padding: 10px; border-bottom: 1px solid #f0f0f0; display: flex; justify-content: space-between; }
        .guest-item:last-child { border-bottom: none; }
        .guest-name { font-weight: bold; font-size: 1.1rem; }
        .guest-highlight { color: var(--primary-color); background: #ffebee; border-radius: 4px; padding: 0 5px;}

        /* 圖例說明 */
        .legend { display: flex; gap: 10px; margin-top: 5px; font-size: 12px; color: #666; justify-content: center; flex-wrap: wrap;}
        .dot { width: 10px; height: 10px; border-radius: 50%; display: inline-block; margin-right: 3px; }
    </style>
</head>
<body>

    <div class="header">
        <h1>🧧 114義消尾牙座次圖</h1>
        <div class="search-box">
            <input type="text" id="searchInput" placeholder="輸入姓名查找座位 (例如：林謙志)">
        </div>
        <div class="legend">
            <span><span class="dot" style="background:var(--table-main-color)"></span>主桌</span>
            <span><span class="dot" style="background:var(--table-vip-color)"></span>貴賓/顧問</span>
            <span><span class="dot" style="background:var(--table-normal-color)"></span>一般桌</span>
            <span><span class="dot" style="background:var(--highlight-color)"></span>搜尋結果</span>
        </div>
    </div>

    <div class="map-container" id="mapContainer">
        <div class="stage">舞 台 (STAGE)</div>
        <div class="tables-grid" id="tablesGrid">
            </div>
    </div>

    <div class="modal-overlay" id="modalOverlay" onclick="closeModal(event)">
        <div class="modal">
            <div class="modal-header">
                <div class="modal-title" id="modalTitle">主桌 1</div>
                <button class="close-btn" onclick="closeModalDirect()">×</button>
            </div>
            <ul class="guest-list" id="guestList">
                </ul>
        </div>
    </div>

    <script>
        // ==========================================
        // 📋 資料庫 (已整合真實資料)
        // ==========================================
        const rawData = [
            { "n": "林謙志", "s": "主桌1" }, { "n": "駱啟明", "s": "主桌1" }, { "n": "孫福佑", "s": "主桌1" }, { "n": "陳高尚", "s": "主桌1" }, { "n": "林志宏", "s": "主桌1" }, { "n": "吳瓊華", "s": "主桌1" }, { "n": "孫文山", "s": "主桌1" }, { "n": "王俊傑", "s": "主桌1" }, { "n": "魏福添", "s": "主桌1" }, { "n": "張家豪", "s": "主桌1" }, { "n": "李文義", "s": "主桌1" }, { "n": "游永中", "s": "主桌1" },
            { "n": "陳俊青", "s": "主桌2" }, { "n": "趙彬然", "s": "主桌2" }, { "n": "林瑞才", "s": "主桌2" }, { "n": "賴俊男", "s": "主桌2" }, { "n": "曾百溪", "s": "主桌2" }, { "n": "林昊佑", "s": "主桌2" }, { "n": "林茂發", "s": "主桌2" }, { "n": "張東玄", "s": "主桌2" }, { "n": "吳進宗", "s": "主桌2" }, { "n": "陳義方", "s": "主桌2" }, { "n": "陳木生", "s": "主桌2" }, { "n": "張家銨", "s": "主桌2" },
            { "n": "曾星明", "s": "貴賓桌3" }, { "n": "林暉智", "s": "貴賓桌3" }, { "n": "沈明賢", "s": "貴賓桌3" }, { "n": "李文彬", "s": "貴賓桌3" }, { "n": "蔡清松", "s": "貴賓桌3" }, { "n": "余凌冲", "s": "貴賓桌3" }, { "n": "張道銘", "s": "貴賓桌3" },
            { "n": "王志龍", "s": "貴賓桌4" }, { "n": "洪瑞添", "s": "貴賓桌4" }, { "n": "黃憲章", "s": "貴賓桌4" }, { "n": "黃國誌", "s": "貴賓桌4" }, { "n": "廖光政", "s": "貴賓桌4" }, { "n": "林永晟", "s": "貴賓桌4" }, { "n": "許鴻茗", "s": "貴賓桌4" }, { "n": "喬永福", "s": "貴賓桌4" }, { "n": "孫境堯", "s": "貴賓桌4" },
            { "n": "林美秀", "s": "副團長桌5" }, { "n": "林曼莉", "s": "副團長桌5" }, { "n": "黃章一", "s": "副團長桌5" }, { "n": "陳德聰", "s": "副團長桌5" }, { "n": "王順元", "s": "副團長桌5" }, { "n": "高貴美", "s": "副團長桌5" },
            { "n": "洪偉欽", "s": "貴賓桌6" }, { "n": "張文烈", "s": "貴賓桌6" }, { "n": "許宥鈞", "s": "貴賓桌6" }, { "n": "許博任", "s": "貴賓桌6" }, { "n": "古崇序", "s": "貴賓桌6" }, { "n": "賴景民", "s": "貴賓桌6" }, { "n": "吳俊毅", "s": "貴賓桌6" }, { "n": "陳寓綸", "s": "貴賓桌6" }, { "n": "余家均", "s": "貴賓桌6" },
            { "n": "張介堂", "s": "顧問桌7" }, { "n": "楊朝凱", "s": "顧問桌7" }, { "n": "劉純娟", "s": "顧問桌7" }, { "n": "蔡青榕", "s": "顧問桌7" }, { "n": "廖世義", "s": "顧問桌7" }, { "n": "施隆昌", "s": "顧問桌7" },
            { "n": "陳世昌", "s": "顧問桌8" }, { "n": "張家華", "s": "顧問桌8" }, { "n": "陳文宗", "s": "顧問桌8" }, { "n": "游世雍", "s": "顧問桌8" }, { "n": "黃盛業", "s": "顧問桌8" }, { "n": "王詠建", "s": "顧問桌8" }, { "n": "陶明揚", "s": "顧問桌8" },
            { "n": "紀穎龍", "s": "顧問桌9" }, { "n": "陳厚諭", "s": "顧問桌9" }, { "n": "李彥鋒", "s": "顧問桌9" }, { "n": "郭明原", "s": "顧問桌9" }, { "n": "張耀潭", "s": "顧問桌9" }, { "n": "郭庭興", "s": "顧問桌9" },
            { "n": "張春洲", "s": "顧問桌10" }, { "n": "童經理", "s": "顧問桌10" }, { "n": "戶張龍一", "s": "顧問桌10" }, { "n": "蕭梓芮", "s": "顧問桌10" }, { "n": "張晉維", "s": "顧問桌10" }, { "n": "江東英", "s": "顧問桌10" },
            { "n": "王慶陸", "s": "貴賓桌11" }, { "n": "張熙坤", "s": "貴賓桌11" }, { "n": "洪崑峯", "s": "貴賓桌11" }, { "n": "張書凱", "s": "貴賓桌11" }, { "n": "廖秀娥", "s": "貴賓桌11" }, { "n": "王泰文", "s": "貴賓桌11" },
            { "n": "基隆港", "s": "四大港桌12" }, { "n": "西碼頭分隊", "s": "四大港桌12" }, { "n": "周朝祥", "s": "四大港桌12" }, { "n": "蔡賢廸", "s": "四大港桌12" },
            { "n": "基隆港", "s": "四大港桌13" }, { "n": "高雄港", "s": "四大港桌13" }, { "n": "黃敬雲", "s": "四大港桌13" }, { "n": "朱榮聰", "s": "四大港桌13" },
            { "n": "大隊長夫人", "s": "親友桌14" }, { "n": "岳父岳母", "s": "親友桌14" }, { "n": "孫媽媽", "s": "親友桌14" }, { "n": "淑君老師", "s": "親友桌14" }, { "n": "緻瑋", "s": "親友桌14" },
            { "n": "謝東曉", "s": "親友桌15" }, { "n": "孫倚文", "s": "親友桌15" }, { "n": "孫倚琳", "s": "親友桌15" }, { "n": "孫文川", "s": "親友桌15" }, { "n": "懿慧", "s": "親友桌15" }, { "n": "吳宏健", "s": "親友桌15" },
            { "n": "第四大隊/海爆", "s": "第四大隊 17-25" }, { "n": "大肚/龍井/沙鹿", "s": "第四大隊 17-25" }, { "n": "梧棲/清水/清泉/犁份", "s": "第四大隊 17-25" },
            { "n": "第一中隊", "s": "第一中隊 26-27" }, { "n": "邦尼國際", "s": "第一中隊 26-27" },
            { "n": "陳思學", "s": "第二中隊 28" }, { "n": "郭丁湖", "s": "第二中隊 28" }, { "n": "陳科賓", "s": "第二中隊 28" }, { "n": "鄭紫妤", "s": "第二中隊 28" }, { "n": "吳慶章", "s": "第二中隊 28" }, { "n": "海巡隨行", "s": "第二中隊 28" },
            { "n": "第一分隊", "s": "第一分隊 29" }, { "n": "第一分隊", "s": "第一分隊 30" },
            { "n": "南堤分隊", "s": "南堤分隊 31" },
            { "n": "防風林分隊", "s": "合桌 32" }, { "n": "第一分隊", "s": "合桌 32" },
            { "n": "防風林分隊", "s": "防風林分隊 33" }, { "n": "防風林分隊", "s": "防風林分隊 34" },
            { "n": "西碼頭分隊", "s": "西碼頭分隊 35" }, { "n": "西碼頭分隊", "s": "西碼頭分隊 36" }, { "n": "西碼頭分隊", "s": "西碼頭分隊 37" }, { "n": "西碼頭分隊", "s": "西碼頭分隊 38" },
            { "n": "中龍分隊", "s": "中龍分隊 39" }, { "n": "中龍分隊", "s": "中龍分隊 40" }
        ];

        // ==========================================
        // 🛠️ 核心邏輯
        // ==========================================
        
        // 1. 整理資料：把平整的名單轉成 { "桌名": [人名, 人名...] }
        const tablesData = {};
        rawData.forEach(item => {
            // 簡化桌名顯示 (移除過長的括號說明，讓球球顯示好看點)
            let tableName = item.s.split('(')[0].trim(); 
            if (!tablesData[tableName]) {
                tablesData[tableName] = [];
            }
            tablesData[tableName].push(item.n);
        });

        // 2. 自定義排序邏輯：主桌 -> 貴賓 -> 顧問 -> 數字
        const sortOrder = ["主桌", "副團長", "貴賓", "顧問", "警察", "四大港", "親友", "第四", "第一", "第二", "南堤", "防風", "西碼頭", "中龍"];
        
        const sortedTableNames = Object.keys(tablesData).sort((a, b) => {
            // 提取關鍵字索引
            let indexA = sortOrder.findIndex(key => a.includes(key));
            let indexB = sortOrder.findIndex(key => b.includes(key));
            
            // 如果都在清單內，照清單順序
            if (indexA !== -1 && indexB !== -1) {
                if (indexA !== indexB) return indexA - indexB;
                // 同類型比數字
                return parseInt(a.replace(/\D/g,'')) - parseInt(b.replace(/\D/g,''));
            }
            // 如果只有一個在清單內，在清單內的排前面
            if (indexA !== -1) return -1;
            if (indexB !== -1) return 1;
            
            // 剩下的照數字或字串排
            return a.localeCompare(b, 'zh-Hant');
        });

        // 3. 生成畫面
        const grid = document.getElementById('tablesGrid');
        
        sortedTableNames.forEach(tableName => {
            const el = document.createElement('div');
            el.className = 'table';
            el.innerText = tableName.replace("分隊", "").replace("中隊", "").replace("四大港桌", "四大港").replace("貴賓桌", "貴賓"); // 簡化顯示
            el.setAttribute('data-name', tableName); // 存完整桌名方便搜尋
            
            // 根據桌名分配顏色樣式
            if (tableName.includes("主桌") || tableName.includes("副團長")) {
                el.classList.add('type-main');
            } else if (tableName.includes("貴賓") || tableName.includes("顧問") || tableName.includes("親友") || tableName.includes("四大港")) {
                el.classList.add('type-vip');
            }

            // 點擊事件
            el.onclick = () => showModal(tableName);
            
            grid.appendChild(el);
        });

        // ==========================================
        // 🔍 搜尋與互動功能
        // ==========================================

        const searchInput = document.getElementById('searchInput');

        // 監聽輸入
        searchInput.addEventListener('input', (e) => {
            const val = e.target.value.trim().toLowerCase();
            const tables = document.querySelectorAll('.table');
            
            // 清除所有亮起狀態
            tables.forEach(t => t.classList.remove('highlight'));

            if (!val) return;

            // 尋找符合的人
            const matchedTables = new Set();
            rawData.forEach(p => {
                if (p.n.toLowerCase().includes(val) || p.s.toLowerCase().includes(val)) {
                    // 找到這個人的桌名，並對應到簡化後的 key
                    let key = p.s.split('(')[0].trim();
                    matchedTables.add(key);
                }
            });

            // 亮起對應的桌子
            let firstMatch = null;
            tables.forEach(t => {
                if (matchedTables.has(t.getAttribute('data-name'))) {
                    t.classList.add('highlight');
                    if (!firstMatch) firstMatch = t;
                }
            });

            // 捲動到第一個結果
            if (firstMatch) {
                firstMatch.scrollIntoView({ behavior: 'smooth', block: 'center' });
            }
        });

        // ==========================================
        // 🖼️ 彈出視窗控制
        // ==========================================
        const modal = document.getElementById('modalOverlay');
        const modalTitle = document.getElementById('modalTitle');
        const list = document.getElementById('guestList');

        function showModal(tableName) {
            modalTitle.innerText = tableName;
            list.innerHTML = ""; // 清空舊資料
            
            const guests = tablesData[tableName] || [];
            
            // 取得搜尋關鍵字，用來標記紅字
            const keyword = searchInput.value.trim();

            guests.forEach(name => {
                const li = document.createElement('li');
                li.className = 'guest-item';
                
                // 如果搜尋中，且名字符合，加強顯示
                let displayName = name;
                if (keyword && name.includes(keyword)) {
                    displayName = `<span class="guest-highlight">${name}</span>`;
                }
                
                li.innerHTML = `<span class="guest-name">${displayName}</span>`;
                list.appendChild(li);
            });

            modal.style.display = 'flex';
        }

        function closeModal(e) {
            if (e.target === modal) {
                modal.style.display = 'none';
            }
        }
        
        function closeModalDirect() {
            modal.style.display = 'none';
        }
    </script>
</body>
</html>

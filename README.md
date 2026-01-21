<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>114義消尾牙 - 桌位查詢系統</title>
    <style>
        :root {
            --primary-gold: #d4af37;
            --primary-red: #c0392b;
            --bg-color: #fdf5e6;
            --table-main: #e74c3c;   /* 主桌紅 */
            --table-vip: #f39c12;    /* 貴賓金 */
            --table-guest: #27ae60;  /* 一般綠 */
            --table-blue: #2980b9;   /* 警消藍 */
            --aisle-width: 60px;
        }

        body {
            font-family: "Microsoft JhengHei", "Heiti TC", sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
        }

        h1 {
            color: #444;
            margin-bottom: 20px;
            text-align: center;
        }

        /* 搜尋框區域 */
        .control-panel {
            position: sticky;
            top: 0;
            background: rgba(253, 245, 230, 0.95);
            padding: 15px;
            z-index: 100;
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        input[type="text"] {
            padding: 12px 20px;
            width: 90%;
            max-width: 400px;
            border: 2px solid #ccc;
            border-radius: 30px;
            font-size: 16px;
            outline: none;
            transition: all 0.3s;
        }

        input[type="text"]:focus {
            border-color: var(--primary-red);
            box-shadow: 0 0 8px rgba(192, 57, 43, 0.3);
        }

        /* 舞台 */
        .stage {
            width: 80%;
            max-width: 800px;
            height: 60px;
            background: linear-gradient(to bottom, #333, #000);
            color: gold;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 0 0 20px 20px;
            margin: 20px 0 40px 0;
            font-weight: bold;
            font-size: 1.2em;
            letter-spacing: 5px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.3);
        }

        /* 座位圖網格 */
        .seating-chart {
            display: grid;
            /* 9列佈局：左1, 左2, [星光大道], 右中1, 右中2, [走道], 右1, 右2, 右3 */
            grid-template-columns: 80px 80px var(--aisle-width) 80px 80px var(--aisle-width) 80px 80px 80px;
            gap: 12px;
            position: relative;
            margin-bottom: 100px;
        }

        /* 桌子樣式 */
        .table {
            border-radius: 50%;
            height: 80px;
            width: 80px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 12px;
            cursor: pointer;
            transition: transform 0.2s;
            text-align: center;
            box-shadow: 0 3px 6px rgba(0,0,0,0.15);
            border: 3px solid rgba(255,255,255,0.5);
            position: relative;
            color: white;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.3);
        }

        .table:hover {
            transform: scale(1.15);
            z-index: 10;
        }

        /* 桌子顏色分類 */
        .t-main { background-color: var(--table-main); } /* 主桌 */
        .t-vip { background-color: var(--table-vip); }   /* 貴賓/顧問/副團 */
        .t-org { background-color: var(--table-blue); }  /* 分隊/單位 */
        .t-guest { background-color: var(--table-guest); } /* 親友/其他 */

        .table-num {
            font-size: 18px;
            font-weight: 900;
            line-height: 1;
        }
        .table-name {
            font-size: 11px;
            margin-top: 2px;
            line-height: 1.1;
            max-width: 90%;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        /* 搜尋高亮 */
        .table.highlight {
            background-color: #e74c3c !important; /* 強制紅色 */
            box-shadow: 0 0 20px #e74c3c;
            transform: scale(1.2);
            z-index: 20;
            animation: pulse 1s infinite alternate;
        }

        @keyframes pulse {
            from { box-shadow: 0 0 10px #e74c3c; }
            to { box-shadow: 0 0 30px #e74c3c; }
        }

        /* 走道文字 */
        .aisle-label {
            grid-row: 1 / span 8;
            display: flex;
            justify-content: center;
            align-items: center;
            writing-mode: vertical-rl;
            color: #aaa;
            font-weight: bold;
            letter-spacing: 8px;
            pointer-events: none;
            font-size: 14px;
        }

        /* 入口標示 */
        .entrance-marker {
            grid-column: 6; /* 對應右邊走道 */
            grid-row: 9;    /* 最下方 */
            text-align: center;
            font-weight: bold;
            color: var(--primary-red);
            margin-top: 10px;
        }

        /* 彈出視窗 */
        #modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.6);
            justify-content: center;
            align-items: center;
            z-index: 1000;
            backdrop-filter: blur(2px);
        }

        .modal-content {
            background: white;
            padding: 25px;
            border-radius: 15px;
            width: 320px;
            max-height: 80vh;
            display: flex;
            flex-direction: column;
            box-shadow: 0 10px 25px rgba(0,0,0,0.3);
            animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }

        @keyframes popIn {
            from { opacity: 0; transform: scale(0.8); }
            to { opacity: 1; transform: scale(1); }
        }

        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
            margin-bottom: 15px;
        }
        
        .modal-header h2 { margin: 0; color: #333; font-size: 1.5em; }
        
        .close-btn {
            background: none; border: none; font-size: 28px; cursor: pointer; color: #999;
        }
        
        .guest-list {
            overflow-y: auto;
            flex-grow: 1;
            padding-right: 5px;
        }

        .guest-item {
            padding: 8px 10px;
            border-bottom: 1px solid #f5f5f5;
            color: #555;
            font-size: 1.1em;
            display: flex;
            justify-content: space-between;
        }
        
        .guest-item:last-child { border-bottom: none; }

        .guest-note {
            font-size: 0.8em;
            color: #888;
            background: #eee;
            padding: 2px 6px;
            border-radius: 10px;
        }

        .highlight-text {
            color: red;
            font-weight: bold;
            background-color: #ffe6e6;
        }

        /* 手機版優化 */
        @media (max-width: 800px) {
            .seating-chart {
                transform: scale(0.9);
                transform-origin: top center;
            }
        }
    </style>
</head>
<body>

    <div class="control-panel">
        <h1>114 義消尾牙座次表</h1>
        <input type="text" id="searchInput" placeholder="搜尋姓名、單位 (例如：林謙志、議員、風電)">
    </div>

    <div class="stage">舞 台 (Stage)</div>

    <div class="seating-chart" id="chart">
        <div class="aisle-label" style="grid-column: 3;">★ 星光大道 ★</div>
        <div class="aisle-label" style="grid-column: 6;">走道</div>
        
        <div class="entrance-marker">
            <div style="font-size:24px;">⬆</div>
            入口
        </div>
    </div>

    <div id="modal">
        <div class="modal-content">
            <div class="modal-header">
                <h2 id="modalTitle">桌號</h2>
                <button class="close-btn" onclick="closeModal()">×</button>
            </div>
            <div class="guest-list" id="modalGuests">
                </div>
        </div>
    </div>

    <script>
        // ==========================================
        // 1. 資料庫 (整合您提供的最新名單)
        // ==========================================
        const tablesData = [
            // --- Row 1 (貼近舞台) ---
            { id: '15', name: '親友桌', type: 't-guest', r: 1, c: 1, guests: ['大隊長夫人','岳父','岳母','孫媽媽','淑君老師*4','緻瑋*3'] },
            { id: '1',  name: '主桌',   type: 't-main',  r: 1, c: 2, guests: ['林謙志','駱啟明','孫福佑','陳高尚','林志宏','王俊傑','孫文山','游永中','魏福添','張家豪','李文義','議員'] },
            { id: '2',  name: '主桌',   type: 't-main',  r: 1, c: 4, guests: ['陳俊青(議員)','曾百溪(議員)','林瑞才(議員)','賴俊男(議員)','吳瓊華(議員)','林昊佑(議員)','陳木生(議員)','張東玄(議員)','許修豪(議員)','陳義方(議員)','林孟令(議員)','張家銨(議員)'] },

            // --- Row 2 (右邊退後一格，與副團長齊平) ---
            { id: '16', name: '親友桌', type: 't-guest', r: 2, c: 1, guests: ['謝東曉*2','孫倚文*4','孫倚琳','孫文川*3','懿慧','吳宏健'] },
            { id: '3',  name: '貴賓',   type: 't-vip',   r: 2, c: 2, guests: ['曾星明','林暉智*2','沈明賢*3','李文彬','蘇泓維','張道銘'] },
            // (Space)
            { id: '4',  name: '副團長', type: 't-vip',   r: 2, c: 4, guests: ['林美秀','林曼莉*2','黃章一','陳德聰','王順元','高貴美*2'] },
            { id: '5',  name: '風電',   type: 't-vip',   r: 2, c: 5, guests: ['王志龍','洪瑞添','黃憲章','黃國誌','廖光政','林永晟','許鴻茗','喬永福','李武勳','王志文'] },
            // (Space)
            { id: '13', name: '四港',   type: 't-org',   r: 2, c: 7, guests: ['基隆港*4','西碼頭分隊*2','警消*2','周朝祥','隨行人員'] },
            { id: 'No1',name: '第四',   type: 't-org',   r: 2, c: 8, guests: ['第四救災救護大隊'] },
            { id: 'No2',name: '清泉',   type: 't-org',   r: 2, c: 9, guests: ['清泉分隊(待補)'] }, // 名單未詳列，依圖保留位置

            // --- Row 3 ---
            { id: '17', name: '守望',   type: 't-org',   r: 3, c: 1, guests: ['大肚守望相助隊'] },
            { id: '6',  name: '貴賓',   type: 't-vip',   r: 3, c: 2, guests: ['洪偉欽','張文烈','許宥鈞','許博任','古崇序','賴景民','吳俊毅','陳寓綸','余家均'] },
            // (Space)
            { id: '7',  name: '顧問',   type: 't-vip',   r: 3, c: 4, guests: ['張介堂','楊朝凱','劉純娟*2','蔡青榕','廖世義','施榮昌*2'] },
            { id: '8',  name: '顧問',   type: 't-vip',   r: 3, c: 5, guests: ['陳世昌','張家華','陳文宗','游世雍','黃盛業*2','王詠建','陶明揚*2'] },
            // (Space)
            { id: '14', name: '四港',   type: 't-org',   r: 3, c: 7, guests: ['基隆港*4','高雄港*2','警消*2','蔡賢廸'] },
            { id: 'No3',name: '大肚',   type: 't-org',   r: 3, c: 8, guests: ['大肚分隊'] },
            { id: 'No4',name: '清水',   type: 't-org',   r: 3, c: 9, guests: ['清水分隊'] },

            // --- Row 4 ---
            { id: '12', name: '貴賓',   type: 't-vip',   r: 4, c: 1, guests: ['江東英','張熙坤','洪崑峯','張書凱','傳令','隨行人員*2'] },
            { id: '9',  name: '顧問',   type: 't-vip',   r: 4, c: 2, guests: ['紀穎龍','陳厚諭*2','李彥鋒','郭明原*2','張耀潭*2','郭庭興'] },
            // (Space)
            { id: '10', name: '顧問',   type: 't-vip',   r: 4, c: 4, guests: ['張春洲','童得彰*3','戶張龍一*2','蕭梓芮*2','張晉維','王慶陸'] },
            { id: '11', name: '港警',   type: 't-org',   r: 4, c: 5, guests: ['黃敬雲','廖秀娥','王泰文','陳信宏*2','黃炯智*4','陳松斌'] },
            // (Space)
            { id: '36', name: '西碼頭', type: 't-org',   r: 4, c: 7, guests: ['西碼頭分隊','王小姐*2'] },
            { id: 'No5',name: '龍井',   type: 't-org',   r: 4, c: 8, guests: ['龍井分隊'] },
            { id: 'No6',name: '犁份',   type: 't-org',   r: 4, c: 9, guests: ['犁份分隊'] },

            // --- Row 5 ---
            { id: '40', name: '中龍',   type: 't-org',   r: 5, c: 1, guests: ['中龍分隊','防風林分隊'] },
            { id: '29', name: '二中隊', type: 't-org',   r: 5, c: 2, guests: ['陳思學','郭丁湖*3','陳科賓','鄭紫妤','德隆倉儲*2','吳慶章'] },
            // (Space)
            { id: '27', name: '一中隊', type: 't-org',   r: 5, c: 4, guests: ['第一中隊'] },
            { id: '28', name: '一中隊', type: 't-org',   r: 5, c: 5, guests: ['第一中隊'] },
            // (Space)
            { id: '37', name: '西碼頭', type: 't-org',   r: 5, c: 7, guests: ['西碼頭分隊'] },
            { id: 'No7',name: '沙鹿',   type: 't-org',   r: 5, c: 8, guests: ['沙鹿分隊'] },
            { id: 'No8',name: '梧棲',   type: 't-org',   r: 5, c: 9, guests: ['梧棲分隊'] },

            // --- Row 6 ---
            { id: '41', name: '中龍',   type: 't-org',   r: 6, c: 1, guests: ['中龍分隊'] },
            { id: '32', name: '南堤',   type: 't-org',   r: 6, c: 2, guests: ['南堤分隊'] },
            // (Space)
            { id: '30', name: '第一',   type: 't-org',   r: 6, c: 4, guests: ['第一分隊'] },
            { id: '34', name: '防風林', type: 't-org',   r: 6, c: 5, guests: ['防風林分隊'] },
            // (Space)
            { id: '38', name: '西碼頭', type: 't-org',   r: 6, c: 7, guests: ['西碼頭分隊'] },
            { id: '39', name: '西碼頭', type: 't-org',   r: 6, c: 8, guests: ['西碼頭分隊'] },
            { id: 'No9',name: '海爆',   type: 't-org',   r: 6, c: 9, guests: ['海爆分隊'] },

            // --- Row 7 (Bottom) ---
            { id: '31', name: '第一',   type: 't-org',   r: 7, c: 4, guests: ['第一分隊'] },
            { id: '33', name: '合桌',   type: 't-org',   r: 7, c: 5, guests: ['南堤分隊','第一分隊'] },
            // (Space)
            { id: '35', name: '防風林', type: 't-org',   r: 7, c: 7, guests: ['防風林分隊'] },
            { id: '42', name: '童綜合', type: 't-org',   r: 7, c: 8, guests: ['童綜合'] },
            { id: '43', name: '光田',   type: 't-org',   r: 7, c: 9, guests: ['光田醫院'] },
        ];

        // ==========================================
        // 2. 程式邏輯
        // ==========================================
        const chartContainer = document.getElementById('chart');
        const searchInput = document.getElementById('searchInput');
        let activeHighlight = null;

        // 處理名字顯示 (例如: "林暉智*2" -> "林暉智", count="2位")
        function parseGuest(rawName) {
            const parts = rawName.split('*');
            if (parts.length > 1) {
                return { name: parts[0], note: parts[1] + "位" };
            }
            return { name: rawName, note: "" };
        }

        // 初始化渲染
        function renderTables() {
            tablesData.forEach(table => {
                const tableDiv = document.createElement('div');
                tableDiv.className = `table ${table.type}`;
                tableDiv.style.gridRow = table.r;
                tableDiv.style.gridColumn = table.c;
                
                // 顯示內容
                tableDiv.innerHTML = `
                    <div class="table-num">${table.id}</div>
                    <div class="table-name">${table.name}</div>
                `;
                
                // 綁定資料
                tableDiv.dataset.id = table.id;
                tableDiv.dataset.searchContent = table.guests.join(',').toLowerCase() + ',' + table.name + ',' + table.id;
                
                // 點擊事件
                tableDiv.onclick = () => openModal(table);

                chartContainer.appendChild(tableDiv);
            });
        }

        // 搜尋功能
        searchInput.addEventListener('input', (e) => {
            const term = e.target.value.toLowerCase().trim();
            const tables = document.querySelectorAll('.table');

            tables.forEach(t => {
                t.classList.remove('highlight');
                if (term && t.dataset.searchContent.includes(term)) {
                    t.classList.add('highlight');
                }
            });
        });

        // 開啟視窗
        function openModal(tableData) {
            document.getElementById('modalTitle').innerText = `${tableData.id} - ${tableData.name}`;
            const listDiv = document.getElementById('modalGuests');
            listDiv.innerHTML = '';
            
            const searchTerm = searchInput.value.toLowerCase().trim();

            if (tableData.guests.length === 0) {
                listDiv.innerHTML = '<div style="padding:10px; color:#888;">本桌尚未提供詳細名單</div>';
            } else {
                tableData.guests.forEach(raw => {
                    const guest = parseGuest(raw);
                    const div = document.createElement('div');
                    div.className = 'guest-item';
                    
                    // 搜尋字串高亮邏輯
                    let displayName = guest.name;
                    if (searchTerm && guest.name.toLowerCase().includes(searchTerm)) {
                        displayName = `<span class="highlight-text">${guest.name}</span>`;
                    }

                    div.innerHTML = `
                        <span>${displayName}</span>
                        ${guest.note ? `<span class="guest-note">${guest.note}</span>` : ''}
                    `;
                    listDiv.appendChild(div);
                });
            }
            
            document.getElementById('modal').style.display = 'flex';
        }

        function closeModal() {
            document.getElementById('modal').style.display = 'none';
        }

        // 點擊背景關閉
        window.onclick = function(event) {
            const modal = document.getElementById('modal');
            if (event.target == modal) {
                closeModal();
            }
        }

        // 執行
        renderTables();
    </script>
</body>
</html>

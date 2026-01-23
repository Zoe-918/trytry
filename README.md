<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>114義消尾牙 - 智慧座位導覽</title>
    <style>
        :root {
            --primary-gold: #d4af37;
            --primary-red: #c0392b;
            --bg-color: #fdf5e6;
            --table-main: #e74c3c;   /* 主桌紅 */
            --table-vip: #f39c12;    /* 貴賓金 */
            --table-org: #2980b9;    /* 單位藍 */
            --table-guest: #27ae60;  /* 親友綠 */
            --aisle-width: 60px;
        }

        body {
            font-family: "Microsoft JhengHei", "Heiti TC", sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            padding: 0;
            display: flex;
            flex-direction: column;
            height: 100vh;
            overflow: hidden;
        }

        /* --- 頂部搜尋與資訊列 --- */
        .control-panel {
            background: white;
            padding: 15px;
            z-index: 50;
            display: flex;
            flex-direction: column;
            align-items: center;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
            flex-shrink: 0;
        }

        h1 { margin: 0 0 10px 0; color: #444; font-size: 1.2rem; }

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

        /* --- 地圖顯示區域 (Viewport) --- */
        .map-viewport {
            flex-grow: 1;
            overflow: hidden; 
            position: relative;
            background-color: #f4f4f4;
            background-image: radial-gradient(#ddd 1px, transparent 1px);
            background-size: 20px 20px;
            cursor: grab; 
            display: flex;
            justify-content: center; /* 讓內容水平置中 */
            align-items: flex-start;
        }

        .map-viewport:active { cursor: grabbing; }

        /* --- 可縮放的內容容器 --- */
        .zoom-wrapper {
            transition: transform 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94); /* 恢復一點平滑過渡 */
            transform-origin: center top; /* 從上方中間開始縮放 */
            padding: 40px;
            padding-bottom: 150px;
            min-width: 900px; /* 確保內容有足夠寬度不被擠壓 */
            box-sizing: border-box;
        }

        /* 舞台 */
        .stage {
            width: 800px;
            height: 60px;
            background: linear-gradient(to bottom, #333, #000);
            color: gold;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 0 0 20px 20px;
            margin: 0 auto 40px auto;
            font-weight: bold;
            font-size: 1.5em;
            letter-spacing: 5px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.3);
        }

        /* 座位圖網格 */
        .seating-chart {
            display: grid;
            grid-template-columns: 80px 80px var(--aisle-width) 80px 80px var(--aisle-width) 80px 80px 80px;
            gap: 15px;
            justify-content: center;
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
            text-align: center;
            box-shadow: 0 3px 6px rgba(0,0,0,0.2);
            border: 3px solid rgba(255,255,255,0.8);
            position: relative;
            color: white;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.5);
            transition: all 0.3s;
            user-select: none; 
        }

        .t-main { background-color: var(--table-main); }
        .t-vip { background-color: var(--table-vip); }
        .t-org { background-color: var(--table-org); }
        .t-guest { background-color: var(--table-guest); }

        .table-num { font-size: 18px; font-weight: 900; line-height: 1; }
        .table-name { font-size: 11px; margin-top: 2px; line-height: 1.1; overflow: hidden; white-space: nowrap; width: 90%; text-overflow: ellipsis;}

        /* --- 搜尋特效 --- */
        .table.dimmed { opacity: 0.15; filter: grayscale(100%); transform: scale(0.9); }
        .table.highlight { opacity: 1 !important; filter: none; z-index: 100; transform: scale(1.3); box-shadow: 0 0 0 5px #fff, 0 0 20px var(--primary-red); animation: bounce 1s infinite; }
        @keyframes bounce { 0%, 100% { transform: scale(1.3); } 50% { transform: scale(1.4); } }

        /* 走道文字 */
        .aisle-label { grid-row: 1 / span 8; display: flex; justify-content: center; align-items: center; writing-mode: vertical-rl; color: #aaa; font-weight: bold; letter-spacing: 10px; pointer-events: none; font-size: 16px; }
        .entrance-marker { grid-column: 6; grid-row: 9; text-align: center; font-weight: bold; color: var(--primary-red); margin-top: 10px; font-size: 1.2em; }

        /* --- 縮放控制按鈕 --- */
        .zoom-controls {
            position: fixed;
            bottom: 20px;
            right: 20px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            z-index: 100;
        }

        .zoom-btn {
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background: white;
            border: none;
            box-shadow: 0 4px 10px rgba(0,0,0,0.2);
            font-size: 24px;
            color: #555;
            cursor: pointer;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: background 0.2s;
        }
        .zoom-btn:active { background: #eee; }
        
        .fit-text { font-size: 14px; font-weight: bold; }

        /* --- 彈出視窗 --- */
        #modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); justify-content: center; align-items: center; z-index: 1000; backdrop-filter: blur(2px); }
        .modal-content { background: white; padding: 25px; border-radius: 15px; width: 85%; max-width: 320px; max-height: 80vh; display: flex; flex-direction: column; box-shadow: 0 10px 25px rgba(0,0,0,0.3); animation: popIn 0.3s; }
        @keyframes popIn { from { transform: scale(0.8); opacity: 0; } to { transform: scale(1); opacity: 1; } }
        .modal-header { display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #eee; padding-bottom: 10px; margin-bottom: 10px; }
        .close-btn { background: none; border: none; font-size: 28px; cursor: pointer; color: #999; }
        .guest-list { overflow-y: auto; flex-grow: 1; }
        .guest-item { padding: 10px; border-bottom: 1px solid #f5f5f5; font-size: 1.1em; display: flex; justify-content: space-between; }
        .guest-note { font-size: 0.8em; color: #888; background: #eee; padding: 2px 6px; border-radius: 10px; }
        .highlight-text { color: red; font-weight: bold; background: #ffe6e6; padding: 0 4px; border-radius: 3px; }
    </style>
</head>
<body>

    <div class="control-panel">
        <h1>114 義消尾牙座次表</h1>
        <input type="text" id="searchInput" placeholder="搜尋姓名、單位... (例如：林謙志、議員)">
    </div>

    <div class="map-viewport" id="mapViewport">
        <div class="zoom-wrapper" id="zoomWrapper">
            <div class="stage">舞 台 (Stage)</div>

            <div class="seating-chart" id="chart">
                <div class="aisle-label" style="grid-column: 3;">★ 星光大道 ★</div>
                <div class="aisle-label" style="grid-column: 6;">走道</div>
                
                <div class="entrance-marker">
                    <div style="font-size:30px;">⬆</div>
                    入 口
                </div>
            </div>
        </div>
    </div>

    <div class="zoom-controls">
        <button class="zoom-btn" onclick="zoomIn()">+</button>
        <button class="zoom-btn" onclick="zoomOut()">−</button>
        <button class="zoom-btn fit-text" onclick="resetZoom()">1：1</button>
    </div>

    <div id="modal">
        <div class="modal-content">
            <div class="modal-header">
                <h2 id="modalTitle">桌號</h2>
                <button class="close-btn" onclick="closeModal()">×</button>
            </div>
            <div class="guest-list" id="modalGuests"></div>
        </div>
    </div>

    <script>
        const tablesData = [
             // Row 1
            { id: '15', name: '親友桌', type: 't-guest', r: 1, c: 1, guests: ['大隊長夫人','岳父','岳母','淑君老師*4','謝東曉*2','緻瑋*3'] },
            { id: '1',  name: '主桌',   type: 't-main',  r: 1, c: 2, guests: ['林謙志','駱啟明','孫福佑','陳高尚','林志宏','王俊傑','孫文山','游永中','魏福添','梁又文','李文義','議員'] },
            { id: '2',  name: '主桌',   type: 't-main',  r: 1, c: 4, guests: ['陳俊青','曾百溪','林瑞才','賴俊男','吳瓊華','林昊佑','陳木生','張東玄','許修豪','陳義方','林孟令','鄭文賓','張家銨'] },
            // Row 2
            { id: '16', name: '親友桌', type: 't-guest', r: 2, c: 1, guests: ['孫倚文*2','楊壹善','孫倚琳','孫文川*3','劉懿慧','孫媽媽','趙宜庭','吳宏健'] },
            { id: '3',  name: '貴賓',   type: 't-vip',   r: 2, c: 2, guests: ['曾星明','黃公甫*2','沈明賢*3','王星皓','李文彬','張道銘'] },
            { id: '4',  name: '副團長', type: 't-vip',   r: 2, c: 4, guests: ['林美秀','林曼莉*2','黃章一','陳德聰','杜俊慧','陳子健','林茂發','李奇峰'] },
            { id: '5',  name: '風電',   type: 't-vip',   r: 2, c: 5, guests: ['王志龍','洪瑞添','黃憲章','黃國誌','廖光政','林永晟','許鴻茗','喬永福','李武勳','王志文'] },
            { id: '13', name: '四港',   type: 't-org',   r: 2, c: 7, guests: ['基隆港*4','西碼頭分隊*2','臺中港警消*2','周朝祥'] },
            { id: '18', name: '第四',   type: 't-org',   r: 2, c: 8, guests: ['第四救災救護大隊'] },
            { id: '19', name: '清泉',   type: 't-org',   r: 2, c: 9, guests: ['清泉分隊'] },
            // Row 3
            { id: '17', name: '守望',   type: 't-org',   r: 3, c: 1, guests: ['大肚守望相助隊'] },
            { id: '6',  name: '貴賓',   type: 't-vip',   r: 3, c: 2, guests: ['洪偉欽','許宥鈞','許博任','古崇序','賴景民','吳俊毅','蘇泓維','陳寓綸','陳建宏'] },
            { id: '7',  name: '顧問',   type: 't-vip',   r: 3, c: 4, guests: ['張介堂','楊朝凱','劉純娟*2','蔡青榕','廖世義','施榮昌*2','余家均'] },
            { id: '8',  name: '顧問',   type: 't-vip',   r: 3, c: 5, guests: ['陳世昌','張家華','陳文宗','林暉智*2','游世雍','黃盛業*2','王詠建','陶明揚*2'] },
            { id: '14', name: '四港',   type: 't-org',   r: 3, c: 7, guests: ['基隆港*4','高雄港*2','臺中港警消*2','蔡賢廸','邦尼國際隨行人員'] },
            { id: '20', name: '大肚',   type: 't-org',   r: 3, c: 8, guests: ['大肚分隊'] },
            { id: '21', name: '清水',   type: 't-org',   r: 3, c: 9, guests: ['清水分隊'] },
            // Row 4
            { id: '12', name: '貴賓',   type: 't-vip',   r: 4, c: 1, guests: ['江東英','張熙坤','洪崑峯','張書凱','傳令','海巡署(中部)隨行人員','第四布雷艇隨行人員','第三海巡隊隨行人員'] },
            { id: '9',  name: '顧問',   type: 't-vip',   r: 4, c: 2, guests: ['紀穎龍','陳厚諭*2','李彥鋒','郭明原*2','張耀潭*2','張文烈','郭庭興'] },
            { id: '10', name: '顧問',   type: 't-vip',   r: 4, c: 4, guests: ['張春洲','童得彰*3','戶張龍一*2','蕭梓芮*2','王慶陸'] },
            { id: '11', name: '港警',   type: 't-org',   r: 4, c: 5, guests: ['黃敬雲','廖秀娥','王泰文','陳信宏*2','黃炯智*4','陳松斌'] },
            { id: '36', name: '西碼頭', type: 't-org',   r: 4, c: 7, guests: ['西碼頭分隊','永聖王小姐*2'] },
            { id: '22', name: '龍井',   type: 't-org',   r: 4, c: 8, guests: ['龍井分隊'] },
            { id: '23', name: '犁份',   type: 't-org',   r: 4, c: 9, guests: ['犁份分隊'] },
            // Row 5
            { id: '40', name: '中龍',   type: 't-org',   r: 5, c: 1, guests: ['中龍分隊','防風林分隊'] },
            { id: '29', name: '二中隊', type: 't-org',   r: 5, c: 2, guests: ['陳思學','郭丁湖*3','陳科賓','鄭紫妤','德隆倉儲*2','吳慶章'] },
            { id: '27', name: '一中隊', type: 't-org',   r: 5, c: 4, guests: ['第一中隊'] },
            { id: '28', name: '一中隊', type: 't-org',   r: 5, c: 5, guests: ['第一中隊'] },
            { id: '37', name: '西碼頭', type: 't-org',   r: 5, c: 7, guests: ['西碼頭分隊'] },
            { id: '24', name: '沙鹿',   type: 't-org',   r: 5, c: 8, guests: ['沙鹿分隊'] },
            { id: '25', name: '梧棲',   type: 't-org',   r: 5, c: 9, guests: ['梧棲分隊'] },
            // Row 6
            { id: '41', name: '中龍',   type: 't-org',   r: 6, c: 1, guests: ['中龍分隊'] },
            { id: '30', name: '第一',   type: 't-org',   r: 6, c: 4, guests: ['第一分隊'] },
            { id: '34', name: '防風林', type: 't-org',   r: 6, c: 5, guests: ['防風林分隊'] },
            { id: '38', name: '西碼頭', type: 't-org',   r: 6, c: 7, guests: ['西碼頭分隊'] },
            { id: '26', name: '海爆',   type: 't-org',   r: 6, c: 9, guests: ['海爆分隊'] },
            // Row 7
            { id: '31', name: '第一',   type: 't-org',   r: 7, c: 4, guests: ['第一分隊'] },
            { id: '32', name: '南堤',   type: 't-org',   r: 7, c: 5, guests: ['南堤分隊'] },
            { id: '39', name: '西碼頭', type: 't-org',   r: 7, c: 7, guests: ['西碼頭分隊'] },
            { id: '35', name: '防風林', type: 't-org',   r: 6, c: 8, guests: ['防風林分隊'] }, // ID 35 從原圖位置推測
            { id: '43', name: '光田',   type: 't-org',   r: 7, c: 9, guests: ['光田醫院'] }, // !!! 原本這裡少逗號
            // Row 8
            { id: '33', name: '合桌',   type: 't-org',   r: 7, c: 8, guests: ['南堤分隊','第一分隊'] }, // 調整位置至走道旁
            { id: '42', name: '童綜合', type: 't-org',   r: 8, c: 8, guests: ['童綜合'] }
        ];

        // ==========================================
        // 2. 縮放與拖曳邏輯
        // ==========================================
        let currentScale = 1.0;
        const zoomWrapper = document.getElementById('zoomWrapper');
        const mapViewport = document.getElementById('mapViewport');

        function updateZoom() {
            zoomWrapper.style.transform = `scale(${currentScale}) translate(0, 0)`;
        }

        function zoomIn() {
            currentScale += 0.2;
            updateZoom();
        }

        function zoomOut() {
            if (currentScale > 0.3) {
                currentScale -= 0.2;
                updateZoom();
            }
        }

        // 修改後的 "最適大小" 邏輯
        function resetZoom() {
            // 我們的內容設計寬度大約是 900px (舞台800 + 邊距)
            const contentWidth = 950; 
            const viewportWidth = mapViewport.offsetWidth;

            // 計算能夠塞滿寬度的比例
            let fitScale = (viewportWidth / contentWidth) * 0.95; // 0.95 是留一點點邊邊

            // 電腦螢幕很大時，不要放太大，保持 1.0 即可
            if (fitScale > 1.2) fitScale = 1.0;
            
            // 手機如果非常窄，也設定一個最小值，避免太小看不見
            if (fitScale < 0.25) fitScale = 0.25;

            currentScale = fitScale;
            updateZoom();

            // 將捲軸歸零，回到最上方中間
            mapViewport.scrollTop = 0;
            mapViewport.scrollLeft = 0;
        }

        // 滑鼠拖曳 (Drag to Scroll)
        let isDown = false;
        let startX;
        let startY;
        let scrollLeft;
        let scrollTop;

        mapViewport.addEventListener('mousedown', (e) => {
            isDown = true;
            mapViewport.style.cursor = 'grabbing';
            startX = e.pageX - mapViewport.offsetLeft;
            startY = e.pageY - mapViewport.offsetTop;
            scrollLeft = mapViewport.scrollLeft;
            scrollTop = mapViewport.scrollTop;
        });

        mapViewport.addEventListener('mouseleave', () => { isDown = false; mapViewport.style.cursor = 'grab'; });
        mapViewport.addEventListener('mouseup', () => { isDown = false; mapViewport.style.cursor = 'grab'; });

        mapViewport.addEventListener('mousemove', (e) => {
            if (!isDown) return;
            e.preventDefault();
            const x = e.pageX - mapViewport.offsetLeft;
            const y = e.pageY - mapViewport.offsetTop;
            const walkX = (x - startX) * 1.5;
            const walkY = (y - startY) * 1.5;
            mapViewport.scrollLeft = scrollLeft - walkX;
            mapViewport.scrollTop = scrollTop - walkY;
        });

        // ==========================================
        // 3. 渲染與搜尋
        // ==========================================
        const chartContainer = document.getElementById('chart');
        
        function parseGuest(rawName) {
            const parts = rawName.split('*');
            if (parts.length > 1) {
                return { name: parts[0], note: parts[1] + "位" };
            }
            return { name: rawName, note: "" };
        }

        function renderTables() {
            tablesData.forEach(table => {
                const div = document.createElement('div');
                div.className = `table ${table.type}`;
                div.style.gridRow = table.r;
                div.style.gridColumn = table.c;
                div.innerHTML = `<div class="table-num">${table.id}</div><div class="table-name">${table.name}</div>`;
                div.dataset.id = table.id;
                div.dataset.searchIndex = [table.id, table.name, ...table.guests].join(',').toLowerCase();
                div.onclick = () => openModal(table);
                chartContainer.appendChild(div);
            });
            
            // 渲染完成後，自動執行一次最適大小調整
            setTimeout(() => {
                resetZoom();
            }, 100);
        }

        const searchInput = document.getElementById('searchInput');
        searchInput.addEventListener('input', (e) => {
            const term = e.target.value.toLowerCase().trim();
            const tables = document.querySelectorAll('.table');

            if (term === "") {
                tables.forEach(t => { t.classList.remove('highlight', 'dimmed'); });
                return;
            }

            tables.forEach(t => {
                const index = t.dataset.searchIndex;
                if (index.includes(term)) {
                    t.classList.add('highlight');
                    t.classList.remove('dimmed');
                } else {
                    t.classList.add('dimmed');
                    t.classList.remove('highlight');
                }
            });
        });

        // ==========================================
        // 4. 彈出視窗
        // ==========================================
        function openModal(tableData) {
            document.getElementById('modalTitle').innerText = `${tableData.id} - ${tableData.name}`;
            const listDiv = document.getElementById('modalGuests');
            listDiv.innerHTML = '';
            
            const searchTerm = searchInput.value.toLowerCase().trim();

            if (!tableData.guests || tableData.guests.length === 0) {
                listDiv.innerHTML = '<div style="padding:10px; color:#999;">本桌無詳細名單</div>';
            } else {
                tableData.guests.forEach(raw => {
                    const guest = parseGuest(raw);
                    const div = document.createElement('div');
                    div.className = 'guest-item';
                    
                    let displayName = guest.name;
                    if (searchTerm && guest.name.toLowerCase().includes(searchTerm)) {
                        displayName = `<span class="highlight-text">${guest.name}</span>`;
                    }

                    div.innerHTML = `<span>${displayName}</span>${guest.note ? `<span class="guest-note">${guest.note}</span>` : ''}`;
                    listDiv.appendChild(div);
                });
            }
            document.getElementById('modal').style.display = 'flex';
        }

        function closeModal() { document.getElementById('modal').style.display = 'none'; }
        window.onclick = function(event) { if (event.target == document.getElementById('modal')) closeModal(); }

        renderTables();
    </script>
</body>
</html>

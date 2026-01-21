<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>114義消尾牙 - 智慧導覽系統</title>
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

        /* --- 地圖顯示區域 --- */
        .map-viewport {
            flex-grow: 1;
            overflow: auto; 
            position: relative;
            background-color: #f4f4f4;
            background-image: radial-gradient(#ddd 1px, transparent 1px);
            background-size: 20px 20px;
            display: flex;
            justify-content: center;
            align-items: flex-start; 
            padding: 40px;
            cursor: grab;
        }

        .map-viewport:active { cursor: grabbing; }

        .zoom-wrapper {
            transition: transform 0.2s cubic-bezier(0.25, 0.46, 0.45, 0.94);
            transform-origin: top center;
            padding-bottom: 100px;
        }

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
        }

        .t-main { background-color: var(--table-main); }
        .t-vip { background-color: var(--table-vip); }
        .t-org { background-color: var(--table-org); }
        .t-guest { background-color: var(--table-guest); }

        .table-num { font-size: 18px; font-weight: 900; line-height: 1; }
        .table-name { font-size: 11px; margin-top: 2px; line-height: 1.1; overflow: hidden; white-space: nowrap; width: 90%; text-overflow: ellipsis;}

        /* --- 搜尋特效 --- */
        .table.dimmed {
            opacity: 0.15;
            filter: grayscale(100%);
            transform: scale(0.9);
        }

        .table.highlight {
            opacity: 1 !important;
            filter: none;
            z-index: 100;
            transform: scale(1.3);
            box-shadow: 0 0 0 5px #fff, 0 0 20px var(--primary-red);
            animation: bounce 1s infinite;
        }

        @keyframes bounce {
            0%, 100% { transform: scale(1.3); }
            50% { transform: scale(1.4); }
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
            letter-spacing: 10px;
            pointer-events: none;
            font-size: 16px;
        }

        .entrance-marker {
            grid-column: 6;
            grid-row: 9;
            text-align: center;
            font-weight: bold;
            color: var(--primary-red);
            margin-top: 10px;
            font-size: 1.2em;
        }

        /* --- 縮放控制 --- */
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

        /* --- 彈出視窗 --- */
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
            animation: popIn 0.3s;
        }
        
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
        <input type="text" id="searchInput" placeholder="請輸入姓名 (例如：林謙志、王志龍)">
    </div>

    <div class="map-viewport" id="mapViewport">
        <div class="zoom-wrapper" id="zoomWrapper">
            <div class="stage">舞 台 (Stage)</div>

            <div class="seating-chart" id="chart">
                <div class="aisle-label" style="grid-column: 3;">★ 星光大道 ★</div>
                <div class="aisle-label" style="grid-column: 6;">走道</div>
                <div class="entrance-marker">
                    <div style="font-size:30px;">⬆</div>
                    入口
                </div>
            </div>
        </div>
    </div>

    <div class="zoom-controls">
        <button class="zoom-btn" onclick="zoomIn()">+</button>
        <button class="zoom-btn" onclick="zoomOut()">−</button>
        <button class="zoom-btn" onclick="resetZoom()" style="font-size:16px; font-weight:bold;">1:1</button>
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
        // ==========================================
        // 資料庫
        // ==========================================
        const tablesData = [
            // Row 1
            { id: '15', name: '親友桌', type: 't-guest', r: 1, c: 1, guests: ['大隊長夫人','岳父','岳母','孫媽媽','淑君老師*4','緻瑋*3'] },
            { id: '1',  name: '主桌',   type: 't-main',  r: 1, c: 2, guests: ['林謙志','駱啟明','孫福佑','陳高尚','林志宏','王俊傑','孫文山','游永中','魏福添','張家豪','李文義','議員'] },
            { id: '2',  name: '主桌',   type: 't-main',  r: 1, c: 4, guests: ['陳俊青(議員)','曾百溪(議員)','林瑞才(議員)','賴俊男(議員)','吳瓊華(議員)','林昊佑(議員)','陳木生(議員)','張東玄(議員)','許修豪(議員)','陳義方(議員)','林孟令(議員)','張家銨(議員)'] },
            // Row 2
            { id: '16', name: '親友桌', type: 't-guest', r: 2, c: 1, guests: ['謝東曉*2','孫倚

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
            overflow: auto;
            position: relative;
            background-color: #f4f4f4;
            background-image:

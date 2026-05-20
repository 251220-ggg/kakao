<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>주변 은행 약탈 지도 (Bank Heist Radar)</title>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@700&family=Noto+Sans+KR:wght@400;700&display=swap" rel="stylesheet">
    <style>
        /* 1. 기본 레이아웃 및 갱단 아지트 모니터 스타일 */
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body, html { width: 100%; height: 100%; overflow: hidden; font-family: 'Noto Sans KR', sans-serif; background-color: #0d0f12; }

        .container { display: flex; width: 100%; height: 100vh; }

        /* 2. 왼쪽 사이드바 (차가운 콘크리트 & 다크 네온 스타일) */
        .sidebar {
            width: 380px;
            height: 100%;
            background-color: #12161a; 
            background-image: url('https://www.transparenttextures.com/patterns/asfalt-dark.png'); /* 거친 질감 */
            border-right: 2px solid #ff3333; /* 경고를 뜻하는 레드 라인 */
            display: flex;
            flex-direction: column;
            z-index: 10;
            box-shadow: 5px 0 25px rgba(0,0,0,0.7);
            color: #e0e6ed; 
        }

        .header { padding: 25px 20px; border-bottom: 1px solid #232b35; }
        .header h2 { 
            margin-bottom: 15px; 
            font-size: 1.6rem; 
            font-family: 'Orbitron', sans-serif; 
            color: #ff3333; 
            text-shadow: 0 0 10px rgba(255, 51, 51, 0.6); 
            text-align: center; 
            letter-spacing: 2px;
        }

        /* 입력창 영역 (군용/해킹 장비 스타일) */
        .search-box { display: flex; }
        .search-box input {
            flex: 1;
            padding: 12px;
            border: 1px solid #3a4655;
            border-radius: 4px 0 0 4px;
            outline: none;
            font-size: 0.95rem;
            background-color: #1a222d;
            color: #00ff66; /* 해킹 모니터 느낌의 녹색 글씨 */
            font-family: 'Noto Sans KR', sans-serif;
            font-weight: bold;
        }
        .search-box input::placeholder { color: #4f6175; }
        .search-box button {
            padding: 0 20px;
            background-color: #ff3333;
            border: none;
            cursor: pointer;
            font-weight: bold;
            border-radius: 0 4px 4px 0;
            color: #ffffff;
            font-family: 'Noto Sans KR', sans-serif;
            font-size: 0.95rem;
            transition: all 0.2s;
            box-shadow: 0 0 10px rgba(255, 51, 51, 0.4);
        }
        .search-box button:hover { background-color: #cc0000; box-shadow: 0 0 15px rgba(255, 51, 51, 0.8); }

        /* 검색 결과 리스트 (작전 타깃 리스트) */
        #result-list {
            flex: 1;
            overflow-y: auto;
            list-style: none;
            padding: 15px;
        }
        .result-item {
            padding: 18px;
            background-color: #171e26; 
            border: 1px solid #283545;
            cursor: pointer;
            margin-bottom: 12px;
            border-radius: 4px;
            transition: all 0.2s;
            color: #cbd5e1; 
        }
        .result-item:hover { 
            background-color: #1e2936; 
            border-color: #ff3333; 
            transform: translateY(-2px); 
            box-shadow: 0 4px 12px rgba(255, 51, 51, 0.15); 
        }
        .result-item strong { 
            display: block; 
            font-size: 1.15rem; 
            color: #00ff66; /* 활성화된 타깃 이름 */
            margin-bottom: 8px; 
            font-weight: 700;
        }
        .result-item p { font-size: 0.85rem; line-height: 1.5; margin-bottom: 4px; color: #94a3b8; }
        .result-item .info-link { 
            display: inline-block;
            margin-top: 5px;
            font-size: 0.85rem; 
            color: #ff3333; 
            text-decoration: none; 
            font-weight: bold; 
        }
        .result-item .info-link:hover { text-decoration: underline; }

        /* 3. 오른쪽 지도 영역 (야간 작전 모드 톤다운 필터) */
        .map-area { flex: 1; position: relative; }
        #map { 
            width: 100%; 
            height: 100%; 
            /* 지도를 어둡고 푸르스름한 야간 작전 스크린 톤으로 변경 */
            filter: invert(90%) hue-rotate(180deg) brightness(85%) contrast(110%);
        }

        /* 4. 작전용 커스텀 오버레이 (조준경/CCTV 데이터 팝업) */
        .custom-overlay { 
            position: relative;
            bottom: 50px; 
            padding: 15px; 
            min-width: 180px; 
            line-height: 1.5; 
            background-color: #0f141c; 
            border: 2px solid #ff3333; 
            border-radius: 4px; 
            box-shadow: 0 0 20px rgba(255, 51, 51, 0.4); 
            color: #ffffff; 
        }
        .custom-overlay strong { color: #00ff66; font-size: 1.05rem; display: block; margin-bottom: 4px; }
        .custom-overlay small { color: #94a3b8; font-size: 0.85rem; display: block; margin-bottom: 8px; }
        .custom-overlay::after {
            content: '';
            position: absolute;
            bottom: -8px;
            left: 50%;
            transform: translateX(-50%);
            border-width: 8px 8px 0;
            border-style: solid;
            border-color: #ff3333 transparent;
            display: block;
            width: 0;
        }
    </style>
</head>
<body>

<div class="container">
    <div class="sidebar">
        <div class="header">
            <h2>🚨 BANK HEIST RADAR</h2>
            <div class="search-box">
                <input type="text" id="keyword" placeholder="금융기관 입력 (예: 은행, 금고)" onkeypress="if
                

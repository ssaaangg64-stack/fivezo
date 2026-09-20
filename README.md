<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>지구과학/환경 탐구: 월별 초미세먼지(PM2.5) 나쁨 일수 분석 도구</title>
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #0f172a;
            --primary-light: #38bdf8;
            --bg: #f8fafc;
            --card-bg: #ffffff;
            --text: #334155;
            --border: #e2e8f0;
            --danger: #ef4444;
            --warning: #f59e0b;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 24px;
            line-height: 1.5;
        }

        .container {
            max-width: 1000px;
            margin: 0 auto;
        }

        header {
            text-align: center;
            margin-bottom: 32px;
        }

        header h1 {
            color: var(--primary);
            margin-bottom: 8px;
            font-size: 1.8rem;
        }

        header p {
            color: #64748b;
            font-size: 0.95rem;
        }

        .card {
            background: var(--card-bg);
            border-radius: 12px;
            padding: 24px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 2px 4px -1px rgba(0, 0, 0, 0.03);
            border: 1px solid var(--border);
            margin-bottom: 24px;
        }

        .drop-zone {
            border: 2px dashed var(--primary-light);
            border-radius: 8px;
            padding: 32px;
            text-align: center;
            background-color: #f0f9ff;
            cursor: pointer;
            transition: background-color 0.2s;
        }

        .drop-zone:hover {
            background-color: #e0f2fe;
        }

        .drop-zone input[type="file"] {
            display: none;
        }

        .btn {
            background-color: #0284c7;
            color: white;
            border: none;
            padding: 10px 18px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 600;
            margin-top: 12px;
        }

        .btn:hover {
            background-color: #0369a1;
        }

        .btn-sample {
            background-color: #64748b;
            font-size: 0.85rem;
            padding: 6px 12px;
            margin-left: 8px;
        }

        .btn-sample:hover {
            background-color: #475569;
        }

        .alert-box {
            background-color: #fef2f2;
            border-left: 4px solid var(--danger);
            padding: 12px 16px;
            border-radius: 4px;
            margin-bottom: 20px;
            display: none;
        }

        .alert-box.active {
            display: block;
        }

        .stats-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
            gap: 16px;
            margin-bottom: 24px;
        }

        .stat-card {
            background-color: #f8fafc;
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 16px;
            text-align: center;
        }

        .stat-card h3 {
            margin: 0 0 8px 0;
            font-size: 0.9rem;
            color: #64748b;
        }

        .stat-card .stat-value {
            font-size: 1.5rem;
            font-weight: 700;
            color: var(--primary);
        }

        .chart-container {
            position: relative;
            height: 380px;
            width: 100%;
        }

        .column-selector {
            display: flex;
            gap: 16px;
            flex-wrap: wrap;
            margin-top: 16px;
            padding: 16px;
            background: #f1f5f9;
            border-radius: 6px;
            align-items: center;
        }

        .column-selector label {
            font-weight: 600;
            font-size: 0.9rem;
        }

        select {
            padding: 6px 10px;
            border-radius: 4px;
            border: 1px solid var(--border);
        }

        .badge-warning {
            background-color: #fef3c7;
            color: #b45309;
            padding: 2px 8px;
            border-radius: 4px;
            font-size: 0.85rem;
            font-weight: 600;
        }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>고등학교 과학탐구: 월별 초미세먼지(PM2.5) '나쁨' 일수 분석</h1>
        <p>CSV 파일(날짜·시각, PM2.5 농도 포함)을 업로드하여 일평균 36 ㎍/㎥ 이상인 '나쁨' 날을 월별로 분석합니다.</p>
    </header>

    <!-- 파일 업로드 카드 -->
    <div class="card">
        <div class="drop-zone" id="dropZone">
            <p style="margin: 0; font-size: 1.1rem; font-weight: 600;">CSV 파일을 드래그하여 넣거나 클릭하여 선택하세요</p>
            <p style="margin: 8px 0 0 0; color: #64748b; font-size: 0.85rem;">(날짜·시각, PM2.5 농도(㎍/㎥) 항목 포함)</p>
            <input type="file" id="fileInput" accept=".csv">
            <button class="btn" onclick="document.getElementById('fileInput').click()">파일 선택</button>
            <button class="btn btn-sample" id="sampleBtn">샘플 데이터로 분석 테스트</button>
        </div>

        <!-- 열 선택기 (자동 감지 실패 시 사용자 선택) -->
        <div class="column-selector" id="columnSelector" style="display: none;">
            <div>
                <label for="dateCol">날짜·시각 열:</label>
                <select id="dateCol"></select>
            </div>
            <div>
                <label for="pm25Col">PM2.5 농도(㎍/㎥) 열:</label>
                <select id="pm25Col"></select>
            </div>
            <button class="btn" style="margin-top:0; padding: 6px 12px;" id="reAnalyzeBtn">다시 분석</button>
        </div>
    </div>

    <!-- 결측치 처리 알림 -->
    <div class="alert-box" id="alertBox">
        <strong>데이터 정제 알림:</strong> 결측치(빈 값) 또는 유효하지 않은 데이터 총 <span id="excludedCount" style="font-weight: bold; color: var(--danger);">0</span>개를 제외하고 분석을 진행했습니다.
    </div>

    <!-- 요약 통계 카드 -->
    <div class="stats-grid" id="statsSection" style="display: none;">
        <div class="stat-card">
            <h3>총 분석 일수</h3>
            <div class="stat-value" id="totalDays">0 일</div>
        </div>
        <div class="stat-card">
            <h3>'나쁨' 기준 (환경부)</h3>
            <div class="stat-value"><span class="badge-warning">36 ㎍/㎥ 이상</span></div>
        </div>
        <div class="stat-card">
            <h3>총 '나쁨' 판정 일수</h3>
            <div class="stat-value" id="totalBadDays" style="color: var(--danger);">0 일</div>
        </div>
        <div class="stat-card">
            <h3>최고 일평균 농도</h3>
            <div class="stat-value" id="maxDailyPm">0 ㎍/㎥</div>
        </div>
    </div>

    <!-- 시각화 그래프 카드 -->
    <div class="card" id="chartSection" style="display: none;">
        <h3 style="margin-top:0; color: var(--primary);">월별 초미세먼지(PM2.5) '나쁨' 발생 일수</h3>
        <div class="chart-container">
            <canvas id="badDaysChart"></canvas>
        </div>
    </div>
</div>

<script>
    let rawCsvHeaders = [];
    let rawCsvRows = [];
    let myChart = null;

    const fileInput = document.getElementById('fileInput');
    const dropZone = document.getElementById('dropZone');

    // 드래그 앤 드롭 이벤트
    dropZone.addEventListener('dragover', (e) => { e.preventDefault(); dropZone.style.backgroundColor = '#e0f2fe'; });
    dropZone.addEventListener('dragleave', () => { dropZone.style.backgroundColor = '#f0f9ff'; });
    dropZone.addEventListener('drop', (e) => {
        e.preventDefault();
        dropZone.style.backgroundColor = '#f0f9ff';
        if (e.dataTransfer.files.length) handleFile(e.dataTransfer.files[0]);
    });
    fileInput.addEventListener('change', (e) => {
        if (e.target.files.length) handleFile(e.target.files[0]);
    });

    // 샘플 데이터 생성 및 분석
    document.getElementById('sampleBtn').addEventListener('click', (e) => {
        e.stopPropagation();
        const sampleData = 
`날짜시각,PM2.5(ug/m3)
2025-01-05 08:00,42.5
2025-01-05 20:00,38.0
2025-01-06 12:00,20.1
2025-01-15 09:00,55.4
2025-01-15 18:00,
2025-02-01 10:00,15.2
2025-02-10 14:00,60.8
2025-02-10 22:00,72.1
2025-02-11 11:00,N/A
2025-03-01 09:00,40.0
2025-03-02 10:00,38.5
2025-03-03 11:00,45.2
2025-03-15 12:00,12.0
2025-04-05 13:00,25.4
2025-04-20 15:00,36.5
2025-05-01 08:00,10.2`;
        
        parseAndAnalyzeCSV(sampleData);
    });

    function handleFile(file) {
        const reader = new FileReader();
        reader.onload = function (e) {
            parseAndAnalyzeCSV(e.target.result);
        };
        reader.readAsText(file, 'UTF-8');
    }

    // CSV 파싱
    function parseAndAnalyzeCSV(csvText) {
        const lines = csvText.trim().split(/\r?\n/);
        if (lines.length < 2) {
            alert('유효한 CSV 데이터가 아닙니다.');
            return;
        }

        function splitCSVLine(line) {
            const result = [];
            let current = '';
            let inQuotes = false;
            for (let i = 0; i < line.length; i++) {
                const char = line[i];
                if (char === '"') inQuotes = !inQuotes;
                else if (char === ',' && !inQuotes) {
                    result.push(current.trim());
                    current = '';
                } else current += char;
            }
            result.push(current.trim());
            return result;
        }

        rawCsvHeaders = splitCSVLine(lines[0]);
        rawCsvRows = lines.slice(1).map(splitCSVLine);

        const dateSelect = document.getElementById('dateCol');
        const pm25Select = document.getElementById('pm25Col');
        dateSelect.innerHTML = '';
        pm25Select.innerHTML = '';

        let detectedDateIdx = -1;
        let detectedPm25Idx = -1;

        rawCsvHeaders.forEach((h, idx) => {
            const headerText = h.toLowerCase();
            dateSelect.add(new Option(`${idx + 1}. ${h}`, idx));
            pm25Select.add(new Option(`${idx + 1}. ${h}`, idx));

            if (headerText.includes('날짜') || headerText.includes('일시') || headerText.includes('time') || headerText.includes('date')) {
                detectedDateIdx = idx;
            }
            if (headerText.includes('pm2.5') || headerText.includes('pm25') || headerText.includes('미세먼지') || headerText.includes('농도')) {
                detectedPm25Idx = idx;
            }
        });

        if (detectedDateIdx !== -1) dateSelect.value = detectedDateIdx;
        if (detectedPm25Idx !== -1) pm25Select.value = detectedPm25Idx;

        document.getElementById('columnSelector').style.display = 'flex';
        
        processData(dateSelect.value, pm25Select.value);
    }

    document.getElementById('reAnalyzeBtn').addEventListener('click', () => {
        const dateIdx = document.getElementById('dateCol').value;
        const pm25Idx = document.getElementById('pm25Col').value;
        processData(dateIdx, pm25Idx);
    });

    // 데이터 분석 처리
    function processData(dateIdx, pm25Idx) {
        let excludedCount = 0;
        
        // 날짜별 농도 모음: { "YYYY-MM-DD": [농도1, 농도2, ...] }
        const dailyData = {};

        rawCsvRows.forEach(row => {
            const dateRaw = row[dateIdx];
            const pm25Raw = row[pm25Idx];

            // 1. 결측치(빈 값) 제거
            if (!dateRaw || !pm25Raw || dateRaw.trim() === '' || pm25Raw.trim() === '') {
                excludedCount++;
                return;
            }

            const pmValue = parseFloat(pm25Raw.replace(/,/g, ''));
            if (isNaN(pmValue) || pmValue < 0) {
                excludedCount++;
                return;
            }

            // 날짜 문자열 정제 (YYYY-MM-DD 추출)
            const dateMatch = dateRaw.match(/\d{4}[-/. ]\d{1,2}[-/. ]\d{1,2}/);
            let dateStr = '';
            if (dateMatch) {
                const parts = dateMatch[0].replace(/[/. ]/g, '-').split('-');
                const year = parts[0];
                const month = parts[1].padStart(2, '0');
                const day = parts[2].padStart(2, '0');
                dateStr = `${year}-${month}-${day}`;
            } else {
                // 단순 날짜 형식 처리 시도
                const d = new Date(dateRaw);
                if (!isNaN(d.getTime())) {
                    const year = d.getFullYear();
                    const month = String(d.getMonth() + 1).padStart(2, '0');
                    const day = String(d.getDate()).padStart(2, '0');
                    dateStr = `${year}-${month}-${day}`;
                } else {
                    excludedCount++;
                    return;
                }
            }

            if (!dailyData[dateStr]) {
                dailyData[dateStr] = [];
            }
            dailyData[dateStr].push(pmValue);
        });

        // 2. 동일한 날짜 데이터 그룹화 -> '일평균 PM2.5 농도' 계산
        const dailyAverages = {};
        let maxDailyPm = 0;

        for (const [dateStr, values] of Object.entries(dailyData)) {
            const sum = values.reduce((acc, v) => acc + v, 0);
            const avg = sum / values.length;
            dailyAverages[dateStr] = avg;
            if (avg > maxDailyPm) {
                maxDailyPm = avg;
            }
        }

        // 3. 일평균 농도 36 ㎍/㎥ 이상인 날을 '나쁨'으로 분류 및 월별 집계
        const monthlyBadDays = {}; // { "YYYY-MM": count }
        let totalBadDays = 0;
        const totalDaysCount = Object.keys(dailyAverages).length;

        for (const [dateStr, avg] of Object.entries(dailyAverages)) {
            const monthKey = dateStr.substring(0, 7); // YYYY-MM
            if (!monthlyBadDays[monthKey]) {
                monthlyBadDays[monthKey] = 0;
            }

            if (avg >= 36) { // '나쁨' 기준: 36 ㎍/㎥ 이상
                monthlyBadDays[monthKey]++;
                totalBadDays++;
            }
        }

        // 결과 화면 표시
        document.getElementById('alertBox').classList.add('active');
        document.getElementById('excludedCount').textContent = excludedCount;

        document.getElementById('totalDays').textContent = `${totalDaysCount} 일`;
        document.getElementById('totalBadDays').textContent = `${totalBadDays} 일`;
        document.getElementById('maxDailyPm').textContent = `${maxDailyPm.toFixed(1)} ㎍/㎥`;

        document.getElementById('statsSection').style.display = 'grid';
        document.getElementById('chartSection').style.display = 'block';

        // 4. 막대그래프 시각화
        renderChart(monthlyBadDays);
    }

    // Chart.js 막대그래프 생성
    function renderChart(monthlyBadDays) {
        const ctx = document.getElementById('badDaysChart').getContext('2d');

        // 월 오름차순 정렬
        const sortedMonths = Object.keys(monthlyBadDays).sort();
        const counts = sortedMonths.map(m => monthlyBadDays[m]);

        if (myChart) {
            myChart.destroy();
        }

        myChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: sortedMonths.map(m => `${m.split('-')[0]}년 ${parseInt(m.split('-')[1])}월`),
                datasets: [{
                    label: "PM2.5 '나쁨' 일수 (일평균 36 ㎍/㎥ 이상)",
                    data: counts,
                    backgroundColor: 'rgba(239, 68, 68, 0.75)',
                    borderColor: 'rgba(185, 28, 28, 1)',
                    borderWidth: 1.5,
                    borderRadius: 4
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                plugins: {
                    legend: {
                        position: 'top',
                    },
                    tooltip: {
                        callbacks: {
                            label: function(context) {
                                return ` 나쁨 발생: ${context.raw} 일 (기준: 36 ㎍/㎥ 이상)`;
                            }
                        }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        ticks: {
                            stepSize: 1,
                            callback: function(value) {
                                return value + ' 일';
                            }
                        },
                        title: {
                            display: true,
                            text: '일수 (일)',
                            font: { weight: 'bold' }
                        }
                    },
                    x: {
                        title: {
                            display: true,
                            text: '월 (Year-Month)',
                            font: { weight: 'bold' }
                        }
                    }
                }
            }
        });
    }
</script>
</body>
</html>

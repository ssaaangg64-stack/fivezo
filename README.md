<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>지구과학탐구: 장마철/비장마철 강수량 분석 도구</title>
    <!-- Chart.js CDN -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --primary: #1e3a8a;
            --primary-light: #3b82f6;
            --bg: #f8fafc;
            --card-bg: #ffffff;
            --text: #1e293b;
            --border: #e2e8f0;
            --danger: #ef4444;
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
            background-color: #eff6ff;
            cursor: pointer;
            transition: background-color 0.2s;
        }

        .drop-zone:hover {
            background-color: #dbeafe;
        }

        .drop-zone input[type="file"] {
            display: none;
        }

        .btn {
            background-color: var(--primary-light);
            color: white;
            border: none;
            padding: 10px 18px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 600;
            margin-top: 12px;
        }

        .btn-sample {
            background-color: #64748b;
            font-size: 0.85rem;
            padding: 6px 12px;
            margin-left: 8px;
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
            grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
            gap: 16px;
            margin-bottom: 24px;
        }

        .stat-card {
            background-color: #f8fafc;
            border: 1px solid var(--border);
            border-radius: 8px;
            padding: 16px;
        }

        .stat-card h3 {
            margin: 0 0 12px 0;
            font-size: 1.1rem;
            color: var(--primary);
            border-bottom: 2px solid var(--border);
            padding-bottom: 6px;
        }

        .stat-item {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
        }

        .stat-value {
            font-weight: 700;
            color: var(--text);
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
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>지구과학탐구: 장마철/비장마철 강수량 비교 분석</h1>
        <p>CSV 파일을 업로드하여 기간별 일 강수량(mm)의 평균 및 최댓값을 자동 산출하고 비교합니다.</p>
    </header>

    <!-- 파일 업로드 카드 -->
    <div class="card">
        <div class="drop-zone" id="dropZone">
            <p style="margin: 0; font-size: 1.1rem; font-weight: 600;">CSV 파일을 드래그하여 넣거나 클릭하여 선택하세요</p>
            <p style="margin: 8px 0 0 0; color: #64748b; font-size: 0.85rem;">(날짜, 시간, 일 강수량(mm), 장마철 여부 항목 포함)</p>
            <input type="file" id="fileInput" accept=".csv">
            <button class="btn" onclick="document.getElementById('fileInput').click()">파일 선택</button>
            <button class="btn btn-sample" id="sampleBtn">샘플 데이터 생성 및 테스트</button>
        </div>

        <!-- 열 선택기 (자동 감지 실패 시 지정) -->
        <div class="column-selector" id="columnSelector" style="display: none;">
            <div>
                <label for="precipCol">일 강수량(mm) 열:</label>
                <select id="precipCol"></select>
            </div>
            <div>
                <label for="statusCol">장마철 여부 열:</label>
                <select id="statusCol"></select>
            </div>
            <button class="btn" style="margin-top:0; padding: 6px 12px;" id="reAnalyzeBtn">다시 분석</button>
        </div>
    </div>

    <!-- 결측치 처리 안내 -->
    <div class="alert-box" id="alertBox">
        <strong>데이터 정제 알림:</strong> 누락되거나 유효하지 않은 데이터 총 <span id="excludedCount" style="font-weight: bold; color: var(--danger);">0</span>개를 분석에서 제외했습니다.
    </div>

    <!-- 결과 통계 카드 -->
    <div class="stats-grid" id="statsSection" style="display: none;">
        <div class="stat-card">
            <h3>장마철 분석 결과</h3>
            <div class="stat-item">
                <span>유효 데이터 수:</span>
                <span class="stat-value" id="monsoonCount">0 일</span>
            </div>
            <div class="stat-item">
                <span>평균 일 강수량:</span>
                <span class="stat-value" id="monsoonAvg">0 mm</span>
            </div>
            <div class="stat-item">
                <span>최댓값 일 강수량:</span>
                <span class="stat-value" id="monsoonMax">0 mm</span>
            </div>
        </div>

        <div class="stat-card">
            <h3>비장마철 분석 결과</h3>
            <div class="stat-item">
                <span>유효 데이터 수:</span>
                <span class="stat-value" id="nonMonsoonCount">0 일</span>
            </div>
            <div class="stat-item">
                <span>평균 일 강수량:</span>
                <span class="stat-value" id="nonMonsoonAvg">0 mm</span>
            </div>
            <div class="stat-item">
                <span>최댓값 일 강수량:</span>
                <span class="stat-value" id="nonMonsoonMax">0 mm</span>
            </div>
        </div>
    </div>

    <!-- 시각화 그래프 카드 -->
    <div class="card" id="chartSection" style="display: none;">
        <h3 style="margin-top:0; color: var(--primary);">일 강수량 분포 및 통계 비교 (mm)</h3>
        <div class="chart-container">
            <canvas id="comparisonChart"></canvas>
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
    dropZone.addEventListener('dragover', (e) => { e.preventDefault(); dropZone.style.backgroundColor = '#dbeafe'; });
    dropZone.addEventListener('dragleave', () => { dropZone.style.backgroundColor = '#eff6ff'; });
    dropZone.addEventListener('drop', (e) => {
        e.preventDefault();
        dropZone.style.backgroundColor = '#eff6ff';
        if (e.dataTransfer.files.length) handleFile(e.dataTransfer.files[0]);
    });
    fileInput.addEventListener('change', (e) => {
        if (e.target.files.length) handleFile(e.target.files[0]);
    });

    // 샘플 데이터 다운로드 및 분석
    document.getElementById('sampleBtn').addEventListener('click', (e) => {
        e.stopPropagation();
        const sampleData = 
`날짜,시간,일 강수량(mm),장마여부
2025-06-20,12:00,0.0,비장마
2025-06-21,12:00,12.5,비장마
2025-06-22,12:00,,비장마
2025-06-25,12:00,45.0,장마
2025-06-26,12:00,110.5,장마
2025-06-27,12:00,88.0,장마
2025-06-28,12:00,15.2,장마
2025-06-29,12:00,N/A,장마
2025-07-10,12:00,5.0,비장마
2025-07-11,12:00,0.0,비장마
2025-07-12,12:00,2.1,비장마`;
        
        parseAndAnalyzeCSV(sampleData);
    });

    function handleFile(file) {
        const reader = new FileReader();
        reader.onload = function (e) {
            parseAndAnalyzeCSV(e.target.result);
        };
        reader.readAsText(file, 'UTF-8');
    }

    // CSV 파싱 및 열 자동 감지
    function parseAndAnalyzeCSV(csvText) {
        const lines = csvText.trim().split(/\r?\n/);
        if (lines.length < 2) {
            alert('유효한 CSV 데이터가 아닙니다.');
            return;
        }

        // CSV 라인 분할 함수 (큰따옴표 처리)
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

        // 열 선택 Dropdown 구성
        const precipSelect = document.getElementById('precipCol');
        const statusSelect = document.getElementById('statusCol');
        precipSelect.innerHTML = '';
        statusSelect.innerHTML = '';

        let detectedPrecipIdx = -1;
        let detectedStatusIdx = -1;

        rawCsvHeaders.forEach((h, idx) => {
            const headerText = h.toLowerCase();
            precipSelect.add(new Option(`${idx + 1}. ${h}`, idx));
            statusSelect.add(new Option(`${idx + 1}. ${h}`, idx));

            if (headerText.includes('강수량') || headerText.includes('강수') || headerText.includes('precip') || headerText.includes('rain')) {
                detectedPrecipIdx = idx;
            }
            if (headerText.includes('장마') || headerText.includes('monsoon') || headerText.includes('구분') || headerText.includes('여부')) {
                detectedStatusIdx = idx;
            }
        });

        // 감지되지 않았을 경우 기본값 할당
        if (detectedPrecipIdx !== -1) precipSelect.value = detectedPrecipIdx;
        if (detectedStatusIdx !== -1) statusSelect.value = detectedStatusIdx;

        document.getElementById('columnSelector').style.display = 'flex';
        
        processData(precipSelect.value, statusSelect.value);
    }

    document.getElementById('reAnalyzeBtn').addEventListener('click', () => {
        const precipIdx = document.getElementById('precipCol').value;
        const statusIdx = document.getElementById('statusCol').value;
        processData(precipIdx, statusIdx);
    });

    // 데이터 정제 및 통계 계산
    function processData(precipIdx, statusIdx) {
        let excludedCount = 0;
        const monsoonData = [];
        const nonMonsoonData = [];

        rawCsvRows.forEach(row => {
            const precipRaw = row[precipIdx];
            const statusRaw = row[statusIdx];

            // 누락값 검사 (빈 문자열, null, undefined, NaN)
            if (precipRaw === undefined || precipRaw === null || precipRaw.trim() === '' ||
                statusRaw === undefined || statusRaw === null || statusRaw.trim() === '') {
                excludedCount++;
                return;
            }

            const precip = parseFloat(precipRaw.replace(/,/g, ''));
            if (isNaN(precip)) {
                excludedCount++;
                return;
            }

            const statusStr = statusRaw.toString().trim().toLowerCase();

            // 장마 / 비장마 구분 logic (장마, 1, true, y 등 대응)
            const isMonsoon = (statusStr === '장마' || statusStr === '장마철' || statusStr === '1' || statusStr === 'true' || statusStr === 'y');
            const isNonMonsoon = (statusStr === '비장마' || statusStr === '비장마철' || statusStr === '0' || statusStr === 'false' || statusStr === 'n');

            if (isMonsoon) {
                monsoonData.push(precip);
            } else if (isNonMonsoon) {
                nonMonsoonData.push(precip);
            } else {
                // 구분 불가능한 라인 처리
                excludedCount++;
            }
        });

        // 결과 업데이트
        document.getElementById('alertBox').classList.add('active');
        document.getElementById('excludedCount').textContent = excludedCount;

        // 통계 계산
        const calcStats = (arr) => {
            if (arr.length === 0) return { avg: 0, max: 0, count: 0 };
            const sum = arr.reduce((acc, v) => acc + v, 0);
            const avg = sum / arr.length;
            const max = Math.max(...arr);
            return { avg: avg.toFixed(2), max: max.toFixed(2), count: arr.length };
        };

        const mStats = calcStats(monsoonData);
        const nmStats = calcStats(nonMonsoonData);

        // UI 통계치 반영 (단위 mm 고정)
        document.getElementById('monsoonCount').textContent = `${mStats.count} 일`;
        document.getElementById('monsoonAvg').textContent = `${mStats.avg} mm`;
        document.getElementById('monsoonMax').textContent = `${mStats.max} mm`;

        document.getElementById('nonMonsoonCount').textContent = `${nmStats.count} 일`;
        document.getElementById('nonMonsoonAvg').textContent = `${nmStats.avg} mm`;
        document.getElementById('nonMonsoonMax').textContent = `${nmStats.max} mm`;

        document.getElementById('statsSection').style.display = 'grid';
        document.getElementById('chartSection').style.display = 'block';

        // 차트 생성
        renderChart(mStats, nmStats);
    }

    // Chart.js 막대그래프 렌더링
    function renderChart(mStats, nmStats) {
        const ctx = document.getElementById('comparisonChart').getContext('2d');

        if (myChart) {
            myChart.destroy();
        }

        myChart = new Chart(ctx, {
            type: 'bar',
            data: {
                labels: ['평균 일 강수량 (mm)', '최댓값 일 강수량 (mm)'],
                datasets: [
                    {
                        label: '장마철',
                        data: [mStats.avg, mStats.max],
                        backgroundColor: 'rgba(59, 130, 246, 0.75)',
                        borderColor: 'rgba(30, 58, 138, 1)',
                        borderWidth: 1.5
                    },
                    {
                        label: '비장마철',
                        data: [nmStats.avg, nmStats.max],
                        backgroundColor: 'rgba(156, 163, 175, 0.75)',
                        borderColor: 'rgba(75, 85, 99, 1)',
                        borderWidth: 1.5
                    }
                ]
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
                                return `${context.dataset.label}: ${context.raw} mm`;
                            }
                        }
                    }
                },
                scales: {
                    y: {
                        beginAtZero: true,
                        title: {
                            display: true,
                            text: '강수량 (mm)',
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

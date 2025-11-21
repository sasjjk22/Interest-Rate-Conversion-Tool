<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>终极全能计算器 (含还款明细)</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

    <style>
        /* --- 核心 UI 样式 --- */
        body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, "Microsoft YaHei", sans-serif; background-color: #eef2f5; display: flex; justify-content: center; padding-top: 20px; margin: 0; color: #333; }
        .container { background: white; width: 94%; max-width: 640px; border-radius: 16px; box-shadow: 0 10px 30px rgba(0,0,0,0.08); overflow: hidden; margin-bottom: 50px; display: flex; flex-direction: column; }
        
        /* 导航 */
        .tabs { display: flex; background: #fff; border-bottom: 1px solid #eee; overflow-x: auto; }
        .tab-btn { flex: 1; min-width: 80px; padding: 15px 5px; border: none; background: transparent; cursor: pointer; font-size: 14px; font-weight: 600; color: #666; white-space: nowrap; transition: 0.2s; }
        .tab-btn:hover { background: #f5faff; color: #007BFF; }
        .tab-btn.active { color: #007BFF; border-bottom: 3px solid #007BFF; background: #f0f7ff; }
        
        /* 内容区 */
        .content { padding: 20px; display: none; animation: fadeIn 0.3s; }
        .content.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }
        
        /* 表单控件 */
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 6px; font-weight: 600; font-size: 13px; color: #555; }
        input, select { width: 100%; padding: 12px; border: 1px solid #dce0e5; border-radius: 8px; font-size: 16px; outline: none; box-sizing: border-box; background: #fff; }
        input:focus, select:focus { border-color: #007BFF; box-shadow: 0 0 0 3px rgba(0,123,255,0.1); }
        
        /* 按钮 */
        button.action-btn { width: 100%; padding: 14px; background: linear-gradient(135deg, #007BFF, #0062cc); color: white; border: none; border-radius: 8px; font-size: 16px; font-weight: bold; cursor: pointer; margin-top: 10px; transition: transform 0.1s; }
        button.action-btn:active { transform: scale(0.98); }
        
        /* 结果区域 */
        .result-box { margin-top: 20px; padding: 15px; background: #f7fdf9; border: 1px solid #c3e6cb; border-radius: 10px; display: none; }
        .result-item { display: flex; justify-content: space-between; margin-bottom: 8px; font-size: 14px; border-bottom: 1px dashed #eee; padding-bottom: 8px; }
        .result-item strong { color: #28a745; font-size: 1.1em; }
        
        /* 图表容器 */
        .chart-wrapper { position: relative; height: 220px; width: 100%; margin-top: 20px; display: none; justify-content: center; }
        
        /* --- 还款计划表样式 (新增) --- */
        .schedule-container { margin-top: 20px; max-height: 300px; overflow-y: auto; border: 1px solid #eee; border-radius: 8px; display: none; }
        table.schedule-tbl { width: 100%; border-collapse: collapse; font-size: 12px; text-align: center; }
        table.schedule-tbl th { background: #f1f3f5; padding: 10px; position: sticky; top: 0; z-index: 1; color: #444; }
        table.schedule-tbl td { padding: 8px; border-bottom: 1px solid #eee; color: #666; }
        table.schedule-tbl tr:nth-child(even) { background-color: #fafbfc; }
        
        /* 反推样式 */
        .reverse-box { background: #fff9db; border-color: #ffeeba; }
        .reverse-box .result-item strong { color: #e6a23c; }

        /* 通用两列布局 */
        .row-input { display: flex; align-items: center; gap: 10px; margin-bottom: 10px; }
        .row-input input { flex: 2; }
        .row-input select { flex: 1; }
    </style>
</head>
<body>

<div class="container">
    <div class="tabs">
        <button class="tab-btn active" onclick="switchTab('loan')">🏠 房贷明细</button>
        <button class="tab-btn" onclick="switchTab('reverse')">📉 反推利率</button>
        <button class="tab-btn" onclick="switchTab('forex')">💱 汇率</button>
        <button class="tab-btn" onclick="switchTab('unit')">⚖️ 工程换算</button>
    </div>

    <div id="loan" class="content active">
        <div class="form-group">
            <label>还款方式</label>
            <select id="repaymentMethod">
                <option value="equal_payment">等额本息 (每月固定)</option>
                <option value="equal_principal">等额本金 (逐月递减)</option>
            </select>
        </div>
        <div class="form-group">
            <label>贷款金额 (万元)</label>
            <input type="number" id="loanAmount" placeholder="100" value="100">
        </div>
        <div class="form-group">
            <label>年利率 (%)</label>
            <input type="number" id="loanRate" placeholder="3.2" value="3.2">
        </div>
        <div class="form-group">
            <label>期限 (年)</label>
            <input type="number" id="loanYears" placeholder="30" value="30">
        </div>
        <button class="action-btn" onclick="calculateLoan()">开始计算 & 生成计划表</button>

        <div id="loanResult" class="result-box">
            <div class="result-item"><span>首月还款：</span><strong id="monthlyPayment">0 元</strong></div>
            <div class="result-item"><span>总利息：</span><strong id="totalInterest">0 万元</strong></div>
            <div class="result-item"><span>本息合计：</span><strong id="totalPayment">0 万元</strong></div>
            
            <div class="chart-wrapper" id="chartBox">
                <canvas id="loanChart"></canvas>
            </div>

            <h4 style="margin: 20px 0 10px; color:#555; border-left: 4px solid #007BFF; padding-left: 10px;">月度还款详情 (下拉查看)</h4>
            <div class="schedule-container" id="scheduleBox">
                <table class="schedule-tbl" id="scheduleTable">
                    <thead>
                        <tr>
                            <th>期数</th>
                            <th>本金</th>
                            <th>利息</th>
                            <th>本期合计</th>
                            <th>剩余本金</th>
                        </tr>
                    </thead>
                    <tbody>
                        </tbody>
                </table>
            </div>
        </div>
    </div>

    <div id="reverse" class="content">
        <div style="background:#eef; padding:10px; border-radius:6px; font-size:13px; color:#557; margin-bottom:15px;">
            💡 输入贷款总额、年限和总利息，反算实际年化利率。
        </div>
        <div class="form-group">
            <label>方式</label>
            <select id="revMethod">
                <option value="equal_payment">等额本息</option>
                <option value="equal_principal">等额本金</option>
            </select>
        </div>
        <div class="form-group">
            <label>本金 (万元)</label>
            <input type="number" id="revPrincipal" placeholder="例如 100">
        </div>
        <div class="form-group">
            <label>期限 (年)</label>
            <input type="number" id="revYears" placeholder="例如 30">
        </div>
        <div class="form-group">
            <label>总利息 (万元)</label>
            <input type="number" id="revInterest" placeholder="例如 55.7">
        </div>
        <button class="action-btn" onclick="calculateReverseRate()" style="background: linear-gradient(135deg, #f6d365, #fda085); color:#444;">反向推算</button>

        <div id="revResult" class="result-box reverse-box">
            <div class="result-item"><span>推算年利率：</span><strong id="revRateResult">0.00%</strong></div>
            <div class="result-item"><span>总还款验证：</span><strong id="revTotalPay">0 万元</strong></div>
        </div>
    </div>

    <div id="forex" class="content">
        <div class="form-group">
            <label>持有</label>
            <div class="row-input">
                <input type="number" id="forexAmount" value="1" oninput="calculateForex()">
                <select id="fromCurrency" onchange="calculateForex()"></select>
            </div>
        </div>
        <div style="text-align:center; color:#ccc;">⬇️</div>
        <div class="form-group">
            <label>目标</label>
            <div class="row-input">
                <input type="text" id="forexResult" readonly style="background:#f4f4f4;">
                <select id="toCurrency" onchange="calculateForex()"></select>
            </div>
        </div>
        <p style="text-align:center; font-size:12px; color:#999;" id="rateStatus">加载中...</p>
        <button class="action-btn" onclick="fetchRates()" style="background:#28a745;">刷新汇率</button>
    </div>

    <div id="unit" class="content">
        <div class="form-group">
            <label>物理量类型</label>
            <select id="unitType" onchange="updateUnitOptions()">
                <option value="pressure">🌡️ 压力 (Pressure)</option>
                <option value="flow">💧 流量 (Flow Rate)</option>
                <option value="velocity">🚀 流速 (Velocity)</option>
                <option value="length">📏 长度 (Length)</option>
                <option value="weight">⚖️ 重量 (Weight)</option>
                <option value="area">⬛ 面积 (Area)</option>
                <option value="volume">🧊 体积 (Volume)</option>
                <option value="data">💾 数据 (Data)</option>
                <option value="time">⏳ 时间 (Time)</option>
            </select>
        </div>
        <div class="row-input">
            <input type="number" id="inputVal" placeholder="输入" oninput="convertUnit()">
            <select id="fromUnit" onchange="convertUnit()"></select>
        </div>
        <div class="row-input">
            <input type="text" id="outputVal" readonly placeholder="结果" style="background:#f4f4f4;">
            <select id="toUnit" onchange="convertUnit()"></select>
        </div>
    </div>
</div>

<script>
    /* --- 全局逻辑 --- */
    function switchTab(name) {
        document.querySelectorAll('.content').forEach(e => e.classList.remove('active'));
        document.querySelectorAll('.tab-btn').forEach(e => e.classList.remove('active'));
        document.getElementById(name).classList.add('active');
        event.currentTarget.classList.add('active');
        if(name === 'forex' && !window.ratesLoaded) fetchRates();
    }

    /* --- 1. 贷款计算 + 计划表 (核心更新) --- */
    let chartInstance = null;
    function calculateLoan() {
        let P = parseFloat(document.getElementById('loanAmount').value) * 10000;
        let r = parseFloat(document.getElementById('loanRate').value) / 100 / 12;
        let n = parseFloat(document.getElementById('loanYears').value) * 12;
        let type = document.getElementById('repaymentMethod').value;
        
        if (!P || !r || !n) return;

        let totalPay=0, totalInt=0, firstPay=0;
        let scheduleHtml = ''; 
        let balance = P;

        // 准备生成表格
        if (type === 'equal_payment') {
            // 等额本息
            let monthly = P * r * Math.pow(1+r, n) / (Math.pow(1+r, n)-1);
            firstPay = monthly;
            totalPay = monthly * n;
            totalInt = totalPay - P;

            for(let i=1; i<=n; i++) {
                let interest = balance * r;
                let principal = monthly - interest;
                balance -= principal;
                if(balance < 0) balance = 0; // 修正精度
                scheduleHtml += `<tr><td>${i}</td><td>${principal.toFixed(2)}</td><td>${interest.toFixed(2)}</td><td>${monthly.toFixed(2)}</td><td>${balance.toFixed(2)}</td></tr>`;
            }
        } else {
            // 等额本金
            let principalMonth = P / n; // 每月本金
            firstPay = principalMonth + P * r;
            totalInt = ((n+1)*P*r)/2;
            totalPay = P + totalInt;

            for(let i=1; i<=n; i++) {
                let interest = balance * r;
                let curPay = principalMonth + interest;
                balance -= principalMonth;
                if(balance < 0) balance = 0;
                scheduleHtml += `<tr><td>${i}</td><td>${principalMonth.toFixed(2)}</td><td>${interest.toFixed(2)}</td><td>${curPay.toFixed(2)}</td><td>${balance.toFixed(2)}</td></tr>`;
            }
        }

        // 填充数据
        document.getElementById('monthlyPayment').innerText = firstPay.toFixed(2) + " 元" + (type==='equal_principal'?' (首月)':'');
        document.getElementById('totalInterest').innerText = (totalInt/10000).toFixed(2) + " 万";
        document.getElementById('totalPayment').innerText = (totalPay/10000).toFixed(2) + " 万";
        
        // 填充表格
        document.querySelector('#scheduleTable tbody').innerHTML = scheduleHtml;
        document.getElementById('loanResult').style.display = 'block';
        document.getElementById('scheduleBox').style.display = 'block';
        document.getElementById('chartBox').style.display = 'flex';

        // 图表
        if (chartInstance) chartInstance.destroy();
        chartInstance = new Chart(document.getElementById('loanChart'), {
            type: 'doughnut',
            data: { labels: ['本金', '总利息'], datasets: [{ data: [P, totalInt], backgroundColor: ['#36A2EB', '#FF6384'] }] },
            options: { maintainAspectRatio: false }
        });
    }

    /* --- 2. 反推利率 --- */
    function calculateReverseRate() {
        let method = document.getElementById('revMethod').value;
        let P = parseFloat(document.getElementById('revPrincipal').value) * 10000;
        let n = parseFloat(document.getElementById('revYears').value) * 12;
        let I = parseFloat(document.getElementById('revInterest').value) * 10000;
        if (!P || !n || !I) { alert("请输入完整数据"); return; }

        let finalRate = 0;
        if (method === 'equal_principal') {
            finalRate = ((2 * I) / ((n + 1) * P)) * 12 * 100;
        } else {
            let target = P + I;
            let min=0, max=1, guess=0;
            for(let i=0; i<100; i++) {
                guess = (min+max)/2;
                if(guess===0) continue;
                let term = Math.pow(1+guess, n);
                let m = P * (guess*term)/(term-1);
                let calc = m * n;
                if(Math.abs(calc - target) < 1) break;
                if(calc > target) max = guess; else min = guess;
            }
            finalRate = guess * 12 * 100;
        }
        document.getElementById('revRateResult').innerText = finalRate.toFixed(3) + " %";
        document.getElementById('revTotalPay').innerText = ((P+I)/10000).toFixed(2) + " 万";
        document.getElementById('revResult').style.display = 'block';
    }

    /* --- 3. 汇率 (含更多币种) --- */
    const api = 'https://api.exchangerate-api.com/v4/latest/CNY';
    const cList = ['CNY','USD','EUR','HKD','JPY','GBP','AUD','CAD','SGD','KRW','THB','TWD','RUB','MYR','INR'];
    const cName = {'CNY':'🇨🇳 人民币','USD':'🇺🇸 美元','EUR':'🇪🇺 欧元','HKD':'🇭🇰 港币','JPY':'🇯🇵 日元','GBP':'🇬🇧 英镑','AUD':'🇦🇺 澳元','CAD':'🇨🇦 加元','SGD':'🇸🇬 新币','KRW':'🇰🇷 韩元','THB':'🇹🇭 泰铢','TWD':'🇹🇼 台币','RUB':'🇷🇺 卢布','MYR':'🇲🇾 马币','INR':'🇮🇳 卢比'};
    let rates = {};

    async function fetchRates() {
        document.getElementById('rateStatus').innerText = "正在更新...";
        try {
            let res = await fetch(api);
            let data = await res.json();
            rates = data.rates;
            window.ratesLoaded = true;
            let ops = cList.map(c => `<option value="${c}">${cName[c]||c} (${c})</option>`).join('');
            let s1 = document.getElementById('fromCurrency');
            let s2 = document.getElementById('toCurrency');
            if(s1.options.length===0) { s1.innerHTML = ops; s2.innerHTML = ops; s1.value='USD'; s2.value='CNY'; }
            document.getElementById('rateStatus').innerText = "更新时间: " + data.date;
            calculateForex();
        } catch(e) { document.getElementById('rateStatus').innerText = "网络错误"; }
    }
    function calculateForex() {
        if(!window.ratesLoaded) return;
        let res = (parseFloat(document.getElementById('forexAmount').value) / rates[document.getElementById('fromCurrency').value]) * rates[document.getElementById('toCurrency').value];
        document.getElementById('forexResult').value = res.toFixed(4);
    }

    /* --- 4. 万能单位 (新增压力/流量/流速) --- */
    const uData = {
        // 新增工程单位
        pressure: { r: {'pa':1, 'mpa':1000000, 'bar':100000, 'psi':6894.76, 'atm':101325, 'mmhg':133.322}, n: {'pa':'帕斯卡 (Pa)','mpa':'兆帕 (MPa)','bar':'巴 (Bar)','psi':'磅力/平方英寸','atm':'标准大气压','mmhg':'毫米汞柱'} },
        flow: { r: {'m3s':1, 'm3h':0.000277778, 'lmin':0.000016667, 'gpm':0.00006309}, n: {'m3s':'立方米/秒','m3h':'立方米/小时','lmin':'升/分','gpm':'加仑/分(美)'} },
        velocity: { r: {'ms':1, 'kmh':0.277778, 'fts':0.3048, 'knot':0.51444}, n: {'ms':'米/秒','kmh':'千米/小时','fts':'英尺/秒','knot':'节 (海里/时)'} },
        
        // 原有单位
        length: { r: {'m':1, 'km':0.001, 'cm':100, 'mm':1000, 'inch':39.37}, n: {'m':'米','km':'千米','cm':'厘米','mm':'毫米','inch':'英寸'} },
        weight: { r: {'kg':1, 'g':1000, 't':0.001, 'lb':2.204}, n: {'kg':'千克','g':'克','t':'吨','lb':'磅'} },
        area: { r: {'m2':1, 'ha':0.0001, 'acre':0.000247}, n: {'m2':'平米','ha':'公顷','acre':'英亩'} },
        volume: { r: {'m3':1, 'l':1000, 'gal':264.17}, n: {'m3':'立方米','l':'升','gal':'加仑'} },
        data: { r: {'gb':1, 'mb':1024, 'tb':0.000976}, n: {'gb':'GB','mb':'MB','tb':'TB'} },
        time: { r: {'min':1, 'hr':0.01666, 'sec':60}, n: {'min':'分','hr':'时','sec':'秒'} }
    };

    function updateUnitOptions() {
        let t = document.getElementById('unitType').value;
        let n = uData[t].n;
        let h = Object.keys(n).map(k => `<option value="${k}">${n[k]}</option>`).join('');
        document.getElementById('fromUnit').innerHTML = h;
        document.getElementById('toUnit').innerHTML = h;
        if(Object.keys(n).length>1) document.getElementById('toUnit').selectedIndex=1;
        convertUnit();
    }

    function convertUnit() {
        let t = document.getElementById('unitType').value;
        let v = parseFloat(document.getElementById('inputVal').value);
        let f = document.getElementById('fromUnit').value;
        let to = document.getElementById('toUnit').value;
        if(isNaN(v)) { document.getElementById('outputVal').value=''; return; }
        
        let r = uData[t].r;
        let res = (v / r[f]) * r[to]; // 基准法：先转为基准单位(系数为1的)，再转目标
        document.getElementById('outputVal').value = parseFloat(res.toPrecision(7));
    }
    updateUnitOptions();
</script>

</body>
</html>

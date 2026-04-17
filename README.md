<!DOCTYPE html>
<html lang="zh-TW">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>OpenClaw 自动化月报</title>
<link href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@300;400;500;700&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
<style>
*{box-sizing:border-box;margin:0;padding:0;}
body{background:#0f1117;color:#e8e6e0;font-family:'M PLUS Rounded 1c',sans-serif;padding:2rem 1.5rem;min-height:100vh;}
@media(max-width:700px){body{padding:1rem;}}

.top-bar{display:flex;align-items:flex-start;justify-content:space-between;flex-wrap:wrap;gap:12px;margin-bottom:1.5rem;}
.live-dot{width:8px;height:8px;border-radius:50%;background:#f472b6;animation:blink 1.5s infinite;flex-shrink:0;}
@keyframes blink{0%,100%{opacity:1;}50%{opacity:0.3;}}
.top-title{font-size:18px;font-weight:500;color:#fff;}
.top-sub{font-size:11px;color:rgba(255,200,220,0.35);margin-top:3px;}
.top-right{display:flex;align-items:center;gap:8px;flex-wrap:wrap;}
.pill{font-size:11px;padding:5px 14px;border-radius:20px;border:0.5px solid rgba(244,114,182,0.35);color:#f9a8d4;background:rgba(244,114,182,0.08);cursor:pointer;transition:background .15s;}
.pill:hover{background:rgba(244,114,182,0.18);}
.pill-time{font-size:11px;padding:5px 12px;border-radius:20px;background:#1e2130;color:rgba(255,255,255,0.35);}
.status-badge{font-size:11px;padding:5px 12px;border-radius:20px;display:flex;align-items:center;gap:6px;background:#1e2130;color:rgba(255,255,255,0.35);}
.status-badge.ok{background:rgba(244,114,182,0.1);color:#f9a8d4;border:0.5px solid rgba(244,114,182,0.25);}
.status-badge.err{background:rgba(251,100,80,0.1);color:#fca5a5;border:0.5px solid rgba(251,100,80,0.25);}
.status-badge.loading{background:rgba(167,139,250,0.1);color:#c4b5fd;border:0.5px solid rgba(167,139,250,0.25);}
.sdot{width:6px;height:6px;border-radius:50%;background:currentColor;}
.sdot.blink{animation:blink 1.2s infinite;}

.sec-head{display:flex;align-items:center;gap:8px;margin-bottom:10px;}
.sec-line{width:3px;height:14px;border-radius:2px;background:#f472b6;flex-shrink:0;}
.sec-title{font-size:11px;font-weight:500;color:rgba(255,255,255,0.38);letter-spacing:.06em;text-transform:uppercase;}

.metrics{display:grid;grid-template-columns:repeat(4,1fr);gap:8px;margin-bottom:1.5rem;}
@media(max-width:600px){.metrics{grid-template-columns:repeat(2,1fr);}}
.mc{background:#1a1d27;border-radius:10px;padding:14px 16px;border:0.5px solid rgba(255,255,255,0.06);}
.mc-bar{height:2px;border-radius:2px;margin-bottom:10px;}
.b1{background:linear-gradient(90deg,#f472b6,#e879f9);}
.b2{background:linear-gradient(90deg,#e879f9,#a78bfa);}
.b3{background:linear-gradient(90deg,#f472b6,#fb923c);}
.b4{background:linear-gradient(90deg,#a78bfa,#60a5fa);}
.mc-label{font-size:11px;color:rgba(255,255,255,0.35);margin-bottom:5px;}
.mc-val{font-size:22px;font-weight:500;color:#fff;}
.mc-note{font-size:10px;margin-top:4px;color:#f9a8d4;}

.kanban{display:grid;gap:10px;margin-bottom:1.5rem;}
.week-col{background:#1a1d27;border-radius:12px;overflow:hidden;border:0.5px solid rgba(255,255,255,0.06);}
.wh-bar{height:2px;}
.week-head{padding:10px 14px;display:flex;align-items:center;gap:8px;font-size:12px;font-weight:500;border-bottom:0.5px solid rgba(255,255,255,0.06);}
.week-dot{width:6px;height:6px;border-radius:50%;flex-shrink:0;}
.week-label{color:rgba(255,255,255,0.75);}
.week-period{font-size:10px;color:rgba(255,255,255,0.3);margin-left:auto;}
.week-body{padding:10px;}
.section{margin-top:8px;}

.stag{display:inline-flex;align-items:center;font-size:10px;font-weight:500;padding:3px 9px;border-radius:20px;margin-bottom:6px;}
.s1{background:rgba(244,114,182,0.1);color:#f9a8d4;border:0.5px solid rgba(244,114,182,0.2);}
.s2{background:rgba(167,139,250,0.1);color:#c4b5fd;border:0.5px solid rgba(167,139,250,0.2);}
.s3{background:rgba(96,165,250,0.1);color:#93c5fd;border:0.5px solid rgba(96,165,250,0.2);}
.s4{background:rgba(251,100,80,0.1);color:#fca5a5;border:0.5px solid rgba(251,100,80,0.2);}
.s5{background:rgba(251,146,60,0.1);color:#fdba74;border:0.5px solid rgba(251,146,60,0.2);}

.card{background:#13151e;border:0.5px solid rgba(255,255,255,0.05);border-radius:8px;padding:10px 12px;margin-bottom:6px;}
.sr{display:flex;justify-content:space-between;padding:3px 0;border-bottom:0.5px solid rgba(255,255,255,0.05);}
.sr:last-child{border-bottom:none;}
.sk{font-size:11px;color:rgba(255,255,255,0.3);}
.sv{font-size:11px;font-weight:500;color:#fff;}
.sv-hi{color:#f472b6;}
.pt{font-size:12px;font-weight:500;color:#fff;margin-bottom:4px;}
.pd{font-size:11px;color:rgba(255,255,255,0.35);line-height:1.55;margin-bottom:3px;}
.pr{font-size:11px;color:#c4b5fd;line-height:1.55;}
.pr::before{content:"→ ";opacity:0.5;}
.ri-label{font-size:10px;font-weight:500;color:#fca5a5;margin-bottom:4px;}
.ri{font-size:11px;color:rgba(252,165,165,0.55);line-height:1.6;margin-bottom:6px;}
.ri:last-child{margin-bottom:0;}
.func-tag{display:inline-block;font-size:10px;padding:2px 8px;border-radius:6px;background:rgba(255,255,255,0.05);border:0.5px solid rgba(255,255,255,0.08);color:rgba(255,255,255,0.35);margin:2px;}
.pi{display:flex;gap:7px;padding:3px 0;border-bottom:0.5px solid rgba(255,255,255,0.04);}
.pi:last-child{border-bottom:none;}
.pn{font-size:10px;color:#fdba74;flex-shrink:0;margin-top:1px;}
.pt2{font-size:11px;color:rgba(255,255,255,0.35);line-height:1.5;}

.two{display:grid;grid-template-columns:repeat(2,1fr);gap:10px;margin-bottom:1.5rem;}
@media(max-width:600px){.two{grid-template-columns:1fr;}}
.chart-card{background:#1a1d27;border:0.5px solid rgba(255,255,255,0.06);border-radius:12px;padding:14px 16px;}
.chart-title{font-size:12px;font-weight:500;color:rgba(255,255,255,0.6);margin-bottom:12px;}
.cw{position:relative;width:100%;height:180px;}

.divider{border:none;border-top:0.5px solid rgba(255,255,255,0.06);margin:1.25rem 0;}
.badges{display:flex;flex-wrap:wrap;gap:5px;margin-top:6px;}
.badge{font-size:10px;padding:2px 9px;border-radius:20px;background:#1a1d27;color:rgba(255,255,255,0.3);border:0.5px solid rgba(255,255,255,0.08);}

.week-colors{--c0:#f472b6;--c1:#a78bfa;--c2:#fb923c;--c3:#60a5fa;--c4:#e879f9;}
</style>
</head>
<body class="week-colors">

<div class="top-bar">
  <div style="display:flex;align-items:center;gap:10px;">
    <span class="live-dot"></span>
    <div>
      <div class="top-title">OpenClaw 自动化月报</div>
      <div class="top-sub" id="topSub">商务中心 · 客服组 · 负责人：雨棠</div>
    </div>
  </div>
  <div class="top-right">
    <div class="status-badge loading" id="statusBadge">
      <span class="sdot blink"></span>
      <span id="statusText">读取中...</span>
    </div>
    <button class="pill" onclick="loadData()">重新整理</button>
  </div>
</div>

<div class="sec-head"><span class="sec-line"></span><span class="sec-title">关键指标总览</span></div>
<div class="metrics" id="metricsArea">
  <div class="mc"><div class="mc-bar b1"></div><div class="mc-label">月度总触发</div><div class="mc-val">—</div></div>
  <div class="mc"><div class="mc-bar b2"></div><div class="mc-label">综合成功率</div><div class="mc-val">—</div></div>
  <div class="mc"><div class="mc-bar b3"></div><div class="mc-label">总字符消耗</div><div class="mc-val">—</div></div>
  <div class="mc"><div class="mc-bar b4"></div><div class="mc-label">活跃智能体</div><div class="mc-val">—</div></div>
</div>

<div class="sec-head"><span class="sec-line"></span><span class="sec-title">周度看板</span></div>
<div class="kanban" id="kanbanArea">
  <div style="text-align:center;padding:3rem;color:rgba(255,255,255,0.2);font-size:13px;">载入中...</div>
</div>

<div class="divider"></div>
<div class="sec-head"><span class="sec-line"></span><span class="sec-title">趋势图表</span></div>
<div class="two">
  <div class="chart-card">
    <div class="chart-title">任务触发次数</div>
    <div class="cw"><canvas id="c1" role="img" aria-label="各周触发次数趋势">各周触发次数</canvas></div>
  </div>
  <div class="chart-card">
    <div class="chart-title">任务执行成功率</div>
    <div class="cw"><canvas id="c2" role="img" aria-label="各周成功率趋势">各周成功率</canvas></div>
  </div>
</div>

<div class="divider"></div>
<div style="font-size:11px;color:rgba(255,255,255,0.25);">高频指令
  <div class="badges" id="badgeArea">
    <span class="badge">/T1</span><span class="badge">/T1E</span><span class="badge">/T1W</span>
    <span class="badge">/daily</span><span class="badge">/dailydd</span>
    <span class="badge">/WS</span><span class="badge">/WR</span>
    <span class="badge">/QWS</span><span class="badge">/CWS</span><span class="badge">/XWS</span>
  </div>
</div>

<script>
const CSV_URL = 'https://docs.google.com/spreadsheets/d/e/2PACX-1vR4MZC5X5HMRykJcEJ79oPYOsld-qLeiDfkPp2vDf661UR6r5Tp15xuJVYtRzQrMC94AUIvx3cSO6XK/pub?output=csv';

const WEEK_COLORS = [
  {dot:'#f472b6', bar:'linear-gradient(90deg,#f472b6,#e879f9,#a78bfa,#60a5fa)'},
  {dot:'#a78bfa', bar:'linear-gradient(90deg,#e879f9,#a78bfa,#60a5fa,#f472b6)'},
  {dot:'#fb923c', bar:'linear-gradient(90deg,#f472b6,#fb923c,#fbbf24,#e879f9)'},
  {dot:'#60a5fa', bar:'linear-gradient(90deg,#60a5fa,#a78bfa,#f472b6,#fb923c)'},
  {dot:'#e879f9', bar:'linear-gradient(90deg,#e879f9,#f472b6,#fb923c,#a78bfa)'},
];
const CHART_COLORS = ['#f472b6','#a78bfa','#fb923c','#60a5fa','#e879f9'];

let chart1 = null, chart2 = null;

function setStatus(type, text){
  const b = document.getElementById('statusBadge');
  b.className = 'status-badge ' + type;
  b.innerHTML = `<span class="sdot${type==='loading'?' blink':''}"></span><span>${text}</span>`;
}

function parseRows(results){
  const rows = results.data;
  if(!rows || rows.length < 2) return null;
  const headers = rows[0].map(h => h.trim());
  return rows.slice(1).filter(r => r.some(c => c && c.trim())).map(r => {
    const obj = {};
    headers.forEach((h, i) => obj[h] = (r[i] || '').trim());
    obj['触发次数'] = parseFloat(obj['触发次数']) || 0;
    obj['成功率'] = parseFloat(obj['成功率']) || 0;
    obj['字符消耗'] = parseFloat(obj['字符消耗']) || 0;
    obj['活跃智能体数'] = parseFloat(obj['活跃智能体数']) || 0;
    return obj;
  });
}

function renderMetrics(data){
  const total = data.reduce((s,r) => s + r['触发次数'], 0);
  const avgRate = (data.reduce((s,r) => s + r['成功率'], 0) / data.length).toFixed(1);
  const totalChar = data.reduce((s,r) => s + r['字符消耗'], 0);
  const maxAgents = Math.max(...data.map(r => r['活跃智能体数']));
  const best = data.reduce((a,b) => a['成功率'] > b['成功率'] ? a : b);
  document.getElementById('metricsArea').innerHTML = `
    <div class="mc"><div class="mc-bar b1"></div><div class="mc-label">月度总触发</div><div class="mc-val">${total.toLocaleString()}</div><div class="mc-note">${data.length} 周累计</div></div>
    <div class="mc"><div class="mc-bar b2"></div><div class="mc-label">综合成功率</div><div class="mc-val">${avgRate}%</div><div class="mc-note">最高 ${best['成功率']}%（${best['周次']}）</div></div>
    <div class="mc"><div class="mc-bar b3"></div><div class="mc-label">总字符消耗</div><div class="mc-val">${totalChar.toLocaleString()}</div><div class="mc-note">${data.length} 周累计</div></div>
    <div class="mc"><div class="mc-bar b4"></div><div class="mc-label">活跃智能体</div><div class="mc-val">${maxAgents}</div><div class="mc-note">当前最大值</div></div>`;
}

function renderWeekCard(row, i){
  const c = WEEK_COLORS[i % WEEK_COLORS.length];
  const allKeys = Object.keys(row);
  const sysKeys = new Set(['周次','汇报周期','负责人','部门','活跃智能体数','触发次数','成功率','字符消耗','平均响应','高频功能调用','异常记录','优化对策']);
  const descKeys = allKeys.filter(k => k.includes('描述'));
  const resultKeys = allKeys.filter(k => k.includes('成果'));
  const planKeys = allKeys.filter(k => k.startsWith('★'));
  const progTitleKeys = allKeys.filter(k => !sysKeys.has(k) && !k.includes('描述') && !k.includes('成果') && !k.startsWith('★'));

  // 一、运行数据
  const sec1 = `<div class="section"><span class="stag s1">一、运行数据概览</span>
    <div class="card">
      <div class="sr"><span class="sk">活跃智能体</span><span class="sv">${row['活跃智能体数']} 个</span></div>
      <div class="sr"><span class="sk">触发次数</span><span class="sv">${row['触发次数']} 次</span></div>
      <div class="sr"><span class="sk">成功率</span><span class="sv sv-hi">${row['成功率']}%</span></div>
      <div class="sr"><span class="sk">字符消耗</span><span class="sv">${Number(row['字符消耗']).toLocaleString()}</span></div>
      <div class="sr"><span class="sk">平均响应</span><span class="sv">${row['平均响应'] || '—'}</span></div>
    </div></div>`;

  // 二、核心进展
  let progHtml = '';
  const maxP = Math.max(progTitleKeys.length, descKeys.length);
  for(let n = 0; n < Math.min(maxP, 3); n++){
    const title = progTitleKeys[n] ? row[progTitleKeys[n]] : '';
    const desc = descKeys[n] ? row[descKeys[n]] : '';
    const res = resultKeys[n] ? row[resultKeys[n]] : '';
    if(!title && !desc && !res) continue;
    progHtml += `<div class="card">
      ${title ? `<div class="pt">${title}</div>` : ''}
      ${desc ? `<div class="pd">${desc}</div>` : ''}
      ${res ? `<div class="pr">${res}</div>` : ''}
    </div>`;
  }
  const sec2 = progHtml ? `<div class="section"><span class="stag s2">二、核心自动化进展</span>${progHtml}</div>` : '';

  // 三、资源消耗
  const funcs = row['高频功能调用'] || '';
  const sec3 = funcs ? `<div class="section"><span class="stag s3">三、资源消耗</span>
    <div class="card"><div style="display:flex;flex-wrap:wrap;">
      ${funcs.split(/[，,、]/).filter(f=>f.trim()).map(f=>`<span class="func-tag">${f.trim()}</span>`).join('')}
    </div></div></div>` : '';

  // 四、风险记录
  const abnormal = row['异常记录'] || '';
  const optimize = row['优化对策'] || '';
  const sec4 = (abnormal || optimize) ? `<div class="section"><span class="stag s4">四、风险记录与优化</span>
    <div class="card">
      ${abnormal ? `<div class="ri-label">异常记录</div><div class="ri">${abnormal}</div>` : ''}
      ${optimize ? `<div class="ri-label">优化对策</div><div class="ri">${optimize}</div>` : ''}
    </div></div>` : '';

  // 五、演进计划
  const plans = planKeys.map(k => row[k]).filter(Boolean);
  const sec5 = plans.length ? `<div class="section"><span class="stag s5">五、下周演进计划</span>
    <div class="card">${plans.map((p,idx) => `<div class="pi"><span class="pn">${idx+1}.</span><span class="pt2">${p}</span></div>`).join('')}</div></div>` : '';

  return `<div class="week-col">
    <div class="wh-bar" style="background:${c.bar};height:2px;"></div>
    <div class="week-head">
      <span class="week-dot" style="background:${c.dot};"></span>
      <span class="week-label">${row['周次']}</span>
      <span class="week-period">${row['汇报周期'] || ''}</span>
    </div>
    <div class="week-body">${sec1}${sec2}${sec3}${sec4}${sec5}</div>
  </div>`;
}

function renderKanban(data){
  const kb = document.getElementById('kanbanArea');
  const cols = Math.min(data.length, 3);
  kb.style.gridTemplateColumns = `repeat(${cols}, 1fr)`;
  kb.innerHTML = data.map((row, i) => renderWeekCard(row, i)).join('');
  const sub = `商务中心 · 客服组 · 负责人：${data[0]['负责人'] || '雨棠'} · 第 ${data[0]['周次']}–${data[data.length-1]['周次']} 周`;
  document.getElementById('topSub').textContent = sub;
}

function renderCharts(data){
  const tc = 'rgba(255,255,255,0.3)';
  const gc = 'rgba(255,255,255,0.05)';
  const labels = data.map(r => r['周次']);
  const base = {responsive:true,maintainAspectRatio:false,plugins:{legend:{display:false}},scales:{x:{ticks:{color:tc,font:{size:11,family:"'M PLUS Rounded 1c'"}},grid:{color:gc},border:{display:false}},y:{ticks:{color:tc,font:{size:11}},grid:{color:gc},border:{display:false}}}};
  const barColors = data.map((_,i) => CHART_COLORS[i % CHART_COLORS.length]);
  if(chart1) chart1.destroy();
  if(chart2) chart2.destroy();
  chart1 = new Chart(document.getElementById('c1'), {type:'bar', data:{labels, datasets:[{data:data.map(r=>r['触发次数']), backgroundColor:barColors.map(c=>c+'99'), borderColor:barColors, borderWidth:1, borderRadius:5}]}, options:base});
  chart2 = new Chart(document.getElementById('c2'), {type:'line', data:{labels, datasets:[{data:data.map(r=>r['成功率']), borderColor:'#f472b6', backgroundColor:'rgba(244,114,182,0.08)', pointBackgroundColor:'#f472b6', pointRadius:5, tension:0.3, fill:true}]}, options:{...base, scales:{...base.scales, y:{...base.scales.y, min:60, max:100, ticks:{...base.scales.y.ticks, callback:v=>v+'%'}}}}});
}

async function loadData(){
  setStatus('loading', '读取中...');
  try{
    const res = await fetch(CSV_URL);
    if(!res.ok) throw new Error('HTTP ' + res.status);
    const text = await res.text();
    const parsed = Papa.parse(text, {skipEmptyLines:true});
    const data = parseRows(parsed);
    if(!data || data.length === 0) throw new Error('无数据，请确认 Sheets 栏位标题正确');
    renderMetrics(data);
    renderKanban(data);
    renderCharts(data);
    const now = new Date().toLocaleTimeString('zh-TW', {hour:'2-digit', minute:'2-digit'});
    setStatus('ok', '已同步 ' + now);
  }catch(e){
    setStatus('err', '读取失败：' + e.message);
  }
}

loadData();
</script>
</body>
</html>

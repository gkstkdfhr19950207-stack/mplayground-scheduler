[매장_일별_스케줄러.html](https://github.com/user-attachments/files/29959971/_._.html)
# mplayground-scheduler
매장 스케쥴 관리 프로그램입니
[Uploading 매장_일별_스케줄러.html…]()
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=5.0">
<title>매장 일별 스케줄러</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/exceljs/4.4.0/exceljs.min.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-database-compat.js"></script>
<style>
  :root{
    --ink:#1c2333;
    --paper:#f6f7fa;
    --card:#ffffff;
    --line:#e2e6ee;
    --accent:#2f4bd8;
    --muted:#7b8496;
  }
  *{box-sizing:border-box; margin:0; padding:0;}
  body{
    font-family:'Pretendard','Apple SD Gothic Neo','Malgun Gothic',sans-serif;
    background:var(--paper); color:var(--ink); padding:24px;
  }
  .wrap{max-width:1500px; margin:0 auto;}

  header{
    display:flex; flex-wrap:wrap; align-items:flex-end; gap:16px;
    margin-bottom:16px;
  }
  h1{font-size:22px; font-weight:800; letter-spacing:-0.5px;}
  h1 span{color:var(--accent);}
  .meta{display:flex; gap:8px; flex-wrap:wrap; align-items:center;}
  .meta input[type=text]{
    border:1px solid var(--line); border-radius:8px; padding:8px 12px;
    font-size:14px; width:180px; background:var(--card);
  }
  .meta input[type=date]{
    border:1px solid var(--line); border-radius:8px; padding:7px 10px;
    font-size:14px; background:var(--card);
  }
  .spacer{flex:1;}
  .btn{
    border:1px solid var(--line); background:var(--card); border-radius:8px;
    padding:8px 14px; font-size:14px; cursor:pointer; font-weight:600;
  }
  .btn:hover{border-color:var(--accent); color:var(--accent);}
  .btn.primary{background:var(--accent); color:#fff; border-color:var(--accent);}
  .btn.primary:hover{opacity:.9; color:#fff;}

  /* 매출 패널 */
  .sales{
    background:var(--card); border:1px solid var(--line); border-radius:12px;
    padding:14px 16px; margin-bottom:14px;
  }
  .sales h2{font-size:15px; margin-bottom:12px;}
  .sales-grid{
    display:grid; grid-template-columns:repeat(auto-fit, minmax(150px, 1fr)); gap:10px;
  }
  .field{display:flex; flex-direction:column; gap:4px;}
  .field label{font-size:12px; font-weight:700; color:var(--muted);}
  .field input{
    border:1px solid var(--line); border-radius:8px; padding:8px 10px;
    font-size:14px; font-weight:700; text-align:right; font-family:inherit;
    background:#fbfcfe; color:var(--ink); width:100%;
  }
  .field input:focus{outline:2px solid var(--accent); outline-offset:-1px;}
  .field.calc input{
    background:#eef4ff; border-color:#c6d4f7; color:var(--accent); cursor:default;
  }
  .field .unit{font-size:11px; color:var(--muted); text-align:right;}
  .progress-wrap{grid-column:1 / -1; margin-top:2px;}
  .progress-bar{
    height:22px; background:#edf0f6; border-radius:999px; overflow:hidden; position:relative;
  }
  .progress-fill{
    height:100%; background:linear-gradient(90deg,#4c6ef5,#2f4bd8);
    border-radius:999px; transition:width .3s; min-width:0;
  }
  .progress-label{
    position:absolute; inset:0; display:flex; align-items:center; justify-content:center;
    font-size:12px; font-weight:800; color:var(--ink);
  }

  /* 업무 팔레트 */
  .palette{
    display:flex; flex-wrap:wrap; gap:8px; align-items:center;
    background:var(--card); border:1px solid var(--line); border-radius:12px;
    padding:12px 14px; margin-bottom:14px;
  }
  .palette .label{font-size:13px; color:var(--muted); font-weight:700; margin-right:4px;}
  .chip{
    display:flex; align-items:center; gap:6px;
    border:2px solid transparent; border-radius:999px;
    padding:6px 13px; font-size:13.5px; font-weight:700; cursor:pointer;
    color:#fff; user-select:none; transition:transform .08s;
  }
  .chip .dot{width:8px; height:8px; border-radius:50%; background:rgba(255,255,255,.85);}
  .chip.selected{border-color:var(--ink); transform:scale(1.06); box-shadow:0 2px 8px rgba(0,0,0,.18);}
  .chip.eraser{background:#fff; color:var(--ink); border:2px dashed var(--line);}
  .chip.eraser.selected{border:2px solid var(--ink);}
  .chip.floor{background:#fff; color:var(--ink); border:2px solid var(--line); padding:5px 12px;}
  .chip.floor.selected{background:var(--ink); color:#fff; border-color:var(--ink); transform:scale(1.06);}
  .pal-break{flex-basis:100%; height:0;}
  .modes{display:flex; gap:6px; margin-right:10px;}
  .mode-btn{
    border:1px solid var(--line); background:#fff; border-radius:8px;
    padding:6px 12px; font-size:13px; font-weight:700; cursor:pointer; color:var(--muted);
    font-family:inherit;
  }
  .mode-btn.active{background:var(--accent); color:#fff; border-color:var(--accent);}
  td.slot.sel{
    outline:2px solid var(--accent); outline-offset:-2px;
    box-shadow:inset 0 0 0 100px rgba(47,75,216,.22);
  }
  .toast{
    position:fixed; bottom:24px; left:50%; transform:translateX(-50%);
    background:var(--ink); color:#fff; padding:10px 18px; border-radius:10px;
    font-size:13.5px; font-weight:700; opacity:0; pointer-events:none;
    transition:opacity .25s; z-index:99;
  }
  .toast.show{opacity:1;}
  .chip.휴게{color:#4a5261;}
  .hint{font-size:12px; color:var(--muted); margin-left:auto;}

  /* 표 */
  .board{
    background:var(--card); border:1px solid var(--line); border-radius:12px;
    overflow:auto; padding-bottom:4px;
  }
  table{border-collapse:collapse; width:100%; min-width:1400px;}
  th,td{border:1px solid var(--line); font-size:12px; text-align:center;}
  thead th{
    background:#eef1f7; padding:5px 2px; font-weight:700; position:sticky; top:0; z-index:2;
  }
  thead tr:nth-child(2) th{top:29px; font-weight:600; font-size:10.5px; color:var(--muted); padding:3px 1px;}
  th.name-col{min-width:100px; position:sticky; left:0; z-index:3; background:#eef1f7;}
  th.rank-col{min-width:78px;}
  th.time-col{min-width:70px;}
  td.name-cell{
    position:sticky; left:0; background:#fff; z-index:1; min-width:100px;
    padding:4px 6px;
  }
  td.name-cell input, td.rank-cell input{
    border:none; width:100%; font-size:13px; font-weight:700; text-align:center;
    font-family:inherit; background:transparent; outline:none;
  }
  td.name-cell input:focus, td.rank-cell input:focus{background:#f0f3ff;}
  td.rank-cell{padding:4px 4px; background:#fff;}
  td.rank-cell input{font-weight:500; color:#3a4152;}
  td.time-cell{padding:2px; background:#fff;}
  td.time-cell select{
    border:1px solid transparent; border-radius:6px; padding:4px 0px;
    font-size:12px; font-family:inherit; background:transparent; cursor:pointer;
    font-weight:600; color:var(--ink); width:100%; text-align:center;
  }
  td.time-cell select:hover{border-color:var(--line); background:#f7f9fd;}
  td.slot{
    height:42px; min-width:36px; max-width:44px; cursor:pointer; font-weight:700; font-size:9.5px;
    color:#fff; user-select:none; -webkit-user-select:none;
    word-break:break-all; line-height:1.15; padding:1px;
  }
  td.slot:hover{outline:2px solid var(--accent); outline-offset:-2px;}
  td.slot.휴게{color:#4a5261;}
  td.slot.off{
    background:repeating-linear-gradient(45deg,#f2f3f7,#f2f3f7 5px,#e8eaf1 5px,#e8eaf1 10px);
    cursor:not-allowed;
  }
  td.slot.off:hover{outline:none;}
  td.del-cell{border:none; background:transparent; padding-left:6px;}
  .del-btn{
    border:none; background:transparent; color:var(--muted); cursor:pointer;
    font-size:15px; padding:4px;
  }
  .del-btn:hover{color:#d64545;}
  tfoot td{background:#f4f6fa; font-weight:700; font-size:10.5px; padding:5px 1px; color:var(--muted);}
  tfoot td.name-cell{position:sticky; left:0; background:#f4f6fa;}

  .toolbar{display:flex; gap:8px; margin:12px 0 18px;}

  /* 전달사항 */
  .memo{
    background:var(--card); border:1px solid var(--line); border-radius:12px;
    padding:16px; margin-bottom:14px;
  }
  .memo h2{font-size:15px; margin-bottom:10px;}
  .memo textarea{
    width:100%; min-height:88px; resize:vertical;
    border:1px solid var(--line); border-radius:8px; padding:10px 12px;
    font-family:inherit; font-size:13.5px; line-height:1.6; color:var(--ink);
    background:#fbfcfe;
  }
  .memo textarea:focus{outline:2px solid var(--accent); outline-offset:-1px;}

  /* 집계 */
  .summary{
    background:var(--card); border:1px solid var(--line); border-radius:12px;
    padding:16px;
  }
  .summary h2{font-size:15px; margin-bottom:10px;}
  .summary table{min-width:0;}
  .summary th, .summary td{padding:6px 10px; font-size:12.5px;}
  .summary thead th{position:static;}
  .badge{
    display:inline-block; width:10px; height:10px; border-radius:3px; margin-right:5px;
    vertical-align:-1px;
  }

  /* 동기화 상태 배지 */
  .sync-badge{
    display:flex; align-items:center; gap:6px; border:1px solid var(--line); background:var(--card);
    border-radius:8px; padding:8px 12px; font-size:12.5px; font-weight:700; cursor:pointer; color:var(--muted);
    white-space:nowrap;
  }
  .sync-dot{width:8px; height:8px; border-radius:50%; background:#c7cbd6; flex:none;}
  .sync-badge.on .sync-dot{background:#2fb267;}
  .sync-badge.off .sync-dot{background:#c7cbd6;}
  .sync-badge.err .sync-dot{background:#d64545;}
  .sync-badge.on{color:#1f9e57;}
  .sync-badge.err{color:#d64545;}

  .modal-back{
    position:fixed; inset:0; background:rgba(20,24,36,.45); display:flex;
    align-items:center; justify-content:center; z-index:200; padding:16px;
  }
  .modal-back.hide{display:none;}
  .modal{
    background:#fff; border-radius:14px; padding:20px; width:100%; max-width:420px;
    max-height:88vh; overflow:auto;
  }
  .modal h3{font-size:16px; margin-bottom:6px;}
  .modal p{font-size:12.5px; color:var(--muted); line-height:1.6; margin-bottom:12px;}
  .modal label{font-size:12px; font-weight:700; color:var(--muted); display:block; margin:10px 0 4px;}
  .modal input[type=text]{
    width:100%; border:1px solid var(--line); border-radius:8px; padding:9px 10px; font-size:13.5px;
    font-family:inherit;
  }
  .modal .modal-btns{display:flex; gap:8px; margin-top:16px;}
  .modal .modal-btns .btn{flex:1;}

  /* 선택·복사 모드 모바일용 액션 바 */
  .sel-actions{
    display:none; gap:8px; margin:0 0 14px; flex-wrap:wrap;
  }
  .sel-actions.show{display:flex;}
  .sel-actions .btn{flex:1; min-width:88px;}

  /* 모바일 대응 */
  @media (max-width: 820px){
    body{padding:12px;}
    header{gap:10px;}
    h1{font-size:18px;}
    .meta input[type=text]{width:130px;}
    .btn{padding:10px 12px; font-size:13px;}
    .sales-grid{grid-template-columns:repeat(2, 1fr);}
    .palette{padding:10px;}
    .hint{width:100%; margin-left:0; order:99;}
    td.slot{height:46px; min-width:40px;}
    .modes .mode-btn{padding:9px 12px;}
    table{min-width:1150px;}
  }
  @media (max-width: 480px){
    .sales-grid{grid-template-columns:repeat(2, 1fr);}
    .meta{width:100%;}
    .meta input[type=text]{flex:1; width:auto;}
    .toolbar{flex-wrap:wrap;}
    .toolbar .btn{flex:1; min-width:120px;}
  }

  @media print{
    body{padding:0; background:#fff;}
    .palette,.toolbar,.btn,.del-cell,.del-btn,.hint{display:none !important;}
    .board,.memo,.summary,.sales{border:1px solid #bbb; overflow:visible; border-radius:0; margin-bottom:8px;}
    td.slot:hover{outline:none;}
    thead th{position:static;}
    td.name-cell, th.name-col, tfoot td.name-cell{position:static;}
    .memo textarea{border:none; background:#fff; padding:0;}
    @page{size:landscape; margin:8mm;}
  }
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>매장 일별 <span>스케줄러</span></h1>
    <div class="meta">
      <input type="text" id="storeName" placeholder="매장명 (예: 대구동성로점)">
      <input type="date" id="workDate">
    </div>
    <div class="spacer"></div>
    <div class="sync-badge off" id="syncBadge" onclick="openSyncModal()">
      <span class="sync-dot"></span><span id="syncBadgeText">동기화 설정</span>
    </div>
    <button class="btn" onclick="saveFile()">💾 저장하기</button>
    <button class="btn" onclick="document.getElementById('loadInput').click()">📂 불러오기</button>
    <input type="file" id="loadInput" accept=".json" style="display:none" onchange="loadFile(this)">
    <button class="btn" onclick="downloadXLSX()">엑셀 저장</button>
    <button class="btn primary" onclick="window.print()">인쇄</button>
  </header>

  <!-- 매출 관리 -->
  <div class="sales">
    <h2>매출 관리</h2>
    <div class="sales-grid">
      <div class="field">
        <label for="todayTarget">금일 목표 매출</label>
        <input type="text" id="todayTarget" inputmode="numeric" placeholder="0">
        <span class="unit">원</span>
      </div>
      <div class="field">
        <label for="targetSales">목표 매출 (월)</label>
        <input type="text" id="targetSales" inputmode="numeric" placeholder="0">
        <span class="unit">원</span>
      </div>
      <div class="field">
        <label for="prevSales">전일 매출</label>
        <input type="text" id="prevSales" inputmode="numeric" placeholder="0">
        <span class="unit">원</span>
      </div>
      <div class="field">
        <label for="prevCount">전일 결제건수</label>
        <input type="text" id="prevCount" inputmode="numeric" placeholder="0">
        <span class="unit">건</span>
      </div>
      <div class="field">
        <label for="prevQty">전일 판매수량</label>
        <input type="text" id="prevQty" inputmode="numeric" placeholder="0">
        <span class="unit">개</span>
      </div>
      <div class="field calc">
        <label for="prevAtv">전일 객단가 (자동)</label>
        <input type="text" id="prevAtv" readonly value="-">
        <span class="unit">원/건 = 전일매출 ÷ 결제건수</span>
      </div>
      <div class="field calc">
        <label for="prevUpt">전일 객수량 (자동)</label>
        <input type="text" id="prevUpt" readonly value="-">
        <span class="unit">개/건 = 판매수량 ÷ 결제건수</span>
      </div>
      <div class="field">
        <label for="cumSales">누적 매출</label>
        <input type="text" id="cumSales" inputmode="numeric" placeholder="0">
        <span class="unit">원</span>
      </div>
      <div class="field calc">
        <label for="progressRate">매출 진척률 (자동)</label>
        <input type="text" id="progressRate" readonly value="-">
        <span class="unit">% = 누적매출 ÷ 목표매출</span>
      </div>
      <div class="field calc">
        <label for="perLabor">인당효율 (자동)</label>
        <input type="text" id="perLabor" readonly value="-">
        <span class="unit">원/h = 금일 목표매출 ÷ 총 근로시간</span>
      </div>
      <div class="progress-wrap">
        <div class="progress-bar">
          <div class="progress-fill" id="progressFill" style="width:0%"></div>
          <div class="progress-label" id="progressLabel">목표 매출과 누적 매출을 입력하면 진척률이 표시됩니다</div>
        </div>
      </div>
    </div>
  </div>

  <div class="palette" id="palette">
    <div class="modes">
      <button class="mode-btn active" id="modePaint" onclick="setMode('paint')">✏️ 칠하기</button>
      <button class="mode-btn" id="modeSelect" onclick="setMode('select')">⬚ 선택·복사</button>
    </div>
    <span class="label">업무 선택 →</span>
    <span class="hint">칠하기: 클릭·드래그, 같은 업무 다시 누르면 지워짐 · 선택·복사: 드래그로 범위 선택 후 Ctrl+C / Ctrl+V, Delete로 지우기</span>
  </div>
  <div class="toast" id="toast"></div>

  <div class="sel-actions" id="selActions">
    <button class="btn" onclick="copySel()">📋 복사</button>
    <button class="btn" onclick="pasteAt()">📥 붙여넣기</button>
    <button class="btn" onclick="deleteSel()">🗑 선택 지우기</button>
    <button class="btn" onclick="sel=null; renderTable();">✕ 선택 해제</button>
  </div>

  <div class="board">
    <table id="scheduleTable">
      <thead>
        <tr id="headRow1"></tr>
        <tr id="headRow2"></tr>
      </thead>
      <tbody id="bodyRows"></tbody>
      <tfoot><tr id="footRow"></tr></tfoot>
    </table>
  </div>

  <div class="toolbar">
    <button class="btn" onclick="addEmployee()">＋ 직원 추가</button>
    <button class="btn" onclick="clearAll()">전체 지우기</button>
  </div>

  <div class="memo">
    <h2>전달사항 / 특이사항</h2>
    <textarea id="memoBox" placeholder="예) 14시 본사 물류 입고 예정 / 신입 김OO OJT 교육 담당: 박매니저 / 마감 시 창고 시건 확인"></textarea>
  </div>

  <div class="summary">
    <h2>개인별 업무 시간 집계</h2>
    <div style="overflow:auto"><table id="summaryTable"></table></div>
  </div>
</div>

<datalist id="rankList">
  <option value="점장"></option>
  <option value="매니저"></option>
  <option value="부매니저"></option>
  <option value="주임"></option>
  <option value="리더"></option>
  <option value="스태프"></option>
  <option value="수습"></option>
  <option value="파트타임"></option>
  <option value="단기스태프"></option>
</datalist>

<div class="modal-back hide" id="syncModal">
  <div class="modal">
    <h3>☁️ 실시간 동기화 설정</h3>
    <p>
      Firebase Realtime Database 정보를 입력하면, <b>매장명 + 날짜</b>가 같은 스케줄을
      여러 기기(매장 PC·본사 등)에서 실시간으로 함께 보고 저장할 수 있어요.<br>
      본사에서 한 번 설정해서 공유해주면, 각 매장은 매장명만 입력하고 사용하면 됩니다.
      설정하지 않아도 기존처럼 이 기기에서만 저장/불러오기 하며 사용할 수 있어요.
    </p>
    <label>apiKey</label>
    <input type="text" id="fbApiKey" placeholder="AIza...">
    <label>authDomain</label>
    <input type="text" id="fbAuthDomain" placeholder="프로젝트ID.firebaseapp.com">
    <label>databaseURL</label>
    <input type="text" id="fbDatabaseURL" placeholder="https://프로젝트ID-default-rtdb.asia-southeast1.firebasedatabase.app">
    <label>projectId</label>
    <input type="text" id="fbProjectId" placeholder="프로젝트ID">
    <div class="modal-btns">
      <button class="btn" onclick="closeSyncModal()">닫기</button>
      <button class="btn" onclick="clearSyncConfig()">연결 해제</button>
      <button class="btn primary" onclick="saveSyncConfig()">저장 및 연결</button>
    </div>
  </div>
</div>

<script>
const TASKS = ["청소","입하","보충","라인변경","매장변경","후방업무","창고정리","교육","RT","반품","재고실사","POS","매장체크","재고주문","FR","브리핑","인수인계","휴게"];
const COLORS = {
  "청소":"#5aa9e6","입하":"#f4a259","보충":"#7bc47f","라인변경":"#b78bd9",
  "매장변경":"#e57fa4","후방업무":"#8d99ae","창고정리":"#c9a227","교육":"#4cc3c8",
  "RT":"#e15b64","반품":"#8c5e58","재고실사":"#2f7d5a","POS":"#3d5a80",
  "매장체크":"#7048b5","재고주문":"#b5179e","FR":"#8f9e2f",
  "브리핑":"#f25c05","인수인계":"#457b9d","휴게":"#d9dee7"
};
const FLOORS = ["B1층","1층","2층","3층","4층"];
const FLOOR_COLORS = {
  "B1층":"#6a4c93", // 보라
  "1층":"#d1495b",  // 레드
  "2층":"#1c7ed6",  // 블루
  "3층":"#2b9348",  // 그린
  "4층":"#e8871e"   // 오렌지
};
const START = 9, END = 24;              // 9:00 ~ 24:00
const SLOTS = [];                       // 30분 단위 슬롯 (총 30개)
for(let t=START*2; t<END*2; t++) SLOTS.push(t); // t = half-hour index (18 = 9:00)

function slotLabel(t){ // 18 -> "9:00"
  const h = Math.floor(t/2), m = t%2 ? '30':'00';
  return `${h}:${m}`;
}
function timeLabel(t){ // 출퇴근 표기 18 -> "09:00", 48 -> "24:00"
  const h = Math.floor(t/2), m = t%2 ? '30':'00';
  return `${String(h).padStart(2,'0')}:${m}`;
}

function newEmp(name, rank, inT, outT, breakT){
  // breakT: 휴게시간(30분 단위 슬롯 수) 0=없음, 1=0:30, 2=1:00
  return {name, rank: rank||'', inT: inT??START*2, outT: outT??END*2, breakT: breakT??2, slots:Array(SLOTS.length).fill(null)};
}
let employees = [ newEmp("직원 1","",18,36), newEmp("직원 2","",24,42), newEmp("직원 3","",30,48) ];
let currentTask = "청소";
let currentFloor = null; // null = 층 표시 없음
let isPainting = false;
let mode = 'paint';      // 'paint' | 'select'
let sel = null;          // {a:{ei,si}, b:{ei,si}} 선택 범위
let selecting = false;
let clipboard = null;    // 복사된 2차원 슬롯 배열

document.getElementById('workDate').valueAsDate = new Date();

/* ================= 실시간 동기화 (Firebase) ================= */
const SYNC_CFG_KEY = 'schedulerFirebaseConfig';
let fbApp = null, fbRef = null, fbListenerOn = false;
let syncing = false;          // 원격 데이터 반영 중 (중복 저장 방지)
let lastPushedJSON = '';      // 직전에 내가 올린 데이터 (에코 무시용)
let syncDebounce = null;

function loadSyncConfig(){
  try{ return JSON.parse(localStorage.getItem(SYNC_CFG_KEY) || 'null'); }catch(e){ return null; }
}
function setSyncBadge(state, text){
  const badge = document.getElementById('syncBadge');
  badge.className = 'sync-badge ' + state;
  document.getElementById('syncBadgeText').textContent = text;
}
function openSyncModal(){
  const cfg = loadSyncConfig();
  if(cfg){
    document.getElementById('fbApiKey').value = cfg.apiKey || '';
    document.getElementById('fbAuthDomain').value = cfg.authDomain || '';
    document.getElementById('fbDatabaseURL').value = cfg.databaseURL || '';
    document.getElementById('fbProjectId').value = cfg.projectId || '';
  }
  document.getElementById('syncModal').classList.remove('hide');
}
function closeSyncModal(){ document.getElementById('syncModal').classList.add('hide'); }
function saveSyncConfig(){
  const cfg = {
    apiKey: document.getElementById('fbApiKey').value.trim(),
    authDomain: document.getElementById('fbAuthDomain').value.trim(),
    databaseURL: document.getElementById('fbDatabaseURL').value.trim(),
    projectId: document.getElementById('fbProjectId').value.trim()
  };
  if(!cfg.apiKey || !cfg.databaseURL){
    alert('apiKey와 databaseURL은 꼭 입력해 주세요.');
    return;
  }
  localStorage.setItem(SYNC_CFG_KEY, JSON.stringify(cfg));
  closeSyncModal();
  initSync();
}
function clearSyncConfig(){
  if(!confirm('실시간 동기화 연결을 해제할까요? (이 기기의 로컬 저장은 그대로 유지됩니다)')) return;
  localStorage.removeItem(SYNC_CFG_KEY);
  fbListenerOn = false;
  fbRef = null; fbApp = null;
  setSyncBadge('off','동기화 설정');
  closeSyncModal();
}
function syncKey(){
  const store = (document.getElementById('storeName').value || '매장').trim();
  const date = document.getElementById('workDate').value || 'nodate';
  // Firebase 경로에 쓸 수 없는 문자(.#$[]/) 제거
  const safeStore = store.replace(/[.#$\[\]\/]/g,'_') || '매장';
  return `schedules/${safeStore}_${date}`;
}
function initSync(){
  const cfg = loadSyncConfig();
  if(!cfg || !cfg.apiKey || !cfg.databaseURL){ setSyncBadge('off','동기화 설정'); return; }
  try{
    if(!fbApp){ fbApp = firebase.initializeApp(cfg); }
    subscribeSync();
  }catch(e){
    console.error(e);
    setSyncBadge('err','연결 오류');
  }
}
function subscribeSync(){
  if(!fbApp) return;
  if(fbRef && fbListenerOn){ fbRef.off(); fbListenerOn = false; }
  setSyncBadge('off','연결 중…');
  const db = firebase.database();
  fbRef = db.ref(syncKey());
  fbListenerOn = true;
  fbRef.on('value', snap=>{
    setSyncBadge('on', `실시간 동기화 중 (${(document.getElementById('storeName').value||'매장')})`);
    const val = snap.val();
    if(!val) return; // 아직 저장된 데이터 없음
    const json = JSON.stringify(val);
    if(json === lastPushedJSON) return; // 내가 방금 올린 것과 같으면 무시(에코)
    syncing = true;
    try{ restore(val); }catch(e){ console.error('원격 데이터 복원 실패', e); }
    syncing = false;
  }, err=>{
    console.error(err);
    setSyncBadge('err','연결 오류 (설정 확인)');
  });
}
function pushSync(){
  if(!fbRef || syncing) return;
  clearTimeout(syncDebounce);
  syncDebounce = setTimeout(()=>{
    const data = serialize();
    const json = JSON.stringify(data);
    lastPushedJSON = json;
    fbRef.set(data).catch(e=> console.error('동기화 저장 실패', e));
  }, 500);
}
// 매장명·날짜가 바뀌면 새 경로로 재구독
['storeName','workDate'].forEach(id=>{
  document.getElementById(id).addEventListener('change', ()=>{ if(fbApp) subscribeSync(); });
});
initSync();

/* ---------- 팔레트 ---------- */
function renderPalette(){
  const pal = document.getElementById('palette');
  pal.querySelectorAll('.chip, .pal-break, .label-floor').forEach(c=>c.remove());
  const hint = pal.querySelector('.hint');
  TASKS.forEach(t=>{
    const chip = document.createElement('div');
    chip.className = 'chip' + (t==="휴게"?" 휴게":"") + (currentTask===t?" selected":"");
    chip.style.background = COLORS[t];
    chip.innerHTML = '<span class="dot"></span>'+t;
    chip.onclick = ()=>{ currentTask = t; renderPalette(); };
    pal.insertBefore(chip, hint);
  });

  // 층 선택 (업무와 조합)
  const br = document.createElement('div');
  br.className = 'pal-break';
  pal.insertBefore(br, hint);
  const fl = document.createElement('span');
  fl.className = 'label label-floor';
  fl.textContent = '층 선택 →';
  pal.insertBefore(fl, hint);
  const noneChip = document.createElement('div');
  noneChip.className = 'chip floor' + (currentFloor===null?" selected":"");
  noneChip.textContent = '층 없음';
  noneChip.onclick = ()=>{ currentFloor = null; renderPalette(); };
  pal.insertBefore(noneChip, hint);
  FLOORS.forEach(f=>{
    const chip = document.createElement('div');
    chip.className = 'chip' + (currentFloor===f?" selected":"");
    chip.style.background = FLOOR_COLORS[f];
    chip.innerHTML = '<span class="dot"></span>'+f;
    chip.onclick = ()=>{ currentFloor = f; renderPalette(); };
    pal.insertBefore(chip, hint);
  });
}

/* ---------- 시간 select (30분 단위) ---------- */
function timeSelect(value, onchange, min, max){
  const sel = document.createElement('select');
  for(let t=min; t<=max; t++){
    const opt = document.createElement('option');
    opt.value = t;
    opt.textContent = timeLabel(t);
    if(t===value) opt.selected = true;
    sel.appendChild(opt);
  }
  sel.onchange = e=> onchange(parseInt(e.target.value));
  return sel;
}

function inRange(emp, si){
  const t = SLOTS[si];
  return t >= emp.inT && t < emp.outT;
}

/* ---------- 표 ---------- */
function renderTable(){
  // 헤더 1행: 시간(시 단위, colspan 2) / 2행: 00·30
  const h1 = document.getElementById('headRow1');
  const h2 = document.getElementById('headRow2');
  let r1 = '<th class="name-col" rowspan="2">이름</th><th class="rank-col" rowspan="2">직급</th>' +
           '<th class="time-col" rowspan="2">출근</th><th class="time-col" rowspan="2">퇴근</th>' +
           '<th class="time-col" rowspan="2">휴게</th>';
  let r2 = '';
  for(let h=START; h<END; h++){
    r1 += `<th colspan="2">${h}시</th>`;
    r2 += `<th>00</th><th>30</th>`;
  }
  r1 += '<th style="border:none;background:transparent" rowspan="2"></th>';
  h1.innerHTML = r1;
  h2.innerHTML = r2;

  const body = document.getElementById('bodyRows');
  body.innerHTML = '';
  employees.forEach((emp, ei)=>{
    const tr = document.createElement('tr');

    // 이름
    const nameTd = document.createElement('td');
    nameTd.className = 'name-cell';
    const nameInp = document.createElement('input');
    nameInp.value = emp.name; nameInp.placeholder = '이름';
    nameInp.onchange = e=>{ emp.name = e.target.value; renderSummary(); };
    nameTd.appendChild(nameInp);
    tr.appendChild(nameTd);

    // 직급
    const rankTd = document.createElement('td');
    rankTd.className = 'rank-cell';
    const rankInp = document.createElement('input');
    rankInp.value = emp.rank; rankInp.placeholder = '직급';
    rankInp.setAttribute('list','rankList');
    rankInp.onchange = e=>{ emp.rank = e.target.value; renderSummary(); };
    rankTd.appendChild(rankInp);
    tr.appendChild(rankTd);

    // 출근 (30분 단위)
    const inTd = document.createElement('td');
    inTd.className = 'time-cell';
    inTd.appendChild(timeSelect(emp.inT, v=>{
      emp.inT = v;
      if(emp.outT <= v) emp.outT = Math.min(v+1, END*2);
      trimSlots(emp); renderTable(); renderSummary();
    }, START*2, END*2-1));
    tr.appendChild(inTd);

    // 퇴근 (30분 단위)
    const outTd = document.createElement('td');
    outTd.className = 'time-cell';
    outTd.appendChild(timeSelect(emp.outT, v=>{
      emp.outT = v;
      if(emp.inT >= v) emp.inT = Math.max(v-1, START*2);
      trimSlots(emp); renderTable(); renderSummary();
    }, START*2+1, END*2));
    tr.appendChild(outTd);

    // 휴게시간 선택 (없음 / 0:30 / 1:00)
    const brTd = document.createElement('td');
    brTd.className = 'time-cell';
    const brSel = document.createElement('select');
    [[0,'없음'],[1,'0:30'],[2,'1:00']].forEach(([v,label])=>{
      const opt = document.createElement('option');
      opt.value = v; opt.textContent = label;
      if(emp.breakT===v) opt.selected = true;
      brSel.appendChild(opt);
    });
    brSel.onchange = e=>{ emp.breakT = parseInt(e.target.value); renderSummary(); };
    brTd.appendChild(brSel);
    tr.appendChild(brTd);

    // 30분 슬롯
    SLOTS.forEach((t, si)=>{
      const td = document.createElement('td');
      const active = inRange(emp, si);
      if(!active){
        td.className = 'slot off';
      } else {
        const slot = emp.slots[si]; // null 또는 {task, floor}
        const task = slot ? slot.task : null;
        td.className = 'slot' + (task==="휴게"?" 휴게":"");
        // 층이 지정되면 층 색상, 아니면 업무 색상
        td.style.background = task ? (slot.floor ? FLOOR_COLORS[slot.floor] : COLORS[task]) : '#fff';
        if(slot){
          td.innerHTML = slot.floor
            ? `<span style="display:block;font-size:9px;font-weight:800;letter-spacing:.3px">${slot.floor}</span>${task}`
            : task;
        } else {
          td.textContent = '';
        }
        const rect = selRect();
        if(rect && ei>=rect.e1 && ei<=rect.e2 && si>=rect.s1 && si<=rect.s2){
          td.classList.add('sel');
        }
        td.onmousedown = e=>{
          e.preventDefault();
          if(mode==='paint'){ isPainting = true; startStroke(ei, si); }
          else { selecting = true; sel = {a:{ei,si}, b:{ei,si}}; renderTable(); }
        };
        td.onmouseover = ()=>{
          if(mode==='paint' && isPainting) applyStroke(ei, si);
          else if(mode==='select' && selecting){ sel.b = {ei,si}; renderTable(); }
        };
        td.dataset.ei = ei; td.dataset.si = si;
        td.ontouchstart = e=>{
          e.preventDefault();
          if(mode==='paint'){ isPainting = true; startStroke(ei, si); }
          else { selecting = true; sel = {a:{ei,si}, b:{ei,si}}; renderTable(); }
        };
        td.ontouchmove = e=>{
          e.preventDefault();
          const touch = e.touches[0];
          const el = document.elementFromPoint(touch.clientX, touch.clientY);
          const cell = el && el.closest('td.slot:not(.off)');
          if(!cell || cell.dataset.ei===undefined) return;
          const tei = parseInt(cell.dataset.ei), tsi = parseInt(cell.dataset.si);
          if(mode==='paint' && isPainting) applyStroke(tei, tsi);
          else if(mode==='select' && selecting){ sel.b = {ei:tei, si:tsi}; renderTable(); }
        };
        td.ontouchend = ()=>{ isPainting = false; selecting = false; };
      }
      tr.appendChild(td);
    });

    // 삭제
    const delTd = document.createElement('td');
    delTd.className = 'del-cell';
    const delBtn = document.createElement('button');
    delBtn.className = 'del-btn';
    delBtn.textContent = '✕';
    delBtn.title = '직원 삭제';
    delBtn.onclick = ()=>{ if(confirm(`'${emp.name}' 행을 삭제할까요?`)){ employees.splice(ei,1); renderTable(); renderSummary(); } };
    delTd.appendChild(delBtn);
    tr.appendChild(delTd);

    body.appendChild(tr);
  });

  renderFoot();
}

function trimSlots(emp){
  SLOTS.forEach((t,si)=>{ if(!inRange(emp,si)) emp.slots[si] = null; });
}

function renderFoot(){
  const foot = document.getElementById('footRow');
  let cells = '<td class="name-cell">시간대 인원</td><td colspan="4" style="font-weight:500">휴게 제외 실근무</td>';
  SLOTS.forEach((t,si)=>{
    const n = employees.filter(e=> inRange(e,si) && e.slots[si] && e.slots[si].task!=="휴게").length;
    cells += `<td>${n>0? n : '·'}</td>`;
  });
  cells += '<td style="border:none;background:transparent"></td>';
  foot.innerHTML = cells;
}

let strokeMode = 'paint'; // 드래그 시작 시 결정: paint 또는 erase

function currentSlotValue(){
  return {task: currentTask, floor: currentTask==="휴게" ? null : currentFloor};
}
function sameAsCurrent(slot){
  if(!slot) return false;
  const cur = currentSlotValue();
  return slot.task===cur.task && slot.floor===cur.floor;
}
function startStroke(ei, si){
  if(!inRange(employees[ei], si)) return;
  // 이미 같은 업무(+층)가 칠해진 칸을 다시 누르면 지우기 모드
  strokeMode = sameAsCurrent(employees[ei].slots[si]) ? 'erase' : 'paint';
  applyStroke(ei, si);
}
function applyStroke(ei, si){
  if(!inRange(employees[ei], si)) return;
  employees[ei].slots[si] = (strokeMode==='erase') ? null : currentSlotValue();
  renderTable();
  renderSummary();
}
document.addEventListener('mouseup', ()=>{ isPainting = false; selecting = false; });

/* ---------- 모드 전환 ---------- */
function setMode(m){
  mode = m;
  sel = null;
  document.getElementById('modePaint').classList.toggle('active', m==='paint');
  document.getElementById('modeSelect').classList.toggle('active', m==='select');
  document.getElementById('selActions').classList.toggle('show', m==='select');
  renderTable();
}

/* ---------- 선택·복사·붙여넣기 ---------- */
function selRect(){
  if(!sel) return null;
  return {
    e1: Math.min(sel.a.ei, sel.b.ei), e2: Math.max(sel.a.ei, sel.b.ei),
    s1: Math.min(sel.a.si, sel.b.si), s2: Math.max(sel.a.si, sel.b.si)
  };
}
function showToast(msg){
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  clearTimeout(t._timer);
  t._timer = setTimeout(()=> t.classList.remove('show'), 1600);
}
function copySel(){
  const r = selRect();
  if(!r) return;
  clipboard = [];
  for(let ei=r.e1; ei<=r.e2; ei++){
    const row = [];
    for(let si=r.s1; si<=r.s2; si++){
      const s = employees[ei]?.slots[si];
      row.push(s ? {task:s.task, floor:s.floor} : null);
    }
    clipboard.push(row);
  }
  showToast(`복사됨 (직원 ${clipboard.length}명 × ${clipboard[0].length}칸) — 붙여넣을 위치를 클릭 후 Ctrl+V`);
}
function pasteAt(){
  const r = selRect();
  if(!r || !clipboard) return;
  let pasted = 0;
  clipboard.forEach((row, di)=>{
    row.forEach((val, dj)=>{
      const ei = r.e1 + di, si = r.s1 + dj;
      if(ei >= employees.length || si >= SLOTS.length) return;
      if(!inRange(employees[ei], si)) return; // 출퇴근 시간 밖은 건너뜀
      employees[ei].slots[si] = val ? {task:val.task, floor:val.floor} : null;
      pasted++;
    });
  });
  renderTable(); renderSummary();
  showToast(`붙여넣기 완료 (${pasted}칸)`);
}
function deleteSel(){
  const r = selRect();
  if(!r) return;
  for(let ei=r.e1; ei<=r.e2; ei++){
    for(let si=r.s1; si<=r.s2; si++){
      if(employees[ei]) employees[ei].slots[si] = null;
    }
  }
  renderTable(); renderSummary();
  showToast('선택 범위 지움');
}

document.addEventListener('keydown', e=>{
  // 입력창에 타이핑 중일 땐 무시
  const tag = document.activeElement?.tagName;
  if(tag==='INPUT' || tag==='TEXTAREA' || tag==='SELECT') return;
  const k = e.key.toLowerCase();
  if((e.ctrlKey || e.metaKey) && k==='c' && sel){ e.preventDefault(); copySel(); }
  else if((e.ctrlKey || e.metaKey) && k==='v' && sel && clipboard){ e.preventDefault(); pasteAt(); }
  else if((k==='delete' || k==='backspace') && sel && mode==='select'){ e.preventDefault(); deleteSel(); }
  else if(k==='escape'){ sel = null; renderTable(); }
});

/* ---------- 집계 ---------- */
function fmtH(halfCnt){ // 슬롯 수 -> 시간 표기 (3 -> 1.5h)
  if(halfCnt===0) return '-';
  return (halfCnt/2) + 'h';
}
function laborHours(emp){ // 총 근로시간 = (퇴근-출근) - 휴게시간
  return Math.max(0, (emp.outT - emp.inT - emp.breakT)) / 2;
}
function totalLaborHours(){
  return employees.reduce((sum,e)=> sum + laborHours(e), 0);
}
function breakLabel(bt){ return bt===0? '없음' : (bt===1? '0:30' : '1:00'); }

function renderSummary(){
  const tbl = document.getElementById('summaryTable');
  let html = '<thead><tr><th style="background:#eef1f7">이름</th><th style="background:#eef1f7">직급</th><th style="background:#eef1f7">출퇴근</th><th style="background:#eef1f7">휴게</th>';
  TASKS.forEach(t=>{
    html += `<th style="background:#eef1f7"><span class="badge" style="background:${COLORS[t]}"></span>${t}</th>`;
  });
  html += '<th style="background:#eef1f7">배정합계</th><th style="background:#eef1f7">총 근로시간</th></tr></thead><tbody>';
  employees.forEach(emp=>{
    html += `<tr><td style="font-weight:700">${emp.name}</td><td>${emp.rank||'-'}</td>`;
    html += `<td>${timeLabel(emp.inT)}~${timeLabel(emp.outT)}</td>`;
    html += `<td>${breakLabel(emp.breakT)}</td>`;
    let total = 0;
    TASKS.forEach(t=>{
      const cnt = emp.slots.filter(s=>s && s.task===t).length;
      if(t!=="휴게") total += cnt;
      html += `<td>${fmtH(cnt)}</td>`;
    });
    html += `<td style="font-weight:700">${fmtH(total)}</td><td style="font-weight:700">${laborHours(emp)}h</td></tr>`;
  });
  // 합계 행
  const laborSum = totalLaborHours();
  html += `<tr><td colspan="${4+TASKS.length+1}" style="text-align:right; font-weight:700; background:#f4f6fa">총 근로시간 합계</td>`;
  html += `<td style="font-weight:800; background:#eef4ff; color:var(--accent)">${laborSum}h</td></tr>`;
  html += '</tbody>';
  tbl.innerHTML = html;
  calcSales(); // 인당효율 갱신
  if(typeof autoSave === 'function') autoSave();
}

/* ---------- 매출 계산 ---------- */
function parseNum(id){
  const raw = document.getElementById(id).value.replace(/[^0-9.]/g,'');
  return raw ? parseFloat(raw) : 0;
}
function fmtComma(n){ return n.toLocaleString('ko-KR'); }

function bindSalesInput(id){
  const el = document.getElementById(id);
  el.addEventListener('input', ()=>{
    // 쉼표 자동 포맷
    const raw = el.value.replace(/[^0-9]/g,'');
    el.value = raw ? fmtComma(parseInt(raw)) : '';
    calcSales();
    if(typeof autoSave === 'function') autoSave();
  });
}
['todayTarget','targetSales','prevSales','prevCount','prevQty','cumSales'].forEach(bindSalesInput);

function calcSales(){
  const todayTarget = parseNum('todayTarget');
  const target = parseNum('targetSales');
  const prevSales = parseNum('prevSales');
  const prevCount = parseNum('prevCount');
  const prevQty = parseNum('prevQty');
  const cum = parseNum('cumSales');

  // 전일 객단가 = 전일매출 ÷ 결제건수
  const atvEl = document.getElementById('prevAtv');
  atvEl.value = (prevSales>0 && prevCount>0) ? fmtComma(Math.round(prevSales/prevCount)) : '-';

  // 전일 객수량 = 판매수량 ÷ 결제건수
  const uptEl = document.getElementById('prevUpt');
  uptEl.value = (prevQty>0 && prevCount>0) ? (prevQty/prevCount).toFixed(2) : '-';

  // 인당효율 = 금일 목표매출 ÷ 총 근로시간
  const laborSum = totalLaborHours();
  const perLaborEl = document.getElementById('perLabor');
  perLaborEl.value = (todayTarget>0 && laborSum>0) ? fmtComma(Math.round(todayTarget/laborSum)) : '-';

  // 진척률 = 누적매출 ÷ 목표매출
  const rateEl = document.getElementById('progressRate');
  const fill = document.getElementById('progressFill');
  const label = document.getElementById('progressLabel');
  if(target>0 && cum>0){
    const rate = cum/target*100;
    rateEl.value = rate.toFixed(1);
    fill.style.width = Math.min(rate,100) + '%';
    label.textContent = `진척률 ${rate.toFixed(1)}%  (누적 ${fmtComma(cum)} / 목표 ${fmtComma(target)})`;
    fill.style.background = rate>=100
      ? 'linear-gradient(90deg,#2fb267,#1f9e57)'
      : 'linear-gradient(90deg,#4c6ef5,#2f4bd8)';
  } else {
    rateEl.value = '-';
    fill.style.width = '0%';
    label.textContent = '목표 매출과 누적 매출을 입력하면 진척률이 표시됩니다';
  }
}

/* ---------- 조작 ---------- */
function addEmployee(){
  employees.push(newEmp(`직원 ${employees.length+1}`,'',START*2,36));
  renderTable(); renderSummary();
}
function clearAll(){
  if(!confirm('모든 배정을 지울까요? (직원 목록은 유지됩니다)')) return;
  employees.forEach(e=> e.slots = Array(SLOTS.length).fill(null));
  renderTable(); renderSummary();
}

/* ---------- 저장 / 불러오기 ---------- */
const SALES_IDS = ['todayTarget','targetSales','prevSales','prevCount','prevQty','cumSales'];

function serialize(){
  const sales = {};
  SALES_IDS.forEach(id=> sales[id] = document.getElementById(id).value);
  return {
    version: 1,
    store: document.getElementById('storeName').value,
    date: document.getElementById('workDate').value,
    memo: document.getElementById('memoBox').value,
    sales,
    employees
  };
}
function restore(data){
  if(!data || !Array.isArray(data.employees)) throw new Error('형식 오류');
  document.getElementById('storeName').value = data.store || '';
  if(data.date) document.getElementById('workDate').value = data.date;
  document.getElementById('memoBox').value = data.memo || '';
  SALES_IDS.forEach(id=>{
    document.getElementById(id).value = (data.sales && data.sales[id]) || '';
  });
  employees = data.employees.map(e=>({
    name: e.name || '', rank: e.rank || '',
    inT: e.inT ?? START*2, outT: e.outT ?? END*2, breakT: e.breakT ?? 2,
    slots: Array.from({length: SLOTS.length}, (_,i)=>{
      const s = e.slots && e.slots[i];
      return s ? {task:s.task, floor:s.floor ?? null} : null;
    })
  }));
  sel = null; clipboard = null;
  renderTable(); renderSummary();
}

function saveFile(){
  const data = serialize();
  const store = data.store || '매장';
  const date = data.date || '';
  const blob = new Blob([JSON.stringify(data, null, 2)], {type:'application/json'});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `${store}_스케줄_${date}.json`;
  a.click();
  URL.revokeObjectURL(a.href);
  showToast('파일로 저장했어요 — 불러오기로 다시 열 수 있어요');
}
function loadFile(input){
  const file = input.files && input.files[0];
  if(!file) return;
  const reader = new FileReader();
  reader.onload = ()=>{
    try{
      restore(JSON.parse(reader.result));
      showToast('불러오기 완료');
      autoSave();
    }catch(err){
      alert('파일을 읽을 수 없어요. 이 프로그램에서 저장한 JSON 파일인지 확인해 주세요.');
    }
  };
  reader.readAsText(file);
  input.value = ''; // 같은 파일 재선택 가능하게
}

/* 브라우저 자동 저장 (지원되는 환경에서만 동작) */
const AUTOSAVE_KEY = 'storeSchedulerAutosave';
let autoTimer = null;
function autoSave(){
  clearTimeout(autoTimer);
  autoTimer = setTimeout(()=>{
    try{ localStorage.setItem(AUTOSAVE_KEY, JSON.stringify(serialize())); }catch(e){}
  }, 400);
  pushSync();
}
function tryAutoRestore(){
  try{
    const raw = localStorage.getItem(AUTOSAVE_KEY);
    if(raw){
      restore(JSON.parse(raw));
      showToast('마지막 작업을 자동으로 불러왔어요');
      return true;
    }
  }catch(e){}
  return false;
}
// 입력 변경 시 자동 저장
['storeName','workDate','memoBox'].forEach(id=>{
  document.getElementById(id).addEventListener('input', autoSave);
});

/* ---------- 엑셀(XLSX) 저장: 색상 포함 ---------- */
function argb(hex){ return 'FF' + hex.replace('#','').toUpperCase(); }
const THIN = {top:{style:'thin'},left:{style:'thin'},bottom:{style:'thin'},right:{style:'thin'}};

async function downloadXLSX(){
  if(typeof ExcelJS === 'undefined'){
    alert('엑셀 저장 기능을 불러오지 못했어요. 인터넷 연결을 확인하고 페이지를 새로고침해 주세요.');
    return;
  }
  const store = document.getElementById('storeName').value || '매장';
  const date = document.getElementById('workDate').value || '';
  const wb = new ExcelJS.Workbook();

  /* --- 시트 1: 매출관리 --- */
  const ws1 = wb.addWorksheet('매출관리');
  ws1.columns = [{width:20},{width:18}];
  ws1.mergeCells('A1:B1');
  const t1 = ws1.getCell('A1');
  t1.value = `${store} 매출관리 (${date})`;
  t1.font = {bold:true, size:13};
  t1.alignment = {horizontal:'center'};
  const salesItems = [
    ['금일 목표 매출', document.getElementById('todayTarget').value],
    ['목표 매출 (월)', document.getElementById('targetSales').value],
    ['전일 매출', document.getElementById('prevSales').value],
    ['전일 결제건수', document.getElementById('prevCount').value],
    ['전일 판매수량', document.getElementById('prevQty').value],
    ['전일 객단가', document.getElementById('prevAtv').value],
    ['전일 객수량', document.getElementById('prevUpt').value],
    ['누적 매출', document.getElementById('cumSales').value],
    ['매출 진척률(%)', document.getElementById('progressRate').value],
    ['총 근로시간(h)', totalLaborHours()],
    ['인당효율(원/h)', document.getElementById('perLabor').value],
  ];
  salesItems.forEach((item, i)=>{
    const row = ws1.getRow(i+2);
    row.getCell(1).value = item[0];
    row.getCell(2).value = item[1] || '-';
    row.getCell(1).fill = {type:'pattern', pattern:'solid', fgColor:{argb:'FFEEF1F7'}};
    row.getCell(1).font = {bold:true, size:10};
    row.getCell(2).font = {size:10};
    row.getCell(2).alignment = {horizontal:'right'};
    row.getCell(1).border = THIN;
    row.getCell(2).border = THIN;
  });

  /* --- 시트 2: 데일리 스케줄 --- */
  const ws2 = wb.addWorksheet('데일리 스케줄');
  const FIXED = 5; // 이름, 직급, 출근, 퇴근, 휴게
  const totalCols = FIXED + SLOTS.length;
  ws2.getColumn(1).width = 11;
  ws2.getColumn(2).width = 9;
  [3,4,5].forEach(c=> ws2.getColumn(c).width = 7);
  for(let c=FIXED+1; c<=totalCols; c++) ws2.getColumn(c).width = 6.5;

  // 제목
  ws2.mergeCells(1, 1, 1, totalCols);
  const t2 = ws2.getCell(1,1);
  t2.value = `${store} 데일리 스케줄 (${date})`;
  t2.font = {bold:true, size:13};
  t2.alignment = {horizontal:'center'};

  // 헤더 (2행: 시간, 3행: 00/30)
  const headFill = {type:'pattern', pattern:'solid', fgColor:{argb:'FFEEF1F7'}};
  const fixedLabels = ['이름','직급','출근','퇴근','휴게'];
  fixedLabels.forEach((label, i)=>{
    ws2.mergeCells(2, i+1, 3, i+1);
    const cell = ws2.getCell(2, i+1);
    cell.value = label;
    cell.fill = headFill;
    cell.font = {bold:true, size:9.5};
    cell.alignment = {horizontal:'center', vertical:'middle'};
    cell.border = THIN;
    ws2.getCell(3, i+1).border = THIN;
  });
  for(let h=START; h<END; h++){
    const c1 = FIXED + 1 + (h-START)*2;
    ws2.mergeCells(2, c1, 2, c1+1);
    const hc = ws2.getCell(2, c1);
    hc.value = `${h}시`;
    hc.fill = headFill; hc.font = {bold:true, size:9}; 
    hc.alignment = {horizontal:'center'}; hc.border = THIN;
    ws2.getCell(2, c1+1).border = THIN;
    [['00',c1],['30',c1+1]].forEach(([m,c])=>{
      const sc = ws2.getCell(3, c);
      sc.value = m; sc.fill = headFill; sc.font = {size:8, color:{argb:'FF7B8496'}};
      sc.alignment = {horizontal:'center'}; sc.border = THIN;
    });
  }

  // 직원 행
  employees.forEach((emp, ei)=>{
    const r = 4 + ei;
    const row = ws2.getRow(r);
    row.height = 28;
    const fixedVals = [emp.name, emp.rank||'-', timeLabel(emp.inT), timeLabel(emp.outT), breakLabel(emp.breakT)];
    fixedVals.forEach((v, i)=>{
      const cell = row.getCell(i+1);
      cell.value = v;
      cell.font = {size:9.5, bold: i===0};
      cell.alignment = {horizontal:'center', vertical:'middle'};
      cell.border = THIN;
    });
    SLOTS.forEach((t, si)=>{
      const cell = row.getCell(FIXED + 1 + si);
      cell.border = THIN;
      cell.alignment = {horizontal:'center', vertical:'middle', wrapText:true};
      if(!inRange(emp, si)){
        cell.fill = {type:'pattern', pattern:'lightUp', fgColor:{argb:'FFB9BFCC'}, bgColor:{argb:'FFF2F3F7'}};
      } else {
        const s = emp.slots[si];
        if(s){
          const bg = s.floor ? FLOOR_COLORS[s.floor] : COLORS[s.task];
          cell.fill = {type:'pattern', pattern:'solid', fgColor:{argb:argb(bg)}};
          cell.value = s.floor ? `${s.floor}\n${s.task}` : s.task;
          cell.font = {size:7.5, bold:true, color:{argb: s.task==="휴게" ? 'FF4A5261' : 'FFFFFFFF'}};
        }
      }
    });
  });

  // 시간대 인원 행
  const fr = 4 + employees.length;
  const frow = ws2.getRow(fr);
  ws2.mergeCells(fr, 1, fr, FIXED);
  const fc = frow.getCell(1);
  fc.value = '시간대 인원 (휴게 제외)';
  fc.fill = headFill; fc.font = {bold:true, size:9};
  fc.alignment = {horizontal:'center'}; 
  for(let c=1; c<=FIXED; c++) frow.getCell(c).border = THIN;
  SLOTS.forEach((t, si)=>{
    const n = employees.filter(e=> inRange(e,si) && e.slots[si] && e.slots[si].task!=="휴게").length;
    const cell = frow.getCell(FIXED + 1 + si);
    cell.value = n>0 ? n : '';
    cell.fill = headFill; cell.font = {size:8.5, bold:true};
    cell.alignment = {horizontal:'center'}; cell.border = THIN;
  });

  // 다운로드
  const buf = await wb.xlsx.writeBuffer();
  const blob = new Blob([buf], {type:'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `${store}_스케줄_${date}.xlsx`;
  a.click();
  URL.revokeObjectURL(a.href);
  showToast('엑셀 파일로 저장했어요 (색상 포함)');
}

/* ---------- CSV ---------- */
function downloadCSV(){
  const store = document.getElementById('storeName').value || '매장';
  const date = document.getElementById('workDate').value || '';
  const memo = document.getElementById('memoBox').value || '';
  let rows = [];
  rows.push([`${store} 일별 스케줄`, date]);
  rows.push([]);
  rows.push(['[매출 관리]']);
  rows.push(['금일 목표 매출','목표 매출(월)','전일 매출','전일 결제건수','전일 판매수량','전일 객단가','전일 객수량','누적 매출','매출 진척률(%)','총 근로시간(h)','인당효율(원/h)']);
  rows.push([
    document.getElementById('todayTarget').value,
    document.getElementById('targetSales').value,
    document.getElementById('prevSales').value,
    document.getElementById('prevCount').value,
    document.getElementById('prevQty').value,
    document.getElementById('prevAtv').value,
    document.getElementById('prevUpt').value,
    document.getElementById('cumSales').value,
    document.getElementById('progressRate').value,
    totalLaborHours(),
    document.getElementById('perLabor').value
  ]);
  rows.push([]);
  rows.push(['[스케줄]']);
  rows.push(['이름','직급','출근','퇴근','휴게', ...SLOTS.map(t=>slotLabel(t)), '배정합계(h)','총 근로시간(h)']);
  employees.forEach(emp=>{
    const total = emp.slots.filter(s=>s && s.task!=="휴게").length/2;
    rows.push([
      emp.name, emp.rank, timeLabel(emp.inT), timeLabel(emp.outT), breakLabel(emp.breakT),
      ...emp.slots.map((s,si)=> inRange(emp,si) ? (s ? (s.floor? s.floor+' '+s.task : s.task) : '') : '×'),
      total, laborHours(emp)
    ]);
  });
  rows.push(['시간대 인원','','','','', ...SLOTS.map((t,si)=> employees.filter(e=>inRange(e,si)&&e.slots[si]&&e.slots[si].task!=="휴게").length), '', totalLaborHours()]);
  rows.push([]);
  rows.push(['전달사항/특이사항', memo]);
  const csv = '\uFEFF' + rows.map(r=>r.map(v=>`"${String(v).replace(/"/g,'""')}"`).join(',')).join('\n');
  const blob = new Blob([csv], {type:'text/csv;charset=utf-8'});
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob);
  a.download = `${store}_스케줄_${date}.csv`;
  a.click();
  URL.revokeObjectURL(a.href);
}

renderPalette();
if(!tryAutoRestore()){
  renderTable();
  renderSummary();
}
calcSales();
</script>
</body>
</html>

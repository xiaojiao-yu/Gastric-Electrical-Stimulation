# Gastric-Electrical-Stimulation
UI software for gastric electrical stimulation devices
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
<title>胃电刺激治疗系统 GES-100</title>
<link rel="manifest" href="./manifest.webmanifest">
<meta name="theme-color" content="#0e7490">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
<meta name="apple-mobile-web-app-title" content="胃电刺激">
<link rel="icon" href="./icons/icon-192.png">
<link rel="apple-touch-icon" href="./icons/icon-192.png">
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  :root {
    --primary: #0e7490; --primary-dark: #0c4a5e; --accent: #14b8a6;
    --danger: #dc2626; --warn: #d97706; --ok: #16a34a;
    --bg: #eef4f7; --card: #ffffff; --line: #dbe6ec;
    --text: #0f172a; --sub: #64748b;
    --sidebar: #0c2f3d; --sidebar2: #0e3a4c;
  }
  html, body { height: 100%; }
  body {
    font-family: "Segoe UI", "Microsoft YaHei", "PingFang SC", sans-serif;
    background: var(--bg); color: var(--text); font-size: 15px; overflow: hidden;
  }
  #app { display: grid; grid-template-rows: 58px 1fr 34px; grid-template-columns: 208px 1fr; height: 100vh; }

  /* ===== 顶栏 ===== */
  header {
    grid-column: 1 / 3; display: flex; align-items: center; gap: 16px; padding: 0 20px;
    background: var(--card); border-bottom: 1px solid var(--line); z-index: 5;
  }
  .logo { display: flex; align-items: center; gap: 10px; font-weight: 700; font-size: 17px; color: var(--primary-dark); }
  .logo .mark { width: 32px; height: 32px; border-radius: 9px; background: linear-gradient(135deg, var(--primary), var(--accent));
    display: flex; align-items: center; justify-content: center; color: #fff; font-size: 15px; font-weight: 800; }
  .logo small { display: block; font-size: 11px; color: var(--sub); font-weight: 400; }
  .spacer { flex: 1; }
  .badge { padding: 3px 12px; border-radius: 999px; font-size: 12.5px; font-weight: 600; }
  .badge.patient { background: #e0f2fe; color: #0369a1; }
  .badge.doctor { background: #fef3c7; color: #92400e; }
  .badge.st-idle { background: #f1f5f9; color: #475569; }
  .badge.st-run { background: #dcfce7; color: #166534; }
  .badge.st-pause { background: #fef9c3; color: #854d0e; }
  .badge.st-fault { background: #fee2e2; color: #991b1b; animation: blink 1s infinite; }
  @keyframes blink { 50% { opacity: .45; } }
  #clock { font-variant-numeric: tabular-nums; color: var(--sub); font-size: 13.5px; }

  /* ===== 侧栏 ===== */
  aside { background: linear-gradient(180deg, var(--sidebar), var(--sidebar2)); padding: 18px 12px; display: flex; flex-direction: column; gap: 6px; }
  .nav-item {
    display: flex; align-items: center; gap: 10px; padding: 12px 14px; border: none; border-radius: 10px;
    background: transparent; color: #b8d4de; font-size: 15px; cursor: pointer; text-align: left; width: 100%;
  }
  .nav-item svg { width: 19px; height: 19px; flex: none; }
  .nav-item:hover { background: rgba(255,255,255,.07); color: #fff; }
  .nav-item.active { background: rgba(20,184,166,.22); color: #fff; box-shadow: inset 3px 0 0 var(--accent); }
  aside .foot { margin-top: auto; color: #5f8494; font-size: 11px; line-height: 1.6; padding: 0 8px; }

  /* ===== 主区 ===== */
  main { overflow-y: auto; padding: 20px 24px; }
  .view { display: none; }
  .view.active { display: block; }
  .card { background: var(--card); border: 1px solid var(--line); border-radius: 14px; padding: 18px 20px; box-shadow: 0 1px 3px rgba(15,40,60,.05); }
  .card h3 { font-size: 14.5px; color: var(--primary-dark); margin-bottom: 12px; display: flex; align-items: center; gap: 8px; }
  .card h3 .dot { width: 8px; height: 8px; border-radius: 50%; background: var(--accent); }

  /* 报警条 */
  #alarmBar { display: none; margin-bottom: 14px; border-radius: 12px; padding: 12px 16px;
    background: #fef2f2; border: 1.5px solid var(--danger); color: #991b1b; align-items: center; gap: 12px; }
  #alarmBar.show { display: flex; }
  #alarmBar.acked { background: #fffbeb; border-color: var(--warn); color: #92400e; }
  #alarmBar .ic { font-size: 20px; }
  #alarmBar b { flex: 1; }

  /* 治疗页布局 */
  .treat-grid { display: grid; grid-template-columns: 400px 1fr; gap: 18px; align-items: start; }
  .ring-wrap { position: relative; width: 300px; height: 300px; margin: 6px auto 4px; }
  .ring-wrap svg { transform: rotate(-90deg); }
  .ring-center { position: absolute; inset: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 4px; }
  #remainText { font-size: 52px; font-weight: 700; font-variant-numeric: tabular-nums; color: var(--primary-dark); letter-spacing: 1px; }
  #stateText { font-size: 15px; color: var(--sub); }
  #stateText.err { color: var(--danger); font-weight: 700; }

  .ctrl-row { display: flex; gap: 10px; justify-content: center; margin-top: 10px; flex-wrap: wrap; }
  .btn { border: none; border-radius: 10px; padding: 11px 20px; font-size: 14.5px; cursor: pointer; font-weight: 600;
    background: #e8eef2; color: #334155; transition: .15s; }
  .btn:hover { filter: brightness(.96); }
  .btn:disabled { opacity: .45; cursor: not-allowed; }
  .btn-primary { background: var(--primary); color: #fff; }
  .btn-go { background: var(--ok); color: #fff; }
  .btn-warn { background: var(--warn); color: #fff; }
  .btn-danger { background: var(--danger); color: #fff; }
  .btn-ghost { background: transparent; border: 1px solid var(--line); color: var(--sub); }
  .btn-lg { padding: 14px 30px; font-size: 16.5px; border-radius: 12px; }
  .btn-sm { padding: 6px 12px; font-size: 13px; border-radius: 8px; }

  /* 强度 */
  .intensity { margin-top: 18px; }
  .intensity .top { display: flex; justify-content: space-between; align-items: baseline; margin-bottom: 8px; }
  .intensity .val { font-size: 30px; font-weight: 700; color: var(--primary-dark); font-variant-numeric: tabular-nums; }
  .intensity .val small { font-size: 14px; color: var(--sub); font-weight: 400; }
  .intensity .cap { font-size: 12.5px; color: var(--sub); }
  .int-row { display: flex; align-items: center; gap: 12px; }
  .ibtn { width: 52px; height: 52px; border-radius: 14px; border: 1.5px solid var(--line); background: #f8fafc;
    font-size: 26px; font-weight: 700; color: var(--primary-dark); cursor: pointer; }
  .ibtn:active { background: #e0f2f5; }
  .ibtn:disabled { opacity: .4; cursor: not-allowed; }
  .ibar { flex: 1; height: 14px; border-radius: 999px; background: #e8eef2; overflow: hidden; }
  .ibar > div { height: 100%; width: 0%; background: linear-gradient(90deg, var(--accent), var(--primary)); border-radius: 999px; transition: width .25s; }
  .hint { font-size: 12px; color: var(--sub); margin-top: 8px; }

  /* 通道与波形 */
  .right-grid { display: grid; gap: 16px; }
  .ch-row { display: grid; grid-template-columns: 64px 1fr 1fr 110px; align-items: center; gap: 10px; padding: 10px 12px; border-radius: 10px; background: #f8fafc; margin-bottom: 8px; }
  .ch-row:last-child { margin-bottom: 0; }
  .ch-name { font-weight: 700; color: var(--primary-dark); font-size: 15px; }
  .kv { font-size: 13.5px; color: var(--sub); }
  .kv b { color: var(--text); font-size: 16px; font-variant-numeric: tabular-nums; }
  .ztag { justify-self: end; font-size: 12.5px; font-weight: 700; padding: 3px 10px; border-radius: 999px; }
  .ztag.ok { background: #dcfce7; color: #166534; }
  .ztag.hi { background: #fee2e2; color: #991b1b; }
  .ztag.lo { background: #fef9c3; color: #854d0e; }
  #waveCv { width: 100%; height: 170px; display: block; background: #fbfdfe; border: 1px solid var(--line); border-radius: 10px; }
  .wave-legend { display: flex; gap: 16px; font-size: 12px; color: var(--sub); margin-top: 8px; flex-wrap: wrap; }
  .wave-legend i { display: inline-block; width: 10px; height: 10px; border-radius: 3px; margin-right: 5px; vertical-align: -1px; }
  .chips { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 10px; }
  .chip { font-size: 12.5px; background: #f1f5f9; border: 1px solid var(--line); border-radius: 8px; padding: 4px 10px; color: #334155; }
  .chip b { color: var(--primary-dark); }
  .preset-line { display: flex; justify-content: space-between; padding: 7px 0; border-bottom: 1px dashed var(--line); font-size: 13.5px; }
  .preset-line:last-child { border-bottom: none; }
  .preset-line span { color: var(--sub); }
  .preset-line b { font-variant-numeric: tabular-nums; }

  /* 表单 */
  .sec { margin-bottom: 26px; }
  .sec > h2 { font-size: 16px; color: var(--primary-dark); margin: 4px 0 12px; padding-left: 10px; border-left: 4px solid var(--accent); }
  .field { margin-bottom: 16px; }
  .field label { display: flex; justify-content: space-between; font-size: 13.5px; color: #334155; margin-bottom: 6px; }
  .field label b { color: var(--primary-dark); font-variant-numeric: tabular-nums; }
  input[type=range] { width: 100%; accent-color: var(--primary); height: 22px; }
  input[type=number], input[type=text], input[type=password], select {
    width: 100%; padding: 9px 12px; border: 1px solid var(--line); border-radius: 9px; font-size: 14px; background: #fbfdfe;
  }
  input:disabled, select:disabled { background: #eef2f5; color: #94a3b8; }
  .grid2 { display: grid; grid-template-columns: 1fr 1fr; gap: 0 26px; }
  .grid3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 0 26px; }
  .note { font-size: 12px; color: var(--sub); margin-top: 6px; line-height: 1.6; }
  .note.warn { color: var(--warn); }
  .seg { display: flex; gap: 0; border: 1.5px solid var(--primary); border-radius: 10px; overflow: hidden; width: fit-content; }
  .seg button { border: none; padding: 9px 22px; background: #fff; color: var(--primary-dark); font-size: 14px; cursor: pointer; font-weight: 600; }
  .seg button.on { background: var(--primary); color: #fff; }
  .switch { position: relative; width: 46px; height: 26px; flex: none; }
  .switch input { display: none; }
  .switch i { position: absolute; inset: 0; background: #cbd5e1; border-radius: 999px; cursor: pointer; transition: .2s; }
  .switch i::after { content: ""; position: absolute; width: 20px; height: 20px; border-radius: 50%; background: #fff; top: 3px; left: 3px; transition: .2s; box-shadow: 0 1px 3px rgba(0,0,0,.25); }
  .switch input:checked + i { background: var(--ok); }
  .switch input:checked + i::after { left: 23px; }
  .swrow { display: flex; align-items: center; justify-content: space-between; padding: 9px 0; }
  .swrow .t { font-size: 14px; }
  .swrow .t small { display: block; color: var(--sub); font-size: 12px; }

  /* 表格 */
  table { width: 100%; border-collapse: collapse; font-size: 13px; }
  th { text-align: left; color: var(--sub); font-weight: 600; padding: 8px 10px; border-bottom: 1.5px solid var(--line); white-space: nowrap; }
  td { padding: 8px 10px; border-bottom: 1px solid #edf2f6; vertical-align: top; }
  tr:hover td { background: #f8fbfd; }
  .tag-ok { color: var(--ok); font-weight: 700; } .tag-bad { color: var(--danger); font-weight: 700; }
  .empty { text-align: center; color: var(--sub); padding: 30px 0; font-size: 13.5px; }

  /* 依从性图 */
  .bars { display: flex; align-items: flex-end; gap: 14px; height: 120px; padding: 0 6px; border-bottom: 1.5px solid var(--line); }
  .bar-col { flex: 1; display: flex; flex-direction: column; align-items: center; gap: 4px; height: 100%; justify-content: flex-end; }
  .bar-col .bar { width: 55%; max-width: 44px; background: linear-gradient(180deg, var(--accent), var(--primary)); border-radius: 6px 6px 0 0; min-height: 2px; }
  .bar-col .lbl { font-size: 11px; color: var(--sub); white-space: nowrap; }
  .bar-col .num { font-size: 11.5px; color: var(--primary-dark); font-weight: 700; }

  .kpis { display: grid; grid-template-columns: repeat(4, 1fr); gap: 14px; margin-bottom: 16px; }
  .kpi { background: var(--card); border: 1px solid var(--line); border-radius: 12px; padding: 14px 16px; }
  .kpi .n { font-size: 26px; font-weight: 700; color: var(--primary-dark); font-variant-numeric: tabular-nums; }
  .kpi .n small { font-size: 13px; color: var(--sub); font-weight: 400; }
  .kpi .t { font-size: 12.5px; color: var(--sub); margin-top: 3px; }

  /* VAS */
  .vas-row { display: grid; grid-template-columns: 120px 1fr 60px; align-items: center; gap: 16px; padding: 12px 0; border-bottom: 1px dashed var(--line); }
  .vas-row .nm { font-size: 14.5px; font-weight: 600; }
  .vas-row .nm small { display: block; font-weight: 400; color: var(--sub); font-size: 11.5px; }
  .vas-row .sc { font-size: 22px; font-weight: 700; color: var(--primary-dark); text-align: right; font-variant-numeric: tabular-nums; }

  /* 模态 */
  #overlay { display: none; position: fixed; inset: 0; background: rgba(9, 30, 41, .55); z-index: 50; align-items: center; justify-content: center; }
  #overlay.show { display: flex; }
  .modal { background: #fff; border-radius: 16px; width: min(680px, 92vw); max-height: 88vh; overflow-y: auto; padding: 24px 26px; }
  .modal h2 { font-size: 17px; color: var(--primary-dark); margin-bottom: 14px; }
  .modal .acts { display: flex; gap: 10px; justify-content: flex-end; margin-top: 18px; }
  .ck-row { display: flex; gap: 10px; align-items: flex-start; background: #f0f9ff; border: 1px solid #bae6fd; padding: 12px 14px; border-radius: 10px; margin: 14px 0; font-size: 13.5px; cursor: pointer; }
  .ck-row input { margin-top: 3px; width: 17px; height: 17px; accent-color: var(--primary); }

  /* toast */
  #toasts { position: fixed; top: 70px; right: 22px; z-index: 99; display: flex; flex-direction: column; gap: 10px; }
  .toast { background: #0f172a; color: #fff; padding: 11px 18px; border-radius: 10px; font-size: 13.5px; box-shadow: 0 6px 18px rgba(0,0,0,.25); animation: tin .25s; max-width: 340px; }
  .toast.ok { background: var(--ok); } .toast.warn { background: var(--warn); } .toast.err { background: var(--danger); }
  @keyframes tin { from { transform: translateX(30px); opacity: 0; } }

  /* 底栏 */
  footer { grid-column: 1 / 3; background: var(--sidebar); color: #7fa8b8; display: flex; align-items: center; gap: 26px; padding: 0 20px; font-size: 12px; }
  footer b { color: #cde7ef; font-weight: 600; }
  .led { display: inline-block; width: 8px; height: 8px; border-radius: 50%; margin-right: 6px; vertical-align: 0; }
  .led.g { background: #22c55e; box-shadow: 0 0 6px #22c55e; }
  .led.r { background: #ef4444; box-shadow: 0 0 6px #ef4444; }
  .pulse { animation: blink 1.2s infinite; }

  .group-item { display: flex; align-items: center; gap: 10px; padding: 10px 12px; border: 1px solid var(--line); border-radius: 10px; margin-bottom: 8px; background: #fbfdfe; }
  .group-item .gn { font-weight: 700; color: var(--primary-dark); flex: 1; }
  .group-item .gn small { display: block; font-weight: 400; color: var(--sub); font-size: 11.5px; margin-top: 2px; }
  .group-item.cur { border-color: var(--accent); background: #f0fdfa; }

  .doc-only-label { font-size: 11px; background: #fef3c7; color: #92400e; padding: 2px 8px; border-radius: 6px; font-weight: 600; }
  .lockbox { text-align: center; padding: 34px 20px; }
  .lockbox .ic { font-size: 44px; margin-bottom: 12px; }

  /* ===== PWA 安装入口 ===== */
  .btn-install { display: none; align-items: center; gap: 6px; border: none; border-radius: 9px;
    background: linear-gradient(135deg, var(--primary), var(--accent)); color: #fff; font-size: 13px;
    font-weight: 600; padding: 8px 14px; cursor: pointer; white-space: nowrap; }
  .btn-install.show { display: inline-flex; }
  .btn-install:active { filter: brightness(.94); }

  /* ===== 中等宽度：主控与状态改为上下排列 ===== */
  @media (max-width: 1100px) {
    .treat-grid { grid-template-columns: 1fr; }
  }

  /* ===== 手机竖屏适配 ===== */
  @media (max-width: 820px) {
    body { font-size: 14.5px; }
    #app {
      grid-template-columns: 1fr;
      grid-template-rows: auto 1fr auto;
      height: 100vh; height: 100dvh;
    }
    header { grid-column: 1 / 2; padding: 0 12px; gap: 8px; height: 54px; }
    .logo { gap: 8px; font-size: 14.5px; min-width: 0; }
    .logo .mark { width: 28px; height: 28px; border-radius: 8px; font-size: 13px; }
    .logo > div:last-child { white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
    .logo small { display: none; }
    #clock { display: none; }
    .badge { font-size: 11px; padding: 3px 8px; }
    #btnInstall { padding: 6px 10px; font-size: 12px; }

    main { grid-row: 2; padding: 12px 12px 16px; }

    /* 侧栏改为底部导航 */
    aside { grid-row: 3; grid-column: 1 / 2; flex-direction: row; align-items: stretch;
      padding: 4px 4px calc(4px + env(safe-area-inset-bottom)); gap: 2px; }
    .nav-item { flex-direction: column; gap: 3px; font-size: 11px; padding: 8px 2px 7px;
      border-radius: 8px; justify-content: center; }
    .nav-item svg { width: 21px; height: 21px; }
    .nav-item.active { box-shadow: inset 0 3px 0 var(--accent); background: rgba(20,184,166,.16); }
    aside .foot { display: none; }
    footer { display: none; }

    .treat-grid { grid-template-columns: 1fr; gap: 12px; }
    .ring-wrap { width: min(285px, 78vw); height: min(285px, 78vw); }
    .ring-wrap svg { width: 100%; height: 100%; }
    #remainText { font-size: clamp(38px, 13vw, 52px); }
    .ctrl-row { gap: 8px; }
    .ctrl-row .btn-lg { flex: 1 1 42%; padding: 13px 12px; font-size: 15.5px; }
    .ibtn { width: 48px; height: 48px; font-size: 24px; }
    .intensity { margin-top: 14px; }
    .intensity .top { flex-wrap: wrap; gap: 4px 10px; }
    .card { padding: 14px; border-radius: 12px; overflow-x: auto; }
    .ch-row { display: flex; flex-wrap: wrap; gap: 6px 10px; }
    .ch-name { width: 38px; }
    .ztag { margin-left: auto; }
    #waveCv { height: 140px; }
    .grid2, .grid3 { grid-template-columns: 1fr; }
    .kpis { grid-template-columns: 1fr 1fr; gap: 10px; }
    .kpi .n { font-size: 22px; }
    .sec { margin-bottom: 20px; }
    .vas-row { grid-template-columns: 1fr auto; gap: 6px 12px; }
    .vas-row .nm { grid-column: 1 / 2; }
    .vas-row .sc { grid-column: 2 / 3; grid-row: 1; font-size: 19px; }
    .vas-row input[type=range] { grid-column: 1 / 3; grid-row: 2; }
    .modal { width: 94vw; padding: 18px 16px; max-height: 90vh; }
    .guide-flex { flex-direction: column; }
    .guide-flex > svg { max-width: 100%; height: auto; margin: 0 auto; }
    #toasts { top: 62px; left: 12px; right: 12px; }
    .toast { max-width: none; }
    table { font-size: 12.5px; }
    th, td { padding: 7px 8px; }
    #logTable, #auditTable { min-width: 620px; }
    #logTable.is-empty, #auditTable.is-empty { min-width: 0; }
    .seg { width: 100%; }
    .seg button { flex: 1; padding: 9px 6px; font-size: 12.5px; }
    .group-item { flex-wrap: wrap; }
    .group-item .gn { flex-basis: 100%; }
  }
</style>
</head>
<body>
<div id="app">

  <!-- ============ 顶栏 ============ -->
  <header>
    <div class="logo">
      <div class="mark">GE</div>
      <div>胃电刺激治疗系统 <small>GES-100 · 经皮干扰电治疗</small></div>
    </div>
    <div class="spacer"></div>
    <span id="roleBadge" class="badge patient">患者模式</span>
    <span id="stBadge" class="badge st-idle">待机</span>
    <span id="clock">--:--:--</span>
  </header>

  <!-- ============ 侧栏 ============ -->
  <aside>
    <button class="nav-item active" data-view="treatment">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 12h4l2-7 4 14 2-7h6"/></svg>
      治疗
    </button>
    <button class="nav-item" data-view="feedback">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 21s-7-4.6-9.5-9C.7 8.6 2.7 5 6.2 5c2 0 3.4 1 4.3 2.4L12 9l1.5-1.6C14.4 6 15.8 5 17.8 5c3.5 0 5.5 3.6 3.7 7-2.5 4.4-9.5 9-9.5 9z"/></svg>
      感受反馈
    </button>
    <button class="nav-item" data-view="records">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 20V10M10 20V4M16 20v-8M22 20H2"/></svg>
      记录与导出
    </button>
    <button class="nav-item" data-view="settings">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="3"/><path d="M19 12a7 7 0 0 0-.1-1.2l2-1.5-2-3.5-2.4 1a7 7 0 0 0-2-1.2L14 3h-4l-.5 2.6a7 7 0 0 0-2 1.2l-2.4-1-2 3.5 2 1.5A7 7 0 0 0 5 12c0 .4 0 .8.1 1.2l-2 1.5 2 3.5 2.4-1a7 7 0 0 0 2 1.2L10 21h4l.5-2.6a7 7 0 0 0 2-1.2l2.4 1 2-3.5-2-1.5c.1-.4.1-.8.1-1.2z"/></svg>
      设置
    </button>
    <div class="foot">软件版本 v0.9.3 原型<br>安全等级：中（B）<br>看门狗 · 联锁 · 审计 就绪</div>
  </aside>

  <!-- ============ 主区 ============ -->
  <main>

    <!-- ===== 治疗 ===== -->
    <section class="view active" id="view-treatment">
      <div id="alarmBar">
        <span class="ic">⚠</span><b id="alarmText">--</b>
        <button class="btn btn-sm" id="btnAck">确认报警</button>
      </div>

      <div class="treat-grid">
        <!-- 左：主控 -->
        <div class="card">
          <h3><span class="dot"></span>治疗控制</h3>
          <div class="ring-wrap">
            <svg width="300" height="300" viewBox="0 0 300 300">
              <circle cx="150" cy="150" r="128" fill="none" stroke="#e8eef2" stroke-width="16"/>
              <circle id="ring" cx="150" cy="150" r="128" fill="none" stroke="#0e7490" stroke-width="16"
                stroke-linecap="round" stroke-dasharray="804.25" stroke-dashoffset="804.25"/>
            </svg>
            <div class="ring-center">
              <div id="remainText">--:--</div>
              <div id="stateText">设备待机 · 输出 0.0 mA</div>
            </div>
          </div>

          <div class="ctrl-row">
            <button class="btn btn-go btn-lg" id="btnStart">开始治疗</button>
            <button class="btn btn-warn btn-lg" id="btnPause" style="display:none">暂停</button>
            <button class="btn btn-go btn-lg" id="btnResume" style="display:none">继续</button>
            <button class="btn btn-danger btn-lg" id="btnStop" style="display:none">停止治疗</button>
            <button class="btn btn-primary btn-lg" id="btnRecheck" style="display:none">重新检测电极</button>
          </div>

          <div class="intensity">
            <div class="top">
              <span class="val"><span id="intVal">0.0</span><small> mA 刺激强度</small></span>
              <span class="cap">医生设定上限：<b id="intCap">--</b> mA</span>
            </div>
            <div class="int-row">
              <button class="ibtn" id="btnMinus">−</button>
              <div class="ibar"><div id="intBar"></div></div>
              <button class="ibtn" id="btnPlus">＋</button>
            </div>
            <div class="hint">· 每次点击微调 0.5 mA，输出经 3~5 秒软启动平滑过渡，防止突然高输出</div>
          </div>
        </div>

        <!-- 右：状态 -->
        <div class="right-grid">
          <div class="card">
            <h3><span class="dot"></span>通道与电极接触状态（实时阻抗监测）</h3>
            <div class="ch-row">
              <span class="ch-name">CH1</span>
              <span class="kv">输出 <b id="ch1Out">0.0</b> mA</span>
              <span class="kv">阻抗 <b id="ch1Z">--</b> kΩ</span>
              <span class="ztag ok" id="ch1Tag">正常</span>
            </div>
            <div class="ch-row">
              <span class="ch-name">CH2</span>
              <span class="kv">输出 <b id="ch2Out">0.0</b> mA</span>
              <span class="kv">阻抗 <b id="ch2Z">--</b> kΩ</span>
              <span class="ztag ok" id="ch2Tag">正常</span>
            </div>
            <div class="note">判定阈值（医生可调）：过高 &gt; <b id="zHiTxt">60</b> kΩ（脱落/干燥），过低 &lt; <b id="zLoTxt">0.8</b> kΩ（短路）。异常时毫秒级切断输出并报警。</div>
          </div>

          <div class="card">
            <h3><span class="dot"></span>输出波形监视</h3>
            <canvas id="waveCv"></canvas>
            <div class="wave-legend">
              <span><i style="background:#0e7490"></i>组织内干涉电流（示意，时间压缩显示）</span>
              <span><i style="background:#94a3b8"></i>拍频包络 ±A</span>
            </div>
            <div class="chips" id="paramChips"></div>
          </div>

          <div class="card">
            <h3><span class="dot"></span>当前处方</h3>
            <div class="preset-line"><span>参数组</span><b id="pGroup">--</b></div>
            <div class="preset-line"><span>治疗模式</span><b id="pMode">--</b></div>
            <div class="preset-line"><span>单次时长 / 每日上限</span><b id="pLen">--</b></div>
            <div class="preset-line"><span>今日已开始治疗</span><b id="pToday">0 / 2 次</b></div>
            <div class="preset-line"><span>阻抗动态校准</span><b id="pCal">--</b></div>
          </div>
        </div>
      </div>
    </section>

    <!-- ===== 反馈 ===== -->
    <section class="view" id="view-feedback">
      <div class="sec"><h2>主观症状评分（VAS 0-10）</h2>
        <div class="card" style="max-width:760px">
          <div class="field" style="max-width:280px">
            <label>记录时机</label>
            <select id="vasPhase">
              <option>治疗前</option><option>治疗后</option><option>治疗中</option><option>日常随访</option>
            </select>
          </div>
          <div class="vas-row">
            <div class="nm">恶心<small>反酸、呕吐感、上腹不适</small></div>
            <input type="range" id="vasN" min="0" max="10" step="1" value="0">
            <div class="sc" id="vasNv">0</div>
          </div>
          <div class="vas-row">
            <div class="nm">腹胀<small>饱胀感、腹部膨隆</small></div>
            <input type="range" id="vasB" min="0" max="10" step="1" value="0">
            <div class="sc" id="vasBv">0</div>
          </div>
          <div class="vas-row" style="border-bottom:none">
            <div class="nm">早饱<small>进食少量即饱、食欲下降</small></div>
            <input type="range" id="vasE" min="0" max="10" step="1" value="0">
            <div class="sc" id="vasEv">0</div>
          </div>
          <div class="note">评分将随时间戳保存，供医生评估疗效并调整处方参数。</div>
          <div class="acts" style="display:flex;justify-content:flex-end;gap:10px;margin-top:14px">
            <button class="btn btn-primary" id="btnSaveVas">保存本次评分</button>
          </div>
        </div>
      </div>
      <div class="sec"><h2>近期评分记录</h2>
        <div class="card" style="max-width:760px"><table id="vasTable"></table></div>
      </div>
    </section>

    <!-- ===== 记录 ===== -->
    <section class="view" id="view-records">
      <div class="kpis" id="kpiRow"></div>
      <div class="sec"><h2>使用依从性（近 7 天）</h2>
        <div class="card">
          <div class="bars" id="compBars"></div>
          <div class="note" style="margin-top:8px">柱高 = 当日完成治疗次数（目标见设置-每日上限）。供医生评估患者依从性。</div>
        </div>
      </div>
      <div class="sec"><h2>刺激日志</h2>
        <div class="card">
          <div style="display:flex;gap:10px;margin-bottom:12px">
            <button class="btn btn-primary btn-sm" id="btnExpCsv">导出治疗记录 CSV</button>
            <button class="btn btn-ghost btn-sm" id="btnExpJson">导出全量备份 JSON</button>
            <span class="spacer"></span>
          </div>
          <table id="logTable"></table>
        </div>
      </div>
    </section>

    <!-- ===== 设置 ===== -->
    <section class="view" id="view-settings">
      <!-- 患者视角 -->
      <div id="patSettings">
        <div class="sec"><h2>处方概览（只读）</h2>
          <div class="card" style="max-width:760px">
            <div class="preset-line"><span>参数组</span><b id="sGroup">--</b></div>
            <div class="preset-line"><span>治疗模式</span><b id="sMode">--</b></div>
            <div class="preset-line"><span>单次时长 / 每日上限</span><b id="sLen">--</b></div>
            <div class="preset-line"><span>强度上限</span><b id="sCap">--</b></div>
            <div class="preset-line"><span>处方锁定</span><b id="sLock">--</b></div>
            <div class="note">核心参数由医生设定并锁定。如需调整频率差、脉宽、治疗时长等，请联系医生在“医生模式”下修改。</div>
          </div>
        </div>
        <div class="sec"><h2>医生模式</h2>
          <div class="card lockbox" style="max-width:760px">
            <div class="ic">🔒</div>
            <p style="color:var(--sub);margin-bottom:16px">频率差、脉宽、治疗时长等核心参数需医生登录后调整。<br>登录与修改均写入审计追踪。</p>
            <button class="btn btn-primary btn-lg" id="btnDocLogin">医生登录</button>
          </div>
        </div>
      </div>

      <!-- 医生视角 -->
      <div id="docSettings" style="display:none">
        <div class="sec"><h2>治疗模式与波形引擎 <span class="doc-only-label">医生模式</span></h2>
          <div class="card" style="max-width:860px">
            <div class="field"><label>输出模式</label>
              <div class="seg" id="modeSeg">
                <button data-m="IFE">干扰电（两路 4 kHz 中频）</button>
                <button data-m="TENS">传统 TENS（低频脉冲）</button>
              </div>
            </div>
            <div class="grid2">
              <div class="field"><label>差频 Δf（两路载波频率差）<b><span id="v-df"></span> Hz</b></label>
                <input type="range" data-param="df" min="1" max="100" step="1">
                <div class="note">范围 1–100 Hz（硬限幅）。CH2 载波 = 4000 Hz + Δf，固定载波 4 kHz 不可改。</div>
              </div>
              <div class="field"><label>两路相位差<b><span id="v-phase"></span>°</b></label>
                <input type="range" data-param="phase" min="0" max="180" step="5">
                <div class="note">精确控制两路输出的相位差，以维持靶组织内稳定干涉电场。</div>
              </div>
            </div>
            <div id="tensFields" style="display:none">
              <div class="grid2">
                <div class="field"><label>TENS 脉冲频率<b><span id="v-tensFreq"></span> Hz</b></label>
                  <input type="range" data-param="tensFreq" min="1" max="150" step="1">
                </div>
                <div class="field"><label>脉冲宽度<b><span id="v-pw"></span> μs</b></label>
                  <input type="range" data-param="pw" min="50" max="1000" step="10">
                  <div class="note">单脉冲净电荷上限校验：当前最大 <b id="chargeTxt">--</b>（监管限值 25 μC 内）。</div>
                </div>
              </div>
            </div>
            <div class="swrow">
              <div class="t">阻抗动态校准<small>根据电极阻抗变化微调输出，维持稳定干涉场</small></div>
              <label class="switch"><input type="checkbox" id="swCal"><i></i></label>
            </div>
          </div>
        </div>

        <div class="sec"><h2>安全限值与联锁</h2>
          <div class="card" style="max-width:860px">
            <div class="grid3">
              <div class="field"><label>最大输出电流（强度上限）<b><span id="v-maxCurrent"></span> mA</b></label>
                <input type="range" data-param="maxCurrent" min="1" max="40" step="0.5">
                <div class="note warn">硬限幅 40 mA，超出监管限值的输入将被拒绝。</div>
              </div>
              <div class="field"><label>阻抗过高阈值<b><span id="v-zHigh"></span> kΩ</b></label>
                <input type="range" data-param="zHigh" min="30" max="200" step="5">
              </div>
              <div class="field"><label>阻抗过低阈值<b><span id="v-zLow"></span> kΩ</b></label>
                <input type="range" data-param="zLow" min="0.2" max="5" step="0.1">
              </div>
            </div>
            <div class="swrow">
              <div class="t">看门狗定时器<small>监控主控循环，异常时复位并使输出归零（故障安全）</small></div>
              <span class="tag-ok">运行中 ●</span>
            </div>
            <div class="swrow">
              <div class="t">故障安全自检<small>每次上电复位后默认输出为 0，需预检通过方可输出</small></div>
              <span class="tag-ok">已启用 ●</span>
            </div>
          </div>
        </div>

        <div class="sec"><h2>处方化治疗设置</h2>
          <div class="card" style="max-width:860px">
            <div class="grid2">
              <div class="field"><label>单次治疗时长<b><span id="v-sessionMin"></span> 分钟</b></label>
                <input type="range" data-param="sessionMin" min="10" max="90" step="5">
              </div>
              <div class="field"><label>每日治疗次数上限<b><span id="v-dailyLimit"></span> 次</b></label>
                <input type="range" data-param="dailyLimit" min="1" max="5" step="1">
                <div class="note">达到上限后软件拒绝启动输出，防止过度使用。</div>
              </div>
            </div>
          </div>
        </div>

        <div class="sec"><h2>参数组管理</h2>
          <div class="card" style="max-width:860px">
            <div id="groupList"></div>
            <div style="display:flex;gap:10px;align-items:center;margin-top:14px">
              <input type="text" id="groupName" placeholder="新参数组名称，如：餐后模式" style="width:260px">
              <button class="btn btn-primary btn-sm" id="btnSaveGroup">将当前参数存为新组</button>
              <span class="spacer"></span>
              <label class="switch" title="锁定处方"><input type="checkbox" id="swLock"><i></i></label>
              <span style="font-size:13.5px">锁定处方<small style="display:block;color:var(--sub)">锁定后参数组不可切换/删除</small></span>
            </div>
          </div>
        </div>

        <div class="sec"><h2>演示 / 验证工具</h2>
          <div class="card" style="max-width:860px">
            <div class="note" style="margin-bottom:10px">用于演示与可用性测试的联锁触发模拟（仅原型提供）：</div>
            <div style="display:flex;gap:10px">
              <button class="btn btn-sm" id="simNormal" style="border:1px solid var(--line)">恢复正常电极</button>
              <button class="btn btn-sm" id="simOpen" style="border:1px solid var(--danger);color:var(--danger)">模拟电极脱落（阻抗过高）</button>
              <button class="btn btn-sm" id="simShort" style="border:1px solid var(--warn);color:var(--warn)">模拟电极短路（阻抗过低）</button>
            </div>
          </div>
        </div>

        <div class="sec"><h2>审计追踪</h2>
          <div class="card" style="max-width:980px">
            <div style="display:flex;gap:10px;margin-bottom:12px">
              <button class="btn btn-primary btn-sm" id="btnExpAudit">导出审计追踪 CSV</button>
              <span class="spacer"></span>
              <button class="btn btn-ghost btn-sm" id="btnDocLogout">退出医生模式</button>
            </div>
            <table id="auditTable"></table>
          </div>
        </div>
      </div>
    </section>
  </main>

  <!-- ============ 底栏 ============ -->
  <footer>
    <span><span class="led g pulse"></span>看门狗 <b>OK</b></span>
    <span><span class="led g" id="ledInterlock"></span>安全联锁 <b id="ftInterlock">就绪</b></span>
    <span>实际输出 <b id="ftOut">0.0 mA</b></span>
    <span>复位后默认输出：<b>0 mA（故障安全）</b></span>
    <span class="spacer"></span>
    <span>界面原型 · 仅用于设计评审与可用性测试 · 不具备实际治疗功能</span>
  </footer>
</div>

<!-- 模态 -->
<div id="overlay"><div class="modal" id="modalBox"></div></div>
<div id="toasts"></div>

<script>
/* ================================================================
   GES-100 胃电刺激治疗系统 · 软件界面原型
   模块：波形引擎 / 阻抗监测与联锁 / 会话管理 / 数据与审计 / 分级权限
================================================================ */
'use strict';
const $ = s => document.querySelector(s);
const $$ = s => [...document.querySelectorAll(s)];
const clamp = (v, a, b) => Math.min(b, Math.max(a, v));
const pad = n => String(n).padStart(2, '0');
const fmtSec = s => pad(Math.floor(s / 60)) + ':' + pad(Math.floor(s % 60));
const ts = d => d.getFullYear() + '-' + pad(d.getMonth()+1) + '-' + pad(d.getDate()) + ' ' + pad(d.getHours()) + ':' + pad(d.getMinutes()) + ':' + pad(d.getSeconds());
const todayStr = (d = new Date()) => d.getFullYear() + '-' + pad(d.getMonth()+1) + '-' + pad(d.getDate());

const store = {
  load(k, def) { try { const v = JSON.parse(localStorage.getItem(k)); return v === null || v === undefined ? def : v; } catch (e) { return def; } },
  save(k, v) { localStorage.setItem(k, JSON.stringify(v)); }
};

/* ---------- 监管硬限幅（软件层不可逾越） ---------- */
const HARD = { carrier: 4000, dfMin: 1, dfMax: 100, tensFreqMax: 150, pwMin: 50, pwMax: 1000,
  currentMax: 40, phaseMax: 180, sessionMax: 90, dailyMax: 5, chargeLimit: 25 };

const PARAM_DEFS = {
  df:         { label: '差频 Δf',        unit: 'Hz',  min: 1,  max: 100, step: 1,   int: true },
  tensFreq:   { label: 'TENS 频率',      unit: 'Hz',  min: 1,  max: 150, step: 1,   int: true },
  pw:         { label: '脉冲宽度',        unit: 'μs',  min: 50, max: 1000, step: 10, int: true },
  phase:      { label: '相位差',          unit: '°',   min: 0,  max: 180, step: 5,   int: true },
  maxCurrent: { label: '最大输出电流',    unit: 'mA',  min: 1,  max: 40,  step: 0.5, int: false },
  sessionMin: { label: '单次治疗时长',    unit: 'min', min: 10, max: 90,  step: 5,   int: true },
  dailyLimit: { label: '每日治疗上限',    unit: '次',  min: 1,  max: 5,   step: 1,   int: true },
  zHigh:      { label: '阻抗过高阈值',    unit: 'kΩ',  min: 30, max: 200, step: 5,   int: true },
  zLow:       { label: '阻抗过低阈值',    unit: 'kΩ',  min: 0.2,max: 5,   step: 0.1, int: false },
};

const DEFAULT_GROUPS = {
  '餐前模式': { mode: 'IFE', df: 40, tensFreq: 100, pw: 250, phase: 0, maxCurrent: 15, sessionMin: 30, dailyLimit: 2, autoCal: true },
  '夜间模式': { mode: 'IFE', df: 10, tensFreq: 80,  pw: 300, phase: 0, maxCurrent: 12, sessionMin: 60, dailyLimit: 1, autoCal: true },
};
let groups = store.load('ges_groups', DEFAULT_GROUPS);
let settings = Object.assign({ activeGroup: '餐前模式', locked: false }, groups['餐前模式'], store.load('ges_settings', {}));
let logs = store.load('ges_logs', []);
let auditL = store.load('ges_audit', []);
let vasL = store.load('ges_vas', []);

const saveSettings = () => store.save('ges_settings', settings);
const saveGroups = () => store.save('ges_groups', groups);

/* ---------- 运行时状态 ---------- */
const rt = {
  state: 'idle',            // idle | rampup | running | paused | rampdown | fault
  role: 'patient',
  intensity: 0, setIntensity: 6, peak: 0,
  elapsed: 0, remain: 0,
  z1: 10, z2: 11, sim: 'normal',
  zSum1: 0, zSum2: 0, zN: 0,
  events: [], currentLog: null,
  alarm: null, alarmAck: false,
};

/* ---------- toast / modal / 声音 ---------- */
function toast(msg, type = '') {
  const d = document.createElement('div');
  d.className = 'toast ' + type; d.textContent = msg;
  $('#toasts').appendChild(d);
  setTimeout(() => { d.style.opacity = '0'; d.style.transition = '.4s'; setTimeout(() => d.remove(), 400); }, 3200);
}
function openModal(html) { $('#modalBox').innerHTML = html; $('#overlay').classList.add('show'); }
function closeModal() { $('#overlay').classList.remove('show'); }
$('#overlay').addEventListener('click', e => { if (e.target === $('#overlay')) closeModal(); });

let audioCtx = null;
function beep() {
  try {
    audioCtx = audioCtx || new (window.AudioContext || window.webkitAudioContext)();
    [0, 0.22].forEach(off => {
      const o = audioCtx.createOscillator(), g = audioCtx.createGain();
      o.type = 'square'; o.frequency.value = 880;
      g.gain.setValueAtTime(0.08, audioCtx.currentTime + off);
      g.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + off + 0.18);
      o.connect(g); g.connect(audioCtx.destination);
      o.start(audioCtx.currentTime + off); o.stop(audioCtx.currentTime + off + 0.2);
    });
  } catch (e) {}
}

/* ---------- 审计 ---------- */
function addAudit(actor, action, target, oldV, newV) {
  auditL.unshift({ time: ts(new Date()), actor, action, target: target || '', old: oldV === undefined ? '' : oldV, new: newV === undefined ? '' : newV });
  if (auditL.length > 500) auditL.length = 500;
  store.save('ges_audit', auditL);
  if (rt.currentView === 'settings') renderAuditTable();
}

/* ---------- 状态渲染 ---------- */
const STATE_TXT = { idle: '设备待机', rampup: '软启动中…', running: '治疗进行中', paused: '已暂停', rampdown: '安全停止中…', fault: '故障锁定' };
function setState(s) { rt.state = s; renderControls(); renderStatus(); }

function renderStatus() {
  const b = $('#stBadge');
  if (rt.state === 'fault') { b.textContent = '报警'; b.className = 'badge st-fault'; }
  else if (rt.state === 'running') { b.textContent = '治疗中'; b.className = 'badge st-run'; }
  else if (rt.state === 'paused' || rt.state === 'rampup' || rt.state === 'rampdown') { b.textContent = STATE_TXT[rt.state]; b.className = 'badge st-pause'; }
  else { b.textContent = '待机'; b.className = 'badge st-idle'; }
  $('#ledInterlock').className = 'led ' + (rt.state === 'fault' ? 'r' : 'g');
  $('#ftInterlock').textContent = rt.state === 'fault' ? '已触发·输出切断' : '就绪';
}

function renderControls() {
  const s = rt.state;
  $('#btnStart').style.display = s === 'idle' ? '' : 'none';
  $('#btnPause').style.display = s === 'running' ? '' : 'none';
  $('#btnResume').style.display = s === 'paused' ? '' : 'none';
  $('#btnStop').style.display = (s === 'running' || s === 'paused' || s === 'rampup') ? '' : 'none';
  $('#btnRecheck').style.display = s === 'fault' ? '' : 'none';

  $('#stateText').className = s === 'fault' ? 'err' : '';
  if (s === 'idle') { $('#remainText').textContent = '--:--'; $('#stateText').textContent = '设备待机 · 输出 0.0 mA'; }
  else if (s === 'fault') { $('#remainText').textContent = 'STOP'; $('#stateText').textContent = '输出已切断 · 请排除故障后重新检测'; }
  else { $('#remainText').textContent = fmtSec(Math.max(0, rt.remain)); $('#stateText').textContent = STATE_TXT[s] + ' · 输出 ' + rt.intensity.toFixed(1) + ' mA'; }

  const prog = (s === 'running' || s === 'paused') && rt.currentLog ? 1 - rt.remain / rt.currentLog.plannedSec : 0;
  const C = 2 * Math.PI * 128;
  $('#ring').style.strokeDashoffset = C * (1 - clamp(prog, 0, 1));
  $('#ring').style.stroke = s === 'fault' ? 'var(--danger)' : (s === 'paused' ? 'var(--warn)' : 'var(--primary)');

  $('#btnMinus').disabled = $('#btnPlus').disabled = (s === 'fault' || rt.role !== 'patient' && false);
  renderIntensity(); renderAlarm();
}

function renderIntensity() {
  $('#intVal').textContent = rt.setIntensity.toFixed(1);
  $('#intCap').textContent = settings.maxCurrent.toFixed(1);
  $('#intBar').style.width = clamp(rt.setIntensity / settings.maxCurrent * 100, 0, 100) + '%';
}

function renderAlarm() {
  const bar = $('#alarmBar');
  if (rt.alarm) {
    bar.classList.add('show');
    bar.classList.toggle('acked', rt.alarmAck);
    $('#alarmText').textContent = (rt.alarmAck ? '[已确认·待排除] ' : '') + rt.alarm;
    $('#btnAck').style.display = rt.alarmAck ? 'none' : '';
  } else bar.classList.remove('show');
}

function updateOutput() {
  rt.peak = Math.max(rt.peak, rt.intensity);
  const v = rt.intensity.toFixed(1);
  $('#ch1Out').textContent = v; $('#ch2Out').textContent = v; $('#ftOut').textContent = v + ' mA';
}

function zTag(el, z) {
  const hi = z > settings.zHigh, lo = z < settings.zLow;
  el.className = 'ztag ' + (hi ? 'hi' : lo ? 'lo' : 'ok');
  el.textContent = hi ? '过高·疑脱落' : lo ? '过低·疑短路' : '正常';
}
function renderChannels() {
  $('#ch1Z').textContent = rt.z1.toFixed(1); $('#ch2Z').textContent = rt.z2.toFixed(1);
  zTag($('#ch1Tag'), rt.z1); zTag($('#ch2Tag'), rt.z2);
  $('#zHiTxt').textContent = settings.zHigh; $('#zLoTxt').textContent = settings.zLow;
}
const zBad = z => z > settings.zHigh || z < settings.zLow;

/* ---------- 处方信息 ---------- */
const MODE_TXT = { IFE: '干扰电（4kHz 中频）', TENS: '传统 TENS' };
function renderPreset() {
  const g = settings.activeGroup;
  $('#pGroup').textContent = g + (settings.locked ? '（已锁定）' : '');
  $('#pMode').textContent = MODE_TXT[settings.mode] + ' · Δf ' + settings.df + ' Hz' + (settings.mode === 'TENS' ? ' · ' + settings.pw + ' μs' : '');
  $('#pLen').textContent = settings.sessionMin + ' min / ' + settings.dailyLimit + ' 次';
  $('#pToday').textContent = logs.filter(L => L.date === todayStr()).length + ' / ' + settings.dailyLimit + ' 次';
  $('#pCal').textContent = settings.autoCal ? '开启（补偿 ' + calComp() + '%）' : '关闭';
  $('#sGroup').textContent = settings.activeGroup + (settings.locked ? '（已锁定）' : '');
  $('#sMode').textContent = MODE_TXT[settings.mode] + ' · Δf ' + settings.df + ' Hz · 相位差 ' + settings.phase + '°';
  $('#sLen').textContent = settings.sessionMin + ' min / ' + settings.dailyLimit + ' 次';
  $('#sCap').textContent = settings.maxCurrent + ' mA';
  $('#sLock').textContent = settings.locked ? '已锁定' : '未锁定';
  $('#paramChips').innerHTML =
    '<span class="chip">模式 <b>' + MODE_TXT[settings.mode] + '</b></span>' +
    '<span class="chip">载波 <b>2 × ' + HARD.carrier + ' Hz</b></span>' +
    '<span class="chip">差频 <b>Δf = ' + settings.df + ' Hz</b></span>' +
    '<span class="chip">相位差 <b>' + settings.phase + '°</b></span>' +
    (settings.mode === 'TENS' ? '<span class="chip">脉宽 <b>' + settings.pw + ' μs</b></span><span class="chip">频率 <b>' + settings.tensFreq + ' Hz</b></span>' : '') +
    '<span class="chip">强度上限 <b>' + settings.maxCurrent + ' mA</b></span>';
}
function calComp() { return (clamp(((rt.z1 + rt.z2) / 2 - 10) / 10 * 3, -8, 8)).toFixed(1); }

/* ---------- 阻抗模拟与安全联锁 ---------- */
function simZ() {
  const j = (a, b) => a + Math.random() * (b - a);
  let t1, t2;
  if (rt.sim === 'open') { t1 = j(75, 150); t2 = j(75, 150); }
  else if (rt.sim === 'short') { t1 = j(0.05, 0.3); t2 = j(0.05, 0.3); }
  else { t1 = j(7, 13); t2 = j(7.5, 13.5); }
  rt.z1 = +(rt.z1 * 0.55 + t1 * 0.45).toFixed(2);
  rt.z2 = +(rt.z2 * 0.55 + t2 * 0.45).toFixed(2);
  renderChannels();
  if (rt.state === 'running' || rt.state === 'rampup') {
    rt.zSum1 += rt.z1; rt.zSum2 += rt.z2; rt.zN++;
    if (zBad(rt.z1) || zBad(rt.z2)) tripFault();
  }
}
setInterval(simZ, 900);

function tripFault() {
  const hi = rt.z1 > settings.zHigh || rt.z2 > settings.zHigh;
  const reason = hi ? '电极接触阻抗过高（可能脱落或皮肤干燥）' : '电极阻抗过低（疑似短路）';
  rt.intensity = 0; updateOutput();
  rt.alarm = '安全联锁触发：' + reason + '，输出已在毫秒级切断'; rt.alarmAck = false;
  addEvent('联锁触发：' + reason + ' → 输出切断');
  addAudit('系统', '安全联锁触发', '输出通道', '正常输出', '切断（' + reason + '）');
  finalizeLog('中断');
  setState('fault');
  beep();
  toast('安全联锁触发，输出已切断！', 'err');
}

$('#btnAck').addEventListener('click', () => { rt.alarmAck = true; renderAlarm(); });

$('#btnRecheck').addEventListener('click', function () {
  this.disabled = true; this.textContent = '检测中…';
  setTimeout(() => {
    this.disabled = false; this.textContent = '重新检测电极';
    if (zBad(rt.z1) || zBad(rt.z2)) toast('检测未通过：电极阻抗仍异常，请重新贴合电极或湿润皮肤后重试', 'warn');
    else { rt.alarm = null; rt.alarmAck = false; renderAlarm(); setState('idle'); toast('电极检测通过，可重新开始治疗', 'ok'); }
  }, 1600);
});

/* ---------- 软启动 / 软停止 ---------- */
function startRamp(up, done) {
  const target = up ? rt.setIntensity : 0;
  const from = rt.intensity, steps = Math.max(1, Math.round(Math.abs(target - from) / 0.5)) || 1;
  let i = 0;
  clearInterval(rt.rampTimer);
  rt.rampTimer = setInterval(() => {
    i++;
    rt.intensity = +(from + (target - from) * i / steps).toFixed(2);
    if (!up && rt.intensity < 0.05) rt.intensity = 0;
    updateOutput();
    if (i >= steps) { clearInterval(rt.rampTimer); rt.rampTimer = null; rt.intensity = +target.toFixed(2); updateOutput(); done && done(); }
  }, 400);
}

/* ---------- 会话生命周期 ---------- */
function addEvent(t) { rt.events.push(pad(new Date().getHours()) + ':' + pad(new Date().getMinutes()) + ' ' + t); }

function startSession() {
  const todayCount = logs.filter(L => L.date === todayStr()).length;
  if (todayCount >= settings.dailyLimit) {
    toast('已达每日治疗上限（' + settings.dailyLimit + ' 次）。如需调整，请联系医生。', 'warn');
    addAudit('系统', '拒绝启动', '治疗次数限制', '', '今日已 ' + todayCount + ' 次');
    return;
  }
  rt.currentLog = {
    id: Date.now(), date: todayStr(), start: ts(new Date()),
    plannedSec: settings.sessionMin * 60,
    params: { mode: settings.mode, df: settings.df, tensFreq: settings.tensFreq, pw: settings.pw, phase: settings.phase, group: settings.activeGroup },
    status: '治疗中'
  };
  rt.elapsed = 0; rt.remain = settings.sessionMin * 60;
  rt.events = ['开始治疗（' + settings.activeGroup + '）']; rt.peak = 0; rt.zSum1 = rt.zSum2 = 0; rt.zN = 0;
  setState('rampup');
  startRamp(true, () => { if (rt.state === 'rampup') setState('running'); });
  toast('软启动完成前请保持静止，如有不适请立即暂停或停止', 'ok');
}

function pauseSession() { setState('paused'); addEvent('暂停'); addAudit('患者', '暂停治疗', '会话', '运行', '暂停'); }
function resumeSession() { setState('running'); addEvent('恢复'); addAudit('患者', '恢复治疗', '会话', '暂停', '运行'); }

function stopSession() {
  if (rt.state === 'rampdown' || rt.state === 'idle' || rt.state === 'fault') return;
  setState('rampdown');
  startRamp(false, () => {
    addEvent('手动停止（软停止完成）');
    finalizeLog('中断');
    toast('治疗已安全停止，输出归零', 'warn');
    setState('idle');
  });
}

function completeSession() {
  setState('rampdown');
  startRamp(false, () => {
    addEvent('达到处方时长，治疗完成');
    finalizeLog('完成');
    setState('idle');
    const L = logs[0];
    openModal('<h2>✅ 本次治疗已完成</h2>' +
      '<div class="preset-line"><span>实际治疗时长</span><b>' + Math.round(L.actualSec / 60) + ' 分钟</b></div>' +
      '<div class="preset-line"><span>峰值强度</span><b>' + L.intensityPeak.toFixed(1) + ' mA</b></div>' +
      '<div class="preset-line"><span>平均接触阻抗</span><b>CH1 ' + L.zAvg1 + ' / CH2 ' + L.zAvg2 + ' kΩ</b></div>' +
      '<div class="note" style="margin-top:10px">建议记录治疗后的症状评分（恶心 / 腹胀 / 早饱），供医生评估疗效。</div>' +
      '<div class="acts"><button class="btn btn-ghost" onclick="closeModal()">稍后再说</button>' +
      '<button class="btn btn-primary" onclick="closeModal();switchView(\'feedback\')">记录治疗后感受</button></div>');
    toast('治疗结束提醒：本次处方治疗已完成', 'ok');
  });
}

function finalizeLog(status) {
  if (!rt.currentLog) return;
  const L = rt.currentLog;
  L.status = status; L.end = ts(new Date()); L.actualSec = rt.elapsed;
  L.intensityPeak = +rt.peak.toFixed(1);
  L.zAvg1 = rt.zN ? +(rt.zSum1 / rt.zN).toFixed(1) : +rt.z1.toFixed(1);
  L.zAvg2 = rt.zN ? +(rt.zSum2 / rt.zN).toFixed(1) : +rt.z2.toFixed(1);
  L.events = rt.events.slice();
  logs.unshift(L); if (logs.length > 300) logs.length = 300;
  store.save('ges_logs', logs);
  rt.currentLog = null;
}

/* ---------- 强度微调（患者权限内） ---------- */
function adjIntensity(d) {
  if (rt.state === 'fault') { toast('故障状态下不可调节强度', 'warn'); return; }
  const v = clamp(+(rt.setIntensity + d).toFixed(1), 0, settings.maxCurrent);
  if (v === rt.setIntensity) { toast(d > 0 ? '已达医生设定的强度上限 ' + settings.maxCurrent + ' mA' : '强度已为最低值 0', 'warn'); return; }
  rt.setIntensity = v;
  addEvent('强度调整 → ' + v.toFixed(1) + ' mA');
  if (rt.state === 'running') startRamp(true, () => {});   // 平滑过渡到新目标
  renderIntensity();
}
$('#btnMinus').addEventListener('click', () => adjIntensity(-0.5));
$('#btnPlus').addEventListener('click', () => adjIntensity(0.5));

/* ---------- 电极贴放引导 + 预检（治疗启动流程） ---------- */
$('#btnStart').addEventListener('click', () => {
  openModal(
    '<h2>第 1 步 · 电极贴放位置引导</h2>' +
    '<div class="guide-flex" style="display:flex;gap:20px;align-items:flex-start">' +
    '<svg width="230" height="270" viewBox="0 0 300 340" style="flex:none;background:#f8fafc;border:1px solid var(--line);border-radius:12px">' +
    '<path d="M85 40 Q150 18 215 40 L228 60 Q252 78 250 120 L238 300 Q236 318 218 318 L82 318 Q64 318 62 300 L50 120 Q48 78 72 60 Z" fill="#fde8dd" stroke="#e8b99a" stroke-width="2"/>' +
    '<path d="M120 34 Q150 62 180 34" fill="none" stroke="#d9a184" stroke-width="2"/>' +
    '<path d="M138 92 L150 104 L162 92" fill="none" stroke="#b0763f" stroke-width="3"/>' +
    '<text x="150" y="84" font-size="11" fill="#92400e" text-anchor="middle">剑突</text>' +
    '<circle cx="150" cy="205" r="6" fill="none" stroke="#b0763f" stroke-width="2"/>' +
    '<text x="150" y="228" font-size="11" fill="#92400e" text-anchor="middle">脐</text>' +
    '<rect x="132" y="112" width="36" height="20" rx="6" fill="#0e7490"/>' +
    '<rect x="76" y="138" width="36" height="20" rx="6" fill="#0e7490"/>' +
    '<rect x="188" y="140" width="36" height="20" rx="6" fill="#14b8a6"/>' +
    '<rect x="132" y="238" width="36" height="20" rx="6" fill="#14b8a6"/>' +
    '<line x1="150" y1="122" x2="94" y2="148" stroke="#0e7490" stroke-width="2.5" stroke-dasharray="5 4"/>' +
    '<line x1="206" y1="150" x2="150" y2="248" stroke="#14b8a6" stroke-width="2.5" stroke-dasharray="5 4"/>' +
    '<text x="150" y="108" font-size="10" fill="#fff" text-anchor="middle" font-weight="bold">CH1-A</text>' +
    '<text x="94" y="152" font-size="10" fill="#fff" text-anchor="middle" font-weight="bold">CH1-B</text>' +
    '<text x="206" y="154" font-size="10" fill="#fff" text-anchor="middle" font-weight="bold">CH2-A</text>' +
    '<text x="150" y="252" font-size="10" fill="#fff" text-anchor="middle" font-weight="bold">CH2-B</text>' +
    '</svg>' +
    '<div style="font-size:13.5px;line-height:1.9;color:#334155">' +
    '<b>CH1（深青）</b>：剑突下 2~3 cm → 左上腹胃体投影区<br>' +
    '<b>CH2（青绿）</b>：右上腹胃窦投影区 → 脐上 3 cm<br>' +
    '· 同通道两电极间距 10–15 cm，避开骨突与疤痕<br>' +
    '· 贴前清洁皮肤，电极凝胶面完全贴合、无翘边<br>' +
    '· 贴放过程中设备保持输出 0 mA（故障安全）' +
    '</div></div>' +
    '<label class="ck-row"><input type="checkbox" id="ckPlaced">' +
    '<span>我已按图示完成电极贴放，电极与皮肤贴合牢固，粘贴处无破损。我理解治疗中感到刺痛、灼热或不适时应立即暂停/停止治疗。</span></label>' +
    '<div class="acts"><button class="btn btn-ghost" onclick="closeModal()">取消</button>' +
    '<button class="btn btn-primary" id="btnPrecheck" disabled>开始预检（2 秒）</button></div>' +
    '<div class="note" id="precheckMsg">预检将测量两对电极的接触阻抗，通过后方可软启动输出。</div>'
  );
  $('#ckPlaced').addEventListener('change', function () { $('#btnPrecheck').disabled = !this.checked; });
  $('#btnPrecheck').addEventListener('click', function () {
    this.disabled = true; this.textContent = '正在检测阻抗…';
    $('#precheckMsg').innerHTML = '检测中，请勿移动身体…';
    setTimeout(() => {
      const bad = [];
      if (zBad(rt.z1)) bad.push('CH1（' + rt.z1.toFixed(1) + ' kΩ）');
      if (zBad(rt.z2)) bad.push('CH2（' + rt.z2.toFixed(1) + ' kΩ）');
      if (bad.length) {
        this.disabled = false; this.textContent = '重新预检';
        $('#precheckMsg').innerHTML = '<span style="color:var(--danger);font-weight:700">预检未通过：</span>' + bad.join('、') + ' 接触阻抗异常。请重新贴合电极或湿润皮肤后重试。';
      } else {
        closeModal();
        toast('预检通过（CH1 ' + rt.z1.toFixed(1) + ' / CH2 ' + rt.z2.toFixed(1) + ' kΩ），开始软启动', 'ok');
        startSession();
      }
    }, 2000);
  });
});
$('#btnPause').addEventListener('click', pauseSession);
$('#btnResume').addEventListener('click', resumeSession);
$('#btnStop').addEventListener('click', stopSession);

/* ---------- 主循环（秒级） ---------- */
setInterval(() => {
  $('#clock').textContent = ts(new Date()).slice(11);
  if (rt.state === 'running') {
    rt.elapsed++; rt.remain--;
    if (rt.remain <= 0) completeSession(); else renderControls();
  }
}, 1000);

/* ---------- 波形引擎（Canvas） ---------- */
const cv = $('#waveCv'), ctx = cv.getContext('2d');
function drawWave(tms) {
  const dpr = window.devicePixelRatio || 1;
  const w = cv.clientWidth, h = cv.clientHeight;
  if (cv.width !== w * dpr) { cv.width = w * dpr; cv.height = h * dpr; }
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  ctx.clearRect(0, 0, w, h);
  ctx.strokeStyle = 'rgba(148,163,184,.18)'; ctx.lineWidth = 1;
  for (let x = 0; x < w; x += 36) { ctx.beginPath(); ctx.moveTo(x, 0); ctx.lineTo(x, h); ctx.stroke(); }
  ctx.beginPath(); ctx.moveTo(0, h / 2); ctx.lineTo(w, h / 2); ctx.stroke();

  const s = rt.state;
  let f;
  if (s === 'running' || s === 'paused' || s === 'rampup' || s === 'rampdown')
    f = clamp(rt.intensity / settings.maxCurrent, 0, 1);
  else if (s === 'fault') f = 0;
  else f = 0.3; // 待机预览
  const A = (h / 2 - 14) * (s === 'running' || s === 'rampup' || s === 'rampdown' ? Math.max(f, 0.06) : s === 'paused' ? Math.max(f, 0.06) : f);
  const t = tms / 1000, mid = h / 2;

  if (settings.mode === 'IFE') {
    const k = 2 * Math.PI * 14 / w, wBar = 2 * Math.PI * 1.1, dw = 2 * Math.PI * (settings.df / 18), ph = settings.phase * Math.PI / 180;
    // 拍频包络（虚线）
    ctx.setLineDash([5, 4]); ctx.strokeStyle = '#94a3b8'; ctx.lineWidth = 1.2;
    ctx.beginPath();
    for (let x = 0; x <= w; x += 2) {
      const env = Math.abs(Math.cos((dw * t - ph) / 2)) * A;
      ctx.lineTo(x, mid - env);
    }
    ctx.moveTo(0, mid);
    for (let x = 0; x <= w; x += 2) {
      const env = Math.abs(Math.cos((dw * t - ph) / 2)) * A;
      ctx.lineTo(x, mid + env);
    }
    ctx.stroke(); ctx.setLineDash([]);
    // 组织内干涉电流（两路叠加）
    ctx.strokeStyle = '#0e7490'; ctx.lineWidth = 2;
    ctx.beginPath();
    for (let x = 0; x <= w; x += 1.5) {
      const y1 = Math.sin(k * x - wBar * t);
      const y2 = Math.sin(k * x - (wBar + dw) * t + ph);
      const y = (y1 + y2) / 2 * A;
      ctx.lineTo(x, mid - y);
    }
    ctx.stroke();
  } else {
    // TENS 双相脉冲串（时间压缩显示）
    const fV = Math.max(0.4, settings.tensFreq / 40), T = 1 / fV;
    const frac = clamp(0.05 + (settings.pw - HARD.pwMin) / (HARD.pwMax - HARD.pwMin) * 0.35, 0.03, 0.4);
    ctx.strokeStyle = '#0e7490'; ctx.lineWidth = 2;
    ctx.beginPath();
    for (let x = 0; x <= w; x += 1) {
      const tt = (x / w * 3 * T + t * fV * T) % T;
      const p = tt / T;
      const y = p < frac ? A * 0.8 : (p < frac * 2 ? -A * 0.8 : 0);
      ctx.lineTo(x, mid - y);
    }
    ctx.stroke();
  }
}
(function loop(t) { drawWave(t); requestAnimationFrame(loop); })(0);
window.addEventListener('resize', () => drawWave(0));

/* ---------- 视图切换 ---------- */
rt.currentView = 'treatment';
function switchView(name) {
  rt.currentView = name;
  $$('.view').forEach(v => v.classList.remove('active'));
  $('#view-' + name).classList.add('active');
  $$('.nav-item').forEach(b => b.classList.toggle('active', b.dataset.view === name));
  if (name === 'records') renderRecords();
  if (name === 'feedback') renderVas();
  if (name === 'settings') renderSettings();
}
$$('.nav-item').forEach(b => b.addEventListener('click', () => switchView(b.dataset.view)));

/* ---------- VAS 反馈 ---------- */
[['vasN', 'vasNv'], ['vasB', 'vasBv'], ['vasE', 'vasEv']].forEach(([a, b]) => {
  $('#' + a).addEventListener('input', function () { $('#' + b).textContent = this.value; });
});
$('#btnSaveVas').addEventListener('click', () => {
  vasL.unshift({ time: ts(new Date()), phase: $('#vasPhase').value, nausea: +$('#vasN').value, bloating: +$('#vasB').value, early: +$('#vasE').value });
  if (vasL.length > 200) vasL.length = 200;
  store.save('ges_vas', vasL);
  addAudit(rt.role === 'doctor' ? '医生' : '患者', '保存症状评分', 'VAS', '', '恶心' + $('#vasN').value + '/腹胀' + $('#vasB').value + '/早饱' + $('#vasE').value + '（' + $('#vasPhase').value + '）');
  toast('评分已保存，可在记录页导出', 'ok');
  renderVas();
});
function renderVas() {
  const tb = $('#vasTable');
  if (!vasL.length) { tb.innerHTML = '<tr><td class="empty">暂无评分记录</td></tr>'; return; }
  tb.innerHTML = '<tr><th>时间</th><th>时机</th><th>恶心</th><th>腹胀</th><th>早饱</th></tr>' +
    vasL.slice(0, 12).map(v =>
      '<tr><td>' + v.time + '</td><td>' + v.phase + '</td><td>' + v.nausea + '</td><td>' + v.bloating + '</td><td>' + v.early + '</td></tr>').join('');
}

/* ---------- 记录 / 依从性 / 导出 ---------- */
function renderRecords() {
  const done = logs.filter(L => L.status === '完成');
  const inter = logs.filter(L => L.status === '中断');
  const totalMin = Math.round(logs.reduce((a, L) => a + (L.actualSec || 0), 0) / 60);
  $('#kpiRow').innerHTML =
    '<div class="kpi"><div class="n">' + logs.length + '</div><div class="t">累计治疗次数</div></div>' +
    '<div class="kpi"><div class="n">' + done.length + '<small> / ' + logs.length + '</small></div><div class="t">按处方完成</div></div>' +
    '<div class="kpi"><div class="n">' + totalMin + '<small> min</small></div><div class="t">累计治疗时长</div></div>' +
    '<div class="kpi"><div class="n">' + inter.length + '</div><div class="t">中断次数</div></div>';

  // 近7天柱状图
  let bars = '';
  for (let i = 6; i >= 0; i--) {
    const d = new Date(); d.setDate(d.getDate() - i);
    const key = todayStr(d);
    const n = logs.filter(L => L.date === key && L.status === '完成').length;
    const hgt = clamp(n / settings.dailyLimit * 100, 2, 100);
    bars += '<div class="bar-col"><span class="num">' + n + '/' + settings.dailyLimit + '</span>' +
      '<div class="bar" style="height:' + hgt + '%"></div><span class="lbl">' + (d.getMonth() + 1) + '/' + d.getDate() + '</span></div>';
  }
  $('#compBars').innerHTML = bars;

  const tb = $('#logTable');
  tb.classList.toggle('is-empty', !logs.length);
  if (!logs.length) { tb.innerHTML = '<tr><td class="empty">暂无治疗记录 · 完成一次治疗后将自动生成刺激日志</td></tr>'; return; }
  tb.innerHTML = '<tr><th>开始时间</th><th>状态</th><th>计划/实际</th><th>参数</th><th>峰值强度</th><th>平均阻抗 (kΩ)</th><th>事件</th></tr>' +
    logs.slice(0, 20).map(L =>
      '<tr><td>' + L.start + '</td>' +
      '<td class="' + (L.status === '完成' ? 'tag-ok' : 'tag-bad') + '">' + L.status + '</td>' +
      '<td>' + Math.round(L.plannedSec / 60) + ' min / ' + Math.round((L.actualSec || 0) / 60) + ' min</td>' +
      '<td>' + (L.params.group || '') + ' · ' + MODE_TXT[L.params.mode].slice(0, 3) + ' · Δf ' + L.params.df + ' Hz</td>' +
      '<td>' + (L.intensityPeak || 0) + ' mA</td>' +
      '<td>' + L.zAvg1 + ' / ' + L.zAvg2 + '</td>' +
      '<td title="' + (L.events || []).join('；') + '">' + (L.events ? L.events.length : 0) + ' 条</td></tr>').join('');
}

function download(name, content, mime) {
  const b = new Blob([content], { type: mime });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(b); a.download = name; a.click();
  URL.revokeObjectURL(a.href);
}
const csvCell = v => '"' + String(v === undefined || v === null ? '' : v).replace(/"/g, '""') + '"';
$('#btnExpCsv').addEventListener('click', () => {
  if (!logs.length) { toast('暂无记录可导出', 'warn'); return; }
  let s = '\ufeff序号,日期,开始时间,结束时间,状态,计划时长(min),实际时长(min),参数组,模式,差频(Hz),脉宽(μs),相位差(°),峰值强度(mA),CH1平均阻抗(kΩ),CH2平均阻抗(kΩ),事件\n';
  logs.forEach((L, i) => {
    s += [i + 1, L.date, L.start, L.end, L.status, Math.round(L.plannedSec / 60), Math.round((L.actualSec || 0) / 60),
      L.params.group, L.params.mode, L.params.df, L.params.pw, L.params.phase, L.intensityPeak, L.zAvg1, L.zAvg2,
      (L.events || []).join('；')].map(csvCell).join(',') + '\n';
  });
  download('GES治疗记录_' + todayStr() + '.csv', s, 'text/csv');
  addAudit(rt.role === 'doctor' ? '医生' : '患者', '数据导出', '治疗记录 CSV', '', logs.length + ' 条');
});
$('#btnExpJson').addEventListener('click', () => {
  download('GES全量备份_' + todayStr() + '.json',
    JSON.stringify({ exportedAt: ts(new Date()), settings, groups, logs, audit: auditL, vas: vasL }, null, 2), 'application/json');
  toast('已导出全量备份（参数、日志、审计、评分）', 'ok');
});
$('#btnExpAudit').addEventListener('click', () => {
  let s = '\ufeff时间,操作者,操作,对象,变更前,变更后\n';
  auditL.forEach(a => { s += [a.time, a.actor, a.action, a.target, a.old, a.new].map(csvCell).join(',') + '\n'; });
  download('GES审计追踪_' + todayStr() + '.csv', s, 'text/csv');
});

/* ---------- 设置（分级权限） ---------- */
function renderSettings() {
  const doc = rt.role === 'doctor';
  $('#patSettings').style.display = doc ? 'none' : '';
  $('#docSettings').style.display = doc ? '' : 'none';
  $('#roleBadge').textContent = doc ? '医生模式' : '患者模式';
  $('#roleBadge').className = 'badge ' + (doc ? 'doctor' : 'patient');
  if (!doc) { renderPreset(); return; }

  // 同步滑杆与数值
  Object.keys(PARAM_DEFS).forEach(k => {
    $$('[data-param="' + k + '"]').forEach(el => { el.value = settings[k]; });
    const v = $('#v-' + k); if (v) v.textContent = settings[k];
  });
  $('#swCal').checked = !!settings.autoCal;
  $('#swLock').checked = !!settings.locked;
  $('#tensFields').style.display = settings.mode === 'TENS' ? '' : 'none';
  $$('#modeSeg button').forEach(b => b.classList.toggle('on', b.dataset.m === settings.mode));
  const charge = settings.maxCurrent * settings.pw / 1000;
  $('#chargeTxt').textContent = charge.toFixed(1) + ' μC';
  renderGroups(); renderAuditTable(); renderPreset();
}

function applyParam(key, raw) {
  const def = PARAM_DEFS[key];
  let v = def.int ? Math.round(+raw) : +raw;
  if (isNaN(v)) { renderSettings(); return; }
  v = clamp(v, def.min, def.max);          // 硬限幅：超出监管范围的输入直接钳位
  if (v === settings[key]) { renderSettings(); return; }
  const old = settings[key];
  settings[key] = v;
  settings.activeGroup = '自定义（未保存）';
  saveSettings();
  addAudit('医生', '参数调整', def.label + ' (' + def.unit + ')', old, v);
  renderSettings();
  toast(def.label + '：' + old + ' → ' + v + ' ' + def.unit + '（已写入审计追踪）', 'ok');
}
$$('[data-param]').forEach(el => el.addEventListener('input', function () { applyParam(this.dataset.param, this.value); }));

$$('#modeSeg button').forEach(b => b.addEventListener('click', () => {
  if (settings.mode === b.dataset.m) return;
  const old = settings.mode;
  settings.mode = b.dataset.m; settings.activeGroup = '自定义（未保存）'; saveSettings();
  addAudit('医生', '模式切换', '输出模式', MODE_TXT[old], MODE_TXT[settings.mode]);
  renderSettings();
}));
$('#swCal').addEventListener('change', function () {
  const old = settings.autoCal; settings.autoCal = this.checked; saveSettings();
  addAudit('医生', '参数调整', '阻抗动态校准', old ? '开启' : '关闭', this.checked ? '开启' : '关闭');
  renderSettings();
});
$('#swLock').addEventListener('change', function () {
  settings.locked = this.checked; saveSettings();
  addAudit('医生', '处方锁定', '处方', this.checked ? '未锁定' : '已锁定', this.checked ? '已锁定' : '未锁定');
  toast(this.checked ? '处方已锁定：参数组不可切换或删除' : '处方锁定已解除', 'ok');
  renderGroups();
});

/* 参数组 */
function renderGroups() {
  const box = $('#groupList');
  box.innerHTML = Object.keys(groups).map(name => {
    const g = groups[name];
    return '<div class="group-item' + (name === settings.activeGroup ? ' cur' : '') + '">' +
      '<div class="gn">' + name + (name === settings.activeGroup ? '（当前）' : '') +
      '<small>' + MODE_TXT[g.mode] + ' · Δf ' + g.df + ' Hz · ' + g.sessionMin + ' min · 上限 ' + g.maxCurrent + ' mA</small></div>' +
      '<button class="btn btn-primary btn-sm" data-apply="' + name + '"' + (settings.locked ? ' disabled' : '') + '>应用</button>' +
      '<button class="btn btn-ghost btn-sm" data-del="' + name + '"' + (settings.locked || Object.keys(groups).length <= 1 ? ' disabled' : '') + '>删除</button></div>';
  }).join('') + (settings.locked ? '<div class="note warn">处方已锁定，如需调整参数组请先解除锁定。</div>' : '');
  $$('[data-apply]').forEach(b => b.addEventListener('click', () => applyGroup(b.dataset.apply)));
  $$('[data-del]').forEach(b => b.addEventListener('click', () => delGroup(b.dataset.del)));
}
function applyGroup(name) {
  if (settings.locked) { toast('处方已锁定', 'warn'); return; }
  const old = settings.activeGroup;
  Object.assign(settings, groups[name], { activeGroup: name });
  saveSettings();
  addAudit('医生', '参数组应用', '参数组', old, name);
  toast('已应用参数组「' + name + '」', 'ok');
  renderSettings();
}
function delGroup(name) {
  if (settings.locked) { toast('处方已锁定', 'warn'); return; }
  if (Object.keys(groups).length <= 1) { toast('至少保留一个参数组', 'warn'); return; }
  delete groups[name]; saveGroups();
  addAudit('医生', '参数组删除', '参数组', name, '（已删除）');
  renderGroups();
}
$('#btnSaveGroup').addEventListener('click', () => {
  const name = $('#groupName').value.trim();
  if (!name) { toast('请输入参数组名称', 'warn'); return; }
  groups[name] = { mode: settings.mode, df: settings.df, tensFreq: settings.tensFreq, pw: settings.pw, phase: settings.phase,
    maxCurrent: settings.maxCurrent, sessionMin: settings.sessionMin, dailyLimit: settings.dailyLimit, autoCal: settings.autoCal };
  settings.activeGroup = name; saveSettings(); saveGroups();
  $('#groupName').value = '';
  addAudit('医生', '参数组保存', '参数组', '', name);
  toast('参数组「' + name + '」已保存', 'ok');
  renderGroups();
});

/* 医生登录（3 次失败锁定 30 秒） */
let loginTries = 0, lockUntil = 0;
$('#btnDocLogin').addEventListener('click', () => {
  if (Date.now() < lockUntil) { toast('已临时锁定，请 ' + Math.ceil((lockUntil - Date.now()) / 1000) + ' 秒后重试', 'warn'); return; }
  openModal('<h2>医生模式登录</h2>' +
    '<div class="field"><label>操作口令</label><input type="password" id="docPwd" placeholder="请输入医生口令（原型默认：1234）" autofocus></div>' +
    '<div class="note warn">连续输错 3 次将临时锁定 30 秒。登录及后续参数修改均写入审计追踪。</div>' +
    '<div class="acts"><button class="btn btn-ghost" onclick="closeModal()">取消</button>' +
    '<button class="btn btn-primary" id="btnDoLogin">登录</button></div>');
  $('#btnDoLogin').addEventListener('click', doLogin);
  $('#docPwd').addEventListener('keydown', e => { if (e.key === 'Enter') doLogin(); });
  setTimeout(() => $('#docPwd').focus(), 60);
});
function doLogin() {
  const v = $('#docPwd').value;
  if (v === '1234') {
    loginTries = 0;
    rt.role = 'doctor';
    addAudit('医生', '医生登录', '权限', '患者模式', '医生模式');
    closeModal(); toast('已进入医生模式，核心参数解锁', 'ok');
    renderSettings();
  } else {
    loginTries++;
    addAudit('未知', '登录失败', '医生口令', '', '第 ' + loginTries + ' 次错误');
    if (loginTries >= 3) { lockUntil = Date.now() + 30000; loginTries = 0; closeModal(); toast('口令错误 3 次，医生登录已临时锁定 30 秒', 'err'); }
    else toast('口令错误，还可尝试 ' + (3 - loginTries) + ' 次', 'err');
  }
}
$('#btnDocLogout').addEventListener('click', () => {
  addAudit('医生', '退出登录', '权限', '医生模式', '患者模式');
  rt.role = 'patient';
  toast('已退出医生模式', 'ok');
  renderSettings();
});

/* 演示工具：模拟电极异常 */
$('#simNormal').addEventListener('click', () => { rt.sim = 'normal'; toast('电极状态恢复正常', 'ok'); });
$('#simOpen').addEventListener('click', () => { rt.sim = 'open'; addAudit('医生', '演示模拟', '电极接触', '正常', '阻抗过高'); toast('已模拟电极脱落，若在输出中将触发联锁', 'warn'); });
$('#simShort').addEventListener('click', () => { rt.sim = 'short'; addAudit('医生', '演示模拟', '电极接触', '正常', '阻抗过低'); toast('已模拟电极短路，若在输出中将触发联锁', 'warn'); });

/* 审计表 */
function renderAuditTable() {
  const tb = $('#auditTable');
  tb.classList.toggle('is-empty', !auditL.length);
  if (!auditL.length) { tb.innerHTML = '<tr><td class="empty">暂无审计记录</td></tr>'; return; }
  tb.innerHTML = '<tr><th>时间</th><th>操作者</th><th>操作</th><th>对象</th><th>变更前</th><th>变更后</th></tr>' +
    auditL.slice(0, 30).map(a =>
      '<tr><td>' + a.time + '</td><td>' + a.actor + '</td><td>' + a.action + '</td><td>' + a.target + '</td><td>' + a.old + '</td><td>' + a.new + '</td></tr>').join('');
}

/* ---------- 初始化（故障安全：上电输出为 0） ---------- */
(function init() {
  rt.intensity = 0;
  renderStatus(); renderControls(); renderChannels(); renderPreset();
  $('#clock').textContent = ts(new Date()).slice(11);
  addAudit('系统', '上电自检', '故障安全', '', '复位完成，输出 0 mA');
})();

/* ---------- PWA：离线缓存与“安装到设备” ---------- */
let installPrompt = null;
const btnInstall = $('#btnInstall');

window.addEventListener('beforeinstallprompt', e => {
  e.preventDefault();
  installPrompt = e;
  btnInstall.classList.add('show');
});

btnInstall.addEventListener('click', async () => {
  if (!installPrompt) {
    openModal('<h2>安装到手机 / 电脑</h2>' +
      '<div class="note" style="line-height:1.9">' +
      '<b>Android（Chrome / Edge）</b>：右上角菜单 → “安装应用”或“添加到主屏幕”。<br>' +
      '<b>iPhone（Safari）</b>：底部“分享”按钮 → “添加到主屏幕”。<br>' +
      '<b>Windows（Edge / Chrome）</b>：地址栏右侧的“安装”图标，或菜单 → “应用” → “安装此站点”。<br><br>' +
      '安装后可从桌面图标直接启动，并以独立窗口运行。</div>' +
      '<div class="acts"><button class="btn btn-primary" onclick="closeModal()">知道了</button></div>');
    return;
  }
  installPrompt.prompt();
  const choice = await installPrompt.userChoice;
  if (choice.outcome === 'accepted') btnInstall.classList.remove('show');
  installPrompt = null;
});

window.addEventListener('appinstalled', () => {
  btnInstall.classList.remove('show');
  toast('已安装到设备，可从桌面图标直接启动', 'ok');
});

if ('serviceWorker' in navigator && location.protocol.indexOf('http') === 0) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('./sw.js').catch(() => {});
  });
} else if (location.protocol === 'file:') {
  // 直接双击打开时提示使用本地服务方式，才能安装为应用
  console.info('当前以文件方式打开；运行“启动应用.bat”后可安装为桌面/手机应用。');
}
</script>
</body>
</html>
/* GES-100 PWA · 离线缓存 */
const CACHE = 'ges-pwa-v2';
const ASSETS = [
  './index.html',
  './manifest.webmanifest',
  './icons/icon-192.png',
  './icons/icon-512.png',
  './icons/icon-maskable-512.png'
];

self.addEventListener('install', e => {
  e.waitUntil(caches.open(CACHE).then(c => c.addAll(ASSETS)).then(() => self.skipWaiting()));
});

self.addEventListener('activate', e => {
  e.waitUntil(
    caches.keys()
      .then(keys => Promise.all(keys.filter(k => k !== CACHE).map(k => caches.delete(k))))
      .then(() => self.clients.claim())
  );
});

self.addEventListener('fetch', e => {
  const req = e.request;
  if (req.method !== 'GET' || new URL(req.url).origin !== location.origin) return;

  // 页面导航：优先网络，断网时回退缓存
  if (req.mode === 'navigate') {
    e.respondWith(
      fetch(req)
        .then(res => {
          const copy = res.clone();
          caches.open(CACHE).then(c => c.put(req, copy));
          return res;
        })
        .catch(() => caches.match('./index.html'))
    );
    return;
  }

  // 静态资源：缓存优先
  e.respondWith(caches.match(req).then(hit => hit || fetch(req)));
});

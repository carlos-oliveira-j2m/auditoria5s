<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025 Enterprise</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --yellow: #ffc107; --green: #10b981; }
    *{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}
    body { margin:0; background-color: #f0f2f5; position: relative; }
    body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.9); z-index: -1; }
    header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }
    .screen { display:none; padding:20px; max-width: 1200px; margin: auto; }
    .active { display:block; animation: fadeIn 0.4s; }
    .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); border-left: 6px solid var(--primary); margin-bottom: 20px; }
    .kpi-container { display: flex; gap: 20px; margin-bottom: 20px; flex-wrap: wrap; }
    .kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; text-align: center; border-bottom: 5px solid var(--primary); }
    .kpi-card h2 { margin: 0; font-size: 35px; color: var(--primary); }
    button { border: none; padding: 12px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; }
    .btn-primary { background: var(--primary); width: 100%; }
    .btn-pdf { background: var(--green); }
    .btn-edit { background: var(--yellow); color: #000; padding: 5px 10px; font-size: 11px; margin-right: 5px; }
    .btn-del { background: var(--red); padding: 5px 10px; font-size: 11px; }
    select, input, textarea { width: 100%; padding: 12px; border-radius: 4px; border: 1px solid #ccc; margin: 8px 0; font-size: 14px; }
    .teia-box { min-height: 700px; width: 100%; display: flex; justify-content: center; align-items: center; padding: 20px; }
    .tabela-historico { width: 100%; border-collapse: collapse; margin-top: 20px; background: white; }
    .tabela-historico th, .tabela-historico td { border: 1px solid #ddd; padding: 12px; text-align: left; }
    .pergunta-label { font-weight: bold; color: var(--secondary); display: block; margin-top: 15px; }
    @media print { .no-print { display:none !important; } .screen { display:block; } body::before { background: white; } }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active">
    <div class="card">
        <h2 id="tituloHome">Nova Auditoria</h2>
        <label>Setor:</label>
        <select id="setor">
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="ENG. SOLVE">ENG. SOLVE</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MANUTENÇÃO">MANUTENÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">

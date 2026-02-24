<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>J2M 5S Enterprise - v1.2025</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --p: #f06639; --s: #3d3d3d; --bg: #f4f7f6; --err: #dc3545; }
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { margin: 0; background: var(--bg); font-size: 16px; }
        header { background: var(--s); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--p); font-weight: bold; }
        .container { max-width: 900px; margin: auto; padding: 15px; }
        .screen { display: none; }
        .active { display: block; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 15px; border-left: 5px solid var(--p); }
        label { display: block; font-weight: bold; margin: 10px 0 5px; }
        .req::after { content: " *"; color: var(--err); }
        input, select, textarea { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 5px; font-size: 16px; margin-bottom: 10px; }
        button { cursor: pointer; border: none; padding: 15px; border-radius: 5px; font-weight: bold; width: 100%; margin-bottom: 10px; transition: 0.2s; }
        .btn-p { background: var(--p); color: white; }
        .btn-s { background: var(--s); color: white; }
        .btn-sync { background: #007bff; color: white; }
        .q-row { border-bottom: 1px solid #eee; padding: 15px 0; }
        .q-text { display: block; margin-bottom: 8px; font-weight: 600; color: #444; }
        select.nota-box { border: 2px solid var(--p); background: #fff9f7; }
        @media print { .no-print { display: none !important; } .screen { display: block !important; } }
    </style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - v1.2025</header>

<div class="container">
    <div id="scr_home" class="screen active">
        <div class="card">
            <h3>Identificação Obrigatória</h3>
            <label class="req">Setor:</label>
            <select id="setor" required>
                <option value="">-- SELECIONE --</option>
                <option value="ADM Filial">ADM Filial</option>
                <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
                <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
                <option value="INJEÇÃO">INJEÇÃO</option>
                <option value="MATRIZARIA">MATRIZARIA</option>
                <option value="MONTAGEM">MONTAGEM</option>
                <option value="PPCP">PPCP</option>
                <option value="QUALIDADE">QUALIDADE</option>
                <option value="SALA DE ELETRONICOS">SALA DE ELETRONICOS</option>
            </select>
            <label class="req">Responsável da Área:</label><input type="text" id="resp" placeholder="Nome do Responsável">
            <label class="req">Auditor:</label><input type="text" id="aud" placeholder="Nome do Auditor">
            <label class="req">Data:</label><input type="date" id="dt">
            
            <button class="btn-p" onclick="iniciar()">INICIAR AVALIAÇÃO</button>
            <button class="

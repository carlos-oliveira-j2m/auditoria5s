<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025 Enterprise</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>

<style>
:root { --primary: #f06639; --secondary: #5d5a51; --green: #28a745; --blue: #007bff; --danger: #dc3545; }
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}

body { 
    margin:0; min-height: 100vh; background-color: #f0f2f5; background-size: cover;
    background-position: center; background-attachment: fixed; transition: background-image 1.5s ease-in-out;
    position: relative;
}
body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.9); z-index: -1; }

header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }
.screen { display:none; padding:20px; max-width: 1150px; margin: 10px auto; }
.active { display:block; animation: fadeIn 0.4s; }

.card { background: white; padding: 25px; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 15px; border-left: 5px solid var(--primary); box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.pergunta-item { margin-bottom: 20px; padding-bottom: 15px; border-bottom: 1px solid #eee; }
.pergunta-item label { font-weight: bold; display: block; margin-bottom: 5px; }
.pergunta-desc { font-size: 13px; color: #666; font-style: italic; display: block; margin-bottom: 8px; }

.filter-bar { display: flex; gap: 10px; background: #fff; padding: 15px; border-radius: 8px; margin-bottom: 20px; flex-wrap: wrap; border: 1px solid #ddd; }
.kpi-container { display: flex; gap: 20px; margin-bottom: 20px; }
.kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; text-align: center; border-bottom: 5px solid var(--primary); }
.kpi-card h2 { margin: 0; font-size: 36px; color: var(--primary); }

button { border: none; padding: 12px 25px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.2s; }
.btn-primary { background: var(--primary); width: 100%; }
.btn-blue { background: var(--blue); width: 100%; margin-bottom: 10px; }

select, input, textarea { width: 100%; padding: 12px; border-radius: 4px; border: 1px solid #ccc; margin-bottom: 10px; }
.invalid { border: 2px solid var(--danger) !important; background: #fff1f1; }

.plano-box { background: #fff8e1; border: 1px solid #ffe082; padding: 15px; border-radius: 5px; margin-top: 10px; border-left: 5px solid #ffb300; }
.tabela-pdf { width: 100%; border-collapse: collapse; margin-top: 20px; background: white; }
.tabela-pdf th, .tabela-pdf td { border: 1px solid #999; padding: 10px; text-align: left; font-size: 13px; }

@media print { .no-print { display:none !important; } .screen { display:block !important; } body::before { background: white; } }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body id="bgBody">

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active">
    <div class="card">
        <h3>Identificação da Auditoria</h3>
        <label>Setor:</label>
        <select id="setor">
            <option value="">-- Selecione o Setor --</option>
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="QUALIDADE">QUALIDADE</option>
            <option value="MANUTENÇÃO">MANUTENÇÃO</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor" placeholder="Nome do Auditor">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-primary" onclick="iniciar()">COMEÇAR AVALIAÇÃO</button>
        <button class="btn-blue" style="margin-top:10px" onclick="sincronizar()">🔄 SINCRONIZAR COM PLANILHA</button>
        <button class="btn" style="background:var(--secondary); width:100%" onclick="abrirDash()">DASHBOARD / HISTÓRICO</button>
    </div>
</div>

<div id="senso" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="no-print">
        <div class="filter-bar">
            <div style="flex:1"><label>Filtrar Setor</label><select id="fSetor" onchange="renderRelatorio()"></select></div>
            <div style="flex:1">

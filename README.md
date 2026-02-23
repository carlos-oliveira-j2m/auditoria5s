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

.filter-bar { display: flex; gap: 10px; background: #fff; padding: 15px; border-radius: 8px; margin-bottom: 20px; flex-wrap: wrap; border: 1px solid #ddd; }
.filter-item { flex: 1; min-width: 150px; }

.kpi-container { display: flex; gap: 20px; margin-bottom: 20px; }
.kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; text-align: center; border-bottom: 5px solid var(--primary); }
.kpi-card h2 { margin: 0; font-size: 36px; color: var(--primary); }

button { border: none; padding: 12px 25px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; }
.btn-primary { background: var(--primary); width: 100%; }
.btn-blue { background: var(--blue); width: 100%; margin-bottom: 10px; }

select, input, textarea { width: 100%; padding: 12px; border-radius: 4px; border: 1px solid #ccc; margin-bottom: 10px; font-size: 14px; }

/* Estilo para Plano de Ação em destaque no relatório */
.plano-acao-box { background: #fff8e1; border: 1px solid #ffe082; padding: 15px; border-radius: 5px; margin-top: 10px; border-left: 5px solid #ffb300; }

.tabela-pdf { width: 100%; border-collapse: collapse; margin-top: 20px; }
.tabela-pdf th, .tabela-pdf td { border: 1px solid #999; padding: 10px; text-align: left; font-size: 13px; }
.tabela-pdf th { background: #f4f4f4; }

@media print { 
    .no-print { display:none !important; } 
    .screen { display:block !important; }
    body::before { background: white; }
    .card { page-break-inside: avoid; box-shadow: none; border: 1px solid #ccc; }
}
</style>
</head>
<body id="bgBody">

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active">
    <div class="card">
        <h3>Identificação</h3>
        <label>Setor:</label>
        <select id="setor">
            <option value="">Selecione...</option>
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-primary" onclick="start()">INICIAR AUDITORIA</button>
        <button class="btn-blue" style="margin-top:10px" onclick="sync()">🔄 SINCRONIZAR</button>
        <button class="btn" style="background:var(--secondary); width:100%" onclick="showDash()">RELATÓRIOS E FILTROS</button>
    </div>
</div>

<div id="senso" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="no-print">
        <div class="filter-bar">
            <div class="filter-item"><label>Setor</label><select id="fSetor" onchange="renderRelatorio()"></select></div>
            <div class="filter-item"><label>Mês</label><select id="fMes" onchange="renderRelatorio()">
                <option value="TODOS">Todos</option><option value="0">Jan</option><option value="1">Fev</option><option value="2">Mar</option>
            </select></div>
            <div class="filter-item"><label>Ano</label><select id="fAno" onchange="renderRelatorio()"><option value="2025">2025</option></select></div>
            <button class="btn-blue" onclick="window.print()">GERAR PDF</button>
            <button class="btn" style="background:#666" onclick="location.reload()">VOLTAR</button>
        </div>
    </div>

    <div class="kpi-container">
        <div class="kpi-card"><h2 id="mediaGeral">0.0</h2><p>MÉDIA DA FÁBRICA</p></div>
        <div class="kpi-card"><h2 id="totalAudits">0</h2><p>AUDITORIAS REALIZADAS</p></div>
    </div>

    <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(450px, 1fr)); gap: 20px;">
        <div class="card"><h3>Radar: Setor vs Média vs Meta</h3><canvas id="cRadar"></canvas></div>
        <div class="card"><h3>Comparativo entre Áreas</h3><canvas id="cBarra"></canvas></div>
    </div>

    <div id="resultadoFinal"></div>
</div>

<script>
const API = "https://script.google.com/macros/s/AKfycbzcntNB7ErwLkye0Y7kAoqneoeMAs_MMe7YvszVdHGrHJUGacpixxAYV9LDppBlUoNx/exec";

const checklist = [
    { s: "SELEÇÃO", p: ["Ferramentas necessárias?", "Itens desnecessários?", "Materiais obsoletos?", "Documentação atual?"] },
    { s: "ORDENAÇÃO", p: ["Identificação de locais?", "Demarcação de solo?", "Acesso rápido?"] },
    { s: "LIMPEZA", p: ["Máquinas limpas?", "Ausência de vazamentos?", "Piso e cantos?"] },
    { s: "PADRONIZAÇÃO", p: ["Uso de EPIs?", "Gestão visual?", "Sinalização?"] },
    { s: "DISCIPLINA", p: ["Rotina 5S mantida?", "Melhorias propostas?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, sensoIdx = 0;
let chartR, chartB;

function start() {
    if(!document.getElementById('setor').value) return alert("Selecione o setor!");
    audit = { id: Date.now(), setor: document.getElementById('setor').value, auditor: document.getElementById('auditor').value, data: document.getElementById('data').value, respostas: [] };
    sensoIdx = 0; renderSenso();
}

function renderSenso() {
    const s = checklist[sensoIdx];
    let h = `<div class="card"><h2>${sensoIdx+1}º Senso: ${s.s}</h2>`;
    s.p.forEach((p, i) => {
        h += `<div class="pergunta-item"><label>${p}</label>
              <select class="nota-q"><option value="">ESCOLHA A NOTA...</option>
              <option value="10">10 - Excelente</option><option value="8">8 - Bom</option>
              <option value="6">6 - Regular</option><option value="4">4 - Ruim</option><option value="2">2 - Crítico</option></select></div>`;
    });
    h += `<label><b>Evidências Encontradas (O que você viu?):</b></label><textarea id="evidencia" placeholder="Ex: Vazamento na prensa, ferramentas no chão..."></textarea>
          <label><b>Plano de Ação:</b></label><textarea id="plano" placeholder="O que será feito para corrigir?"></textarea>
          <button class="btn-primary" onclick="saveSenso()">PRÓXIMO SENSO</button></div>`;
    document.getElementById('senso').innerHTML = h;
    show('senso');
}

function saveSenso() {
    const selects = document.querySelectorAll('.nota-q');
    let notas = [];
    for(let s of selects) {
        if(!s.value) return alert("Por favor, preencha TODAS as notas antes de continuar!");
        notas.push(Number(s.value));
    }
    
    const media = (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1);
    const plano = document.getElementById('plano').value;
    const evid = document.getElementById('evidencia').value;

    // Se nota < 6, plano de ação é obrigatório
    if(media < 6 && plano.length < 5) return alert("Para notas abaixo de 6, o PLANO DE AÇÃO é obrigatório!");

    audit.respostas.push({ media, plano, evid });
    sensoIdx++;
    if(sensoIdx < checklist.length) renderSenso();
    else {
        db.push(audit);
        localStorage.setItem("j2m_db", JSON.stringify(db));
        fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
        alert("Salvo com sucesso!");
        showDash();
    }
}

function showDash() {
    const sel = document.getElementById("fSetor");
    sel.innerHTML = '<option value="TODOS">Todos os Setores</option>';
    [...new Set(db.map(a => a.setor))].forEach(s => sel.innerHTML += `<option value="${s}">${s}</option>`);
    renderRelatorio();
    show("dashboard");
}

function renderRelatorio() {
    const fS = document.getElementById("fSetor").value;
    const fM = document.getElementById("fMes").value;
    
    const filtrados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return (fS === "TODOS" || a.setor === fS) && (fM === "TODOS" || d.getMonth() == fM);
    });

    if(!filtrados.length) return;
    const ult = filtrados[filtrados.length - 1];

    document.getElementById('totalAudits').innerText = db.length;
    let somaF = db.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById('mediaGeral').innerText = (somaF / db.length).toFixed(1);

    // Radar Comparativo
    if(chartR) chartR.destroy();
    const mediaFabricaSensos = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / db.length).toFixed(1));
    
    chartR = new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets:

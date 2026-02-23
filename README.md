<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025 Enterprise</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
<style>
:root { --primary: #f06639; --secondary: #5d5a51; --green: #28a745; --blue: #007bff; }
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}
body { margin:0; min-height: 100vh; background-color: #f0f2f5; background-size: cover; background-position: center; background-attachment: fixed; transition: background-image 1.5s ease-in-out; position: relative; z-index: 1;}
body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.85); z-index: -1; }
header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }
.screen { display:none; padding:20px; max-width: 1150px; margin: 10px auto; }
.active { display:block; animation: fadeIn 0.4s; }
.card { background: rgba(255,255,255,0.95); padding: 15px; border: 1px solid #ddd; border-radius: 6px; margin-bottom: 15px; border-left: 5px solid var(--primary); box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
.kpi-container { display: flex; gap: 20px; margin-bottom: 20px; flex-wrap: wrap; }
.kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; min-width: 200px; text-align: center; border-bottom: 5px solid var(--primary); }
.kpi-card h2 { margin: 0; font-size: 32px; color: var(--primary); }
button { border: none; padding: 10px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.2s; }
.btn-primary { background: var(--primary); width: 100%; margin-top: 10px; }
.btn-secondary { background: var(--secondary); width: 100%; margin-top: 10px; }
select, input, textarea { width: 100%; padding: 10px; border-radius: 4px; border: 1px solid #ccc; margin-bottom: 10px; }
.tabela-pdf { width: 100%; border-collapse: collapse; margin-top: 10px; background: white; }
.tabela-pdf th, .tabela-pdf td { border: 1px solid #999; padding: 8px; text-align: left; font-size: 13px; }
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
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="ENG. SOLVE">ENG. SOLVE</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="QUALIDADE">QUALIDADE</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-primary" onclick="start()">Iniciar Avaliação</button>
        <button class="btn-secondary" onclick="openDashboard()">Dashboard / Histórico</button>
    </div>
</div>

<div id="senso" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="kpi-container">
        <div class="kpi-card"><h2 id="mediaGeralFabrica">0.0</h2><p>Média Geral Fábrica</p></div>
        <div class="kpi-card"><h2 id="totalAuditorias">0</h2><p>Total de Auditorias</p></div>
    </div>
    <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 20px;">
        <div class="card"><canvas id="cRadar"></canvas></div>
        <div class="card"><canvas id="cBarra"></canvas></div>
    </div>
    <div id="relatorioView"></div>
    <button class="btn-secondary" onclick="show('home')">Voltar</button>
</div>

<script>
const checklist = [
    { senso: "Utilização", perguntas: ["Itens desnecessários na área?", "Estoque excessivo?", "Documentos atualizados?", "Ferramentas acessíveis?", "Descarte correto?"] },
    { senso: "Organização", perguntas: ["Local fixo identificado?", "Piso demarcado?", "Busca rápida (<30s)?", "Armários organizados?", "Fluxo facilitado?"] },
    { senso: "Limpeza", perguntas: ["Chão/Máquinas limpos?", "Causas de sujeira?", "Materiais disponíveis?", "Lixeiras corretas?", "Equipamentos conservados?"] },
    { senso: "Padronização", perguntas: ["Sinalização clara?", "Cores padrão?", "Uso de EPIs?", "Ambiente higienizado?", "Padrões seguidos?"] },
    { senso: "Disciplina", perguntas: ["Rotinas seguidas?", "Checklists preenchidos?", "Melhoria contínua?", "Prazos cumpridos?", "Equipe engajada?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, sensoIndex = 0;
let chartRadar, chartBar;

function show(id) { document.querySelectorAll(".screen").forEach(s => s.classList.remove("active")); document.getElementById(id).classList.add("active"); }

function start() {
    audit = { id: Date.now(), setor: setor.value, auditor: auditor.value, data: data.value, respostas: [] };
    sensoIndex = 0; renderSenso(); show("senso");
}

function renderSenso() {
    const s = checklist[sensoIndex];
    let html = `<div class="card"><h2>${s.senso}</h2>`;
    s.perguntas.forEach((p, i) => {
        html += `<label>${p}</label><select id="n${i}"><option value="10">Excelente (10)</option><option value="8">Bom (8)</option><option value="5">Regular (5)</option><option value="0">Péssimo (0)</option></select>`;
    });
    html += `<label>Plano de Ação:</label><textarea id="plano"></textarea><button class="btn-primary" onclick="salvarSenso()">${sensoIndex < 4 ? 'Próximo' : 'Finalizar'}</button></div>`;
    document.getElementById("senso").innerHTML = html;
}

function salvarSenso() {
    let notas = [];
    for(let i=0; i<5; i++) notas.push(Number(document.getElementById("n"+i).value));
    audit.respostas.push({ media: (notas.reduce((a,b)=>a+b)/5).toFixed(1), plano: plano.value });
    sensoIndex++;
    if(sensoIndex < 5) renderSenso();
    else { db.push(audit); localStorage.setItem("j2m_db", JSON.stringify(db)); openDashboard(); }
}

function openDashboard() {
    renderRelatorio(); show("dashboard");
}

function renderRelatorio() {
    let somaTotal = 0;
    db.forEach(a => somaTotal += a.respostas.reduce((acc, r) => acc + Number(r.media), 0) / 5);
    mediaGeralFabrica.innerText = db.length > 0 ? (somaTotal / db.length).toFixed(1) : "0.0";
    totalAuditorias.innerText = db.length;

    let html = "<h3>Histórico</h3><table class='tabela-pdf'><tr><th>Data</th><th>Setor</th><th>Nota</th></tr>";
    db.forEach(a => {
        const notaF = (a.respostas.reduce((acc,r)=>acc+Number(r.media),0)/5).toFixed(1);
        html += `<tr><td>${a.data}</td><td>${a.setor}</td><td><b>${notaF}</b></td></tr>`;
    });
    relatorioView.innerHTML = html + "</table>";
    atualizarGraficos();
}

function atualizarGraficos() {
    if(chartRadar) chartRadar.destroy();
    const mSensos = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / db.length).toFixed(1));
    chartRadar = new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Utilização", "Organização", "Limpeza", "Padronização", "Disciplina"],
            datasets: [{ label: 'Média Atual', data: mSensos, borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' }]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });
}
</script>
</body>
</html>

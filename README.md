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

body { 
    margin:0; min-height: 100vh; background-color: #f0f2f5;
    background-size: cover; background-position: center; background-attachment: fixed;
    transition: background-image 1.5s ease-in-out; position: relative;
}
body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.85); z-index: -1; }

header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }
.screen { display:none; padding:20px; max-width: 1150px; margin: 10px auto; }
.active { display:block; animation: fadeIn 0.4s; }

.kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; min-width: 200px; text-align: center; border-bottom: 5px solid var(--primary); }
.card { background: rgba(255,255,255,0.95); padding: 15px; border: 1px solid #ddd; border-radius: 6px; margin-bottom: 15px; border-left: 5px solid var(--primary); }

button { border: none; padding: 12px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; }
.btn-primary { background: var(--primary); }
.btn-secondary { background: var(--secondary); }

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
            <option value="LOGISTICA FILIAL">LOGISTICA FILIAL</option>
            <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MANUTENÇÃO">MANUTENÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
            <option value="SALA ELETRONICOS">SALA ELETRONICOS</option>
            <option value="SALA IMPETUS">SALA IMPETUS</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data">
        <br><br>
        <button class="btn-primary" onclick="start()">Iniciar Avaliação</button>
        <button class="btn-secondary" onclick="openDashboard()">Dashboard</button>
    </div>
</div>

<div id="senso" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="card">
        <div style="display:flex; gap:10px; margin-bottom:15px;">
            <button class="btn-secondary" onclick="show('home')">Novo</button>
            <button style="background:#10b981" onclick="window.print()">Salvar PDF</button>
        </div>
        <div style="display:flex; gap:20px;">
            <div class="kpi-card"><h2 id="mediaGeralFabrica" style="color:#f06639">0.0</h2><p>Média Geral Fábrica</p></div>
            <div class="kpi-card"><h2 id="totalAuditorias" style="color:#f06639">0</h2><p>Total Auditorias</p></div>
        </div>
        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-top:20px;">
            <canvas id="cRadar"></canvas>
            <canvas id="cBarra"></canvas>
        </div>
        <div id="relatorioView"></div>
    </div>
</div>

<script>
const checklist = [
    { senso: "Utilização", perguntas: ["Itens desnecessários na área?", "Estoque excessivo ou obsoleto?", "Quadros e documentos atualizados?", "Ferramentas de uso constante acessíveis?", "Descarte de itens inúteis correto?"] },
    { senso: "Organização", perguntas: ["Local fixo e identificado?", "Piso demarcado?", "Fácil localização (< 30 seg)?", "Armários e gavetas organizados?", "Fluxo de trabalho facilitado?"] },
    { senso: "Limpeza", perguntas: ["Chão e máquinas limpos?", "Causas de sujeira eliminadas?", "Materiais de limpeza disponíveis?", "Lixeiras identificadas?", "Equipamentos conservados?"] },
    { senso: "Padronização", perguntas: ["Identificação visual clara?", "Padrão de cores seguido?", "Uso de EPIs e Uniforme?", "Ambiente higienizado?", "Manual de 5S seguido?"] },
    { senso: "Disciplina", perguntas: ["Rotinas de limpeza seguidas?", "Checklists preenchidos?", "Melhoria contínua ativa?", "Prazos cumpridos?", "Equipe engajada?"] }
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
        html += `<label>${p}</label>
        <select id="n${i}">
            <option value="10">Excelente (10)</option>
            <option value="8">Bom (8)</option>
            <option value="5">Regular (5)</option>
            <option value="0">Péssimo (0)</option>
        </select>`;
    });
    html += `<label>Plano de Ação:</label><textarea id="plano"></textarea>
    <label>Foto do Desvio:</label><input type="file" accept="image/*" capture="environment">
    <button class="btn-primary" onclick="salvarSenso()">${sensoIndex < 4 ? 'Próximo' : 'Finalizar'}</button></div>`;
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
    // Lógica de cálculo de médias e renderização de tabelas e gráficos (Radar Comparativo)
    // Atualização dos KPIs de Média e Total
}
</script>
</body>
</html>

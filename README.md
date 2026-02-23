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

/* Plano de Fundo Dinâmico */
body { 
    margin:0; 
    min-height: 100vh;
    background-color: #f0f2f5;
    background-size: cover;
    background-position: center;
    background-attachment: fixed;
    transition: background-image 1.5s ease-in-out;
    position: relative;
}

/* Overlay para garantir leitura */
body::before {
    content: "";
    position: fixed;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(255, 255, 255, 0.85);
    z-index: -1;
}

header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; position: relative; }

.screen { display:none; padding:20px; max-width: 1150px; margin: 10px auto; position: relative; }
.active { display:block; animation: fadeIn 0.4s; }

.filter-bar { display: flex; gap: 10px; background: rgba(255,255,255,0.9); padding: 15px; border-radius: 8px; margin-bottom: 20px; flex-wrap: wrap; border: 1px solid #ddd; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
.filter-item { flex: 1; min-width: 140px; }
.filter-item label { font-size: 11px; font-weight: bold; display: block; margin-bottom: 4px; color: var(--secondary); }

.kpi-container { display: flex; gap: 20px; margin-bottom: 20px; flex-wrap: wrap; }
.kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; min-width: 200px; text-align: center; border-bottom: 5px solid var(--primary); box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.kpi-card h2 { margin: 0; font-size: 32px; color: var(--primary); }

.card { background: rgba(255,255,255,0.95); padding: 15px; border: 1px solid #ddd; border-radius: 6px; margin-bottom: 15px; border-left: 5px solid var(--primary); box-shadow: 0 2px 4px rgba(0,0,0,0.05); }

button { border: none; padding: 10px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.2s; }
.btn-primary { background: var(--primary); }
.btn-primary:hover { opacity: 0.9; }
.btn-secondary { background: var(--secondary); }
.btn-danger { background: #dc3545; font-size: 10px; padding: 5px 10px; }
.btn-edit { background: #ffc107; color: #000; font-size: 10px; padding: 5px 10px; margin-right: 5px; }

select, input, textarea { width: 100%; padding: 10px; border-radius: 4px; border: 1px solid #ccc; background: white; }

.tabela-pdf { width: 100%; border-collapse: collapse; margin-top: 10px; background: white; }
.tabela-pdf th, .tabela-pdf td { border: 1px solid #999; padding: 8px; text-align: left; font-size: 13px; }
.tabela-pdf th { background: #eee; }

@media print { .no-print { display:none !important; } body::before { background: white; } }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body id="bgBody">

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active">
    <div class="card">
        <h3 id="tituloHome">Identificação da Auditoria</h3>
        <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px;">
            <div><label>Setor:</label>
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
            </div>
            <div><label>Auditor:</label><input type="text" id="auditor"></div>
            <div><label>Responsável:</label><input type="text" id="responsavel"></div>
            <div><label>Data:</label><input type="date" id="data"></div>
        </div><br>
        <button class="btn-primary" onclick="start()">Iniciar Avaliação</button>
        <button class="btn-secondary" onclick="openDashboard()">Dashboard / Histórico</button>
    </div>
</div>

<div id="senso" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="no-print">
        <div class="filter-bar">
            <div class="filter-item"><label>SETOR</label><select id="fSetor" onchange="renderRelatorio()"></select></div>
            <div class="filter-item"><label>MÊS</label>
                <select id="fMes" onchange="renderRelatorio()">
                    <option value="GERAL">Todos</option>
                    <option value="0">Janeiro</option><option value="1">Fevereiro</option><option value="2">Março</option>
                </select>
            </div>
            <div style="display:flex; align-items:flex-end; gap:5px;">
                <button class="btn-secondary" onclick="resetForm()">Novo</button>
                <button style="background:#10b981" onclick="window.print()">Salvar PDF</button>
            </div>
        </div>
    </div>

    <div class="kpi-container">
        <div class="kpi-card"><h2 id="mediaGeralFabrica">0.0</h2><p>Média Geral Fábrica</p></div>
        <div class="kpi-card"><h2 id="totalAuditorias">0</h2><p>Total de Auditorias</p></div>
    </div>

    <div id="areaGraficos" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); gap: 20px; margin-bottom: 40px;">
        <div style="background:white; padding:10px; border-radius:8px;"><canvas id="cRadar"></canvas></div>
        <div style="background:white; padding:10px; border-radius:8px;"><canvas id="cBarra"></canvas></div>
    </div>

    <div id="relatorioView"></div>
</div>

<script>
Chart.register(ChartDataLabels);

const imagensFundo = [
    "https://images.unsplash.com/photo-1504917595217-d4dc5ebe6122?auto=format&fit=crop&q=80&w=1920",
    "https://images.unsplash.com/photo-1581092160562-40aa08e78837?auto=format&fit=crop&q=80&w=1920",
    "https://images.unsplash.com/photo-1565608438257-fac3c27beb36?auto=format&fit=crop&q=80&w=1920"
];
let imgIndex = 0;
function mudarFundo() {
    document.getElementById('bgBody').style.backgroundImage = `url('${imagensFundo[imgIndex]}')`;
    imgIndex = (imgIndex + 1) % imagensFundo.length;
}
setInterval(mudarFundo, 10000);
mudarFundo();

const checklist = [
    { senso: "Utilização", perguntas: ["Itens desnecessários na área?", "Estoque excessivo/obsoleto?", "Quadros/docs atualizados?", "Ferramentas acessíveis?", "Descarte correto?"] },
    { senso: "Organização", perguntas: ["Local fixo e identificado?", "Piso demarcado?", "Localização rápida (<30s)?", "Armários organizados?", "Fluxo facilitado?"] },
    { senso: "Limpeza", perguntas: ["Chão e máquinas limpos?", "Fontes de sujeira controladas?", "Materiais de limpeza OK?", "Lixeiras identificadas?", "Equipamentos conservados?"] },
    { senso: "Padronização", perguntas: ["Identificação visual clara?", "Padrões de cores seguidos?", "Uso correto de EPIs?", "Ambiente agradável?", "Manutenções em dia?"] },
    { senso: "Disciplina", perguntas: ["Rotinas seguidas?", "Checklists preenchidos?", "Melhoria contínua ativa?", "Prazos cumpridos?", "Equipe engajada?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, sensoIndex = 0, editandoId = null;
let chartRadar, chartBar;

function show(id) { document.querySelectorAll(".screen").forEach(s => s.classList.remove("active")); document.getElementById(id).classList.add("active"); }

function start() {
    audit = { id: Date.now(), setor: document.getElementById('setor').value, auditor: document.getElementById('auditor').value, responsavel: document.getElementById('responsavel').value, data: document.getElementById('data').value, respostas: [] };
    sensoIndex = 0; renderSenso(); show("senso");
}

function renderSenso() {
    const s = checklist[sensoIndex];
    let html = `<div class="card"><h2>${s.senso}</h2>`;
    s.perguntas.forEach((p, i) => {
        html += `<label>${p}</label><select id="n${i}"><option value="10">Excelente (10)</option><option value="8">Bom (8)</option><option value="5">Regular (5)</option><option value="0">Péssimo (0)</option></select>`;
    });
    html += `<label>Plano de Ação:</label><textarea id="plano"></textarea><br><button class="btn-primary" onclick="salvarSenso()">Próximo</button></div>`;
    document.getElementById("senso").innerHTML = html;
}

function salvarSenso() {
    let notas = [];
    for(let i=0; i<5; i++) notas.push(Number(document.getElementById("n"+i).value));
    audit.respostas.push({ media: (notas.reduce((a,b)=>a+b)/5).toFixed(1), plano: document.getElementById("plano").value });
    sensoIndex++;
    if(sensoIndex < 5) renderSenso();
    else { db.push(audit); localStorage.setItem("j2m_db", JSON.stringify(db)); openDashboard(); }
}

function openDashboard() {
    const sel = document.getElementById("fSetor");
    sel.innerHTML = '<option value="GERAL">Visão Geral Fábrica</option>';
    [...new Set(db.map(a => a.setor))].forEach(s => sel.innerHTML += `<option value="${s}">${s}</option>`);
    renderRelatorio(); show("dashboard");
}

function renderRelatorio() {
    const fS = document.getElementById("fSetor").value;
    const filtrados = db.filter(a => fS === "GERAL" || a.setor === fS);
    atualizarGraficos(filtrados);
}

function atualizarGraficos(dados) {
    if(chartRadar) chartRadar.destroy(); if(chartBar) chartBar.destroy();
    if(dados.length === 0) return;
    const mSensos = [0,1,2,3,4].map(i => (dados.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / dados.length).toFixed(1));
    chartRadar = new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Utilização", "Organização", "Limpeza", "Padronização", "Disciplina"],
            datasets: [
                { label: 'Setor', data: mSensos, borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Meta', data: [8,8,8,8,8], borderColor: '#28a745', borderDash: [5,5], fill: false }
            ]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });
}
</script>
</body>
</html>

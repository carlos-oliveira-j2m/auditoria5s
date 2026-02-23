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

/* MANTENDO O TEU CSS ORIGINAL E FUNDO DINÂMICO */
body { 
    margin:0; min-height: 100vh; background-color: #f0f2f5; background-size: cover;
    background-position: center; background-attachment: fixed; transition: background-image 1.5s ease-in-out;
    position: relative;
}
body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.85); z-index: -1; }

header { background: var(--secondary); color: white; padding: 20px; text-align: center; border-bottom: 6px solid var(--primary); font-size: 24px; font-weight: bold; }
.container { max-width: 1100px; margin: 30px auto; padding: 0 20px; }
.card { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 10px 25px rgba(0,0,0,0.1); margin-bottom: 30px; border-top: 5px solid var(--primary); }

.stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 25px; }
.stat-card { background: white; padding: 20px; border-radius: 10px; text-align: center; border: 1px solid #eee; }
.stat-card h1 { margin: 10px 0 0 0; font-size: 42px; color: var(--secondary); }

.btn { padding: 12px 25px; border: none; border-radius: 6px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; }
.btn-primary { background: var(--primary); width: 100%; }
.btn-sync { background: var(--blue); width: 100%; margin-bottom: 10px; }

.screen { display: none; }
.active { display: block; }

@media print { .no-print { display: none !important; } .screen { display: block !important; } }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div class="container">
    
    <div id="screen-home" class="screen active no-print">
        <div class="card">
            <h2 style="text-align:center">Painel de Gestão 5S</h2>
            <button class="btn btn-sync" onclick="sincronizarDados()">🔄 SINCRONIZAR CELULAR / PC</button>
            <button class="btn btn-primary" onclick="irPara('screen-setup')">Nova Auditoria</button>
            <button class="btn" style="background: var(--secondary); width: 100%; margin-top: 10px;" onclick="irPara('screen-relatorio')">Visualizar Relatórios</button>
        </div>
    </div>

    <div id="screen-setup" class="screen no-print">
        <div class="card">
            <h3>Configurações</h3>
            <label>Setor:</label>
            <select id="setor">
                <option value="ADM Filial">ADM Filial</option>
                <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
                <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
                <option value="ENGENHARIA SOLVE">ENGENHARIA SOLVE</option>
                <option value="INJEÇÃO">INJEÇÃO</option>
                <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
                <option value="MANUTENÇÃO">MANUTENÇÃO</option>
                <option value="MATRIZARIA">MATRIZARIA</option>
                <option value="MONTAGEM">MONTAGEM</option>
                <option value="PPCP">PPCP</option>
                <option value="QUALIDADE">QUALIDADE</option>
                <option value="SALA DE ELETRONICOS">SALA DE ELETRONICOS</option>
                <option value="SALA DO IMPETUS">SALA DO IMPETUS</option>
            </select>
            <label>Auditor:</label>
            <input type="text" id="auditor">
            <button class="btn btn-primary" style="margin-top:20px" onclick="iniciarAuditoria()">Começar</button>
        </div>
    </div>

    <div id="screen-audit" class="screen no-print">
        <div id="pergunta-container"></div>
    </div>

    <div id="screen-relatorio" class="screen">
        <div class="no-print" style="margin-bottom: 20px; display: flex; gap: 10px;">
            <select id="filtro-setor" onchange="renderRelatorio()" style="flex: 2;"></select>
            <button class="btn" style="background: var(--green); flex: 1;" onclick="window.print()">PDF</button>
        </div>

        <div class="stats-grid">
            <div class="stat-card"><p>MÉDIA GERAL</p><h1 id="avg-factory">0.0</h1></div>
            <div class="stat-card"><p>TOTAL AUDITORIAS</p><h1 id="total-audits">0</h1></div>
        </div>

        <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 20px;">
            <div class="card"><h3>Setor vs Média</h3><canvas id="cRadar"></canvas></div>
            <div class="card"><h3>Ranking Setores</h3><canvas id="cBarra"></canvas></div>
        </div>

        <div class="card">
            <h3>Histórico</h3>
            <table>
                <thead><tr><th>Data</th><th>Setor</th><th>Nota</th><th>Ações</th></tr></thead>
                <tbody id="lista-hist"></tbody>
            </table>
        </div>
        <button class="btn no-print" style="background:#666; width:100%" onclick="irPara('screen-home')">Voltar</button>
    </div>
</div>

<script>
Chart.register(ChartDataLabels);
const API = "https://script.google.com/macros/s/AKfycbwPjqUwMg-YiCxlMhBujpMDUZZqZ_mZFFRIwyXUkojM40H_IaIjSr6utfevshUg5WFv/exec";

// MANTENDO TODAS AS TUAS PERGUNTAS E IMAGENS ORIGINAIS
const imagens = ['https://images.unsplash.com/photo-1581091226825-a6a2a5aee158?w=1920', 'https://images.unsplash.com/photo-1504328332780-bebf217e0653?w=1920'];
const checklist = [
    { s: "Seleção", p: ["Ferramentas necessárias?", "Itens desnecessários?", "Materiais próximos?", "Docs atualizados?", "Lixo/Sucata?"] },
    { s: "Ordenação", p: ["Piso demarcado?", "Fluxo visível?", "Identificação?", "Limpeza organizada?", "Objetos pessoais?"] },
    { s: "Limpeza", p: ["Piso/Máquinas limpos?", "Vazamentos?", "Condições de uso?", "Coleta seletiva?", "Higiene pessoal?"] },
    { s: "Padronização", p: ["Cores/Sinais?", "Docs técnicos?", "Lixeiras?", "Quadros?"] },
    { s: "Disciplina", p: ["Segurança/EPI?", "Melhorias executadas?", "Padrões mantidos?", "Engajamento?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, etapa = 0, chartRadar, chartBar;

function mudarFundo() { document.body.style.backgroundImage = `url('${imagens[Math.floor(Math.random() * imagens.length)]}')`; }
setInterval(mudarFundo, 10000); mudarFundo();

function irPara(id) { 
    document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
    if(id === 'screen-relatorio') { preencherFiltros(); renderRelatorio(); }
}

function iniciarAuditoria() {
    audit = { id: Date.now(), setor: document.getElementById('setor').value, auditor: document.getElementById('auditor').value, data: new Date().toLocaleDateString(), respostas: [] };
    etapa = 0; gerarPergunta(); irPara('screen-audit');
}

function gerarPergunta() {
    const s = checklist[etapa];
    let h = `<div class="card"><h2>Senso de ${s.s}</h2>`;
    s.p.forEach((p, i) => {
        h += `<label>${p}</label><select class="q-nota"><option value="10">10</option><option value="8">8</option><option value="6">6</option><option value="4">4</option><option value="2">2</option></select>`;
    });
    h += `<label>Ação Corretiva:</label><textarea id="plano-txt"></textarea><button class="btn btn-primary" onclick="proximo()">Próximo</button></div>`;
    document.getElementById('pergunta-container').innerHTML = h;
}

function proximo() {
    const notas = Array.from(document.querySelectorAll('.q-nota')).map(s => Number(s.value));
    audit.respostas.push({ media: notas.reduce((a,b)=>a+b,0)/notas.length, plano: document.getElementById('plano-txt').value });
    etapa++;
    if(etapa < checklist.length) gerarPergunta();
    else {
        db.push(audit);
        localStorage.setItem("j2m_db", JSON.stringify(db));
        // SINCRONIZA AO FINALIZAR
        fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
        alert("Salvo!"); irPara('screen-relatorio');
    }
}

async function sincronizarDados() {
    alert("Sincronizando com a nuvem...");
    try {
        const r = await fetch(API);
        const data = await r.json();
        if(data.length > 1) {
            db = data.slice(1).map(row => ({ id: row[10], setor: row[1], auditor: row[3], data: row[9], respostas: JSON.parse(row[11]) }));
            localStorage.setItem("j2m_db", JSON.stringify(db));
            alert("Sucesso!"); renderRelatorio();
        }
    } catch(e) { alert("Erro de conexão."); }
}

function preencherFiltros() {
    const f = document.getElementById('filtro-setor');
    f.innerHTML = '<option value="TODOS">Todos os Setores</option>';
    [...new Set(db.map(x => x.setor))].forEach(s => f.innerHTML += `<option value="${s}">${s}</option>`);
}

function renderRelatorio() {
    const filtro = document.getElementById('filtro-setor').value;
    const dados = filtro === "TODOS" ? db : db.filter(x => x.setor === filtro);
    if(!dados.length) return;
    const ult = dados[dados.length - 1];

    document.getElementById('total-audits').innerText = db.length;
    const somaF = db.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById('avg-factory').innerText = (somaF / db.length).toFixed(1);

    if(chartRadar) chartRadar.destroy();
    chartRadar = new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets: [{ label: ult.setor, data: ult.respostas.map(r => r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' }]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    const setores = [...new Set(db.map(a => a.setor))];
    const notas = setores.map(s => {
        const d = db.filter(x => x.setor === s);
        return (d.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0) / d.length).toFixed(1);
    });

    if(chartBar) chartBar.destroy();
    chartBar = new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: {

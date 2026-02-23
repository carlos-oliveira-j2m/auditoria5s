<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>

<style>
:root { --primary: #f06639; --secondary: #5d5a51; --green: #28a745; --blue: #007bff; --danger: #dc3545; }
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}

body { margin:0; min-height:100vh; background:#f0f2f5 url('https://images.unsplash.com/photo-1504917595217-d4dc5ebe6122?w=1600') no-repeat center center fixed; background-size:cover; }
body::before { content: ""; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(255,255,255,0.9); z-index:-1; }

header { background:var(--secondary); padding:15px; text-align:center; border-bottom:5px solid var(--primary); color:white; font-weight:bold; font-size:22px; }
.screen { display:none; padding:20px; max-width:1100px; margin:auto; }
.active { display:block; }

.card { background:white; padding:20px; border-radius:8px; margin-bottom:15px; border-left:5px solid var(--primary); box-shadow:0 2px 5px rgba(0,0,0,0.1); }
.pergunta { margin-bottom:15px; padding-bottom:10px; border-bottom:1px solid #eee; }
.desc { font-size:12px; color:#666; display:block; margin-bottom:5px; }

select, input, textarea { width:100%; padding:10px; border-radius:4px; border:1px solid #ccc; margin-bottom:10px; }
.erro { border:2px solid var(--danger) !important; background:#fff1f1; }

button { border:none; padding:12px; border-radius:4px; cursor:pointer; font-weight:bold; text-transform:uppercase; color:white; width:100%; margin-bottom:10px; }
.btn-p { background:var(--primary); }
.btn-b { background:var(--blue); }

.kpis { display:flex; gap:15px; margin-bottom:20px; }
.kpi { background:var(--secondary); color:white; padding:15px; border-radius:8px; flex:1; text-align:center; border-bottom:4px solid var(--primary); }
.kpi h2 { margin:0; color:var(--primary); font-size:32px; }

.tabela { width:100%; border-collapse:collapse; margin-top:20px; }
.tabela th, .tabela td { border:1px solid #ccc; padding:10px; text-align:left; font-size:13px; }
.plano-box { background:#fff8e1; border:1px solid #ffe082; padding:8px; border-radius:4px; font-size:12px; margin-top:5px; }

@media print { .no-print { display:none; } .screen { display:block!important; } body { background:white; } }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="scr_home" class="screen active">
    <div class="card">
        <h3>Identificação</h3>
        <label>Setor:</label>
        <select id="setor">
            <option value="">-- SELECIONE --</option>
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="ALMOXARIFADO">ALMOXARIFADO</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-p" onclick="abrirAuditoria()">INICIAR AVALIAÇÃO</button>
        <button class="btn-b" onclick="sincronizar()">🔄 SINCRONIZAR NUVEM</button>
        <button style="background:#666" onclick="abrirDash()">ABRIR DASHBOARD</button>
    </div>
</div>

<div id="scr_audit" class="screen">
    <div id="box_perguntas"></div>
</div>

<div id="scr_dash" class="screen">
    <div class="no-print card">
        <div style="display:flex; gap:10px">
            <select id="f_setor" onchange="renderRelatorio()"></select>
            <button class="btn-b" onclick="window.print()" style="width:200px">GERAR PDF</button>
            <button style="background:#666; width:200px" onclick="location.reload()">VOLTAR</button>
        </div>
    </div>

    <div class="kpis">
        <div class="kpi"><h2 id="k_media">0.0</h2>MÉDIA FÁBRICA</div>
        <div class="kpi"><h2 id="k_total">0</h2>AUDITORIAS</div>
    </div>

    <div style="display:grid; grid-template-columns:repeat(auto-fit, minmax(400px, 1fr)); gap:20px;">
        <div class="card"><canvas id="chartRadar"></canvas></div>
        <div class="card"><canvas id="chartBarra"></canvas></div>
    </div>
    <div id="relatorio_final"></div>
</div>

<script>
const API = "https://script.google.com/macros/s/AKfycbzcntNB7ErwLkye0Y7kAoqneoeMAs_MMe7YvszVdHGrHJUGacpixxAYV9LDppBlUoNx/exec";

const sensos = [
    { s: "SELEÇÃO", p: ["Ferramentas necessárias?", "Área limpa de itens obsoletos?"] },
    { s: "ORDENAÇÃO", p: ["Identificação de locais?", "Layout organizado?"] },
    { s: "LIMPEZA", p: ["Máquinas e postos limpos?", "Ausência de vazamentos?"] },
    { s: "PADRONIZAÇÃO", p: ["Uso de EPIs?", "Gestão visual e avisos?"] },
    { s: "DISCIPLINA", p: ["Cumprimento das normas?", "Melhoria contínua?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, step = 0;
let rad, bar;

function abrirAuditoria() {
    if(!document.getElementById('setor').value) return alert("Selecione o setor!");
    audit = { 
        id: Date.now(), 
        setor: document.getElementById('setor').value, 
        auditor: document.getElementById('auditor').value, 
        data: document.getElementById('data').value, 
        respostas: [] 
    };
    step = 0;
    renderSenso();
}

function renderSenso() {
    const s = sensos[step];
    let h = `<div class="card"><h2>${step+1}º Senso: ${s.s}</h2>`;
    s.p.forEach((p, i) => {
        h += `<div class="pergunta"><strong>${p}</strong>
              <select class="nt"><option value="">-- NOTA --</option>
              <option value="10">10</option><option value="8">8</option>
              <option value="6">6</option><option value="4">4</option><option value="2">2</option></select></div>`;
    });
    h += `<label>Evidências:</label><textarea id="evid"></textarea>
          <label>Plano de Ação (Obrigatório se nota < 6):</label><textarea id="plano"></textarea>
          <button class="btn-p" onclick="gravarSenso()">PRÓXIMO</button></div>`;
    document.getElementById('box_perguntas').innerHTML = h;
    show('scr_audit');
}

function gravarSenso() {
    const selects = document.querySelectorAll('.nt');
    let notas = [];
    for(let sel of selects) {
        if(!sel.value) { sel.classList.add('erro'); return alert("Preencha todas as notas!"); }
        notas.push(Number(sel.value));
    }
    const media = (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1);
    const plano = document.getElementById('plano').value.trim();
    
    if(media < 6 && plano.length < 3) return alert("Média baixa! Plano de Ação obrigatório.");

    audit.respostas.push({ media, plano, evid: document.getElementById('evid').value });
    step++;
    
    if(step < sensos.length) renderSenso();
    else finalizar();
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("j2m_db", JSON.stringify(db));
    fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
    alert("Concluído!");
    abrirDash();
}

function abrirDash() {
    const f = document.getElementById('f_setor');
    f.innerHTML = '<option value="TODOS">Todos os Setores</option>';
    [...new Set(db.map(x=>x.setor))].forEach(s => f.innerHTML += `<option value="${s}">${s}</option>`);
    renderRelatorio();
    show('scr_dash');
}

function renderRelatorio() {
    if(!db.length) return;
    const fs = document.getElementById('f_setor').value;
    const filtered = db.filter(a => fs === "TODOS" || a.setor === fs);
    const ult = filtered[filtered.length-1];

    document.getElementById('k_total').innerText = db.length;
    let somaF = db.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById('k_media').innerText = (somaF/db.length).toFixed(1);

    if(rad) rad.destroy();
    const mediaF = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / db.length).toFixed(1));
    rad = new Chart(document.getElementById('chartRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets: [
                { label: ult.setor, data: ult.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Fábrica', data: mediaF, borderColor: '#007bff', fill: false, borderDash:[5,5] },
                { label: 'Meta (8.0)', data: [8,8,8,8,8], borderColor: '#28a745', fill: false, borderDash:[2,2] }
            ]
        },
        options: { scales: { r: { min:0, max:10 } } }
    });

    if(bar) bar.destroy();
    const sets = [...new Set(db.map(d=>d.setor))];
    const nts = sets.map(s => (db.filter(x=>x.setor===s).reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0) / db.filter(x=>x.setor===s).length).toFixed(1));
    bar = new Chart(document.getElementById('chartBarra'), {
        type: 'bar',
        data: { labels: sets, datasets: [{ label: 'Média por Área', data: nts, backgroundColor: '#5d5a51' }] }
    });

    let h = `<h3>Detalhes: ${ult.setor}</h3><table class="tabela"><tr><th>Senso</th><th>Nota</th><th>Evidência / Plano</th></tr>`;
    ult.respostas.forEach((r, i) => {
        h += `<tr><td>${sensos[i].s}</td><td><b>${r.media}</b></td><td>${r.evid}<br>${r.plano ? '<div class="plano-box"><b>PLANO:</b> '+r.plano+'</div>' : ''}</td></tr>`;
    });
    document.getElementById('relatorio_final').innerHTML = h + "</table>";
}

function show(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }

async function sincronizar() {
    alert("Sincronizando...");
    try {
        const r = await fetch(API);
        const data = await r.json();
        db = data.slice(1).map(row => ({ id: row[10], setor: row[1], auditor: row[3], data: row[9], respostas: JSON.parse(row[11]) }));
        localStorage.setItem("j2m_db", JSON.stringify(db));
        abrirDash();
    } catch(e) { alert("Erro ao carregar dados."); }
}
</script>
</body>
</html>

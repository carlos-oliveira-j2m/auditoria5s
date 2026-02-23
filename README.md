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
body::before { content: ""; position:fixed; top:0; left:0; width:100%; height:100%; background:rgba(255,255,255,0.92); z-index:-1; }

header { background:var(--secondary); padding:15px; text-align:center; border-bottom:5px solid var(--primary); color:white; font-weight:bold; font-size:22px; }
.screen { display:none; padding:20px; max-width:1100px; margin:auto; }
.active { display:block; animation: fadeIn 0.3s; }

.card { background:white; padding:20px; border-radius:8px; margin-bottom:15px; border-left:5px solid var(--primary); box-shadow:0 2px 8px rgba(0,0,0,0.1); }
.pergunta { margin-bottom:15px; padding-bottom:10px; border-bottom:1px solid #eee; }
.desc { font-size:12px; color:#666; display:block; margin-bottom:5px; font-style: italic; }

select, input, textarea { width:100%; padding:12px; border-radius:4px; border:1px solid #ccc; margin-bottom:10px; font-size:14px; }
.campo-erro { border: 2px solid var(--danger) !important; background: #fff1f1; }

button { border:none; padding:12px; border-radius:4px; cursor:pointer; font-weight:bold; text-transform:uppercase; color:white; width:100%; margin-bottom:10px; transition: 0.2s; }
.btn-p { background:var(--primary); }
.btn-p:hover { background: #d4562d; }
.btn-b { background:var(--blue); }

.kpis { display:flex; gap:15px; margin-bottom:20px; }
.kpi { background:var(--secondary); color:white; padding:15px; border-radius:8px; flex:1; text-align:center; border-bottom:4px solid var(--primary); }
.kpi h2 { margin:0; color:var(--primary); font-size:32px; }

.tabela { width:100%; border-collapse:collapse; margin-top:20px; background: white; }
.tabela th, .tabela td { border:1px solid #ccc; padding:12px; text-align:left; font-size:13px; }
.plano-box { background:#fffde7; border:1px solid #ffd600; padding:10px; border-radius:4px; font-size:13px; margin-top:8px; color: #333; border-left: 5px solid #ffd600; }

@media print { .no-print { display:none !important; } .screen { display:block !important; } body { background:white; } .card { box-shadow:none; border: 1px solid #ccc; } }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="scr_home" class="screen active">
    <div class="card">
        <h3>Nova Auditoria</h3>
        <label>Setor:</label>
        <select id="setor">
            <option value="">-- SELECIONE O SETOR --</option>
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor" placeholder="Nome do responsável">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-p" onclick="iniciar()">INICIAR AVALIAÇÃO</button>
        <button class="btn-b" onclick="sincronizar()">🔄 SINCRONIZAR COM NUVEM</button>
        <button style="background:var(--secondary)" onclick="abrirDash()">DASHBOARD E FILTROS</button>
    </div>
</div>

<div id="scr_audit" class="screen">
    <div id="box_perguntas"></div>
</div>

<div id="scr_dash" class="screen">
    <div class="no-print card">
        <div style="display:flex; gap:10px; flex-wrap: wrap;">
            <div style="flex:1; min-width:150px;"><label>Setor</label><select id="f_setor" onchange="renderRelatorio()"></select></div>
            <div style="flex:1; min-width:100px;"><label>Mês</label><select id="f_mes" onchange="renderRelatorio()">
                <option value="T">Todos</option><option value="0">Jan</option><option value="1">Fev</option><option value="2">Mar</option>
            </select></div>
            <div style="flex:1; min-width:100px;"><label>Ano</label><select id="f_ano" onchange="renderRelatorio()"><option value="2026">2026</option><option value="2025">2025</option></select></div>
            <button class="btn-b" onclick="window.print()" style="width:150px; margin-top:25px;">PDF</button>
            <button style="background:#666; width:150px; margin-top:25px;" onclick="location.reload()">VOLTAR</button>
        </div>
    </div>

    <div class="kpis">
        <div class="kpi"><h2 id="k_media">0.0</h2>MÉDIA DA FÁBRICA</div>
        <div class="kpi"><h2 id="k_total">0</h2>AUDITORIAS</div>
    </div>

    <div style="display:grid; grid-template-columns:repeat(auto-fit, minmax(450px, 1fr)); gap:20px;">
        <div class="card"><h3>Radar: Setor vs Fábrica vs Meta</h3><canvas id="chartRadar"></canvas></div>
        <div class="card"><h3>Comparativo entre Áreas</h3><canvas id="chartBarra"></canvas></div>
    </div>
    <div id="relatorio_final"></div>
</div>

<script>
const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbzcntNB7ErwLkye0Y7kAoqneoeMAs_MMe7YvszVdHGrHJUGacpixxAYV9LDppBlUoNx/exec";

const estrutura = [
    { s: "UTILIZAÇÃO", p: ["Apenas itens necessários no local?", "Descarte de materiais obsoletos?", "Organização de documentos/papéis?"] },
    { s: "ORDENAÇÃO", p: ["Identificação de locais e etiquetas?", "Demarcação de solo e áreas?", "Facilidade de acesso aos itens?"] },
    { s: "LIMPEZA", p: ["Máquinas e postos sem sujeira/óleo?", "Piso e lixeiras em ordem?", "Conservação geral do ambiente?"] },
    { s: "SAÚDE/PADRONIZAÇÃO", p: ["Uso correto de EPIs?", "Gestão visual atualizada?"] },
    { s: "DISCIPLINA", p: ["Padrões mantidos pela equipe?", "Execução de melhorias contínuas?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, passo = 0;
let rChart, bChart;

function iniciar() {
    const s = document.getElementById('setor').value;
    if(!s) return alert("Selecione o Setor!");
    audit = { id: Date.now(), setor: s, auditor: document.getElementById('auditor').value, data: document.getElementById('data').value, respostas: [] };
    passo = 0;
    renderPasso();
}

function renderPasso() {
    const senso = estrutura[passo];
    let h = `<div class="card"><h2>${passo+1}º Senso: ${senso.s}</h2>`;
    senso.p.forEach((p, i) => {
        h += `<div class="pergunta"><strong>${p}</strong>
              <select class="nota-aud"><option value="">-- SELECIONE A NOTA --</option>
              <option value="10">10 - Excelente</option><option value="8">8 - Bom</option>
              <option value="6">6 - Regular</option><option value="4">4 - Ruim</option><option value="2">2 - Crítico</option></select></div>`;
    });
    h += `<label><b>Respostas Explicativas (Evidências):</b></label>
          <textarea id="txt_evid" placeholder="Descreva o que foi observado neste senso..."></textarea>
          <label><b>Plano de Ação (Obrigatório p/ nota < 6):</b></label>
          <textarea id="txt_plano" placeholder="O que será feito para corrigir?"></textarea>
          <button class="btn-p" onclick="proximo()">GRAVAR SENSO</button></div>`;
    document.getElementById('box_perguntas').innerHTML = h;
    show('scr_audit');
}

function proximo() {
    const notasUI = document.querySelectorAll('.nota-aud');
    let notas = [];
    for(let n of notasUI) {
        if(!n.value) { n.classList.add('campo-erro'); return alert("Nota Obrigatória! Preencha todos os campos."); }
        notas.push(Number(n.value));
    }
    
    const media = (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1);
    const plano = document.getElementById('txt_plano').value.trim();
    const evid = document.getElementById('txt_evid').value.trim();

    if(media < 6 && plano.length < 5) {
        document.getElementById('txt_plano').classList.add('campo-erro');
        return alert("Média inferior a 6.0! O Plano de Ação é obrigatório para este senso.");
    }

    audit.respostas.push({ media, plano, evid });
    passo++;
    
    if(passo < estrutura.length) renderPasso();
    else finalizar();
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("j2m_db", JSON.stringify(db));
    fetch(SCRIPT_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
    alert("Auditoria finalizada com sucesso!");
    abrirDash();
}

function abrirDash() {
    const f = document.getElementById('f_setor');
    f.innerHTML = '<option value="T">Todos os Setores</option>';
    [...new Set(db.map(x=>x.setor))].forEach(s => f.innerHTML += `<option value="${s}">${s}</option>`);
    renderRelatorio();
    show('scr_dash');
}

function renderRelatorio() {
    if(!db.length) return;
    const fs = document.getElementById('f_setor').value;
    const fm = document.getElementById('f_mes').value;
    const fa = document.getElementById('f_ano').value;

    const filtrados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return (fs === "T" || a.setor === fs) && (fm === "T" || d.getMonth() == fm) && (d.getFullYear() == fa);
    });

    if(!filtrados.length) { document.getElementById('relatorio_final').innerHTML = "<h4>Nenhum dado encontrado para este filtro.</h4>"; return; }
    
    const ult = filtrados[filtrados.length-1];

    // KPIs
    document.getElementById('k_total').innerText = db.length;
    let somaF = db.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById('k_media').innerText = (somaF/db.length).toFixed(1);

    // Radar
    if(rChart) rChart.destroy();
    const mediaFabrica = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / db.length).toFixed(1));
    rChart = new Chart(document.getElementById('chartRadar'), {
        type: 'radar',
        data: {
            labels: ["Utilização", "Ordenação", "Limpeza", "Saúde", "Disciplina"],
            datasets: [
                { label: ult.setor, data: ult.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Média Fábrica', data: mediaFabrica, borderColor: '#007bff', fill: false, borderDash:[5,5] },
                { label: 'Meta (8.0)', data: [8,8,8,8,8], borderColor: '#28a745', fill: false, borderDash:[2,2], pointRadius: 0 }
            ]
        },
        options: { scales: { r: { min:0, max:10 } } }
    });

    // Barra
    if(bChart) bChart.destroy();
    const sets = [...new Set(db.map(d=>d.setor))];
    const nts = sets.map(s => (db.filter(x=>x.setor===s).reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0) / db.filter(x=>x.setor===s).length).toFixed(1));
    bChart = new Chart(document.getElementById('chartBarra'), {
        type: 'bar',
        data: { labels: sets, datasets: [{ label: 'Nota Final', data: nts, backgroundColor: '#5d5a51' }] },
        options: { plugins: { datalabels: { display: true, anchor: 'end', align: 'top' } } }
    });

    // Tabela Relatório
    let h = `<h3>Detalhamento: ${ult.setor} | Data: ${ult.data}</h3><table class="tabela">
             <tr><th>Senso</th><th>Nota</th><th>Evidências / Plano de Ação</th></tr>`;
    ult.respostas.forEach((r, i) => {
        h += `<tr><td>${estrutura[i].s}</td><td><b style="font-size:16px">${r.media}</b></td>
              <td><i>Evidências:</i> ${r.evid || 'Nenhuma observação.'}
              ${r.plano ? '<div class="plano-box"><b>PLANO DE AÇÃO:</b> '+r.plano+'</div>' : ''}</td></tr>`;
    });
    document.getElementById('relatorio_final').innerHTML = h + "</table>";
}

function show(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }

async function sincronizar() {
    alert("Sincronizando dados...");
    try {
        const r = await fetch(SCRIPT_URL);
        const data = await r.json();
        db = data.slice(1).map(row => ({ id: row[10], setor: row[1], auditor: row[3], data: row[9], respostas: JSON.parse(row[11]) }));
        localStorage.setItem("j2m_db", JSON.stringify(db));
        abrirDash();
    } catch(e) { alert("Erro de conexão com a planilha."); }
}
</script>
</body>
</html>

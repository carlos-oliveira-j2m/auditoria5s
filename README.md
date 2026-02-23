<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Auditoria 5S J2M - Enterprise v3.0</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
    <style>
        :root { --primary: #f06639; --secondary: #3d3d3d; --accent: #5d5a51; --danger: #dc3545; --success: #28a745; --bg: #f8f9fa; }
        * { box-sizing: border-box; font-family: 'Segoe UI', Roboto, Helvetica, sans-serif; }
        body { margin: 0; background: var(--bg); color: #333; line-height: 1.5; }
        
        /* Layout */
        header { background: var(--secondary); color: white; padding: 20px; text-align: center; border-bottom: 6px solid var(--primary); }
        .container { max-width: 1100px; margin: 0 auto; padding: 20px; }
        .screen { display: none; }
        .active { display: block; animation: fadeIn 0.3s ease; }
        
        /* UI Components */
        .card { background: white; padding: 25px; border-radius: 8px; box-shadow: 0 4px 15px rgba(0,0,0,0.08); margin-bottom: 25px; border-top: 4px solid var(--primary); }
        h2 { margin-top: 0; color: var(--secondary); border-bottom: 1px solid #eee; padding-bottom: 10px; }
        
        /* Form */
        label { display: block; font-weight: bold; margin: 15px 0 5px; }
        select, input, textarea { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 6px; font-size: 16px; margin-bottom: 10px; }
        .pergunta-card { background: #fdfdfd; padding: 18px; border: 1px solid #eee; border-radius: 8px; margin-bottom: 15px; border-left: 5px solid #ddd; }
        .pergunta-card.valid { border-left-color: var(--success); }
        .pergunta-card.invalid { border-left-color: var(--danger); background: #fff5f5; }
        .legenda-nota { font-size: 12px; color: #666; margin-top: 5px; display: block; }

        /* Buttons */
        .btn-group { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-top: 20px; }
        button { border: none; padding: 16px; border-radius: 6px; cursor: pointer; font-weight: bold; text-transform: uppercase; transition: 0.3s; color: white; font-size: 14px; }
        .btn-main { background: var(--primary); width: 100%; }
        .btn-sync { background: var(--blue); width: 100%; background-color: #007bff; }
        .btn-sec { background: var(--accent); width: 100%; }
        
        /* Dashboard & Charts */
        .kpi-row { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 25px; }
        .kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; text-align: center; }
        .kpi-card span { display: block; font-size: 36px; font-weight: bold; color: var(--primary); }
        .charts-row { display: grid; grid-template-columns: repeat(auto-fit, minmax(450px, 1fr)); gap: 20px; }

        /* Relatório */
        .table-rep { width: 100%; border-collapse: collapse; margin-top: 20px; }
        .table-rep th, .table-rep td { border: 1px solid #ddd; padding: 12px; text-align: left; }
        .table-rep th { background: #f4f4f4; }
        .plano-acao { background: #fff8e1; border-left: 4px solid #ffc107; padding: 10px; font-size: 14px; margin-top: 5px; }

        @media print { .no-print { display: none !important; } .screen { display: block !important; } body { background: white; } .card { box-shadow: none; border: 1px solid #ddd; } }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body>

<header>SISTEMA DE AUDITORIA 5S J2M - v3.0</header>

<div class="container">
    
    <div id="scr_home" class="screen active">
        <div class="card">
            <h2>Nova Auditoria</h2>
            <label>Setor:</label>
            <select id="setor">
                <option value="">-- Selecione o Setor --</option>
                <option value="ADM FILIAL">ADM FILIAL</option>
                <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
                <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
                <option value="ESPAÇO MAKER">ESPAÇO MAKER</option>
                <option value="INJEÇÃO">INJEÇÃO</option>
                <option value="MANUTENÇÃO">MANUTENÇÃO</option>
                <option value="MATRIZARIA">MATRIZARIA</option>
                <option value="MONTAGEM">MONTAGEM</option>
                <option value="PPCP">PPCP</option>
                <option value="QUALIDADE">QUALIDADE</option>
            </select>
            <label>Auditor Responsável:</label>
            <input type="text" id="auditor" placeholder="Nome do auditor">
            <label>Data da Auditoria:</label>
            <input type="date" id="data_audit">
            
            <button class="btn-main" style="margin-top:20px" onclick="iniciarAudit()">COMEÇAR AVALIAÇÃO</button>
            <div class="btn-group">
                <button class="btn-sync" onclick="syncData()">🔄 SINCRONIZAR NUVEM</button>
                <button class="btn-sec" onclick="abrirDash()">DASHBOARD / HISTÓRICO</button>
            </div>
        </div>
    </div>

    <div id="scr_audit" class="screen">
        <div id="questoes_area"></div>
    </div>

    <div id="scr_dash" class="screen">
        <div class="no-print card">
            <h2>Filtros de Análise</h2>
            <div style="display:grid; grid-template-columns: 1fr 1fr 1fr; gap:10px">
                <select id="f_setor" onchange="renderRelatorio()"></select>
                <select id="f_mes" onchange="renderRelatorio()">
                    <option value="ALL">Todos os Meses</option>
                    <option value="0">Janeiro</option><option value="1">Fevereiro</option><option value="2">Março</option>
                    <option value="3">Abril</option><option value="4">Maio</option><option value="5">Junho</option>
                </select>
                <button class="btn-sync" onclick="window.print()">GERAR PDF</button>
            </div>
            <button class="btn-sec" style="margin-top:10px" onclick="location.reload()">VOLTAR AO INÍCIO</button>
        </div>

        <div class="kpi-row">
            <div class="kpi-card"><span id="k_media_setor">0.0</span>MÉDIA DO FILTRO</div>
            <div class="kpi-card"><span id="k_media_fabrica">0.0</span>MÉDIA FÁBRICA</div>
            <div class="kpi-card"><span id="k_meta">8.0</span>META ESTABELECIDA</div>
        </div>

        <div class="charts-row">
            <div class="card"><h3>Radar: Setor vs Fábrica vs Meta</h3><canvas id="chartRadar"></canvas></div>
            <div class="card"><h3>Comparativo entre Áreas</h3><canvas id="chartBarra"></canvas></div>
        </div>

        <div id="relatorio_final" class="card"></div>
    </div>

</div>

<script>
// --- CONFIGURAÇÃO E DADOS ---
const API_URL = "https://script.google.com/macros/s/AKfycbzcntNB7ErwLkye0Y7kAoqneoeMAs_MMe7YvszVdHGrHJUGacpixxAYV9LDppBlUoNx/exec";

const perguntas_5s = [
    { s: "1 - SELEÇÃO", p: [
        "Ferramentas e equipamentos são necessários para o trabalho diário?",
        "Existem itens duplicados sobre a bancada (ferramentas, estopas, etc)?",
        "As ferramentas são acondicionadas corretamente (um lugar para cada coisa)?",
        "Quadros de gestão/checklists estão disponíveis e atualizados?",
        "Avisos e quadros informativos são todos necessários e atuais?"
    ]},
    { s: "2 - ORDENAÇÃO", p: [
        "Locais de armazenamento (paletes/caixas) estão demarcados?",
        "Linhas e marcações de caminhos e áreas restritas são visíveis?",
        "Prateleiras e armários possuem identificação do conteúdo?",
        "Ferramentas de uso comum retornam ao lugar após o uso?",
        "Documentos e arquivos estão organizados de forma que qualquer um ache em 30s?"
    ]},
    { s: "3 - LIMPEZA", p: [
        "O piso está limpo e livre de objetos/lixo?",
        "Máquinas e equipamentos estão isentos de vazamentos e sujeira?",
        "As lixeiras estão limpas e com os sacos corretos?",
        "Iluminação e janelas estão limpas e conservadas?",
        "Existem fontes de sujeira que precisam ser eliminadas?"
    ]},
    { s: "4 - PADRONIZAÇÃO / SAÚDE", p: [
        "Os colaboradores utilizam os EPIs corretamente?",
        "Quadros de avisos estão organizados e sem papéis vencidos?",
        "Extintores e equipamentos de emergência estão desobstruídos?",
        "Ambientes comuns (banheiros/refeitório) estão organizados?",
        "Existe padrão de cores e identificação seguido por todos?",
        "Ações corretivas da última auditoria foram atendidas?"
    ]},
    { s: "5 - AUTODISCIPLINA", p: [
        "Existe rotina de checklist de autoavaliação no setor?",
        "Os funcionários demonstram conhecer a política da qualidade?",
        "A equipe mantém os padrões sem necessidade de cobrança?",
        "Melhorias foram implementadas desde a última visita?",
        "O setor mantém o nível de organização durante todo o turno?"
    ] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, sensoIdx = 0;
let radChart, barChart;

// --- LÓGICA DE AUDITORIA ---
function iniciarAudit() {
    const s = document.getElementById('setor').value;
    const a = document.getElementById('auditor').value;
    const d = document.getElementById('data_audit').value;
    if(!s || !a || !d) return alert("Por favor, preencha todos os campos de identificação.");
    
    audit = { id: Date.now(), setor: s, auditor: a, data: d, respostas: [] };
    sensoIdx = 0;
    renderSenso();
}

function renderSenso() {
    const senso = perguntas_5s[sensoIdx];
    let html = `<div class="card"><h2>${senso.s}</h2>`;
    
    senso.p.forEach((pergunta, i) => {
        html += `<div class="pergunta-card" id="pc_${i}">
            <strong>${i+1}. ${pergunta}</strong>
            <select class="nota-val" onchange="validarCor(${i})">
                <option value="">-- Selecione a Nota --</option>
                <option value="10">10 (Excelente - Sem evidências)</option>
                <option value="8">8 (Bom - 1 evidência)</option>
                <option value="6">6 (Média - 2 evidências)</option>
                <option value="4">4 (Ruim - 3 evidências)</option>
                <option value="2">2 (Crítico - 4+ evidências)</option>
            </select>
        </div>`;
    });

    html += `<h3>Evidências e Planos</h3>
             <label>Observações / Evidências:</label>
             <textarea id="obs" rows="3" placeholder="Descreva o que foi encontrado..."></textarea>
             <label>Ações Corretivas (Obrigatório para médias < 6):</label>
             <textarea id="plano" rows="3" placeholder="O que será feito para resolver?"></textarea>
             <button class="btn-main" onclick="proximoSenso()">GRAVAR E CONTINUAR</button></div>`;
    
    document.getElementById('questoes_area').innerHTML = html;
    document.getElementById('scr_audit').classList.add('active');
    document.getElementById('scr_home').classList.remove('active');
    window.scrollTo(0,0);
}

function validarCor(i) {
    const sel = document.querySelectorAll('.nota-val')[i];
    const card = document.getElementById('pc_'+i);
    if(sel.value) card.classList.add('valid');
}

function proximoSenso() {
    const selects = document.querySelectorAll('.nota-val');
    let notas = [];
    let erro = false;

    selects.forEach((s, i) => {
        if(!s.value) { document.getElementById('pc_'+i).classList.add('invalid'); erro = true; }
        else notas.push(Number(s.value));
    });

    if(erro) return alert("Todas as perguntas precisam de nota!");

    const media = (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1);
    const plano = document.getElementById('plano').value.trim();
    const obs = document.getElementById('obs').value.trim();

    if(media < 6 && plano.length < 5) return alert("Média inferior a 6.0 exige um Plano de Ação detalhado!");

    audit.respostas.push({ media, plano, obs });
    sensoIdx++;

    if(sensoIdx < perguntas_5s.length) renderSenso();
    else finalizar();
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("j2m_db", JSON.stringify(db));
    fetch(API_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
    alert("Auditoria salva com sucesso!");
    abrirDash();
}

// --- DASHBOARD E RELATÓRIO ---
function abrirDash() {
    const fs = document.getElementById('f_setor');
    fs.innerHTML = '<option value="ALL">Todos os Setores</option>';
    [...new Set(db.map(x=>x.setor))].forEach(s => fs.innerHTML += `<option value="${s}">${s}</option>`);
    
    document.getElementById('scr_dash').classList.add('active');
    document.getElementById('scr_home').classList.remove('active');
    document.getElementById('scr_audit').classList.remove('active');
    renderRelatorio();
}

function renderRelatorio() {
    const selSetor = document.getElementById('f_setor').value;
    const selMes = document.getElementById('f_mes').value;
    
    const dados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return (selSetor === "ALL" || a.setor === selSetor) && (selMes === "ALL" || d.getMonth() == selMes);
    });

    if(!dados.length) { document.getElementById('relatorio_final').innerHTML = "Nenhuma auditoria encontrada para este filtro."; return; }
    
    const ult = dados[dados.length-1];

    // KPIs
    let somaF = db.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById('k_media_fabrica').innerText = (somaF/db.length).toFixed(1);
    
    let somaSetor = dados.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById('k_media_setor').innerText = (somaSetor/dados.length).toFixed(1);

    // Radar
    if(radChart) radChart.destroy();
    const mediaFabricaSensos = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + Number(a.respostas[i]?.media || 0), 0) / db.length).toFixed(1));
    
    radChart = new Chart(document.getElementById('chartRadar'), {
        type: 'radar',
        data: {
            labels: ["Utilização", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets: [
                { label: ult.setor, data: ult.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Fábrica', data: mediaFabricaSensos, borderColor: '#007bff', fill: false, borderDash: [5,5] },
                { label: 'Meta', data: [8,8,8,8,8], borderColor: '#28a745', fill: false, borderDash: [2,2], pointRadius: 0 }
            ]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    // Barra
    if(barChart) barChart.destroy();
    const setores = [...new Set(db.map(d=>d.setor))];
    const notas = setores.map(s => {
        const d = db.filter(x=>x.setor===s);
        return (d.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0) / d.length).toFixed(1);
    });
    
    barChart = new Chart(document.getElementById('chartBarra'), {
        type: 'bar',
        data: { labels: setores, datasets: [{ label: 'Nota Atual', data: notas, backgroundColor: '#5d5a51' }] }
    });

    // Tabela Relatório
    let h = `<h2>Relatório: ${ult.setor} | Data: ${ult.data}</h2>
             <table class="table-rep"><tr><th>Senso</th><th>Nota</th><th>Evidências e Ações</th></tr>`;
    ult.respostas.forEach((r, i) => {
        h += `<tr>
            <td><b>${perguntas_5s[i].s}</b></td>
            <td style="text-align:center"><b>${r.media}</b></td>
            <td>
                <i>Evidência:</i> ${r.obs || 'Nenhuma.'}<br>
                ${r.plano ? `<div class="plano-acao"><b>AÇÃO CORRETIVA:</b> ${r.plano}</div>` : ''}
            </td>
        </tr>`;
    });
    document.getElementById('relatorio_final').innerHTML = h + "</table>";
}

async function syncData() {
    alert("Iniciando sincronização inteligente...");
    try {
        const r = await fetch(API_URL);
        const cloud = await r.json();
        const parsed = cloud.slice(1).map(row => ({
            id: row[10], setor: row[1], auditor: row[3], data: row[9],
            respostas: JSON.parse(row[11] || "[]")
        }));
        
        // Mesclar (Merge)
        const localIds = new Set(db.map(a => a.id));
        parsed.forEach(item => { if(!localIds.has(item.id)) db.push(item); });
        
        localStorage.setItem("j2m_db", JSON.stringify(db));
        alert("Sincronização concluída! Dados locais e nuvem estão iguais.");
        abrirDash();
    } catch(e) { alert("Erro de rede ao sincronizar."); }
}
</script>
</body>
</html>

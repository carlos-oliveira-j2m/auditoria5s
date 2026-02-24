<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>J2M 5S Enterprise - v1.2025</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --primary: #f06639; --dark: #3d3d3d; --bg: #f4f7f6; --danger: #dc3545; }
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { margin: 0; background: var(--bg); color: #333; }
        header { background: var(--dark); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; }
        .container { max-width: 900px; margin: auto; padding: 15px; }
        .screen { display: none; }
        .active { display: block; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 15px; border-left: 5px solid var(--primary); }
        label { display: block; font-weight: bold; margin: 10px 0 5px; }
        input, select, textarea { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 5px; font-size: 16px; margin-bottom: 10px; }
        button { cursor: pointer; border: none; padding: 15px; border-radius: 5px; font-weight: bold; width: 100%; margin-bottom: 10px; }
        .btn-p { background: var(--primary); color: white; }
        .btn-s { background: var(--dark); color: white; }
        .btn-sync { background: #007bff; color: white; }
        .q-row { border-bottom: 1px solid #eee; padding: 12px 0; display: flex; justify-content: space-between; align-items: center; gap: 10px; }
        .q-text { flex: 1; font-size: 14px; font-weight: 500; }
        .nota-box { width: 110px; border: 2px solid var(--primary); font-weight: bold; }
        .chart-container { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-top: 20px; }
        .chart-box { background: white; padding: 10px; height: 300px; border: 1px solid #ddd; }
        @media print {
            .no-print { display: none !important; }
            .screen { display: block !important; }
            .folha-rosto { height: 98vh; text-align: center; display: flex !important; flex-direction: column; justify-content: center; page-break-after: always; }
            .card { border: none; box-shadow: none; }
        }
    </style>
</head>
<body>

<header class="no-print">SISTEMA 5S J2M - v1.2025</header>

<div class="container">
    <div id="scr_home" class="screen active">
        <div class="card">
            <h3>Página Inicial</h3>
            <label>Setor:</label>
            <select id="setor">
                <option value="">-- SELECIONE --</option>
                <option value="ADM FILIAL">ADM FILIAL</option>
                <option value="ALMOXARIFADO">ALMOXARIFADO</option>
                <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
                <option value="ESPAÇO MAKER">ESPAÇO MAKER</option>
                <option value="ESTOQUE">ESTOQUE</option>
                <option value="INJEÇÃO">INJEÇÃO</option>
                <option value="MANUTENÇÃO">MANUTENÇÃO</option>
                <option value="MATRIZARIA">MATRIZARIA</option>
                <option value="MONTAGEM">MONTAGEM</option>
                <option value="PPCP">PPCP</option>
                <option value="QUALIDADE">QUALIDADE</option>
                <option value="RECEBIMENTO">RECEBIMENTO</option>
            </select>
            <label>Responsável:</label><input type="text" id="resp">
            <label>Auditor:</label><input type="text" id="aud">
            <label>Data:</label><input type="date" id="dt">
            
            <button class="btn-p" onclick="iniciarAudit()">INICIAR FORMULÁRIO</button>
            <button class="btn-s" onclick="abrirDash()">DASHBOARD</button>
            <button class="btn-sync" onclick="sincronizarAuto()">🔄 SINCRONIZAR AGORA</button>
        </div>
    </div>

    <div id="scr_form" class="screen"><div id="form_content"></div></div>

    <div id="scr_dash" class="screen">
        <div class="no-print card" id="filtros_dash"></div>
        <div id="relatorio_print"></div>
        <div class="no-print">
            <button class="btn-p" onclick="window.print()">GERAR PDF</button>
            <button class="btn-s" onclick="location.reload()">VOLTAR</button>
        </div>
    </div>
</div>

<script>
const WEB_APP_URL = "SUA_URL_AQUI"; // SUBSTITUA PELA SUA URL DO GOOGLE APPS SCRIPT

const perguntas5S = [
    { s: "1 - SELEÇÃO", q: ["Ferramentas e equipamentos são necessários para o trabalho?","Existem itens duplicados sobre a bancada?","Ferramentas acondicionadas corretamente?","Gestão à vista e documentos estão atualizados?","Avisos e quadros atuais são necessários?"]},
    { s: "2 - ORDENAÇÃO", q: ["Locais de paletes e caixas estão marcados?","Linhas e marcações são claramente visíveis?","Prateleiras e caixas estão identificadas?","Ferramentas retornam ao lugar após uso?","Arquivos e documentos fáceis de encontrar?"]},
    { s: "3 - LIMPEZA", q: ["Piso, corredores e escadas estão limpos?","Máquinas e equipamentos sem vazamentos?","Lixeiras identificadas e limpas?","Iluminação, janelas e paredes limpas?","Fontes de sujeira foram eliminadas?"]},
    { s: "4 - PADRONIZAÇÃO", q: ["Funcionários usam EPIs corretamente?","Extintores e emergência desobstruídos?","Quadros organizados sem papéis obsoletos?","Padrão de cores e etiquetas respeitado?","Placas de segurança em bom estado?"]},
    { s: "5 - AUTODISCIPLINA", q: ["Gestão mantém os padrões de 5S?","Checklist de autoavaliação é realizado?","Missão, visão e política conhecidos?","Melhorias implementadas desde a última?","Ambientes comuns limpos e organizados?","Ações corretivas anteriores atendidas?"]}
];

let db = JSON.parse(localStorage.getItem("db_j2m_5s") || "[]");
let tempAudit = {};
let etapa = 0;

function iniciarAudit() {
    const s = document.getElementById('setor').value;
    if(!s) return alert("Selecione um setor!");
    tempAudit = {
        id: Date.now(),
        setor: s,
        responsavel: document.getElementById('resp').value,
        auditor: document.getElementById('aud').value,
        data: document.getElementById('dt').value,
        respostas: []
    };
    etapa = 0;
    renderEtapa();
}

function renderEtapa() {
    const senso = perguntas5S[etapa];
    let html = `<div class="card"><h2>${senso.s}</h2>`;
    senso.q.forEach((q, i) => {
        html += `<div class="q-row"><span class="q-text">${i+1}. ${q}</span>
            <select class="nota-box" id="n_${i}">
                <option value="10">10</option><option value="8">8</option>
                <option value="6">6</option><option value="4">4</option><option value="2">2</option>
            </select></div>`;
    });
    html += `<label>Plano de Ação Corretiva (Obrigatório se nota < 6):</label>
             <textarea id="p_acao" rows="3"></textarea>
             <button class="btn-p" onclick="salvarEtapa()">PRÓXIMO</button>
             ${etapa > 0 ? `<button class="btn-s" onclick="etapa--; renderEtapa()">VOLTAR</button>` : ''}</div>`;
    document.getElementById('form_content').innerHTML = html;
    showScreen('scr_form');
}

function salvarEtapa() {
    const notas = [];
    let soma = 0;
    const qCount = perguntas5S[etapa].q.length;
    for(let i=0; i<qCount; i++) {
        const v = parseInt(document.getElementById(`n_${i}`).value);
        notas.push(v);
        soma += v;
    }
    const plano = document.getElementById('p_acao').value;
    if(notas.some(n => n < 6) && plano.length < 5) return alert("Plano de ação obrigatório para notas baixas!");
    
    tempAudit.respostas[etapa] = { notas, plano, media: (soma/qCount).toFixed(1) };
    
    if(etapa < 4) { etapa++; renderEtapa(); } 
    else { db.push(tempAudit); localStorage.setItem("db_j2m_5s", JSON.stringify(db)); location.reload(); }
}

async function sincronizarAuto() {
    const btn = document.querySelector('.btn-sync');
    btn.innerText = "⏳ SINCRONIZANDO...";
    try {
        await fetch(WEB_APP_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(db) });
        alert("✅ Sincronização automática enviada!");
    } catch (e) { alert("Erro de conexão."); }
    btn.innerText = "🔄 SINCRONIZAR AGORA";
}

function abrirDash() {
    const fs = document.getElementById('filtros_dash');
    fs.innerHTML = `<select id="f_setor" onchange="renderDash()"><option value="ALL">TODOS OS SETORES</option>` + 
                   [...new Set(db.map(d=>d.setor))].map(s=>`<option value="${s}">${s}</option>`).join('') + `</select>`;
    renderDash();
    showScreen('scr_dash');
}

function renderDash() {
    const sel = document.getElementById('f_setor').value;
    const filtrados = db.filter(d => sel === "ALL" || d.setor === sel);
    if(!filtrados.length) return;
    const a = filtrados[filtrados.length-1];
    const nFinal = (a.respostas.reduce((acc, r) => acc + parseFloat(r.media), 0) / 5).toFixed(1);

    document.getElementById('relatorio_print').innerHTML = `
        <div class="folha-rosto">
            <h1 style="color:var(--primary); font-size:50px;">RELATÓRIO 5S</h1>
            <h2 style="font-size:30px;">${a.setor}</h2>
            <div style="font-size:120px; font-weight:bold; color:var(--primary); margin:20px 0;">${nFinal}</div>
            <p>Data: ${a.data} | Auditor: ${a.auditor}</p>
        </div>
        <div class="chart-container">
            <div class="chart-box"><canvas id="cRadar"></canvas></div>
            <div class="chart-box"><canvas id="cBarra"></canvas></div>
        </div>
        <div class="card"><h3>Planos de Ação</h3>${a.respostas.map((r,i)=>`<p><b>Senso ${i+1}:</b> ${r.plano || 'N/A'}</p>`).join('')}</div>`;
    
    renderCharts(a, nFinal);
}

function renderCharts(a, nFinal) {
    new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ['Seleção', 'Ordenação', 'Limpeza', 'Padronização', 'Disciplina'],
            datasets: [{ label: 'Setor', data: a.respostas.map(r=>r.media), borderColor: '#f06639', fill: true }]
        },
        options: { maintainAspectRatio: false, scales: { r: { min: 0, max: 10 } } }
    });
    new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: {
            labels: [a.setor, 'Meta'],
            datasets: [{ data: [nFinal, 8], backgroundColor: ['#f06639', '#28a745'] }]
        },
        options: { maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { y: { min: 0, max: 10 } } }
    });
}

function showScreen(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }
</script>
</body>
</html>

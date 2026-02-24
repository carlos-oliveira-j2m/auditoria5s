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
        
        label { display: block; font-weight: bold; margin: 10px 0 5px; font-size: 14px; }
        input, select, textarea { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 5px; font-size: 16px; margin-bottom: 10px; }
        .required::after { content: " *"; color: var(--danger); }
        
        button { cursor: pointer; border: none; padding: 15px; border-radius: 5px; font-weight: bold; width: 100%; margin-bottom: 10px; transition: 0.3s; }
        .btn-p { background: var(--primary); color: white; }
        .btn-s { background: var(--dark); color: white; }
        .btn-sync { background: #007bff; color: white; }

        .q-row { border-bottom: 1px solid #eee; padding: 12px 0; }
        .q-text { display: block; margin-bottom: 8px; font-weight: 500; font-size: 14px; }
        
        /* Gráficos Menores como solicitado */
        .chart-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin: 20px 0; }
        .chart-box { background: white; padding: 10px; border-radius: 8px; height: 280px; border: 1px solid #eee; }

        @media print {
            .no-print { display: none !important; }
            .screen { display: block !important; }
            .folha-rosto { height: 95vh; text-align: center; display: flex !important; flex-direction: column; justify-content: center; page-break-after: always; }
            .card { border: none; box-shadow: none; page-break-inside: avoid; }
            .chart-grid { grid-template-columns: 1fr 1fr; }
        }
    </style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - v1.2025</header>

<div class="container">
    <div id="scr_home" class="screen active">
        <div class="card">
            <h3>Página Inicial</h3>
            <label class="required">Setor:</label>
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
                <option value="SALA DE ELETRÔNICOS">SALA DE ELETRÔNICOS</option>
            </select>
            <label class="required">Responsável:</label><input type="text" id="resp">
            <label class="required">Auditor:</label><input type="text" id="aud">
            <label class="required">Data:</label><input type="date" id="dt">
            
            <button class="btn-p" onclick="iniciar()">INICIAR FORMULÁRIO</button>
            <button class="btn-s" onclick="abrirDash()">DASHBOARD / HISTÓRICO</button>
            <button class="btn-sync" onclick="sincronizarAuto()">🔄 SINCRONIZAR AUTOMÁTICO</button>
        </div>
    </div>

    <div id="scr_form" class="screen">
        <div id="form_content"></div>
    </div>

    <div id="scr_dash" class="screen">
        <div class="no-print card">
            <div style="display:grid; grid-template-columns: 1fr 1fr 1fr; gap:10px">
                <select id="f_setor" onchange="renderDash()"></select>
                <select id="f_mes" onchange="renderDash()"></select>
                <select id="f_ano" onchange="renderDash()"></select>
            </div>
        </div>
        <div id="relatorio_final"></div>
        <div class="no-print">
            <button class="btn-p" onclick="window.print()">SALVAR RELATÓRIO PDF</button>
            <button class="btn-s" onclick="location.reload()">VOLTAR</button>
        </div>
    </div>
</div>

<script>
// URL DO SEU APPS SCRIPT QUE JÁ TEMOS
const WEB_APP_URL = "SUA_URL_AQUI"; 

const perguntas = [
    { s: "1 - SELEÇÃO", q: ["Ferramentas e equipamentos são necessários?","Existem itens duplicados na bancada?","Ferramentas acondicionadas corretamente?","Gestão à vista e documentos atualizados?","Avisos e quadros são necessários?"]},
    { s: "2 - ORDENAÇÃO", q: ["Locais de paletes e caixas marcados?","Linhas e marcações visíveis?","Prateleiras e caixas identificadas?","Ferramentas retornam ao lugar?","Documentos fáceis de encontrar?"]},
    { s: "3 - LIMPEZA", q: ["Piso, corredores e escadas limpos?","Máquinas sem vazamentos?","Lixeiras identificadas e limpas?","Iluminação e janelas limpas?","Fontes de sujeira eliminadas?"]},
    { s: "4 - PADRONIZAÇÃO", q: ["Funcionários usam EPIs?","Equip. Emergência desobstruídos?","Quadros organizados?","Padrão de cores respeitado?","Placas em bom estado?"]},
    { s: "5 - AUTODISCIPLINA", q: ["Gestão mantém os padrões?","Checklist realizado no setor?","Missão e Política conhecidos?","Melhorias implementadas?","Áreas comuns limpas?","Ações anteriores atendidas?"]}
];

const legenda = { 10:"Excelente", 8:"Bom (1 desvio)", 6:"Médio (2 desvios)", 4:"Ruim (3 desvios)", 2:"Crítico (4+ desvios)" };

let db = JSON.parse(localStorage.getItem("db_j2m_5s") || "[]");
let audit = JSON.parse(localStorage.getItem("cache_j2m") || "{}");
let etapa = 0;

function iniciar() {
    const s = document.getElementById('setor').value;
    const r = document.getElementById('resp').value;
    const a = document.getElementById('aud').value;
    const d = document.getElementById('dt').value;
    if(!s || !r || !a || !d) return alert("Preencha todos os campos obrigatórios!");
    
    if(!audit.id) audit = { id: Date.now(), setor: s, responsavel: r, auditor: a, data: d, respostas: [] };
    renderEtapa();
}

function renderEtapa() {
    const senso = perguntas[etapa];
    let html = `<div class="card"><h2>${senso.s}</h2>`;
    senso.q.forEach((q, i) => {
        const nota = audit.respostas[etapa]?.notas[i] || "";
        html += `<div class="q-row"><span class="q-text">${i+1}. ${q}</span>
            <select onchange="salvarNota(${i}, this.value)">
                <option value="">-- NOTA --</option>
                ${[10,8,6,4,2].map(n => `<option value="${n}" ${nota == n ? 'selected' : ''}>${n} - ${legenda[n]}</option>`).join('')}
            </select></div>`;
    });
    html += `<label class="required">Plano de Ação (${senso.s}):</label>
        <textarea oninput="audit.respostas[${etapa}].acao = this.value; salvarCache()">${audit.respostas[etapa]?.acao || ""}</textarea>
        <div style="display:flex; gap:10px">
        ${etapa > 0 ? `<button class="btn-s" onclick="etapa--; renderEtapa()">VOLTAR</button>` : ''}
        <button class="btn-p" onclick="validarEtapa()">PRÓXIMO</button></div></div>`;
    document.getElementById('form_content').innerHTML = html;
    show('scr_form');
}

function salvarNota(idx, val) {
    if(!audit.respostas[etapa]) audit.respostas[etapa] = { notas: [], acao: "" };
    audit.respostas[etapa].notas[idx] = Number(val);
    salvarCache();
}

function salvarCache() { localStorage.setItem("cache_j2m", JSON.stringify(audit)); }

function validarEtapa() {
    const r = audit.respostas[etapa];
    if(!r || r.notas.filter(n=>n).length < perguntas[etapa].q.length) return alert("Responda todas as perguntas!");
    if(r.notas.some(n=>n<6) && !r.acao) return alert("Plano de ação obrigatório para notas baixas!");
    
    audit.respostas[etapa].media = (r.notas.reduce((a,b)=>a+b,0)/r.notas.length).toFixed(1);
    etapa++;
    if(etapa < 5) renderEtapa(); else finalizar();
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("db_j2m_5s", JSON.stringify(db));
    localStorage.removeItem("cache_j2m");
    alert("Auditoria salva!");
    location.reload();
}

function renderDash() {
    const selS = document.getElementById('f_setor').value;
    const filtrados = db.filter(a => (selS === "ALL" || a.setor === selS));
    if(!filtrados.length) return;

    const aud = filtrados[filtrados.length-1];
    const nSetor = (aud.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);
    const mGeral = (db.reduce((acc,curr)=>acc+(curr.respostas.reduce((a,b)=>a+Number(b.media),0)/5),0)/db.length).toFixed(1);

    document.getElementById('relatorio_final').innerHTML = `
        <div class="folha-rosto">
            <h1 style="color:var(--primary); font-size:45px">RELATÓRIO 5S</h1>
            <h2 style="font-size:30px">${aud.setor}</h2>
            <div style="font-size:100px; font-weight:bold; color:var(--primary); margin:20px 0">${nSetor}</div>
            <p>Data: ${aud.data} | Auditor: ${aud.auditor}</p>
        </div>
        <div class="chart-grid">
            <div class="chart-box"><canvas id="cRadar"></canvas></div>
            <div class="chart-box"><canvas id="cBarra"></canvas></div>
        </div>
        <div class="card">
            <h3>Planos de Ação por Senso</h3>
            ${aud.respostas.map((r, i) => `<p><b>${perguntas[i].s}:</b> ${r.acao || "N/A"}</p>`).join('')}
        </div>`;
    
    // Gráficos (Radar e Barra) são renderizados aqui...
    renderCharts(aud, nSetor, mGeral);
}

// SINCRONIZAÇÃO AUTOMÁTICA (APENAS UM CLIQUE)
async function sincronizarAuto() {
    if(!WEB_APP_URL || WEB_APP_URL.includes("SUA_URL")) return alert("Cole a URL do seu Apps Script no código!");
    const btn = document.querySelector('.btn-sync');
    btn.innerText = "⏳ Sincronizando...";
    
    try {
        const resp = await fetch(WEB_APP_URL, {
            method: 'POST',
            mode: 'no-cors',
            body: JSON.stringify(db)
        });
        alert("✅ Sincronização concluída!");
    } catch (e) {
        alert("Erro na sincronização.");
    } finally {
        btn.innerText = "🔄 SINCRONIZAR AUTOMÁTICO";

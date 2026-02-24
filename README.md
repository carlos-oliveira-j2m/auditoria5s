<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>J2M 5S - v1.2026</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --primary: #f06639; --dark: #3d3d3d; --bg: #f4f7f6; --danger: #dc3545; --success: #28a745; }
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
        .btn-danger { background: var(--danger); color: white; }
        
        .q-row { border-bottom: 1px solid #eee; padding: 15px 0; }
        .q-text { display: block; margin-bottom: 8px; font-weight: 500; }
        .legenda-nota { font-size: 11px; color: #666; margin-bottom: 5px; display: block; }

        .chart-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin: 20px 0; }
        .chart-box { background: white; padding: 10px; border-radius: 8px; height: 300px; border: 1px solid #eee; }

        @media print {
            .no-print { display: none !important; }
            .screen { display: block !important; }
            .folha-rosto { height: 98vh; text-align: center; padding-top: 150px; page-break-after: always; display: flex !important; flex-direction: column; justify-content: center; }
            .card { border: none; box-shadow: none; page-break-inside: avoid; }
            .chart-grid { grid-template-columns: 1fr 1fr; }
        }
    </style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - v1.2026</header>

<div class="container">
    <div id="scr_home" class="screen active">
        <div class="card">
            <h3>Dados da Auditoria</h3>
            <label class="required">Setor:</label>
            <select id="setor" required>
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
            <label class="required">Responsável da Área:</label>
            <input type="text" id="resp" placeholder="Nome Completo">
            <label class="required">Auditor:</label>
            <input type="text" id="aud" placeholder="Seu Nome">
            <label class="required">Data:</label>
            <input type="date" id="dt">
            
            <button class="btn-p" onclick="iniciar()">INICIAR FORMULÁRIO</button>
            <button class="btn-s" onclick="abrirDash()">DASHBOARD / HISTÓRICO</button>
            <button class="btn-s" style="background:#007bff" onclick="sincronizar()">SINCRONIZAR DADOS (CELULAR/PC)</button>
        </div>
    </div>

    <div id="scr_form" class="screen">
        <div id="form_content"></div>
    </div>

    <div id="scr_dash" class="screen">
        <div class="no-print card">
            <h3>Filtros de Pesquisa</h3>
            <div style="display:grid; grid-template-columns: 1fr 1fr 1fr; gap:10px">
                <select id="f_setor" onchange="renderDash()"></select>
                <select id="f_mes" onchange="renderDash()"></select>
                <select id="f_ano" onchange="renderDash()"></select>
            </div>
        </div>
        <div id="relatorio_print"></div>
        <div class="no-print" id="dash_actions"></div>
        <div class="no-print">
            <button class="btn-p" onclick="window.print()">SALVAR EM PDF</button>
            <button class="btn-s" onclick="location.reload()">VOLTAR AO INÍCIO</button>
        </div>
    </div>
</div>

<script>
const perguntas = [
    { s: "1 - SELEÇÃO", q: [
        "Todas as ferramentas, dispositivos de medição e equipamentos são necessários para o trabalho?",
        "Existem itens duplicados sobre a bancada de trabalho?",
        "As ferramentas são usadas e acondicionadas corretamente?",
        "Quadros de gestão à vista e documentos estão atualizados?",
        "Todos os avisos e quadros informativos atuais são necessários?"
    ]},
    { s: "2 - ORDENAÇÃO", q: [
        "Os locais de armazenamento para paletes e caixas estão marcados?",
        "As linhas e marcações necessárias existem e são visíveis?",
        "Todas as prateleiras e caixas estão identificadas?",
        "As ferramentas retornam ao lugar após o uso?",
        "Arquivos e documentos são fáceis de encontrar?"
    ]},
    { s: "3 - LIMPEZA", q: [
        "Piso, corredores e escadas estão limpos?",
        "Máquinas e equipamentos limpos e sem vazamentos?",
        "Lixeiras identificadas e limpas?",
        "Iluminação, janelas e paredes estão limpas?",
        "As fontes de sujeira foram eliminadas?"
    ]},
    { s: "4 - PADRONIZAÇÃO / SAÚDE", q: [
        "Os funcionários utilizam os EPI's necessários?",
        "Extintores e equipamentos de emergência estão desobstruídos?",
        "Quadros de avisos organizados e sem papéis obsoletos?",
        "Existe padrão de cores e etiquetas e o mesmo é respeitado?",
        "As placas de segurança estão em bom estado?"
    ]},
    { s: "5 - AUTODISCIPLINA", q: [
        "A consciência para necessidade de padrões existe e a gestão mantém?",
        "Check list de autoavaliação é realizado no setor?",
        "Missão, visão e política da qualidade são conhecidos?",
        "Melhorias foram implementadas desde a última auditoria?",
        "Ambientes de uso comum (banheiros/refeitório) estão limpos?",
        "Ações corretivas da auditoria anterior foram atendidas?"
    ]}
];

const legenda = { 10:"Excelente", 8:"Bom (1 desvio)", 6:"Médio (2 desvios)", 4:"Melhorar (3 desvios)", 2:"Crítico (4+ desvios)" };

let db = JSON.parse(localStorage.getItem("db_j2m_final") || "[]");
let audit = JSON.parse(localStorage.getItem("cache_j2m") || "{}");
let etapa = 0;

function iniciar() {
    const s = document.getElementById('setor').value;
    const r = document.getElementById('resp').value;
    const a = document.getElementById('aud').value;
    const d = document.getElementById('dt').value;
    if(!s || !r || !a || !d) return alert("Erro: Todos os campos da capa são obrigatórios!");
    
    if(!audit.id) audit = { id: Date.now(), setor: s, responsavel: r, auditor: a, data: d, respostas: [] };
    renderEtapa();
}

function renderEtapa() {
    const senso = perguntas[etapa];
    let html = `<div class="card"><h2>${senso.s}</h2>`;
    senso.q.forEach((q, i) => {
        const nota = audit.respostas[etapa]?.notas[i] || "";
        html += `<div class="q-row">
            <span class="q-text"><b>${i+1}.</b> ${q}</span>
            <select class="nota-item" onchange="salvarNota(${i}, this.value)">
                <option value="">-- SELECIONE A NOTA --</option>
                ${[10,8,6,4,2].map(n => `<option value="${n}" ${nota == n ? 'selected' : ''}>${n} - ${legenda[n]}</option>`).join('')}
            </select>
        </div>`;
    });
    html += `
        <label class="required">Ação Corretiva para ${senso.s}:</label>
        <textarea id="acao" placeholder="Obrigatório para notas abaixo de 6" oninput="audit.respostas[${etapa}].acao = this.value; cache()">${audit.respostas[etapa]?.acao || ""}</textarea>
        <div style="display:flex; gap:10px">
            ${etapa > 0 ? `<button class="btn-s" onclick="etapa--; renderEtapa()">VOLTAR</button>` : ''}
            <button class="btn-p" onclick="validarEtapa()">PRÓXIMO</button>
        </div>
    </div>`;
    document.getElementById('form_content').innerHTML = html;
    show('scr_form');
}

function salvarNota(idx, val) {
    if(!audit.respostas[etapa]) audit.respostas[etapa] = { notas: [], acao: "" };
    audit.respostas[etapa].notas[idx] = Number(val);
    cache();
}

function cache() { localStorage.setItem("cache_j2m", JSON.stringify(audit)); }

function validarEtapa() {
    const r = audit.respostas[etapa];
    if(!r || r.notas.filter(n => n).length < perguntas[etapa].q.length) return alert("Erro: Todas as perguntas do senso devem ser respondidas!");
    if(r.notas.some(n => n < 6) && (!r.acao || r.acao.length < 5)) return alert("Erro: Ação corretiva é obrigatória para notas menores que 6!");
    
    audit.respostas[etapa].media = (r.notas.reduce((a,b)=>a+b,0)/r.notas.length).toFixed(1);
    etapa++;
    if(etapa < 5) renderEtapa();
    else finalizar();
}

function finalizar() {
    const index = db.findIndex(x => x.id === audit.id);
    if(index > -1) db[index] = audit; else db.push(audit);
    localStorage.setItem("db_j2m_final", JSON.stringify(db));
    localStorage.removeItem("cache_j2m");
    alert("Auditoria Finalizada com Sucesso!");
    location.reload();
}

function abrirDash() {
    const fs = document.getElementById('f_setor');
    fs.innerHTML = `<option value="ALL">TODOS OS SETORES</option>` + [...new Set(db.map(a=>a.setor))].map(s=>`<option value="${s}">${s}</option>`).join('');
    document.getElementById('f_mes').innerHTML = `<option value="ALL">TODOS MESES</option>` + ["Jan","Fev","Mar","Abr","Mai","Jun","Jul","Ago","Set","Out","Nov","Dez"].map((m,i)=>`<option value="${i}">${m}</option>`).join('');
    document.getElementById('f_ano').innerHTML = `<option value="ALL">TODOS ANOS</option>` + [...new Set(db.map(a=>new Date(a.data).getFullYear()))].map(y=>`<option value="${y}">${y}</option>`).join('');
    renderDash();
    show('scr_dash');
}

function renderDash() {
    const selS = document.getElementById('f_setor').value;
    const selM = document.getElementById('f_mes').value;
    const filtrados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return (selS === "ALL" || a.setor === selS) && (selM === "ALL" || d.getMonth() == selM);
    });

    if(!filtrados.length) return document.getElementById('relatorio_print').innerHTML = "Nenhum dado encontrado para o filtro.";

    const aud = filtrados[filtrados.length-1];
    const nSetor = (aud.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);
    const mGeral = (db.reduce((acc,curr)=>acc+(curr.respostas.reduce((a,b)=>a+Number(b.media),0)/5),0)/db.length).toFixed(1);

    document.getElementById('relatorio_print').innerHTML = `
        <div class="folha-rosto">
            <h1 style="color:var(--primary); font-size:50px">RELATÓRIO 5S</h1>
            <h2 style="font-size:35px">${aud.setor}</h2>
            <div style="font-size:110px; font-weight:bold; color:var(--primary); margin:30px 0">${nSetor}</div>
            <p style="font-size:18px"><b>Responsável:</b> ${aud.responsavel} | <b>Auditor:</b> ${aud.auditor}</p>
            <p>Data da Auditoria: ${aud.data}</p>
        </div>
        <div class="card">
            <div class="chart-grid">
                <div class="chart-box"><canvas id="cRadar"></canvas></div>
                <div class="chart-box"><canvas id="cBarra"></canvas></div>
            </div>
        </div>
        <div class="card">
            <h3>Planos de Ação por Senso</h3>
            ${aud.respostas.map((r, i) => `<div style="margin-bottom:10px"><b>${perguntas[i].s}:</b> ${r.acao || "Nenhum desvio relatado."}</div>`).join('')}
        </div>`;

    // Botões de Ação
    document.getElementById('dash_actions').innerHTML = `
        <button class="btn-s" onclick="editarAud(${aud.id})">EDITAR ESTA AUDITORIA</button>
        <button class="btn-danger" onclick="excluirAud(${aud.id})">EXCLUIR ESTA AUDITORIA</button>`;

    new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção","Ordenação","Limpeza","Padronização","Autodisciplina"],
            datasets: [
                { label: 'Setor', data: aud.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Fábrica', data: Array(5).fill(mGeral), borderColor: '#333', borderDash: [5,5], fill: false },
                { label: 'Meta', data: Array(5).fill(8), borderColor: '#28a745', borderDash: [2,2], fill: false }
            ]
        },
        options: { maintainAspectRatio: false, scales: { r: { min: 0, max: 10 } } }
    });

    new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: {
            labels: [aud.setor, "Média Fábrica", "Meta"],
            datasets: [{ label: 'Notas', data: [nSetor, mGeral, 8], backgroundColor: ['#f06639','#5d5a51','#28a745'] }]
        },
        options: { maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { y: { min: 0, max: 10 } } }
    });
}

function editarAud(id) {
    audit = db.find(x => x.id === id);
    etapa = 0;
    renderEtapa();
}

function excluirAud(id) {
    if(confirm("Tem certeza que deseja excluir permanentemente esta auditoria?")) {
        db = db.filter(x => x.id !== id);
        localStorage.setItem("db_j2m_final", JSON.stringify(db));
        location.reload();
    }
}

function sincronizar() {
    const m = prompt("Digite 'E' para EXPORTAR código ou 'I' para IMPORTAR código:");
    if(m?.toUpperCase() === 'E') prompt("Copie este código para o outro dispositivo:", JSON.stringify(db));
    if(m?.toUpperCase() === 'I') {
        const c = prompt("Cole o código aqui:");
        if(c) { db = JSON.parse(c); localStorage.setItem("db_j2m_final", JSON.stringify(db)); location.reload(); }
    }
}

function show(id) { document.querySelectorAll('.screen').forEach(s=>s.classList.remove('active')); document.getElementById(id).classList.add('active'); }
</script>
</body>
</html>

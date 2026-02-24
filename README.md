<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>J2M 5S Enterprise - v1.2025</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --p: #f06639; --s: #3d3d3d; --bg: #f4f7f6; --err: #dc3545; }
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { margin: 0; background: var(--bg); color: #333; }
        header { background: var(--s); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--p); font-weight: bold; }
        .container { max-width: 900px; margin: auto; padding: 15px; }
        .screen { display: none; }
        .active { display: block; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 15px; border-left: 5px solid var(--p); }
        
        label { display: block; font-weight: bold; margin: 10px 0 5px; font-size: 14px; }
        .req::after { content: " (obrigatório)"; color: var(--err); font-size: 11px; }
        input, select, textarea { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 5px; font-size: 16px; margin-bottom: 10px; }
        
        button { cursor: pointer; border: none; padding: 15px; border-radius: 5px; font-weight: bold; width: 100%; margin-bottom: 10px; }
        .btn-p { background: var(--p); color: white; }
        .btn-s { background: var(--s); color: white; }
        .btn-sync { background: #007bff; color: white; }

        .q-row { border-bottom: 1px solid #eee; padding: 15px 0; }
        .q-text { display: block; margin-bottom: 8px; font-weight: 600; font-size: 15px; color: #444; }
        .legenda { font-size: 12px; color: #666; font-style: italic; margin-bottom: 5px; display: block; }
        
        .chart-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin: 20px 0; }
        .chart-box { background: white; padding: 10px; border-radius: 8px; height: 300px; border: 1px solid #ddd; }

        @media print {
            .no-print { display: none !important; }
            .screen { display: block !important; }
            .folha-rosto { height: 95vh; text-align: center; display: flex !important; flex-direction: column; justify-content: center; page-break-after: always; }
        }
    </style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - v1.2025</header>

<div class="container">
    <div id="scr_home" class="screen active">
        <div class="card">
            <h3>Página Inicial - Dados Obrigatórios</h3>
            <label class="req">Setor:</label>
            <select id="setor" required>
                <option value="">-- SELECIONE --</option>
                <option value="ADM FILIAL">ADM FILIAL</option>
                <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
                <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
                <option value="INJEÇÃO">INJEÇÃO</option>
                <option value="MANUTENÇÃO">MANUTENÇÃO</option>
                <option value="MATRIZARIA">MATRIZARIA</option>
                <option value="MONTAGEM">MONTAGEM</option>
                <option value="PPCP">PPCP</option>
                <option value="QUALIDADE">QUALIDADE</option>
                <option value="SALA DE ELETRÔNICOS">SALA DE ELETRÔNICOS</option>
            </select>
            <label class="req">Responsável da Área:</label><input type="text" id="resp" required>
            <label class="req">Auditor:</label><input type="text" id="aud" required>
            <label class="req">Data:</label><input type="date" id="dt" required>
            
            <button class="btn-p" onclick="iniciar()">INICIAR FORMULÁRIO</button>
            <button class="btn-s" onclick="abrirDash()">DASHBOARD</button>
            <button class="btn-sync" onclick="sincronizar()">🔄 SINCRONIZAÇÃO AUTOMÁTICA</button>
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
            <button class="btn-sync" onclick="sincronizar()">SINCRONIZAR FORMULÁRIOS</button>
        </div>
        <div id="relatorio_print"></div>
        <div class="no-print">
            <button class="btn-p" onclick="window.print()">SALVAR RELATÓRIO PDF</button>
            <button class="btn-s" onclick="location.reload()">VOLTAR</button>
        </div>
    </div>
</div>

<script>
// --- CONFIGURAÇÃO ---
const WEB_APP_URL = "SUA_URL_DO_APPS_SCRIPT_AQUI"; 

const perguntas = [
    { s: "1 - SELEÇÃO", q: [
        "Todas as ferramentas, dispositivos de medição e equipamentos são necessários para o trabalho?",
        "Existem itens duplicados sobre a bancada de trabalho?",
        "As ferramentas são usadas e acondicionadas corretamente?",
        "Os quadros de gestão à vista e documentos estão atualizados?",
        "Todos os avisos e quadros informativos atuais são necessários?"
    ]},
    { s: "2 - ORDENAÇÃO", q: [
        "Os locais de armazenamento para paletes e caixas estão marcados?",
        "As linhas e marcações necessárias existem e são visíveis?",
        "Todas as prateleiras e caixas de armazenamento estão identificadas?",
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
    { s: "4 - PADRONIZAÇÃO", q: [
        "Os funcionários utilizam os EPI's necessários corretamente?",
        "Extintores e equipamentos de emergência estão desobstruídos?",
        "Os quadros de avisos estão organizados e sem papéis obsoletos?",
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

const explicacaoNota = {
    10: "Excelente: Sem evidências de desvios.",
    8: "Bom: Apenas 1 evidência de desvio encontrada.",
    6: "Médio: 2 evidências de desvios encontradas.",
    4: "Ruim: 3 evidências de desvios encontradas.",
    2: "Crítico: 4 ou mais evidências encontradas."
};

let db = JSON.parse(localStorage.getItem("db_j2m_5s_v2025") || "[]");
let audit = { respostas: [] };
let etapa = 0;

function iniciar() {
    const s = document.getElementById('setor').value;
    const r = document.getElementById('resp').value;
    const a = document.getElementById('aud').value;
    const d = document.getElementById('dt').value;
    if(!s || !r || !a || !d) return alert("ERRO: Todos os campos da página inicial são obrigatórios!");
    
    audit = { id: Date.now(), setor: s, responsavel: r, auditor: a, data: d, respostas: [] };
    etapa = 0;
    renderEtapa();
}

function renderEtapa() {
    const senso = perguntas[etapa];
    let html = `<div class="card"><h2>${senso.s} (Obrigatório responder todas)</h2>`;
    
    senso.q.forEach((pergunta, i) => {
        html += `<div class="q-row">
            <span class="q-text">${etapa*5 + i + 1}. ${pergunta}</span>
            <span class="legenda">Explicação: Selecione a nota conforme as evidências</span>
            <select class="nota-box" id="n_${etapa}_${i}" required onchange="validarAcaoCorretiva()">
                <option value="">-- SELECIONE A NOTA --</option>
                ${[10,8,6,4,2].map(n => `<option value="${n}">${n} - ${explicacaoNota[n]}</option>`).join('')}
            </select>
        </div>`;
    });

    html += `
        <label id="lbl_acao" class="req">Plano de Ação Corretiva para ${senso.s}:</label>
        <textarea id="acao_txt" placeholder="Descreva a ação... (Obrigatório se nota < 6)"></textarea>
        <div style="display:flex; gap:10px">
            ${etapa > 0 ? `<button class="btn-s" onclick="etapa--; renderEtapa()">VOLTAR</button>` : ''}
            <button class="btn-p" onclick="proximo()">PRÓXIMO</button>
        </div>
    </div>`;
    
    document.getElementById('form_content').innerHTML = html;
    show('scr_form');
}

function proximo() {
    const notas = [];
    const senso = perguntas[etapa];
    
    for(let i=0; i<senso.q.length; i++){
        const v = document.getElementById(`n_${etapa}_${i}`).value;
        if(!v) return alert("Responda TODAS as perguntas para continuar!");
        notas.push(Number(v));
    }

    const acao = document.getElementById('acao_txt').value;
    const mediaSenso = notas.reduce((a,b)=>a+b,0)/notas.length;
    
    if(mediaSenso < 6 && acao.length < 5) {
        document.getElementById('acao_txt').style.borderColor = "red";
        return alert("Ação Corretiva é OBRIGATÓRIA quando a nota do senso é menor que 6!");
    }

    audit.respostas[etapa] = { notas, acao, media: mediaSenso.toFixed(1) };

    if(etapa < 4) {
        etapa++;
        renderEtapa();
    } else {
        finalizar();
    }
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("db_j2m_5s_v2025", JSON.stringify(db));
    alert("Auditoria finalizada com sucesso!");
    location.reload();
}

function renderDash() {
    const selS = document.getElementById('f_setor').value;
    const filtrados = db.filter(a => selS === "ALL" || a.setor === selS);
    if(!filtrados.length) return;

    const a = filtrados[filtrados.length-1];
    const nSetor = (a.respostas.reduce((acc, r) => acc + Number(r.media), 0) / 5).toFixed(1);
    const mGeral = (db.reduce((acc,curr) => acc + (curr.respostas.reduce((x,y)=>x+Number(y.media),0)/5), 0) / db.length).toFixed(1);

    document.getElementById('relatorio_print').innerHTML = `
        <div class="folha-rosto">
            <h1 style="color:var(--p); font-size:45px">AUDITORIA 5S J2M</h1>
            <h2 style="font-size:30px">${a.setor}</h2>
            <div style="font-size:120px; font-weight:bold; color:var(--p); margin:20px 0">${nSetor}</div>
            <p>Auditor: ${a.auditor} | Data: ${a.data}</p>
        </div>
        <div class="chart-grid">
            <div class="chart-box"><canvas id="cRadar"></canvas></div>
            <div class="chart-box"><canvas id="cBarra"></canvas></div>
        </div>
        <div class="card">
            <h3>Planos de Ação por Sensor</h3>
            ${a.respostas.map((r, i) => `<p><b>${perguntas[i].s}:</b> ${r.acao || "Nenhuma ação necessária"}</p>`).join('')}
        </div>`;

    renderCharts(a, nSetor, mGeral);
}

function renderCharts(a, nSetor, mGeral) {
    new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção","Ordenação","Limpeza","Padronização","Disciplina"],
            datasets: [
                { label: 'Setor', data: a.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Fábrica', data: Array(5).fill(mGeral), borderColor: '#333', borderDash: [5,5], fill: false }
            ]
        },
        options: { maintainAspectRatio: false, scales: { r: { min: 0, max: 10 } } }
    });

    new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: {
            labels: [a.setor, "Média Fábrica", "Meta"],
            datasets: [{ data: [nSetor, mGeral, 8.0], backgroundColor: ['#f06639','#555','#28a745'] }]
        },
        options: { maintainAspectRatio: false, plugins: { legend: { display: false } }, scales: { y: { min: 0, max: 10 } } }
    });
}

async function sincronizar() {
    if(WEB_APP_URL.includes("SUA_URL")) return alert("Configure a URL do Apps Script primeiro!");
    try {
        await fetch(WEB_APP_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(db) });
        alert("Sincronização concluída!");
    } catch(e) { alert("Erro ao sincronizar."); }
}

function show(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }
function abrirDash() { 
    document.getElementById('f_setor').innerHTML = `<option value="ALL">TODOS OS SETORES</option>` + [...new Set(db.map(d=>d.setor))].map(s=>`<option value="${s}">${s}</option>`).join('');
    renderDash(); 
    show('scr_dash'); 
}
</script>
</body>
</html>

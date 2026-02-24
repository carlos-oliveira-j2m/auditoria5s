<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Auditoria 5S J2M - Versão Final v1.2025</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
    <style>
        :root { --primary: #f06639; --secondary: #3d3d3d; --bg: #f4f7f6; --danger: #dc3545; }
        * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { margin: 0; background: var(--bg); color: #333; }
        header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; font-size: 20px; }
        .container { max-width: 1000px; margin: 20px auto; padding: 0 15px; }
        .screen { display: none; }
        .active { display: block; }
        .card { background: white; padding: 25px; border-radius: 8px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); margin-bottom: 20px; border-left: 5px solid var(--primary); }
        label { display: block; font-weight: bold; margin-bottom: 5px; font-size: 14px; }
        select, input, textarea { width: 100%; padding: 12px; border-radius: 4px; border: 1px solid #ccc; margin-bottom: 15px; font-size: 16px; }
        .required::after { content: " *"; color: var(--danger); }
        button { cursor: pointer; border: none; padding: 15px; border-radius: 4px; font-weight: bold; text-transform: uppercase; transition: 0.3s; }
        .btn-p { background: var(--primary); color: white; width: 100%; }
        .btn-s { background: var(--secondary); color: white; width: 100%; }
        .btn-edit { background: #ffc107; padding: 5px 10px; font-size: 12px; }
        .btn-del { background: var(--danger); color: white; padding: 5px 10px; font-size: 12px; }
        
        /* Formulário de Auditoria */
        .pergunta-box { border-bottom: 1px solid #eee; padding: 15px 0; }
        .pergunta-texto { font-weight: bold; margin-bottom: 8px; display: block; }
        .legenda-nota { font-size: 12px; color: #666; font-style: italic; margin-bottom: 10px; display: block; }
        .acao-obrigatoria { border: 2px solid var(--danger); padding: 10px; background: #fff5f5; border-radius: 4px; }

        /* Dashboard */
        .flex-filtros { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; background: #fff; padding: 15px; border-radius: 8px; margin-bottom: 20px; }
        .kpi-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px; }
        .kpi-card { background: var(--secondary); color: white; text-align: center; padding: 15px; border-radius: 8px; }
        .kpi-card h2 { margin: 0; color: var(--primary); font-size: 32px; }

        /* Impressão PDF */
        @media print {
            .no-print { display: none !important; }
            .screen { display: block !important; background: white; }
            .active { display: block !important; }
            .card { box-shadow: none; border: 2px solid #333; page-break-inside: avoid; }
            body { background: white; }
            .folha-rosto { text-align: center; page-break-after: always; height: 100vh; padding-top: 100px; }
        }

        .table-print { width: 100%; border-collapse: collapse; margin-top: 20px; }
        .table-print th, .table-print td { border: 1px solid #333; padding: 8px; text-align: left; font-size: 12px; }
        .table-print th { background: #eee; }
    </style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - SISTEMA DE GESTÃO</header>

<div class="container">
    
    <div id="scr_home" class="screen active">
        <div class="card">
            <h3>Nova Auditoria</h3>
            <label class="required">Setor:</label>
            <select id="setor" required>
                <option value="">-- SELECIONE O SETOR --</option>
                <option value="ADM FILIAL">ADM FILIAL</option>
                <option value="ALMOXARIFADO">ALMOXARIFADO</option>
                <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
                <option value="ESPAÇO MAKER">ESPAÇO MAKER</option>
                <option value="ESTOQUE">ESTOQUE</option>
                <option value="INJEÇÃO">INJEÇÃO</option>
                <option value="MANUTENÇÃO">MANUTENÇÃO</option>
                <option value="MATRIZARIA">MATRIZARIA</option>
                <option value="MONTAGEM">MONTAGEM</option>
                <option value="MONTAGEM 2 (IMPETUS E CM GERADOR)">MONTAGEM 2 (IMPETUS E CM GERADOR)</option>
                <option value="PPCP">PPCP</option>
                <option value="QUALIDADE">QUALIDADE</option>
                <option value="RECEBIMENTO">RECEBIMENTO</option>
                <option value="SALA DE ELETRÔNICOS">SALA DE ELETRÔNICOS</option>
            </select>
            
            <label class="required">Responsável da Área:</label>
            <input type="text" id="responsavel" placeholder="Nome do gestor da área" required>
            
            <label class="required">Auditor:</label>
            <input type="text" id="auditor" placeholder="Seu nome" required>
            
            <label class="required">Data:</label>
            <input type="date" id="data" required>
            
            <button class="btn-p" onclick="iniciarAudit()">Iniciar Avaliação</button>
            <button class="btn-s" style="margin-top:10px" onclick="abrirDash()">Acessar Dashboard</button>
        </div>
        
        <div class="card no-print">
            <h4>Registros Locais</h4>
            <div id="lista_home"></div>
        </div>
    </div>

    <div id="scr_audit" class="screen">
        <div id="pergunta_container"></div>
    </div>

    <div id="scr_dash" class="screen">
        <div class="no-print">
            <div class="flex-filtros">
                <div><label>Setor:</label><select id="f_setor" onchange="renderRelatorio()"></select></div>
                <div><label>Mês:</label><select id="f_mes" onchange="renderRelatorio()"></select></div>
                <div><label>Ano:</label><select id="f_ano" onchange="renderRelatorio()"></select></div>
            </div>
            
            <div class="kpi-grid">
                <div class="kpi-card"><h2 id="nota_v_setor">0.0</h2><p>Nota do Setor</p></div>
                <div class="kpi-card"><h2 id="media_v_geral">0.0</h2><p>Média Geral Fábrica</p></div>
            </div>
        </div>

        <div id="print_area">
            </div>

        <div class="no-print" style="margin-top:20px">
            <button class="btn-p" onclick="window.print()">Salvar Relatório em PDF</button>
            <button class="btn-s" style="margin-top:10px" onclick="location.reload()">Voltar ao Início</button>
        </div>
    </div>

</div>

<script>
// --- BANCO DE DADOS DAS 26 PERGUNTAS (CONFORME EXCEL) ---
const questionario = [
    { s: "1 - SELEÇÃO", q: [
        "Todas as ferramentas, dispositivos de medição e equipamentos são necessários para o trabalho diário?",
        "Existem itens duplicados sobre a bancada de trabalho?",
        "As ferramentas são usadas e acondicionadas corretamente?",
        "Os quadros de gestão à vista e documentos estão atualizados?",
        "Todos os avisos e quadros informativos atuais são necessários?"
    ]},
    { s: "2 - ORDENAÇÃO", q: [
        "Os locais de armazenamento para paletes, caixas e carrinhos estão marcados?",
        "As linhas e marcações necessárias existem e são claramente visíveis?",
        "Todas as prateleiras e caixas de armazenamento estão identificadas?",
        "As ferramentas retornam ao lugar após o uso?",
        "Arquivos e documentos são fáceis de encontrar?"
    ]},
    { s: "3 - LIMPEZA", q: [
        "Piso, corredores e escadas estão limpos?",
        "As máquinas e equipamentos estão limpos e sem vazamentos?",
        "As lixeiras estão corretamente identificadas e limpas?",
        "Iluminação, janelas e paredes estão limpas?",
        "As fontes de sujeira foram eliminadas?"
    ]},
    { s: "4 - PADRONIZAÇÃO / SAÚDE", q: [
        "Os funcionários utilizam os EPIs necessários?",
        "Os quadros de avisos estão organizados e sem papéis obsoletos?",
        "Extintores e equipamentos de emergência estão desobstruídos e sinalizados?",
        "Ambientes comuns (banheiros/refeitório) estão limpos?",
        "O padrão de cores e etiquetas é respeitado?",
        "As placas de segurança encontram-se em bom estado?"
    ]},
    { s: "5 - AUTODISCIPLINA", q: [
        "A consciência para padrões existe e a gestão os mantém?",
        "Checklist de autoavaliação é realizado no setor?",
        "Missão, visão e política da qualidade são conhecidos?",
        "Melhorias foram implementadas desde a última auditoria?",
        "As ações corretivas da auditoria anterior foram atendidas?"
    ]}
];

const legendas = {
    10: "Excelente: Sem evidências de desvio",
    8: "Bom: 1 evidência encontrada",
    6: "Média: 2 evidências encontradas",
    4: "Melhorar: 3 evidências encontradas",
    2: "Crítico: 4 ou mais evidências"
};

let db = JSON.parse(localStorage.getItem("j2m_5s_db") || "[]");
let auditAtual = {};
let passoSenso = 0;
let editIdx = -1;

// --- LÓGICA DE NAVEGAÇÃO ---
function iniciarAudit() {
    const s = document.getElementById('setor').value;
    const r = document.getElementById('responsavel').value;
    const a = document.getElementById('auditor').value;
    const d = document.getElementById('data').value;

    if(!s || !r || !a || !d) return alert("Todos os campos de identificação são obrigatórios!");

    if(editIdx === -1) {
        auditAtual = { id: Date.now(), setor: s, responsavel: r, auditor: a, data: d, respostas: [] };
    }
    passoSenso = 0;
    renderPasso();
}

function renderPasso() {
    const senso = questionario[passoSenso];
    let html = `<div class="card"><h2>${senso.s}</h2><p style="color:#666">Responda todas as perguntas abaixo:</p>`;
    
    senso.q.forEach((pergunta, i) => {
        const valAnterior = auditAtual.respostas[passoSenso]?.notas[i] || "";
        html += `
            <div class="pergunta-box">
                <span class="pergunta-texto">${i+1}. ${pergunta}</span>
                <span class="legenda-nota">Selecione a nota baseada nas evidências:</span>
                <select class="nota-item" data-idx="${i}" onchange="checkPlanoObrigatorio(${passoSenso})">
                    <option value="">-- SELECIONE --</option>
                    <option value="10" ${valAnterior==10?'selected':''}>10 - ${legendas[10]}</option>
                    <option value="8" ${valAnterior==8?'selected':''}>8 - ${legendas[8]}</option>
                    <option value="6" ${valAnterior==6?'selected':''}>6 - ${legendas[6]}</option>
                    <option value="4" ${valAnterior==4?'selected':''}>4 - ${legendas[4]}</option>
                    <option value="2" ${valAnterior==2?'selected':''}>2 - ${legendas[2]}</option>
                </select>
            </div>`;
    });

    const planoAnterior = auditAtual.respostas[passoSenso]?.plano || "";
    html += `
        <div id="box_plano_${passoSenso}" style="margin-top:20px">
            <label id="lbl_plano">Plano de Ação Corretiva:</label>
            <textarea id="plano_txt" placeholder="Descreva as ações para os desvios encontrados...">${planoAnterior}</textarea>
        </div>
        <button class="btn-p" onclick="salvarSenso()">Próximo Senso</button>
    </div>`;
    
    document.getElementById('pergunta_container').innerHTML = html;
    show('scr_audit');
    checkPlanoObrigatorio(passoSenso);
}

function checkPlanoObrigatorio(sIdx) {
    const notas = Array.from(document.querySelectorAll('.nota-item')).map(sel => Number(sel.value));
    const temNotaBaixa = notas.some(n => n > 0 && n < 6);
    const box = document.getElementById('plano_txt');
    const lbl = document.getElementById('lbl_plano');
    
    if(temNotaBaixa) {
        box.classList.add('acao-obrigatoria');
        lbl.innerHTML = "Plano de Ação Corretiva <span style='color:red'>(OBRIGATÓRIO PARA NOTAS 2 OU 4)</span>:";
    } else {
        box.classList.remove('acao-obrigatoria');
        lbl.innerHTML = "Plano de Ação Corretiva:";
    }
}

function salvarSenso() {
    const selects = document.querySelectorAll('.nota-item');
    const notas = Array.from(selects).map(s => Number(s.value));
    const plano = document.getElementById('plano_txt').value.trim();

    if(notas.includes(0)) return alert("Você deve responder todas as perguntas do senso!");
    
    const temNotaBaixa = notas.some(n => n < 6);
    if(temNotaBaixa && !plano) return alert("Plano de ação é obrigatório quando há notas menores que 6!");

    const media = (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1);
    auditAtual.respostas[passoSenso] = { notas, plano, media };

    passoSenso++;
    if(passoSenso < questionario.length) {
        renderPasso();
    } else {
        finalizarAuditoria();
    }
}

function finalizarAuditoria() {
    if(editIdx > -1) db[editIdx] = auditAtual;
    else db.push(auditAtual);
    
    localStorage.setItem("j2m_5s_db", JSON.stringify(db));
    alert("Auditoria salva com sucesso!");
    editIdx = -1;
    abrirDash();
}

// --- DASHBOARD E FILTROS ---
function abrirDash() {
    renderFiltros();
    renderRelatorio();
    show('scr_dash');
}

function renderFiltros() {
    const fs = document.getElementById('f_setor');
    const fm = document.getElementById('f_mes');
    const fa = document.getElementById('f_ano');

    const setores = ["ADM FILIAL", "ALMOXARIFADO", "CONFORMAÇÃO", "ESPAÇO MAKER", "ESTOQUE", "INJEÇÃO", "MANUTENÇÃO", "MATRIZARIA", "MONTAGEM", "MONTAGEM 2 (IMPETUS E CM GERADOR)", "PPCP", "QUALIDADE", "RECEBIMENTO", "SALA DE ELETRÔNICOS"];
    fs.innerHTML = setores.map(s => `<option value="${s}">${s}</option>`).join('');

    const meses = ["Janeiro", "Fevereiro", "Março", "Abril", "Maio", "Junho", "Julho", "Agosto", "Setembro", "Outubro", "Novembro", "Dezembro"];
    fm.innerHTML = meses.map((m, i) => `<option value="${i}">${m}</option>`).join('');

    const anos = [...new Set(db.map(a => new Date(a.data + "T00:00:00").getFullYear()))];
    if(anos.length === 0) anos.push(new Date().getFullYear());
    fa.innerHTML = anos.map(a => `<option value="${a}">${a}</option>`).join('');
}

function renderRelatorio() {
    const selS = document.getElementById('f_setor').value;
    const selM = document.getElementById('f_mes').value;
    const selA = document.getElementById('f_ano').value;

    const filtrados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return a.setor === selS && d.getMonth() == selM && d.getFullYear() == selA;
    });

    const area = document.getElementById('print_area');
    if(filtrados.length === 0) {
        area.innerHTML = "<p style='text-align:center; padding:50px;'>Nenhuma auditoria encontrada para este filtro.</p>";
        return;
    }

    const aud = filtrados[filtrados.length-1]; // Pega a última do mês
    const notaFinal = (aud.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);
    
    // Média Geral de todos os setores no filtro selecionado (mês/ano)
    const todasMes = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return d.getMonth() == selM && d.getFullYear() == selA;
    });
    const mediaGeral = todasMes.length > 0 ? (todasMes.reduce((acc, curr) => acc + (curr.respostas.reduce((a,b)=>a+Number(b.media),0)/5), 0) / todasMes.length).toFixed(1) : "0.0";

    document.getElementById('nota_v_setor').innerText = notaFinal;
    document.getElementById('media_v_geral').innerText = mediaGeral;

    let html = `
        <div class="folha-rosto">
            <h1 style="font-size:40px; color:var(--primary)">RELATÓRIO 5S</h1>
            <h2 style="font-size:30px">${aud.setor}</h2>
            <div style="font-size:80px; font-weight:bold; margin:30px 0;">${notaFinal}</div>
            <p><strong>Responsável:</strong> ${aud.responsavel} | <strong>Auditor:</strong> ${aud.auditor}</p>
            <p><strong>Data:</strong> ${aud.data}</p>
            
            <div style="display:grid; grid-template-columns:1fr 1fr; gap:20px; margin-top:30px;">
                <canvas id="cRadar"></canvas>
                <canvas id="cBarra"></canvas>
            </div>
        </div>

        <div class="card" style="margin-top:20px">
            <h3>Checklist Detalhado</h3>
            <table class="table-print">
                <tr><th>Senso / Pergunta</th><th>Nota</th><th>Evidência / Plano de Ação</th></tr>`;

    questionario.forEach((senso, sIdx) => {
        html += `<tr><td colspan="3" style="background:#444; color:white; font-weight:bold">${senso.s} (Média: ${aud.respostas[sIdx].media})</td></tr>`;
        senso.q.forEach((q, qIdx) => {
            const nt = aud.respostas[sIdx].notas[qIdx];
            html += `<tr>
                <td>${q}</td>
                <td style="text-align:center; font-weight:bold">${nt}</td>
                ${qIdx === 0 ? `<td rowspan="${senso.q.length}" style="vertical-align:top"><b>Plano:</b> ${aud.respostas[sIdx].plano || "N/A"}</td>` : ''}
            </tr>`;
        });
    });

    area.innerHTML = html + `</table></div>`;
    
    renderGraficos(aud, todasMes, notaFinal);
}

function renderGraficos(aud, todasMes, notaFinal) {
    // Radar
    new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Saúde", "Disciplina"],
            datasets: [
                { label: 'Nota Setor', data: aud.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Meta', data: [8,8,8,8,8], borderColor: '#28a745', borderDash: [5,5], fill: false }
            ]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    // Barra Comparativa
    const setores = [...new Set(todasMes.map(a => a.setor))];
    const notasSetores = setores.map(s => {
        const auditsSetor = todasMes.filter(a => a.setor === s);
        return (auditsSetor.reduce((acc, curr) => acc + (curr.respostas.reduce((a,b)=>a+Number(b.media),0)/5), 0) / auditsSetor.length).toFixed(1);
    });

    new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: {
            labels: setores,
            datasets: [{ label: 'Nota Final', data: notasSetores, backgroundColor: '#5d5a51' }]
        },
        options: { 
            scales: { y: { min: 0, max: 10 } },
            plugins: { datalabels: { display: true, anchor: 'end', align: 'top' } }
        }
    });
}

function renderListaHome() {
    let html = "";
    db.slice().reverse().forEach((a, idx) => {
        const realIdx = db.length - 1 - idx;
        html += `
            <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid #eee; padding:10px 0;">
                <div><b>${a.data}</b> - ${a.setor}</div>
                <div>
                    <button class="btn-edit" onclick="editarAuditoria(${realIdx})">✎</button>
                    <button class="btn-del" onclick="excluirAuditoria(${realIdx})">X</button>
                </div>
            </div>`;
    });
    document.getElementById('lista_home').innerHTML = html || "<p>Nenhum registro.</p>";
}

function editarAuditoria(idx) {
    editIdx = idx;
    const a = db[idx];
    document.getElementById('setor').value = a.setor;
    document.getElementById('responsavel').value = a.responsavel;
    document.getElementById('auditor').value = a.auditor;
    document.getElementById('data').value = a.data;
    auditAtual = JSON.parse(JSON.stringify(a));
    iniciarAudit();
}

function excluirAuditoria(idx) {
    if(confirm("Tem certeza que deseja excluir?")) {
        db.splice(idx, 1);
        localStorage.setItem("j2m_5s_db", JSON.stringify(db));
        renderListaHome();
        if(document.getElementById('scr_dash').classList.contains('active')) abrirDash();
    }
}

function show(id) {
    document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
}

renderListaHome();
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>J2M Auditoria 5S - v1.2025 Enterprise</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --primary: #f06639; --secondary: #3d3d3d; --danger: #dc3545; --success: #28a745; --bg: #f8f9fa; }
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { margin: 0; background: var(--bg); color: #333; }
        header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; }
        .container { max-width: 900px; margin: auto; padding: 15px; }
        .screen { display: none; }
        .active { display: block; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 15px; border-left: 5px solid var(--primary); }
        
        label { display: block; font-weight: bold; margin: 10px 0 5px; font-size: 14px; }
        input, select, textarea { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 5px; font-size: 16px; }
        .required::after { content: " *"; color: var(--danger); }
        
        button { cursor: pointer; border: none; padding: 15px; border-radius: 5px; font-weight: bold; transition: 0.3s; width: 100%; margin-bottom: 10px; }
        .btn-p { background: var(--primary); color: white; }
        .btn-s { background: var(--secondary); color: white; }
        .btn-back { background: #6c757d; color: white; }
        
        .pergunta { border-bottom: 1px solid #eee; padding: 15px 0; }
        .evidencia { font-size: 12px; color: #666; font-style: italic; display: block; margin-bottom: 5px; }
        .obrigatorio { border: 2px solid var(--danger) !important; background: #fff5f5; }

        .kpi-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px; }
        .kpi-card { background: var(--secondary); color: white; padding: 15px; border-radius: 8px; text-align: center; }
        .kpi-card h2 { color: var(--primary); margin: 0; }

        @media print {
            .no-print { display: none !important; }
            .screen { display: block !important; }
            .card { border: 1px solid #333; box-shadow: none; page-break-inside: avoid; }
            .folha-rosto { text-align: center; page-break-after: always; padding-top: 50px; }
        }
        
        /* Celular */
        @media (max-width: 600px) {
            .kpi-row { grid-template-columns: 1fr; }
            button { padding: 12px; }
        }
    </style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div class="container">

    <div id="scr_home" class="screen active">
        <div class="card">
            <h3>Identificação Obrigatória</h3>
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
                <option value="MONTAGEM 2 (IMPETUS E CM GERADOR)">MONTAGEM 2 (IMPETUS E CM GERADOR)</option>
                <option value="PPCP">PPCP</option>
                <option value="QUALIDADE">QUALIDADE</option>
                <option value="RECEBIMENTO">RECEBIMENTO</option>
                <option value="SALA DE ELETRÔNICOS">SALA DE ELETRÔNICOS</option>
            </select>
            
            <label class="required">Responsável da Área:</label>
            <input type="text" id="responsavel" placeholder="Nome do Responsável">
            
            <label class="required">Auditor:</label>
            <input type="text" id="auditor" placeholder="Nome do Auditor">
            
            <label class="required">Data:</label>
            <input type="date" id="data">
            
            <button class="btn-p" onclick="iniciarAudit()">INICIAR AVALIAÇÃO</button>
            <button class="btn-s" onclick="abrirDash()">DASHBOARD / RELATÓRIOS</button>
            <button class="btn-s" style="background:#007bff" onclick="sincronizarDados()">SINCRONIZAR (CELULAR/PC)</button>
        </div>
    </div>

    <div id="scr_audit" class="screen">
        <div id="pergunta_cont"></div>
    </div>

    <div id="scr_dash" class="screen">
        <div class="no-print card">
            <h3>Filtros</h3>
            <div style="display:grid; grid-template-columns: 1fr 1fr 1fr; gap:10px">
                <select id="f_setor" onchange="renderRelatorio()"></select>
                <select id="f_mes" onchange="renderRelatorio()"></select>
                <select id="f_ano" onchange="renderRelatorio()"></select>
            </div>
        </div>

        <div class="kpi-row no-print">
            <div class="kpi-card"><h2 id="kpi_setor">0.0</h2><p>Nota Setor</p></div>
            <div class="kpi-card"><h2 id="kpi_geral">0.0</h2><p>Média Geral</p></div>
        </div>

        <div id="area_print"></div>

        <div class="no-print">
            <button class="btn-p" onclick="window.print()">SALVAR PDF</button>
            <button class="btn-back" onclick="location.reload()">VOLTAR</button>
        </div>
    </div>
</div>

<script>
// --- BANCO DE DADOS (26 PERGUNTAS EXTRAÍDAS DO EXCEL) ---
const questionario = [
    { s: "1 - SELEÇÃO", q: [
        "Todas as ferramentas, dispositivos de medição e equipamentos são necessários?",
        "Existem itens duplicados sobre a bancada de trabalho?",
        "As ferramentas são usadas e acondicionadas corretamente?",
        "Quadros de gestão à vista e documentos estão atualizados?",
        "Todos os avisos e quadros informativos são necessários?"
    ]},
    { s: "2 - ORDENAÇÃO", q: [
        "Locais de paletes e caixas estão marcados?",
        "Linhas e marcações são claramente visíveis?",
        "Prateleiras e caixas estão identificadas?",
        "As ferramentas retornam ao lugar após o uso?",
        "Arquivos e documentos são fáceis de encontrar?"
    ]},
    { s: "3 - LIMPEZA", q: [
        "Piso, corredores e escadas estão limpos?",
        "Máquinas e equipamentos sem vazamentos e limpos?",
        "Lixeiras identificadas e limpas?",
        "Iluminação, janelas e paredes estão limpas?",
        "As fontes de sujeira foram eliminadas?"
    ]},
    { s: "4 - PADRONIZAÇÃO / SAÚDE", q: [
        "Funcionários utilizam os EPI's necessários?",
        "Quadros de avisos organizados e sem papéis obsoletos?",
        "Extintores e equipamentos de emergência desobstruídos?",
        "Ambientes comuns (banheiros/refeitório) limpos?",
        "Padrão de cores e etiquetas é respeitado?",
        "Placas de segurança em bom estado?"
    ]},
    { s: "5 - AUTODISCIPLINA", q: [
        "A gestão mantém os padrões estabelecidos?",
        "Checklist de autoavaliação é realizado no setor?",
        "Missão, visão e política da qualidade são conhecidos?",
        "Melhorias implementadas desde a última auditoria?",
        "Ações corretivas da auditoria anterior foram atendidas?"
    ]}
];

const legendas = { 10:"Excelente (0 evidências)", 8:"Bom (1 evidência)", 6:"Média (2 evidências)", 4:"Melhorar (3 evidências)", 2:"Crítico (4+ evidências)" };

let db = JSON.parse(localStorage.getItem("j2m_5s_db") || "[]");
let audit = JSON.parse(localStorage.getItem("j2m_temp") || "{}");
let step = 0;

// --- FUNÇÕES DE NAVEGAÇÃO ---
function iniciarAudit() {
    const s = document.getElementById('setor').value;
    const r = document.getElementById('responsavel').value;
    const a = document.getElementById('auditor').value;
    const d = document.getElementById('data').value;

    if(!s || !r || !a || !d) return alert("Preencha todos os campos obrigatórios!");

    if(!audit.id) {
        audit = { id: Date.now(), setor: s, responsavel: r, auditor: a, data: d, respostas: [] };
    }
    renderPasso();
}

function renderPasso() {
    const senso = questionario[step];
    let html = `<div class="card">
        <h3>${senso.s}</h3>
        <p style="font-size:12px; color:var(--primary)">Obrigatório: Plano de ação se houver notas 2 ou 4.</p>`;
    
    senso.q.forEach((pergunta, i) => {
        const notaSalva = (audit.respostas[step]?.notas[i]) || "";
        html += `<div class="pergunta">
            <span><b>${i+1}.</b> ${pergunta}</span>
            <select class="nota-item" onchange="autoSalvar(${i}, this.value)">
                <option value="">-- NOTA --</option>
                ${[10,8,6,4,2].map(n => `<option value="${n}" ${notaSalva == n ? 'selected' : ''}>${n} - ${legendas[n]}</option>`).join('')}
            </select>
        </div>`;
    });

    html += `
        <label>Plano de Ação Corretiva (Senso ${step+1}):</label>
        <textarea id="plano_txt" oninput="audit.respostas[${step}].plano = this.value; autoSalvar()">${audit.respostas[step]?.plano || ""}</textarea>
        <div style="display:flex; gap:10px; margin-top:20px">
            ${step > 0 ? `<button class="btn-back" onclick="voltar()">VOLTAR</button>` : ''}
            <button class="btn-p" onclick="proximo()">PRÓXIMO</button>
        </div>
    </div>`;

    document.getElementById('pergunta_cont').innerHTML = html;
    show('scr_audit');
}

function autoSalvar(qIdx, val) {
    if(!audit.respostas[step]) audit.respostas[step] = { notas: [], plano: "" };
    if(qIdx !== undefined) audit.respostas[step].notas[qIdx] = Number(val);
    localStorage.setItem("j2m_temp", JSON.stringify(audit));
}

function voltar() { step--; renderPasso(); }

function proximo() {
    const notas = Array.from(document.querySelectorAll('.nota-item')).map(s => Number(s.value));
    const plano = document.getElementById('plano_txt').value.trim();

    if(notas.includes(0)) return alert("Responda todas as perguntas!");
    if(notas.some(n => n < 6) && !plano) {
        document.getElementById('plano_txt').classList.add('obrigatorio');
        return alert("Plano de ação obrigatório para notas menores que 6!");
    }

    audit.respostas[step].media = (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1);
    step++;

    if(step < questionario.length) renderPasso();
    else finalizar();
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("j2m_5s_db", JSON.stringify(db));
    localStorage.removeItem("j2m_temp");
    alert("Auditoria Finalizada!");
    location.reload();
}

// --- DASHBOARD ---
function abrirDash() {
    renderFiltros();
    renderRelatorio();
    show('scr_dash');
}

function renderFiltros() {
    const fs = document.getElementById('f_setor');
    const fm = document.getElementById('f_mes');
    const fa = document.getElementById('f_ano');

    const setores = [...new Set(db.map(a => a.setor))].sort();
    fs.innerHTML = `<option value="ALL">Todos Setores</option>` + setores.map(s => `<option value="${s}">${s}</option>`).join('');

    const meses = ["Jan", "Fev", "Mar", "Abr", "Mai", "Jun", "Jul", "Ago", "Set", "Out", "Nov", "Dez"];
    fm.innerHTML = `<option value="ALL">Todos Meses</option>` + meses.map((m, i) => `<option value="${i}">${m}</option>`).join('');

    const anos = [...new Set(db.map(a => new Date(a.data + "T00:00:00").getFullYear()))];
    fa.innerHTML = `<option value="ALL">Todos Anos</option>` + anos.map(a => `<option value="${a}">${a}</option>`).join('');
}

function renderRelatorio() {
    const selS = document.getElementById('f_setor').value;
    const selM = document.getElementById('f_mes').value;
    const selA = document.getElementById('f_ano').value;

    const filtrados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return (selS === "ALL" || a.setor === selS) && (selM === "ALL" || d.getMonth() == selM) && (selA === "ALL" || d.getFullYear() == selA);
    });

    if(filtrados.length === 0) return document.getElementById('area_print').innerHTML = "Nenhum dado.";

    const aud = filtrados[filtrados.length-1];
    const notaSetor = (aud.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);
    const mediaGeral = (db.reduce((acc, curr) => acc + (curr.respostas.reduce((a,b)=>a+Number(b.media),0)/5), 0) / db.length).toFixed(1);

    document.getElementById('kpi_setor').innerText = notaSetor;
    document.getElementById('kpi_geral').innerText = mediaGeral;

    let html = `
        <div class="folha-rosto">
            <h1 style="color:var(--primary)">RELATÓRIO AUDITORIA 5S</h1>
            <h2>${aud.setor}</h2>
            <div style="font-size:60px; font-weight:bold; color:var(--primary)">${notaSetor}</div>
            <p>Auditor: ${aud.auditor} | Responsável: ${aud.responsavel} | Data: ${aud.data}</p>
        </div>
        <div class="card">
            <div style="display:grid; grid-template-columns: 1fr 1fr; gap:10px">
                <canvas id="cRadar"></canvas>
                <canvas id="cBarra"></canvas>
            </div>
        </div>
        <div class="card">
            <h3>Planos de Ação</h3>
            <table style="width:100%; border-collapse:collapse; font-size:12px">
                <tr style="background:#eee"><th>Senso</th><th>Plano de Ação</th></tr>
                ${aud.respostas.map((r, i) => `<tr><td>Senso ${i+1}</td><td>${r.plano || "N/A"}</td></tr>`).join('')}
            </table>
        </div>`;

    document.getElementById('area_print').innerHTML = html;
    setTimeout(() => gerGraficos(aud, filtrados, notaSetor, mediaGeral), 200);
}

function gerGraficos(aud, filtrados, notaSetor, mediaGeral) {
    new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Saúde", "Disciplina"],
            datasets: [
                { label: 'Setor', data: aud.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Média Geral', data: [mediaGeral,mediaGeral,mediaGeral,mediaGeral,mediaGeral], borderColor: '#333', borderDash: [5,5], fill: false },
                { label: 'Meta 8.0', data: [8,8,8,8,8], borderColor: '#28a745', borderDash: [2,2], fill: false }
            ]
        },
        options: { scales: { r: { min:0, max:10 } } }
    });

    const setores = [...new Set(filtrados.map(a => a.setor))];
    const notas = setores.map(s => {
        const x = filtrados.filter(a => a.setor === s);
        return (x.reduce((acc, curr) => acc + (curr.respostas.reduce((a,b)=>a+Number(b.media),0)/5), 0) / x.length).toFixed(1);
    });

    new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: {
            labels: setores,
            datasets: [
                { label: 'Notas Setores', data: notas, backgroundColor: '#5d5a51' },
                { label: 'Meta', data: setores.map(()=>8), type: 'line', borderColor: '#28a745', fill: false }
            ]
        },
        options: { scales: { y: { min:0, max:10 } } }
    });
}

function sincronizarDados() {
    const modo = prompt("Digite 'EXPORTAR' para gerar código ou 'IMPORTAR' para colar dados:");
    if(modo?.toUpperCase() === 'EXPORTAR') {
        prompt("Copie este código e cole no outro dispositivo:", JSON.stringify(db));
    } else if(modo?.toUpperCase() === 'IMPORTAR') {
        const dados = prompt("Cole o código aqui:");
        if(dados) {
            db = JSON.parse(dados);
            localStorage.setItem("j2m_5s_db", JSON.stringify(db));
            alert("Dados Importados!");
            location.reload();
        }
    }
}

function show(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }
</script>
</body>
</html>

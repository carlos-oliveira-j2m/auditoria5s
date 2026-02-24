<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>J2M 5S - v1.2025</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --primary: #f06639; --secondary: #3d3d3d; --danger: #dc3545; --bg: #f8f9fa; }
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { margin: 0; background: var(--bg); }
        .no-print { display: block; }
        header { background: var(--secondary); color: white; padding: 10px; text-align: center; border-bottom: 4px solid var(--primary); }
        .container { max-width: 800px; margin: auto; padding: 10px; }
        .screen { display: none; }
        .active { display: block; }
        .card { background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin-bottom: 15px; border-left: 5px solid var(--primary); }
        
        label { display: block; font-weight: bold; margin: 8px 0 4px; font-size: 13px; }
        input, select, textarea { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; font-size: 15px; }
        button { cursor: pointer; border: none; padding: 12px; border-radius: 5px; font-weight: bold; width: 100%; margin-bottom: 8px; }
        .btn-p { background: var(--primary); color: white; }
        .btn-s { background: var(--secondary); color: white; }
        
        /* Gráficos Menores */
        .chart-box { max-width: 400px; margin: auto; height: 300px; }
        
        .pergunta { border-bottom: 1px solid #eee; padding: 10px 0; font-size: 14px; }
        .nota-item { width: auto; min-width: 120px; float: right; }

        @media print {
            .no-print { display: none !important; }
            .screen { display: block !important; }
            .folha-rosto { height: 100vh; text-align: center; padding-top: 80px; page-break-after: always; }
            .card { border: none; }
        }
    </style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M</header>

<div class="container">
    <div id="scr_home" class="screen active">
        <div class="card">
            <label class="required">Setor:</label>
            <select id="setor" onchange="autoSalvarCache()">
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
            </select>
            <label>Responsável:</label><input type="text" id="responsavel" oninput="autoSalvarCache()">
            <label>Auditor:</label><input type="text" id="auditor" oninput="autoSalvarCache()">
            <label>Data:</label><input type="date" id="data" oninput="autoSalvarCache()">
            <button class="btn-p" onclick="iniciar()">INICIAR</button>
            <button class="btn-s" onclick="abrirDash()">DASHBOARD</button>
            <button class="btn-s" style="background:#007bff" onclick="sinc()">SINCRONIZAR</button>
        </div>
    </div>

    <div id="scr_audit" class="screen">
        <div id="pergunta_cont"></div>
    </div>

    <div id="scr_dash" class="screen">
        <div class="no-print card">
            <div style="display:grid; grid-template-columns: 1fr 1fr 1fr; gap:8px">
                <select id="f_setor" onchange="renderRelatorio()"></select>
                <select id="f_mes" onchange="renderRelatorio()"></select>
                <select id="f_ano" onchange="renderRelatorio()"></select>
            </div>
        </div>
        <div id="relatorio_final"></div>
        <div class="no-print">
            <button class="btn-p" onclick="window.print()">SALVAR PDF</button>
            <button class="btn-s" onclick="location.reload()">VOLTAR</button>
        </div>
    </div>
</div>

<script>
const questoes = [
    { s: "1 - SELEÇÃO", q: ["Ferramentas necessárias?", "Itens duplicados?", "Acondicionamento correto?", "Gestão à vista atualizada?", "Avisos necessários?"]},
    { s: "2 - ORDENAÇÃO", q: ["Marcação de paletes?", "Sinalização visível?", "Identificação de prateleiras?", "Retorno ao lugar?", "Documentos fáceis de achar?"]},
    { s: "3 - LIMPEZA", q: ["Piso e corredores?", "Máquinas sem vazamento?", "Lixeiras limpas?", "Iluminação/Janelas?", "Eliminação de sujeira?"]},
    { s: "4 - PADRONIZAÇÃO", q: ["Uso de EPIs?", "Quadros organizados?", "Emergência desobstruída?", "Ambientes comuns?", "Padrão de cores?", "Placas em bom estado?"]},
    { s: "5 - AUTODISCIPLINA", q: ["Gestão mantém padrão?", "Checklist realizado?", "Missão/Política conhecidos?", "Melhorias feitas?", "Ações anteriores atendidas?"]}
];

let db = JSON.parse(localStorage.getItem("db_5s") || "[]");
let audit = JSON.parse(localStorage.getItem("cache_5s") || "{}");
let etapa = 0;

function autoSalvarCache() {
    audit.setor = document.getElementById('setor').value;
    audit.responsavel = document.getElementById('responsavel').value;
    audit.auditor = document.getElementById('auditor').value;
    audit.data = document.getElementById('data').value;
    localStorage.setItem("cache_5s", JSON.stringify(audit));
}

function iniciar() {
    if(!audit.setor || !audit.data) return alert("Preencha Setor e Data!");
    if(!audit.id) audit.id = Date.now();
    if(!audit.respostas) audit.respostas = [];
    renderEtapa();
}

function renderEtapa() {
    const senso = questoes[etapa];
    let html = `<div class="card"><h3>${senso.s}</h3>`;
    senso.q.forEach((q, i) => {
        const nota = audit.respostas[etapa]?.notas[i] || "";
        html += `<div class="pergunta"><span>${q}</span>
            <select class="nota-item" onchange="gravarNota(${i}, this.value)">
                <option value="">--</option>
                <option value="10" ${nota==10?'selected':''}>10 - Exc</option>
                <option value="8" ${nota==8?'selected':''}>8 - Bom</option>
                <option value="6" ${nota==6?'selected':''}>6 - Méd</option>
                <option value="4" ${nota==4?'selected':''}>4 - Ruim</option>
                <option value="2" ${nota==2?'selected':''}>2 - Crít</option>
            </select></div>`;
    });
    html += `<label>Ação Corretiva:</label><textarea id="txt_acao" oninput="audit.respostas[${etapa}].plano = this.value; autoSalvarCache()">${audit.respostas[etapa]?.plano || ""}</textarea>
        <div style="display:flex; gap:5px; margin-top:10px">
        ${etapa>0?'<button class="btn-s" onclick="etapa--;renderEtapa()">VOLTAR</button>':''}
        <button class="btn-p" onclick="proximo()">PRÓXIMO</button></div></div>`;
    document.getElementById('pergunta_cont').innerHTML = html;
    document.querySelectorAll('.screen').forEach(s=>s.classList.remove('active'));
    document.getElementById('scr_audit').classList.add('active');
}

function gravarNota(i, v) {
    if(!audit.respostas[etapa]) audit.respostas[etapa] = { notas: [], plano: "" };
    audit.respostas[etapa].notas[i] = Number(v);
    autoSalvarCache();
}

function proximo() {
    const n = audit.respostas[etapa]?.notas || [];
    const p = audit.respostas[etapa]?.plano || "";
    if(n.filter(x=>x).length < questoes[etapa].q.length) return alert("Responda todas!");
    if(n.some(x=>x<6) && p.length < 5) return alert("Ação corretiva obrigatória!");
    
    audit.respostas[etapa].media = (n.reduce((a,b)=>a+b,0)/n.length).toFixed(1);
    etapa++;
    if(etapa < 5) renderEtapa(); else { db.push(audit); localStorage.setItem("db_5s", JSON.stringify(db)); localStorage.removeItem("cache_5s"); location.reload(); }
}

function abrirDash() {
    const fs = document.getElementById('f_setor');
    fs.innerHTML = `<option value="TODOS">TODOS OS SETORES</option>` + [...new Set(db.map(a=>a.setor))].map(s=>`<option value="${s}">${s}</option>`).join('');
    document.getElementById('f_mes').innerHTML = ["Jan","Fev","Mar","Abr","Mai","Jun","Jul","Ago","Set","Out","Nov","Dez"].map((m,i)=>`<option value="${i}">${m}</option>`).join('');
    document.getElementById('f_ano').innerHTML = `<option value="2026">2026</option>`;
    renderRelatorio();
    document.querySelectorAll('.screen').forEach(s=>s.classList.remove('active'));
    document.getElementById('scr_dash').classList.add('active');
}

function renderRelatorio() {
    const selS = document.getElementById('f_setor').value;
    const selM = document.getElementById('f_mes').value;
    const filtrados = db.filter(a => (selS === "TODOS" || a.setor === selS) && new Date(a.data+"T00:00:00").getMonth() == selM);
    
    if(!filtrados.length) { document.getElementById('relatorio_final').innerHTML = "Sem registros."; return; }
    
    const aud = filtrados[filtrados.length-1];
    const nFinal = (aud.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);
    const mGeral = (db.reduce((acc,curr)=>acc+(curr.respostas.reduce((a,b)=>a+Number(b.media),0)/5),0)/db.length).toFixed(1);

    let html = `
        <div class="folha-rosto">
            <h1 style="color:var(--primary)">RELATÓRIO 5S</h1>
            <h2>${selS === "TODOS" ? "MÉDIA GERAL EMPRESA" : aud.setor}</h2>
            <div style="font-size:80px; font-weight:bold; color:var(--primary)">${selS === "TODOS" ? mGeral : nFinal}</div>
            <p>Data: ${aud.data}</p>
        </div>
        <div class="card">
            <div class="chart-box"><canvas id="cRadar"></canvas></div>
            <div class="chart-box"><canvas id="cBarra"></canvas></div>
        </div>`;

    // SÓ EXIBE PLANO DE AÇÃO SE UM SETOR ESPECÍFICO FOR SELECIONADO
    if(selS !== "TODOS") {
        html += `<div class="card"><h3>Planos de Ação - ${aud.setor}</h3>
        ${aud.respostas.map((r,i)=>`<p><b>Senso ${i+1}:</b> ${r.plano||"Sem desvios."}</p>`).join('')}</div>`;
    }

    document.getElementById('relatorio_final').innerHTML = html;
    
    setTimeout(() => {
        new Chart(document.getElementById('cRadar'), {
            type: 'radar',
            data: {
                labels: ["Seleção","Ordenação","Limpeza","Saúde","Disciplina"],
                datasets: [{ label: 'Setor', data: aud.respostas.map(r=>r.media), borderColor: '#f06639', fill: true },
                           { label: 'Meta', data: [8,8,8,8,8], borderColor: '#28a745', borderDash: [5,5], fill: false }]
            },
            options: { maintainAspectRatio: false, scales: { r: { min:0, max:10 } } }
        });
        new Chart(document.getElementById('cBarra'), {
            type: 'bar',
            data: {
                labels: [aud.setor, "Média Fábrica", "Meta"],
                datasets: [{ data: [nFinal, mGeral, 8], backgroundColor: ['#f06639','#5d5a51','#28a745'] }]
            },
            options: { maintainAspectRatio: false, plugins:{legend:{display:false}}, scales: { y: { min:0, max:10 } } }
        });
    }, 200);
}

function sinc() {
    const op = prompt("1: Exportar | 2: Importar");
    if(op=="1") prompt("Código:", JSON.stringify(db));
    if(op=="2") { const c = prompt("Cole:"); if(c){ db=JSON.parse(c); localStorage.setItem("db_5s", JSON.stringify(db)); location.reload(); }}
}
</script>
</body>
</html>

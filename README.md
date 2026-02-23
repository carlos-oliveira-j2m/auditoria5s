<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v2.5 Enterprise</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --green: #10b981; }
    *{box-sizing:border-box; font-family: 'Segoe UI', sans-serif;}
    body { margin:0; background: #f4f7f6; color: #333; }
    header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-size: 20px; font-weight: bold; }
    .screen { display:none; padding:20px; max-width: 1200px; margin: auto; }
    .active { display:block; animation: fadeIn 0.4s; }
    .card { background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); border-left: 8px solid var(--primary); margin-bottom: 25px; }
    
    .kpi-container { display: flex; gap: 15px; margin-bottom: 20px; }
    .kpi-card { background: var(--secondary); color: white; padding: 15px; border-radius: 10px; flex: 1; text-align: center; border-bottom: 5px solid var(--primary); }
    .kpi-card h2 { margin: 0; font-size: 35px; color: var(--primary); }

    .filter-bar { background: #eee; padding: 15px; border-radius: 8px; display: grid; grid-template-columns: repeat(auto-fit, minmax(180px, 1fr)); gap: 10px; margin-bottom: 20px; }
    
    button { border: none; padding: 12px 20px; border-radius: 6px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.3s; }
    .btn-next { background: var(--primary); width: 100%; font-size: 16px; margin-top:10px;}
    .btn-pdf { background: var(--green); }
    .btn-edit { background: #ffc107; color: black; font-size: 11px; padding: 5px 10px; }
    .btn-del { background: var(--red); font-size: 11px; padding: 5px 10px; }
    
    label { font-weight: bold; display: block; margin-top: 10px; color: #444; }
    select, input, textarea { width: 100%; padding: 10px; border: 2px solid #ddd; border-radius: 6px; margin-top: 5px; font-size: 14px; }

    .teia-box { height: 650px; width: 100%; background: white; padding: 10px; border-radius: 12px; }
    .plano-item { padding: 10px; border-bottom: 1px solid #eee; }
    .plano-item b { color: var(--primary); text-transform: uppercase; font-size: 12px; }

    /* ESTILO PARA PDF */
    @media print { 
        .no-print, header, .btn-next, .btn-pdf, .filter-bar { display:none !important; } 
        .screen { display:block !important; padding: 0; }
        .card { border: 1px solid #ddd; box-shadow: none; margin-bottom: 10px; page-break-inside: avoid; }
        .teia-box { height: 500px !important; }
        body { background: white; }
        #lista_relatorios { display: none; } /* Esconde o histórico no PDF do gráfico */
    }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header id="headerMain">J2M - AUDITORIA 5S v2.5</header>

<div id="home" class="screen active">
    <div class="card">
        <h2>Identificação</h2>
        <label>Setor:</label>
        <select id="setor">
            <option value="">Selecione...</option>
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="ENG. SOLVE">ENG. SOLVE</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="LOGISTICA FILIAL">LOGISTICA FILIAL</option>
            <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MANUTENÇÃO">MANUTENÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
            <option value="SALA ELETRONICOS">SALA ELETRONICOS</option>
            <option value="SALA IMPETUS">SALA IMPETUS</option>
            <option value="EXPEDIÇÃO">EXPEDIÇÃO</option>
        </select>
        <label>Responsável pela Área:</label><input type="text" id="responsavel">
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data_audit">
        <button class="btn-next" onclick="iniciarAuditoria()">Iniciar</button>
        <button style="background:var(--secondary); width:100%; margin-top:10px" onclick="abrirDashboard()">Dashboard</button>
    </div>
</div>

<div id="senso_screen" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="card no-print">
        <h3>Filtros de Visão</h3>
        <div class="filter-bar">
            <select id="fSetor" onchange="filtrar()"></select>
            <select id="fMes" onchange="filtrar()"></select>
            <button class="btn-pdf" onclick="window.print()">Gerar PDF do Setor</button>
            <button style="background:#444" onclick="location.reload()">Início</button>
        </div>
    </div>

    <div class="card" id="pdf_header" style="display:none; text-align:center;">
        <h1 style="color:var(--primary); margin:0;">RELATÓRIO DE AUDITORIA 5S</h1>
        <p id="pdf_info" style="font-weight:bold;"></p>
    </div>

    <div class="kpi-container no-print">
        <div class="kpi-card"><h2 id="kpi_media">0.0</h2>Média Setor</div>
        <div class="kpi-card"><h2 id="kpi_total">0</h2>Auditorias</div>
    </div>

    <div class="card">
        <div class="teia-box"><canvas id="chartRadar"></canvas></div>
    </div>

    <div class="card" id="box_acoes">
        <h3>Ações Corretivas por Senso</h3>
        <div id="lista_acoes"></div>
    </div>
    
    <div class="card no-print">
        <h3>Histórico de Registros (Editar/Excluir)</h3>
        <div id="lista_relatorios"></div>
    </div>
</div>

<script>
Chart.register(ChartDataLabels);

const nomesSensos = ["Seleção", "Ordenação", "Limpeza", "Padronização", "Autodisciplina"];
const checklist = [
    { s: "1 - SELEÇÃO", p: ["Ferramentas necessárias?", "Itens duplicados?", "Acondicionamento?", "Docs OK?", "Informativos?"] },
    { s: "2 - ORDENAÇÃO", p: ["Locais marcados?", "Linhas visíveis?", "Etiquetagem?", "Material limpeza?", "Identificação pessoal?", "Objetos pessoais?"] },
    { s: "3 - LIMPEZA", p: ["Bancadas/Máquinas?", "Piso/Calçadas?", "Condições técnicas?", "Salas descanso?", "Coleta seletiva?"] },
    { s: "4 - PADRONIZAÇÃO", p: ["Documentação?", "Ergonomia?", "Lixeiras?", "Segurança/Placas?"] },
    { s: "5 - AUTODISCIPLINA", p: ["Consciência?", "Autoavaliação?", "Missão/Visão?", "Uso de EPIs?", "Gestão à vista?", "Ações anteriores?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db_v3") || "[]");
let audit = {}, etapa = 0, editandoId = null;
let myChart;

function iniciarAuditoria() {
    if(!setor.value || !auditor.value || !data_audit.value) return alert("Preencha os campos obrigatórios!");
    
    if(!editandoId) {
        audit = { 
            id: Date.now(), setor: setor.value, responsavel: responsavel.value, auditor: auditor.value, 
            data: data_audit.value, mes: new Date(data_audit.value + 'T00:00:00').getMonth(), respostas: [] 
        };
    }
    etapa = 0; mostrarSenso();
}

function mostrarSenso() {
    const senso = checklist[etapa];
    let html = `<div class="card"><h2>${senso.s}</h2>`;
    senso.p.forEach((p, i) => {
        html += `<label>${p}</label><select class="pergunta-item" id="p_${i}">
            <option value="">Nota...</option>
            <option value="10">10 (Excelente)</option><option value="8">8 (Bom)</option>
            <option value="6">6 (Médio)</option><option value="4">4 (Melhorar)</option><option value="2">2 (Crítico)</option>
        </select>`;
    });
    html += `<label>Plano de Ação (Obrigatório p/ Notas < 6):</label>
             <textarea id="plano_txt" rows="3"></textarea>
             <button class="btn-next" onclick="validarProximo()">Próximo</button></div>`;
    document.getElementById("senso_screen").innerHTML = html;
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById("senso_screen").classList.add("active");
}

function validarProximo() {
    const selects = document.querySelectorAll(".pergunta-item");
    let soma = 0, respondidas = true;
    selects.forEach(s => { if(s.value==="") respondidas=false; soma += Number(s.value); });
    
    if(!respondidas) return alert("Responda todas as perguntas!");
    
    const media = soma / selects.length;
    const plano = document.getElementById("plano_txt").value.trim();
    if(media < 6 && plano === "") return alert("Plano de ação obrigatório para média abaixo de 6!");

    if(!editandoId) audit.respostas.push({ media, plano });
    else audit.respostas[etapa] = { media, plano };

    etapa++;
    if(etapa < 5) mostrarSenso();
    else {
        if(editandoId) db[db.findIndex(x => x.id === editandoId)] = audit;
        else db.push(audit);
        localStorage.setItem("j2m_db_v3", JSON.stringify(db));
        editandoId = null; abrirDashboard();
    }
}

function abrirDashboard() {
    const fSet = document.getElementById("fSetor");
    const fMes = document.getElementById("fMes");
    fSet.innerHTML = '<option value="TODOS">Todos Setores</option>';
    [...new Set(db.map(x => x.setor))].forEach(s => fSet.innerHTML += `<option value="${s}">${s}</option>`);
    fMes.innerHTML = '<option value="TODOS">Todos Meses</option>';
    const meses = ["Jan","Fev","Mar","Abr","Mai","Jun","Jul","Ago","Set","Out","Nov","Dez"];
    [...new Set(db.map(x => x.mes))].sort().forEach(m => fMes.innerHTML += `<option value="${m}">${meses[m]}</option>`);
    
    filtrar();
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById("dashboard").classList.add("active");
}

function filtrar() {
    const fs = document.getElementById("fSetor").value;
    const fm = document.getElementById("fMes").value;
    let filtrados = db.filter(a => (fs === "TODOS" || a.setor === fs) && (fm === "TODOS" || a.mes == fm));
    renderizar(filtrados, fs);
}

function renderizar(dados, setorSel) {
    if(dados.length === 0) return;
    const ult = dados[dados.length - 1];
    
    // Média do Setor no Filtro
    const mSetor = [0,1,2,3,4].map(i => (dados.reduce((acc, a) => acc + a.respostas[i].media, 0) / dados.length).toFixed(1));
    // Média de TODOS os setores (Geral)
    const mGeral = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + a.respostas[i].media, 0) / db.length).toFixed(1));

    document.getElementById("kpi_media").innerText = (mSetor.reduce((a,b)=>Number(a)+Number(b),0)/5).toFixed(1);
    document.getElementById("kpi_total").innerText = dados.length;
    document.getElementById("pdf_info").innerText = `Setor: ${ult.setor} | Responsável: ${ult.responsavel} | Data: ${ult.data}`;
    document.getElementById("pdf_header").style.display = window.matchMedia('print').matches ? "block" : "none";

    if(myChart) myChart.destroy();
    myChart = new Chart(document.getElementById("chartRadar"), {
        type: 'radar',
        data: {
            labels: nomesSensos,
            datasets: [
                { label: 'Nota Setor', data: mSetor, borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.3)', pointRadius: 5, datalabels: { display: true } },
                { label: 'Média Fábrica', data: mGeral, borderColor: '#5d5a51', borderDash: [5,5], fill: false, pointRadius: 0 },
                { label: 'Meta 8.0', data: [8,8,8,8,8], borderColor: '#10b981', borderDash: [2,2], fill: false, pointRadius: 0 }
            ]
        },
        options: { 
            maintainAspectRatio: false, 
            scales: { r: { min: 0, max: 10, ticks: { display: false } } },
            plugins: { datalabels: { backgroundColor: '#fff', borderRadius: 4, font: { weight: 'bold' } } }
        }
    });

    // Ações Corretivas (Somente as do último relatório do filtro)
    let acoesHtml = "";
    ult.respostas.forEach((r, i) => {
        if(r.plano) acoesHtml += `<div class="plano-item"><b>${nomesSensos[i]}:</b> ${r.plano}</div>`;
    });
    document.getElementById("lista_acoes").innerHTML = acoesHtml || "Nenhuma ação corretiva pendente.";

    // Histórico (Para Editar/Excluir)
    let histHtml = '<table style="width:100%; font-size:12px; border-collapse:collapse;">';
    dados.slice().reverse().forEach(a => {
        histHtml += `<tr style="border-bottom:1px solid #eee"><td>${a.data}</td><td>${a.setor}</td>
        <td><button class="btn-edit" onclick="editar(${a.id})">E</button><button class="btn-del" onclick="excluir(${a.id})">X</button></td></tr>`;
    });
    document.getElementById("lista_relatorios").innerHTML = histHtml + '</table>';
}

function excluir(id) { if(confirm("Excluir?")) { db = db.filter(x => x.id !== id); localStorage.setItem("j2m_db_v3", JSON.stringify(db)); filtrar(); } }

function editar(id) {
    editandoId = id;
    audit = JSON.parse(JSON.stringify(db.find(x => x.id === id)));
    document.getElementById("setor").value = audit.setor;
    document.getElementById("responsavel").value = audit.responsavel;
    document.getElementById("auditor").value = audit.auditor;
    document.getElementById("data_audit").value = audit.data;
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById("home").classList.add("active");
    alert("Dados carregados. Clique em 'Iniciar' para editar os sensos.");
}

window.onbeforeprint = () => { document.getElementById("pdf_header").style.display = "block"; };
window.onafterprint = () => { document.getElementById("pdf_header").style.display = "none"; };
</script>
</body>
</html>

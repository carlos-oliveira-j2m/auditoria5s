<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v3.0 Sincronizada</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --green: #10b981; }
    *{box-sizing:border-box; font-family: 'Segoe UI', sans-serif;}
    body { margin:0; background: #f4f7f6; color: #333; }
    header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-size: 20px; font-weight: bold; }
    .screen { display:none; padding:15px; max-width: 1000px; margin: auto; }
    .active { display:block; animation: fadeIn 0.4s; }
    .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); border-left: 8px solid var(--primary); margin-bottom: 20px; }
    .filter-bar { background: #eee; padding: 15px; border-radius: 8px; display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 10px; margin-bottom: 15px; }
    button { border: none; padding: 12px; border-radius: 6px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.3s; }
    .btn-next { background: var(--primary); width: 100%; font-size: 16px; margin-top:15px;}
    .btn-pdf { background: var(--green); }
    .btn-sync { background: #007bff; }
    .btn-edit { background: #ffc107; color: black; padding: 5px 8px; font-size: 10px; }
    .btn-del { background: var(--red); padding: 5px 8px; font-size: 10px; }
    label { font-weight: bold; display: block; margin-top: 12px; color: #444; font-size: 14px; }
    select, input, textarea { width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 6px; margin-top: 5px; font-size: 14px; }
    .teia-box { height: 500px; width: 100%; background: white; display: flex; justify-content: center; }
    .plano-item { padding: 8px; border-bottom: 1px solid #eee; font-size: 13px; }
    .plano-item b { color: var(--primary); text-transform: uppercase; font-size: 11px; display: block; }
    @media print { .no-print, header, .filter-bar, button { display:none !important; } .screen { display:block !important; padding: 0; } .card { border: 1px solid #eee; box-shadow: none; margin-bottom: 10px; border-left: 4px solid var(--primary); } .teia-box { height: 450px !important; } body { background: white; } #lista_relatorios_box { display: none !important; } }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M</header>

<div id="home" class="screen active">
    <div class="card">
        <h2>Identificação da Auditoria</h2>
        <label>Setor:</label>
        <select id="setor">
            <option value="">Selecione...</option>
            <option value="ADM FILIAL">ADM FILIAL</option><option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="ENG. SOLVE">ENG. SOLVE</option><option value="INJEÇÃO">INJEÇÃO</option>
            <option value="LOGISTICA FILIAL">LOGISTICA FILIAL</option><option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MANUTENÇÃO">MANUTENÇÃO</option><option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option><option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option><option value="EXPEDIÇÃO">EXPEDIÇÃO</option>
            <option value="SALA ELETRONICOS">SALA ELETRONICOS</option><option value="SALA IMPETUS">SALA IMPETUS</option>
        </select>
        <label>Responsável pela Área:</label><input type="text" id="responsavel">
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data_audit">
        <button class="btn-next" onclick="iniciarAuditoria()">Iniciar Avaliação</button>
        <button class="btn-sync" style="width:100%; margin-top:10px" onclick="sincronizarComNuvem()">Sincronizar Dados Nuvem</button>
        <button style="background:var(--secondary); width:100%; margin-top:10px" onclick="abrirDashboard()">Ver Dashboard</button>
    </div>
</div>

<div id="senso_screen" class="screen"></div>

<div id="dashboard" class="screen">
    <div id="pdf_header" style="display:none; text-align:center; margin-bottom:20px; border-bottom:2px solid var(--primary); padding-bottom:10px;">
        <h2 style="margin:0; color:var(--primary);">RELATÓRIO DE AUDITORIA 5S - J2M</h2>
        <p id="pdf_dados_top" style="font-weight:bold; font-size:14px;"></p>
    </div>

    <div class="card no-print">
        <div class="filter-bar">
            <select id="fSetor" onchange="filtrar()"></select>
            <select id="fMes" onchange="filtrar()"></select>
            <button class="btn-pdf" onclick="window.print()">Gerar PDF</button>
            <button style="background:#444" onclick="location.reload()">Início</button>
        </div>
    </div>

    <div class="card">
        <div class="teia-box"><canvas id="chartRadar"></canvas></div>
    </div>

    <div class="card" id="box_acoes">
        <h3>Ações Corretivas por Senso</h3>
        <div id="lista_acoes"></div>
    </div>
    
    <div class="card no-print" id="lista_relatorios_box">
        <h3>Histórico e Gestão</h3>
        <div id="lista_relatorios"></div>
    </div>
</div>

<script>
Chart.register(ChartDataLabels);
const URL_GOOGLE = "https://script.google.com/macros/s/AKfycbxmSwLVu1PQoT5LWtKFWr0BKfWc6L2fuI2FFiNPuhsGRWIUOMNlVonKNkCfIQVrglKB/exec";
const nomesSensos = ["Seleção", "Ordenação", "Limpeza", "Padronização", "Autodisciplina"];

const checklist = [
    { s: "SELEÇÃO", p: ["Ferramentas necessárias?", "Itens duplicados?", "Acondicionamento correto?", "Docs atualizados?", "Informativos necessários?"]},
    { s: "ORDENAÇÃO", p: ["Demarcação de piso?", "Corredores visíveis?", "Prateleiras identificadas?", "Limpeza organizada?", "Setores identificados?", "Uso pessoal em local próprio?"]},
    { s: "LIMPEZA", p: ["Máquinas limpas?", "Piso sem resíduos?", "Condições técnicas OK?", "Áreas comuns limpas?", "Coleta seletiva OK?"]},
    { s: "PADRONIZAÇÃO", p: ["Documentação padrão?", "Ergonomia atendida?", "Lixeiras corretas?", "Segurança OK?"]},
    { s: "AUTODISCIPLINA", p: ["Consciência de padrões?", "Autoavaliação feita?", "Missão/Visão conhecidas?", "Uso de EPIs?", "Gestão à vista?", "Ações anteriores feitas?"]}
];

let db = JSON.parse(localStorage.getItem("j2m_db_v3") || "[]");
let audit = {}, etapa = 0, editandoId = null;
let myChart;

function iniciarAuditoria() {
    if(!setor.value || !auditor.value || !data_audit.value) return alert("Preencha os campos!");
    if(!editandoId) {
        audit = { id: Date.now(), setor: setor.value, responsavel: responsavel.value, auditor: auditor.value, data: data_audit.value, mes: new Date(data_audit.value + 'T00:00:00').getMonth(), respostas: [] };
    }
    etapa = 0; mostrarSenso();
}

function mostrarSenso() {
    const senso = checklist[etapa];
    let html = `<div class="card"><h2>${etapa + 1}º Senso: ${senso.s}</h2>`;
    senso.p.forEach((p, i) => {
        html += `<label>${p}</label><select class="pergunta-item" id="p_${i}">
            <option value="">Nota...</option>
            <option value="10">Excelente: Sem evidências (10)</option><option value="8">Bom: 1 evidência (8)</option>
            <option value="6">Média: 2 evidências (6)</option><option value="4">Melhorar: 3 evidências (4)</option>
            <option value="2">Crítico: 4+ evidências (2)</option>
        </select>`;
    });
    html += `<label>Plano de Ação (Obrigatório p/ Notas < 6):</label><textarea id="plano_txt" rows="3"></textarea>
             <button class="btn-next" onclick="validarProximo()">Próximo</button></div>`;
    document.getElementById("senso_screen").innerHTML = html;
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById("senso_screen").classList.add("active");
}

function validarProximo() {
    const selects = document.querySelectorAll(".pergunta-item");
    let soma = 0, ok = true;
    selects.forEach(s => { if(s.value==="") ok=false; soma += Number(s.value); });
    if(!ok) return alert("Responda tudo!");
    const media = soma / selects.length;
    const plano = document.getElementById("plano_txt").value.trim();
    if(media < 6 && plano === "") return alert("Plano de ação obrigatório!");

    if(!editandoId) audit.respostas.push({ media, plano });
    else audit.respostas[etapa] = { media, plano };

    etapa++;
    if(etapa < 5) mostrarSenso();
    else {
        if(editandoId) { db[db.findIndex(x => x.id === editandoId)] = audit; audit.editandoId = editandoId; }
        else db.push(audit);
        localStorage.setItem("j2m_db_v3", JSON.stringify(db));
        fetch(URL_GOOGLE, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
        editandoId = null; abrirDashboard();
    }
}

function sincronizarComNuvem() {
    alert("A ligar à nuvem... Por favor, aguarde.");
    fetch(URL_GOOGLE).then(res => res.json()).then(data => {
        // Converte dados do Sheets (Array) para o formato do App
        const formatados = data.slice(1).map(row => ({
            data: row[9], setor: row[1], responsavel: row[2], auditor: row[3], id: row[10],
            respostas: [ {media:row[4]}, {media:row[5]}, {media:row[6]}, {media:row[7]}, {media:row[8]} ]
        }));
        db = formatados;
        localStorage.setItem("j2m_db_v3", JSON.stringify(db));
        alert("Sincronização concluída!");
        abrirDashboard();
    });
}

function abrirDashboard() {
    const fSet = document.getElementById("fSetor");
    const fMes = document.getElementById("fMes");
    fSet.innerHTML = '<option value="TODOS">Todos Setores</option>';
    [...new Set(db.map(x => x.setor))].forEach(s => fSet.innerHTML += `<option value="${s}">${s}</option>`);
    fMes.innerHTML = '<option value="TODOS">Todos Meses</option>';
    const meses = ["Jan","Fev","Mar","Abr","Mai","Jun","Jul","Ago","Set","Out","Nov","Dez"];
    [...new Set(db.map(x => new Date(x.data + 'T00:00:00').getMonth()))].sort().forEach(m => fMes.innerHTML += `<option value="${m}">${meses[m]}</option>`);
    filtrar();
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById("dashboard").classList.add("active");
}

function filtrar() {
    const fs = document.getElementById("fSetor").value;
    const fm = document.getElementById("fMes").value;
    let filtrados = db.filter(a => (fs === "TODOS" || a.setor === fs) && (fm === "TODOS" || new Date(a.data + 'T00:00:00').getMonth() == fm));
    renderizar(filtrados);
}

function renderizar(dados) {
    if(dados.length === 0) return;
    const ult = dados[dados.length - 1];
    const mSetor = [0,1,2,3,4].map(i => (dados.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / dados.length).toFixed(1));
    const mGeral = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / db.length).toFixed(1));

    document.getElementById("pdf_dados_top").innerText = `Setor: ${ult.setor} | Responsável: ${ult.responsavel} | Data: ${ult.data}`;

    if(myChart) myChart.destroy();
    myChart = new Chart(document.getElementById("chartRadar"), {
        type: 'radar',
        data: {
            labels: nomesSensos,
            datasets: [
                { label: 'Nota Atual', data: mSetor, borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.3)', pointRadius: 5, datalabels: { display: true } },
                { label: 'Média Fábrica', data: mGeral, borderColor: '#5d5a51', borderDash: [5,5], fill: false, pointRadius: 0 },
                { label: 'Meta 8.0', data: [8,8,8,8,8], borderColor: '#10b981', borderDash: [2,2], fill: false, pointRadius: 0 }
            ]
        },
        options: { maintainAspectRatio: false, scales: { r: { min: 0, max: 10, ticks: { display: false } } }, plugins: { datalabels: { backgroundColor: '#fff', borderRadius: 4, font: { weight: 'bold' } } } }
    });

    let acoesHtml = "";
    ult.respostas.forEach((r, i) => { if(r.plano) acoesHtml += `<div class="plano-item"><b>${nomesSensos[i]}</b> ${r.plano}</div>`; });
    document.getElementById("lista_acoes").innerHTML = acoesHtml || "Nenhuma ação pendente.";

    let histHtml = '<table style="width:100%; border-collapse:collapse; font-size:12px;">';
    dados.slice().reverse().forEach(a => {
        histHtml += `<tr style="border-bottom:1px solid #eee"><td>${a.data}</td><td>${a.setor}</td><td style="text-align:right"><button class="btn-edit" onclick="editar(${a.id})">E</button> <button class="btn-del" onclick="excluir(${a.id})">X</button></td></tr>`;
    });
    document.getElementById("lista_relatorios").innerHTML = histHtml + '</table>';
}

function excluir(id) { if(confirm("Excluir?")) { db = db.filter(x => x.id !== id); localStorage.setItem("j2m_db_v3", JSON.stringify(db)); filtrar(); } }

function editar(id) {
    editandoId = id; audit = JSON.parse(JSON.stringify(db.find(x => x.id === id)));
    document.getElementById("setor").value = audit.setor; document.getElementById("responsavel").value = audit.responsavel;
    document.getElementById("auditor").value = audit.auditor; document.getElementById("data_audit").value = audit.data;
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById("home").classList.add("active");
}
window.onbeforeprint = () => { document.getElementById("pdf_header").style.display = "block"; };
window.onafterprint = () => { document.getElementById("pdf_header").style.display = "none"; };
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v2.2025</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --green: #10b981; }
    *{box-sizing:border-box; font-family: 'Segoe UI', sans-serif;}
    body { margin:0; background: #f4f7f6; color: #333; }
    header { background: var(--secondary); color: white; padding: 20px; text-align: center; border-bottom: 5px solid var(--primary); font-size: 24px; font-weight: bold; }
    .screen { display:none; padding:20px; max-width: 1200px; margin: auto; }
    .active { display:block; animation: fadeIn 0.4s; }
    .card { background: white; padding: 25px; border-radius: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); border-left: 8px solid var(--primary); margin-bottom: 25px; }
    
    .kpi-container { display: flex; gap: 20px; margin-bottom: 20px; flex-wrap: wrap; }
    .kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 10px; flex: 1; text-align: center; border-bottom: 5px solid var(--primary); }
    .kpi-card h2 { margin: 0; font-size: 45px; color: var(--primary); }

    .filter-bar { background: #eee; padding: 15px; border-radius: 8px; display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 10px; margin-bottom: 20px; }
    
    button { border: none; padding: 15px 25px; border-radius: 6px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.3s; }
    .btn-next { background: var(--primary); width: 100%; font-size: 18px; }
    .btn-pdf { background: var(--green); }
    
    label { font-weight: bold; display: block; margin-top: 15px; color: #444; }
    select, input, textarea { width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 6px; margin-top: 5px; font-size: 15px; }
    select:focus, textarea:focus { border-color: var(--primary); outline: none; }

    .teia-box { height: 700px; width: 100%; background: white; padding: 20px; border-radius: 12px; margin-top: 20px; }
    .tabela-resumo { width: 100%; border-collapse: collapse; margin-top: 20px; background: white; }
    .tabela-resumo th, .tabela-resumo td { border: 1px solid #ddd; padding: 12px; text-align: left; }
    .tabela-resumo th { background: #5d5a51; color: white; }

    .erro-alerta { color: var(--red); font-weight: bold; font-size: 14px; margin-top: 5px; display: none; }
    .obrigatorio { border: 2px solid var(--red) !important; background: #fff1f1; }

    @media print { .no-print { display:none !important; } .screen { display:block; } }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header>J2M - SISTEMA DE AUDITORIA 5S</header>

<div id="home" class="screen active">
    <div class="card">
        <h2>Identificação da Auditoria</h2>
        <label>Setor:</label>
        <select id="setor">
            <option value="">Selecione o Setor...</option>
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="ENG. SOLVE">ENG. SOLVE</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MANUTENÇÃO">MANUTENÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
        </select>
        <label>Responsável pela Área:</label><input type="text" id="responsavel" placeholder="Nome do responsável">
        <label>Auditor:</label><input type="text" id="auditor" placeholder="Nome do auditor">
        <label>Data:</label><input type="date" id="data_audit">
        <br><br>
        <button class="btn-next" onclick="iniciarAuditoria()">Iniciar Auditoria</button>
        <button style="background:var(--secondary); width:100%; margin-top:10px" onclick="abrirDashboard()">Ir para Dashboard</button>
    </div>
</div>

<div id="senso_screen" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="no-print filter-bar">
        <div><label>Filtrar Setor</label><select id="fSetor" onchange="filtrar()"></select></div>
        <div><label>Filtrar Mês</label><select id="fMes" onchange="filtrar()"></select></div>
        <div><label>Filtrar Ano</label><select id="fAno" onchange="filtrar()"></select></div>
        <div style="display:flex; align-items:flex-end;"><button class="btn-pdf" onclick="window.print()">Gerar PDF</button></div>
        <div style="display:flex; align-items:flex-end;"><button style="background:#444" onclick="location.reload()">Novo</button></div>
    </div>

    <div class="kpi-container">
        <div class="kpi-card"><h2 id="kpi_media">0.0</h2>Média Geral</div>
        <div class="kpi-card"><h2 id="kpi_total">0</h2>Auditorias Filtro</div>
    </div>

    <div class="card teia-box">
        <canvas id="chartRadar"></canvas>
    </div>
    
    <div class="card">
        <h3>Histórico e Planos de Ação</h3>
        <div id="lista_relatorios"></div>
    </div>
</div>

<script>
Chart.register(ChartDataLabels);

const checklist = [
    { s: "1 - SELEÇÃO", p: ["Ferramentas necessárias?", "Itens duplicados?", "Acondicionamento correto?", "Docs/Checklists OK?", "Informativos necessários?"] },
    { s: "2 - ORDENAÇÃO", p: ["Locais marcados?", "Linhas visíveis?", "Etiquetagem OK?", "Material limpeza OK?", "Identificação pessoal?", "Objetos pessoais OK?"] },
    { s: "3 - LIMPEZA", p: ["Bancadas/Máquinas?", "Piso/Calçadas?", "Condições técnicas?", "Salas descanso?", "Coleta seletiva?"] },
    { s: "4 - PADRONIZAÇÃO", p: ["Documentação padrão?", "Ergonomia?", "Lixeiras OK?", "Segurança/Placas?"] },
    { s: "5 - AUTODISCIPLINA", p: ["Consciência?", "Autoavaliação?", "Missão/Visão?", "Uso de EPIs?", "Gestão à vista?", "Ações anteriores?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db_v2") || "[]");
let audit = {}, etapa = 0;
let myChart;

function iniciarAuditoria() {
    if(!setor.value || !responsavel.value || !auditor.value || !data_audit.value) {
        alert("Preencha todos os campos da identificação!");
        return;
    }
    audit = { 
        id: Date.now(), 
        setor: setor.value, 
        responsavel: responsavel.value, 
        auditor: auditor.value, 
        data: data_audit.value,
        mes: new Date(data_audit.value + 'T00:00:00').getMonth(),
        ano: new Date(data_audit.value + 'T00:00:00').getFullYear(),
        respostas: [] 
    };
    etapa = 0; mostrarSenso();
}

function mostrarSenso() {
    const senso = checklist[etapa];
    let html = `<div class="card"><h2>Senso de ${senso.s}</h2>`;
    
    senso.p.forEach((pergunta, i) => {
        html += `<label>${pergunta}</label>
        <select class="pergunta-item" id="p_${i}">
            <option value="">Selecione uma nota...</option>
            <option value="10">Excelente: Sem evidências (10)</option>
            <option value="8">Bom: 1 evidência (8)</option>
            <option value="6">Média: 2 evidências (6)</option>
            <option value="4">Melhorar: 3 evidências (4)</option>
            <option value="2">Crítico: 4+ evidências (2)</option>
        </select>`;
    });

    html += `<label>Plano de Ação (Obrigatório se Média < 6):</label>
             <textarea id="plano_txt" rows="3" placeholder="Descreva as melhorias..."></textarea>
             <p id="msg_erro" class="erro-alerta">Média baixa! O preenchimento do Plano de Ação é obrigatório.</p>
             <button class="btn-next" style="margin-top:20px" onclick="validarESalvar()">Próximo Passo</button></div>`;
    
    const div = document.getElementById("senso_screen");
    div.innerHTML = html;
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    div.classList.add("active");
}

function validarESalvar() {
    const selects = document.querySelectorAll(".pergunta-item");
    let todasRespondidas = true;
    let soma = 0;

    selects.forEach(s => {
        if(s.value === "") todasRespondidas = false;
        soma += Number(s.value);
    });

    if(!todasRespondidas) {
        alert("Responda todas as perguntas para prosseguir!");
        return;
    }

    const mediaSenso = soma / selects.length;
    const plano = document.getElementById("plano_txt").value.trim();

    if(mediaSenso < 6 && plano === "") {
        document.getElementById("plano_txt").classList.add("obrigatorio");
        document.getElementById("msg_erro").style.display = "block";
        return;
    }

    audit.respostas.push({ media: mediaSenso, plano: plano });
    etapa++;

    if(etapa < 5) mostrarSenso();
    else {
        db.push(audit);
        localStorage.setItem("j2m_db_v2", JSON.stringify(db));
        abrirDashboard();
    }
}

function abrirDashboard() {
    const fSet = document.getElementById("fSetor");
    const fMes = document.getElementById("fMes");
    const fAno = document.getElementById("fAno");

    // Preencher Filtros
    fSet.innerHTML = '<option value="TODOS">Todos Setores</option>';
    [...new Set(db.map(x => x.setor))].forEach(s => fSet.innerHTML += `<option value="${s}">${s}</option>`);
    
    fMes.innerHTML = '<option value="TODOS">Todos Meses</option>';
    const meses = ["Jan","Fev","Mar","Abr","Mai","Jun","Jul","Ago","Set","Out","Nov","Dez"];
    [...new Set(db.map(x => x.mes))].sort().forEach(m => fMes.innerHTML += `<option value="${m}">${meses[m]}</option>`);

    fAno.innerHTML = '<option value="TODOS">Todos Anos</option>';
    [...new Set(db.map(x => x.ano))].sort().forEach(a => fAno.innerHTML += `<option value="${a}">${a}</option>`);

    filtrar();
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById("dashboard").classList.add("active");
}

function filtrar() {
    const fS = document.getElementById("fSetor").value;
    const fM = document.getElementById("fMes").value;
    const fA = document.getElementById("fAno").value;

    let filtrados = db.filter(a => {
        return (fS === "TODOS" || a.setor === fS) &&
               (fM === "TODOS" || a.mes == fM) &&
               (fA === "TODOS" || a.ano == fA);
    });

    renderCharts(filtrados);
}

function renderCharts(dados) {
    if(dados.length === 0) {
        document.getElementById("kpi_media").innerText = "0.0";
        document.getElementById("kpi_total").innerText = "0";
        return;
    }

    const mediasSensos = [0,1,2,3,4].map(i => {
        let soma = dados.reduce((acc, a) => acc + a.respostas[i].media, 0);
        return (soma / dados.length).toFixed(1);
    });

    const mediaGeral = (mediasSensos.reduce((a,b) => Number(a)+Number(b), 0) / 5).toFixed(1);
    document.getElementById("kpi_media").innerText = mediaGeral;
    document.getElementById("kpi_total").innerText = dados.length;

    if(myChart) myChart.destroy();
    myChart = new Chart(document.getElementById("chartRadar"), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Autodisciplina"],
            datasets: [{
                label: 'Desempenho 5S',
                data: mediasSensos,
                borderColor: '#f06639',
                backgroundColor: 'rgba(240,102,57,0.3)',
                pointRadius: 8,
                pointBackgroundColor: '#f06639',
                datalabels: { color: '#000', font: { weight: 'bold', size: 14 }, align: 'top' }
            }]
        },
        options: {
            maintainAspectRatio: false,
            scales: { r: { min: 0, max: 10, ticks: { stepSize: 2 } } },
            plugins: { datalabels: { display: true } }
        }
    });

    // Tabela de Histórico
    let html = `<table class="tabela-resumo"><tr><th>Data</th><th>Setor</th><th>Média</th><th>Planos de Ação</th><th>Ação</th></tr>`;
    dados.slice().reverse().forEach(a => {
        const nf = (a.respostas.reduce((x,y)=>x+y.media,0)/5).toFixed(1);
        let planos = a.respostas.map((r,i) => r.plano ? `<b>S${i+1}:</b> ${r.plano}` : "").filter(x => x !== "").join("<br>");
        html += `<tr>
            <td>${a.data}</td>
            <td>${a.setor}</td>
            <td><b>${nf}</b></td>
            <td style="font-size:12px">${planos || "Nenhum"}</td>
            <td class="no-print"><button class="btn-del" style="background:red; padding:5px" onclick="excluir(${a.id})">X</button></td>
        </tr>`;
    });
    document.getElementById("lista_relatorios").innerHTML = html + `</table>`;
}

function excluir(id) {
    if(confirm("Deseja apagar esta auditoria?")) {
        db = db.filter(x => x.id !== id);
        localStorage.setItem("j2m_db_v2", JSON.stringify(db));
        abrirDashboard();
    }
}
</script>
</body>
</html>

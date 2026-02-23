<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - Sincronização Nuvem</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --green: #10b981; --blue: #007bff; }
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
    .btn-sync { background: var(--blue); width: 100%; margin-top: 10px; }
    .btn-edit { background: #ffc107; color: black; padding: 5px 8px; font-size: 10px; }
    .btn-del { background: var(--red); padding: 5px 8px; font-size: 10px; }
    
    label { font-weight: bold; display: block; margin-top: 12px; color: #444; font-size: 14px; }
    select, input, textarea { width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 6px; margin-top: 5px; font-size: 14px; }

    .teia-box { height: 450px; width: 100%; background: white; display: flex; justify-content: center; }
    .plano-item { padding: 8px; border-bottom: 1px solid #eee; font-size: 13px; }
    .plano-item b { color: var(--primary); text-transform: uppercase; font-size: 11px; display: block; }

    @media print { 
        .no-print, header, .filter-bar, button { display:none !important; } 
        .screen { display:block !important; padding: 0; }
        .card { border: 1px solid #eee; box-shadow: none; margin-bottom: 10px; border-left: 4px solid var(--primary); }
        .teia-box { height: 400px !important; width: 100% !important; }
        body { background: white; }
    }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M</header>

<div id="home" class="screen active">
    <div class="card">
        <h2>Iniciar Auditoria</h2>
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
        <label>Responsável:</label><input type="text" id="responsavel" placeholder="Nome do dono da área">
        <label>Auditor:</label><input type="text" id="auditor" placeholder="Seu nome">
        <label>Data:</label><input type="date" id="data_audit">
        <button class="btn-next" onclick="iniciar()">Começar Agora</button>
        <button class="btn-sync" onclick="sincronizar()">Sincronizar Celular e PC</button>
        <button style="background:var(--secondary); width:100%; margin-top:10px" onclick="abrirDash()">Aceder ao Dashboard</button>
    </div>
</div>

<div id="senso_screen" class="screen"></div>

<div id="dashboard" class="screen">
    <div id="pdf_header" style="display:none; text-align:center; margin-bottom:20px; border-bottom:2px solid var(--primary); padding-bottom:10px;">
        <h2 style="margin:0; color:var(--primary);">RELATÓRIO 5S - J2M</h2>
        <p id="pdf_dados_top" style="font-weight:bold;"></p>
    </div>

    <div class="card no-print">
        <div class="filter-bar">
            <select id="fSetor" onchange="filtrar()"></select>
            <button class="btn-pdf" onclick="window.print()">Imprimir PDF</button>
            <button style="background:#444" onclick="location.reload()">Sair</button>
        </div>
    </div>

    <div class="card"><div class="teia-box"><canvas id="chartRadar"></canvas></div></div>
    <div class="card" id="box_acoes"><h3>Planos de Ação</h3><div id="lista_acoes"></div></div>
    <div class="card no-print"><h3>Histórico de Auditorias</h3><div id="lista_hist"></div></div>
</div>

<script>
Chart.register(ChartDataLabels);

// TEU LINK ATUALIZADO
const API = "https://script.google.com/macros/s/AKfycbwPjqUwMg-YiCxlMhBujpMDUZZqZ_mZFFRIwyXUkojM40H_IaIjSr6utfevshUg5WFv/exec";

const nomesSensos = ["Seleção", "Ordenação", "Limpeza", "Padronização", "Autodisciplina"];
const sensos = [
    { s: "SELEÇÃO", p: ["Ferramentas necessárias?", "Itens duplicados?", "Acondicionamento?", "Docs atualizados?", "Informativos?"]},
    { s: "ORDENAÇÃO", p: ["Demarcação de piso?", "Etiquetagem?", "Limpeza organizada?", "Identificação?", "Objetos pessoais?"]},
    { s: "LIMPEZA", p: ["Máquinas limpas?", "Piso OK?", "Condições técnicas?", "Áreas comuns?", "Coleta seletiva?"]},
    { s: "PADRONIZAÇÃO", p: ["Documentação?", "Ergonomia?", "Lixeiras?", "Segurança?"]},
    { s: "AUTODISCIPLINA", p: ["Consciência?", "Uso de EPIs?", "Gestão à vista?", "Ações anteriores?"]}
];

let db = JSON.parse(localStorage.getItem("j2m_db_v3") || "[]");
let audit = {}, etapa = 0;
let myChart;

function iniciar() {
    if(!setor.value || !auditor.value || !data_audit.value) return alert("Preencha os campos obrigatórios!");
    audit = { id: Date.now(), setor: setor.value, responsavel: responsavel.value, auditor: auditor.value, data: data_audit.value, respostas: [] };
    etapa = 0; mostrarSenso();
}

function mostrarSenso() {
    const s = sensos[etapa];
    let h = `<div class="card"><h2>${s.s}</h2>`;
    s.p.forEach((p, i) => {
        h += `<label>${p}</label><select class="p-item" id="p_${i}">
            <option value="">Nota...</option>
            <option value="10">10 (Excelente)</option><option value="8">8 (Bom)</option>
            <option value="6">6 (Médio)</option><option value="4">4 (Melhorar)</option><option value="2">2 (Crítico)</option>
        </select>`;
    });
    h += `<label>Ação Corretiva (Obrigatório se Nota < 6):</label><textarea id="txt_plano" rows="3"></textarea>
          <button class="btn-next" onclick="proximo()">Próximo Senso</button></div>`;
    document.getElementById("senso_screen").innerHTML = h;
    trocarTela("senso_screen");
}

function proximo() {
    const sels = document.querySelectorAll(".p-item");
    let soma = 0, preenchido = true;
    sels.forEach(s => { if(!s.value) preenchido = false; soma += Number(s.value); });
    
    if(!preenchido) return alert("Responda todas as perguntas!");
    
    const media = soma / sels.length;
    const plano = document.getElementById("txt_plano").value.trim();
    if(media < 6 && !plano) return alert("Plano de ação obrigatório para notas baixas!");

    audit.respostas.push({ media, plano });
    etapa++;

    if(etapa < 5) mostrarSenso();
    else finalizar();
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("j2m_db_v3", JSON.stringify(db));
    
    // Enviar para nuvem
    fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) })
    .then(() => { alert("Auditoria Salva e Sincronizada!"); abrirDash(); });
}

function sincronizar() {
    alert("A aceder à nuvem... Aguarde.");
    fetch(API).then(r => r.json()).then(data => {
        if(data.length > 1) {
            db = data.slice(1).map(row => ({
                id: row[10], setor: row[1], responsavel: row[2], auditor: row[3], data: row[9],
                respostas: JSON.parse(row[11])
            }));
            localStorage.setItem("j2m_db_v3", JSON.stringify(db));
            alert("Sincronização concluída! Dados atualizados.");
            abrirDash();
        }
    }).catch(() => alert("Erro ao sincronizar. Verifique a internet."));
}

function abrirDash() {
    const fs = document.getElementById("fSetor");
    fs.innerHTML = '<option value="TODOS">Todos os Setores</option>';
    [...new Set(db.map(x => x.setor))].forEach(s => fs.innerHTML += `<option value="${s}">${s}</option>`);
    filtrar();
    trocarTela("dashboard");
}

function filtrar() {
    const sel = document.getElementById("fSetor").value;
    const dados = sel === "TODOS" ? db : db.filter(x => x.setor === sel);
    render(dados);
}

function render(dados) {
    if(dados.length === 0) return;
    const ult = dados[dados.length - 1];
    const mSetor = [0,1,2,3,4].map(i => (dados.reduce((acc, a) => acc + a.respostas[i].media, 0) / dados.length).toFixed(1));
    const mGeral = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + a.respostas[i].media, 0) / db.length).toFixed(1));

    document.getElementById("pdf_dados_top").innerText = `Setor: ${ult.setor} | Responsável: ${ult.responsavel} | Data: ${ult.data}`;

    if(myChart) myChart.destroy();
    myChart = new Chart(document.getElementById("chartRadar"), {
        type: 'radar',
        data: {
            labels: nomesSensos,
            datasets: [
                { label: 'Nota Setor', data: mSetor, borderColor: '#f06639',

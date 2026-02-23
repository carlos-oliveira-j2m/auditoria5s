<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v3.5 Anti-Travamento</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --green: #10b981; --blue: #007bff; }
    *{box-sizing:border-box; font-family: 'Segoe UI', sans-serif;}
    body { margin:0; background: #f4f7f6; color: #333; }
    header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; }
    .screen { display:none; padding:15px; max-width: 1000px; margin: auto; }
    .active { display:block; animation: fadeIn 0.4s; }
    .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); border-left: 8px solid var(--primary); margin-bottom: 20px; }
    button { border: none; padding: 12px; border-radius: 6px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; }
    .btn-next { background: var(--primary); width: 100%; margin-top:15px;}
    .btn-sync { background: var(--blue); width: 100%; margin-top: 10px; }
    .btn-pdf { background: var(--green); }
    label { font-weight: bold; display: block; margin-top: 12px; font-size: 14px; }
    select, input, textarea { width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 6px; margin-top: 5px; font-size: 14px; }
    .teia-box { height: 400px; width: 100%; }
    @media print { .no-print { display:none !important; } .screen { display:block !important; } .card { border: 1px solid #eee; } }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M</header>

<div id="home" class="screen active">
    <div class="card">
        <h2>Identificação</h2>
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
        <label>Responsável:</label><input type="text" id="responsavel">
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data_audit">
        <button class="btn-next" onclick="iniciar()">Começar Auditoria</button>
        <button id="btnSync" class="btn-sync" onclick="sincronizar()">Sincronizar Celular/PC</button>
        <button style="background:var(--secondary); width:100%; margin-top:10px" onclick="abrirDash()">Relatórios Salvos</button>
    </div>
</div>

<div id="senso_screen" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="card no-print">
        <div style="display:flex; gap:10px">
            <select id="fSetor" onchange="filtrar()"></select>
            <button class="btn-pdf" onclick="window.print()">PDF</button>
            <button style="background:#444" onclick="location.reload()">Sair</button>
        </div>
    </div>
    <div class="card"><div class="teia-box"><canvas id="chartRadar"></canvas></div></div>
    <div class="card"><h3>Ações Corretivas</h3><div id="lista_acoes"></div></div>
    <div class="card no-print"><h3>Histórico</h3><div id="lista_hist"></div></div>
</div>

<script>
Chart.register(ChartDataLabels);

// SEU LINK GOOGLE
const API = "https://script.google.com/macros/s/AKfycbwPjqUwMg-YiCxlMhBujpMDUZZqZ_mZFFRIwyXUkojM40H_IaIjSr6utfevshUg5WFv/exec";

const nomesSensos = ["Seleção", "Ordenação", "Limpeza", "Padronização", "Autodisciplina"];
const perguntas = [
    { s: "SELEÇÃO", p: ["Ferramentas necessárias?", "Itens duplicados?", "Acondicionamento?", "Docs OK?", "Informativos?"] },
    { s: "ORDENAÇÃO", p: ["Piso marcado?", "Etiquetagem?", "Limpeza organizada?", "Identificação?", "Objetos pessoais?"] },
    { s: "LIMPEZA", p: ["Máquinas limpas?", "Piso OK?", "Condições técnicas?", "Áreas comuns?", "Coleta seletiva?"] },
    { s: "PADRONIZAÇÃO", p: ["Documentação?", "Ergonomia?", "Lixeiras?", "Segurança?"] },
    { s: "AUTODISCIPLINA", p: ["Consciência?", "Uso de EPIs?", "Gestão à vista?", "Ações anteriores?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db_v3") || "[]");
let audit = {}, etapa = 0;
let myChart;

function iniciar() {
    if(!setor.value || !auditor.value) return alert("Preencha os campos!");
    audit = { id: Date.now(), setor: setor.value, responsavel: responsavel.value, auditor: auditor.value, data: data_audit.value, respostas: [] };
    etapa = 0; mostrarSenso();
}

function mostrarSenso() {
    const s = perguntas[etapa];
    let h = `<div class="card"><h2>${s.s}</h2>`;
    s.p.forEach((p, i) => {
        h += `<label>${p}</label><select class="p-item" id="p_${i}">
            <option value="">Nota...</option>
            <option value="10">10 (Excelente)</option><option value="8">8 (Bom)</option>
            <option value="6">6 (Médio)</option><option value="4">4 (Melhorar)</option><option value="2">2 (Crítico)</option>
        </select>`;
    });
    h += `<label>Plano de Ação:</label><textarea id="txt_plano" rows="2"></textarea>
          <button class="btn-next" onclick="proximo()">Continuar</button></div>`;
    document.getElementById("senso_screen").innerHTML = h;
    trocarTela("senso_screen");
}

function proximo() {
    const sels = document.querySelectorAll(".p-item");
    let soma = 0, ok = true;
    sels.forEach(s => { if(!s.value) ok = false; soma += Number(s.value); });
    if(!ok) return alert("Responda todas!");
    
    audit.respostas.push({ media: soma/sels.length, plano: document.getElementById("txt_plano").value });
    etapa++;
    if(etapa < 5) mostrarSenso();
    else {
        db.push(audit);
        localStorage.setItem("j2m_db_v3", JSON.stringify(db));
        alert("Salvo no aparelho! Tentando sincronizar com a nuvem...");
        fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) })
        .finally(() => abrirDash());
    }
}

async function sincronizar() {
    const btn = document.getElementById("btnSync");
    btn.innerText = "Sincronizando...";
    btn.disabled = true;

    try {
        const resp = await fetch(API);
        const data = await resp.json();
        if(data && data.length > 1) {
            db = data.slice(1).map(row => ({
                id: row[10], setor: row[1], responsavel: row[2], auditor: row[3], data: row[9],
                respostas: JSON.parse(row[11])
            }));
            localStorage.setItem("j2m_db_v3", JSON.stringify(db));
            alert("Sincronizado!");
            abrirDash();
        }
    } catch (e) {
        alert("Erro de conexão. Verifique se o Google Script está ativo.");
    } finally {
        btn.innerText = "Sincronizar Celular/PC";
        btn.disabled = false;
    }
}

function abrirDash() {
    const fs = document.getElementById("fSetor");
    fs.innerHTML = '<option value="TODOS">Setores</option>';
    [...new Set(db.map(x => x.setor))].forEach(s => fs.innerHTML += `<option value="${s}">${s}</option>`);
    filtrar();
    trocarTela("dashboard");
}

function filtrar() {
    const sel = document.getElementById("fSetor").value;
    render(sel === "TODOS" ? db : db.filter(x => x.setor === sel));
}

function render(dados) {
    if(!dados.length) return;
    const ult = dados[dados.length - 1];
    const mSetor = [0,1,2,3,4].map(i => (dados.reduce((acc, a) => acc + a.respostas[i].media, 0) / dados.length).toFixed(1));
    const mGeral = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + a.respostas[i].media, 0) / db.length).toFixed(1));

    if(myChart) myChart.destroy();
    myChart = new Chart(document.getElementById("chartRadar"), {
        type: 'radar',
        data: {
            labels: nomesSensos,
            datasets: [
                { label: 'Setor', data: mSetor, borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Fábrica', data: mGeral, borderColor: '#5d5a51', borderDash: [5,5], fill: false },
                { label: 'Meta', data: [8,8,8,8,8], borderColor: '#10b981', fill: false }
            ]
        },
        options: { maintainAspectRatio: false, scales: { r: { min: 0, max: 10, ticks: { display: false } } } }
    });

    let hAcoes = "";
    ult.respostas.forEach((r, i) => { if(r.plano) hAcoes += `<div class='plano-item'><b>${nomesSensos[i]}</b> ${r.plano}</div>`; });
    document.getElementById("lista_acoes").innerHTML = hAcoes || "Sem ações.";
    
    let hHist = "<table>";
    dados.slice().reverse().forEach(a => {
        hHist += `<tr><td>${a.data}</td><td>${a.setor}</td><td><button onclick="excluir(${a.id})" style='background:red;padding:2px 5px'>X</button></td></tr>`;
    });
    document.getElementById("lista_hist").innerHTML = hHist + "</table>";
}

function excluir(id) { if(confirm("Eliminar?")) { db = db.filter(x => x.id !== id); localStorage.setItem("j2m_db_v3", JSON.stringify(db)); filtrar(); } }
function trocarTela(id) { document.querySelectorAll(".screen").forEach(s => s.classList.remove("active")); document.getElementById(id).classList.add("active"); }
</script>
</body>
</html>

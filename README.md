<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v5.0 (Oficial)</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --green: #10b981; }
    * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, sans-serif; }
    body { margin: 0; background: #f4f7f6; }
    header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; font-size: 20px; }
    
    .screen { display: none; padding: 15px; max-width: 1000px; margin: auto; }
    .active { display: block; }
    .card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); margin-bottom: 20px; border-left: 6px solid var(--primary); }
    
    /* Indicadores Estilo PDF Aprovado */
    .stats { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px; }
    .stat-box { background: white; padding: 15px; border-radius: 8px; text-align: center; border-top: 4px solid var(--primary); }
    .stat-box h1 { margin: 5px 0; color: var(--primary); font-size: 32px; }
    .stat-box p { margin: 0; font-size: 12px; font-weight: bold; color: #666; }

    label { font-weight: bold; display: block; margin-top: 15px; font-size: 14px; }
    select, input, textarea { width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 6px; margin-top: 5px; font-size: 15px; }
    
    button { border: none; padding: 12px; border-radius: 6px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; width: 100%; margin-top: 10px; transition: 0.3s; }
    .btn-main { background: var(--primary); }
    .btn-sync { background: #007bff; }
    .btn-pdf { background: var(--green); }

    .teia-box { height: 450px; width: 100%; margin-bottom: 20px; }
    table { width: 100%; border-collapse: collapse; font-size: 13px; margin-top: 10px; }
    table th { background: #eee; padding: 10px; text-align: left; }
    table td { padding: 8px 10px; border-bottom: 1px solid #eee; }

    @media print {
        .no-print { display: none !important; }
        .screen { display: block !important; }
        .card { box-shadow: none; border: 1px solid #eee; page-break-inside: avoid; }
        body { background: white; }
    }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M</header>

<div id="home" class="screen active no-print">
    <div class="card">
        <h2>Painel de Auditoria</h2>
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
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data_audit">
        
        <button class="btn-main" onclick="iniciar()">Iniciar Nova Auditoria</button>
        <button class="btn-sync" onclick="sincronizar()">Sincronizar Celular / PC</button>
        <button style="background:var(--secondary)" onclick="abrirDash()">Ver Relatório e Gráficos</button>
    </div>
</div>

<div id="senso_screen" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="stats">
        <div class="stat-box"><p>TOTAL DE AUDITORIAS</p><h1 id="count-audits">0</h1></div>
        <div class="stat-box"><p>MÉDIA GERAL FÁBRICA</p><h1 id="avg-factory">0.0</h1></div>
    </div>

    <div class="card">
        <h3 style="margin:0 0 15px 0">Gráfico de Radar (5 Senso)</h3>
        <div class="teia-box"><canvas id="chartRadar"></canvas></div>
    </div>

    <div class="card no-print">
        <h3 style="margin:0 0 15px 0">Comparativo entre Setores</h3>
        <div style="height:300px"><canvas id="chartBarras"></canvas></div>
    </div>

    <div class="card">
        <h3>Histórico e Notas</h3>
        <div id="lista_hist"></div>
    </div>

    <div class="no-print" style="display:flex; gap:10px;">
        <button class="btn-pdf" onclick="window.print()">Imprimir PDF</button>
        <button style="background:#666" onclick="location.reload()">Voltar</button>
    </div>
</div>

<script>
Chart.register(ChartDataLabels);
const API = "https://script.google.com/macros/s/AKfycbwPjqUwMg-YiCxlMhBujpMDUZZqZ_mZFFRIwyXUkojM40H_IaIjSr6utfevshUg5WFv/exec";

const perguntas = [
    { s: "SELEÇÃO", p: ["Ferramentas necessárias?", "Itens duplicados?", "Acondicionamento?", "Docs OK?", "Informativos?"] },
    { s: "ORDENAÇÃO", p: ["Piso marcado?", "Etiquetagem?", "Limpeza organizada?", "Identificação?", "Objetos pessoais?"] },
    { s: "LIMPEZA", p: ["Máquinas limpas?", "Piso OK?", "Condições técnicas?", "Áreas comuns?", "Coleta seletiva?"] },
    { s: "PADRONIZAÇÃO", p: ["Documentação?", "Ergonomia?", "Lixeiras?", "Segurança?"] },
    { s: "AUTODISCIPLINA", p: ["Consciência?", "Uso de EPIs?", "Gestão à vista?", "Ações anteriores?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db_v3") || "[]");
let audit = {}, etapa = 0;
let chartRadar, chartBarras;

function iniciar() {
    if(!setor.value || !auditor.value) return alert("Preencha o setor e auditor!");
    audit = { id: Date.now(), setor: setor.value, auditor: auditor.value, data: data_audit.value, respostas: [] };
    etapa = 0; mostrarSenso();
}

function mostrarSenso() {
    const s = perguntas[etapa];
    let h = `<div class="card"><h2>${s.s}</h2>`;
    s.p.forEach((p, i) => {
        h += `<label>${p}</label><select class="p-item">
            <option value="">Nota...</option>
            <option value="10">10 (Excelente)</option><option value="8">8 (Bom)</option>
            <option value="6">6 (Médio)</option><option value="4">4 (Melhorar)</option><option value="2">2 (Crítico)</option>
        </select>`;
    });
    h += `<label>Plano de Ação:</label><textarea id="txt_plano"></textarea>
          <button class="btn-main" onclick="proximo()">Próximo</button></div>`;
    document.getElementById("senso_screen").innerHTML = h;
    trocarTela("senso_screen");
}

function proximo() {
    const sels = document.querySelectorAll(".p-item");
    let soma = 0, ok = true;
    sels.forEach(s => { if(!s.value) ok = false; soma += Number(s.value); });
    if(!ok) return alert("Responda tudo!");
    
    audit.respostas.push({ media: soma/sels.length, plano: document.getElementById("txt_plano").value });
    etapa++;
    if(etapa < 5) mostrarSenso();
    else {
        db.push(audit);
        localStorage.setItem("j2m_db_v3", JSON.stringify(db));
        fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
        abrirDash();
    }
}

async function sincronizar() {
    alert("Buscando dados na nuvem...");
    try {
        const r = await fetch(API);
        const data = await r.json();
        if(data && data.length > 1) {
            db = data.slice(1).map(row => ({ id: row[10], setor: row[1], auditor: row[3], data: row[9], respostas: JSON.parse(row[11]) }));
            localStorage.setItem("j2m_db_v3", JSON.stringify(db));
            alert("Sincronizado!");
            abrirDash();
        }
    } catch(e) { alert("Erro de rede."); }
}

function abrirDash() {
    trocarTela("dashboard");
    render();
}

function render() {
    if(!db.length) return;
    const ult = db[db.length - 1];
    
    // Indicadores
    document.getElementById("count-audits").innerText = db.length;
    const somaGeral = db.reduce((acc, a) => acc + (a.respostas.reduce((s, r) => s + r.media, 0) / 5), 0);
    document.getElementById("avg-factory").innerText = (somaGeral / db.length).toFixed(1);

    // Gráfico Radar (Original)
    if(chartRadar) chartRadar.destroy();
    chartRadar = new Chart(document.getElementById("chartRadar"), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets: [{ label: ult.setor, data: ult.respostas.map(r => r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' }]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    // Gráfico Barras (Estilo PDF Aprovado)
    const setores = [...new Set(db.map(a => a.setor))];
    const medias = setores.map(s => {
        const filtra = db.filter(a => a.setor === s);
        return (filtra.reduce((acc, a) => acc + (a.respostas.reduce((s, r) => s + r.media, 0) / 5), 0) / filtra.length).toFixed(1);
    });

    if(chartBarras) chartBarras.destroy();
    chartBarras = new Chart(document.getElementById("chartBarras"), {
        type: 'bar',
        data: { labels: setores, datasets: [{ label: 'Nota Final', data: medias, backgroundColor: '#f06639' }] },
        options: { scales: { y: { min: 0, max: 10 } } }
    });

    // Tabela Histórico
    let h = "<table><tr><th>Data</th><th>Setor</th><th>Nota</th></tr>";
    db.slice().reverse().forEach(a => {
        const nota = (a.respostas.reduce((s, r) => s + r.media, 0) / 5).toFixed(1);
        h += `<tr><td>${a.data}</td><td>${a.setor}</td><td><b>${nota}</b></td></tr>`;
    });
    document.getElementById("lista_hist").innerHTML = h + "</table>";
}

function trocarTela(id) {
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById(id).classList.add("active");
}
</script>
</body>
</html>

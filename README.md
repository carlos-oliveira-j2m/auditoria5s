<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025 Enterprise</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --meta: #10b981; }
    * { box-sizing: border-box; font-family: 'Segoe UI', Arial, sans-serif; }
    body { margin: 0; background: #f4f7f6; color: #333; }
    
    header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; font-size: 20px; }
    .screen { display: none; padding: 15px; max-width: 1200px; margin: auto; }
    .active { display: block; }

    /* Estilo dos Quadrados de Indicadores (Igual ao PDF) */
    .stats-container { display: flex; gap: 20px; margin-bottom: 20px; }
    .stat-box { flex: 1; background: white; padding: 20px; border-radius: 4px; text-align: center; border-top: 5px solid var(--primary); box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
    .stat-box h1 { margin: 10px 0; font-size: 48px; color: #333; }
    .stat-box p { margin: 0; font-weight: bold; color: #666; text-transform: uppercase; font-size: 14px; }

    /* Grid de Gráficos */
    .charts-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 20px; }
    .card { background: white; padding: 15px; border-radius: 4px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); position: relative; }
    .card h3 { margin-top: 0; border-bottom: 1px solid #eee; padding-bottom: 10px; font-size: 16px; }

    /* Tabelas */
    table { width: 100%; border-collapse: collapse; background: white; font-size: 13px; }
    table th { background: #f8f8f8; padding: 12px; text-align: left; border-bottom: 2px solid #ddd; }
    table td { padding: 10px; border-bottom: 1px solid #eee; }

    /* Botões */
    .nav-btns { display: flex; gap: 10px; margin-bottom: 20px; }
    button { cursor: pointer; border: none; padding: 12px 20px; font-weight: bold; border-radius: 4px; color: white; transition: 0.3s; }
    .btn-orange { background: var(--primary); }
    .btn-blue { background: #007bff; }
    .btn-green { background: var(--meta); }

    /* Inputs Auditoria */
    .form-group { margin-bottom: 15px; }
    label { font-weight: bold; display: block; margin-bottom: 5px; }
    select, input, textarea { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 4px; }

    @media print {
        .no-print { display: none !important; }
        .screen { display: block !important; padding: 0; }
        body { background: white; }
        .card { box-shadow: none; border: 1px solid #ddd; }
        .charts-grid { grid-template-columns: 1fr 1fr; }
    }
</style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active no-print">
    <div class="card">
        <h2>Painel Principal</h2>
        <div class="nav-btns">
            <button class="btn-orange" onclick="novaAudit()">Nova Auditoria</button>
            <button class="btn-blue" onclick="sincronizar()">Sincronizar Nuvem</button>
            <button style="background:var(--secondary)" onclick="abrirDash()">Ver Dashboard/PDF</button>
        </div>
        <p style="font-size: 12px; color: #666;">*Utilize o botão Sincronizar para atualizar dados entre telemóvel e PC.</p>
    </div>
</div>

<div id="audit_screen" class="screen">
    <div class="card" id="pergunta_cont"></div>
</div>

<div id="dashboard" class="screen">
    <div class="stats-container">
        <div class="stat-box">
            <p>Média Geral da Fábrica</p>
            <h1 id="avg_geral">0.0</h1>
        </div>
        <div class="stat-box">
            <p>Total de Auditorias</p>
            <h1 id="total_count">0</h1>
        </div>
    </div>

    <div class="charts-grid">
        <div class="card">
            <h3>Setor Selecionado vs Média/Meta</h3>
            <div style="height: 350px;"><canvas id="chartRadar"></canvas></div>
        </div>
        <div class="card">
            <h3>Nota Final por Setor</h3>
            <div style="height: 350px;"><canvas id="chartBarras"></canvas></div>
        </div>
    </div>

    <div class="card">
        <h3>Histórico</h3>
        <table id="tabela_hist">
            <thead>
                <tr><th>Data</th><th>Setor</th><th>Nota</th></tr>
            </thead>
            <tbody id="corpo_hist"></tbody>
        </table>
    </div>

    <div class="no-print" style="margin-top: 20px; text-align: center;">
        <button class="btn-green" onclick="window.print()">Imprimir PDF</button>
        <button style="background:#666" onclick="location.reload()">Voltar</button>
    </div>
</div>

<script>
Chart.register(ChartDataLabels);
const API = "https://script.google.com/macros/s/AKfycbwPjqUwMg-YiCxlMhBujpMDUZZqZ_mZFFRIwyXUkojM40H_IaIjSr6utfevshUg5WFv/exec";

const checklists = [
    { s: "Seleção", p: ["Itens desnecessários?", "Ferramentas em excesso?", "Materiais obsoletos?", "Documentos antigos?", "Lixo/Sucata?"] },
    { s: "Ordenação", p: ["Piso demarcado?", "Identificação de locais?", "Facilidade de acesso?", "Ordem de uso?", "Layout funcional?"] },
    { s: "Limpeza", p: ["Piso limpo?", "Bancadas/Máquinas?", "Fontes de sujidade?", "Material de limpeza disponível?", "Higiene geral?"] },
    { s: "Padronização", p: ["Cores e placas?", "Uniformes/EPIs?", "Instruções visuais?", "Estado de conservação?", "Sinalização de segurança?"] },
    { s: "Disciplina", p: ["Cumprimento de horários?", "Respeito às normas?", "Engajamento da equipa?", "Manutenção do 5S?", "Sugestões de melhoria?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db_v3") || "[]");
let audit = {}, etapa = 0;
let radar, barras;

function novaAudit() {
    let setor = prompt("Setor:");
    if(!setor) return;
    audit = { id: Date.now(), setor: setor, data: new Date().toLocaleDateString(), respostas: [] };
    etapa = 0;
    mostrarEtapa();
}

function mostrarEtapa() {
    const c = checklists[etapa];
    let h = `<h3>${etapa+1}. Senso de ${c.s}</h3>`;
    c.p.forEach((p, i) => {
        h += `<div class="form-group"><label>${p}</label>
              <select class="nota-in"><option value="10">10</option><option value="8">8</option><option value="6">6</option><option value="4">4</option><option value="2">2</option></select></div>`;
    });
    h += `<button class="btn-orange" onclick="salvarEtapa()">Próximo</button>`;
    document.getElementById("pergunta_cont").innerHTML = h;
    trocarTela("audit_screen");
}

function salvarEtapa() {
    let notas = document.querySelectorAll(".nota-in");
    let soma = 0;
    notas.forEach(n => soma += Number(n.value));
    audit.respostas.push(soma / notas.length);
    etapa++;
    if(etapa < 5) mostrarEtapa();
    else {
        db.push(audit);
        localStorage.setItem("j2m_db_v3", JSON.stringify(db));
        fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
        abrirDash();
    }
}

async function sincronizar() {
    alert("A ligar à nuvem...");
    try {
        const r = await fetch(API);
        const data = await r.json();
        if(data.length > 1) {
            db = data.slice(1).map(row => ({
                id: row[10], setor: row[1], data: row[9],
                respostas: JSON.parse(row[11]).map(r => r.media || r)
            }));
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

    // Indicadores Topo
    document.getElementById("total_count").innerText = db.length;
    let somaTotal = db.reduce((a, b) => a + (b.respostas.reduce((s, r) => s + r, 0) / 5), 0);
    document.getElementById("avg_geral").innerText = (somaTotal / db.length).toFixed(1);

    // Gráfico Radar (Teia)
    if(radar) radar.destroy();
    radar = new Chart(document.getElementById("chartRadar"), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets: [
                { label: 'Setor Atual', data: ult.respostas, borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Meta', data: [8,8,8,8,8], borderColor: '#10b981', fill: false, borderDash: [5,5] }
            ]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    // Gráfico Barras (Comparativo)
    const setores = [...new Set(db.map(a => a.setor))];
    const medias = setores.map(s => {
        const filtra = db.filter(a => a.setor === s);
        return (filtra.reduce((acc, a) => acc + (a.respostas.reduce((s, r) => s + r, 0) / 5), 0) / filtra.length).toFixed(1);
    });

    if(barras) barras.destroy();
    barras = new Chart(document.getElementById("chartBarras"), {
        type: 'bar',
        data: {
            labels: setores,
            datasets: [{ label: 'Nota Final', data: medias, backgroundColor: '#f06639' }]
        },
        options: { 
            maintainAspectRatio: false,
            scales: { y: { min: 0, max: 10 } },
            plugins: { datalabels: { anchor: 'end', align: 'top', display: true } }
        }
    });

    // Tabela
    let h = "";
    db.slice().reverse().forEach(a => {
        const nota = (a.respostas.reduce((s, r) => s + r, 0) / 5).toFixed(1);
        h += `<tr><td>${a.data}</td><td>${a.setor}</td><td><b>${nota}</b></td></tr>`;
    });
    document.getElementById("corpo_hist").innerHTML = h;
}

function trocarTela(id) {
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById(id).classList.add("active");
}
</script>
</body>
</html>

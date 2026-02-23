<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - Versão PDF Aprovada</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --bg: #f4f7f6; }
    * { box-sizing: border-box; font-family: 'Segoe UI', Arial, sans-serif; }
    body { margin: 0; background: var(--bg); }
    
    /* Layout Interface */
    header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; }
    .screen { display: none; padding: 15px; max-width: 1100px; margin: auto; }
    .active { display: block; }
    .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); margin-bottom: 20px; }
    
    /* Estilos PDF (Impressão) */
    @media print {
        body { background: white; }
        .no-print, button, header, .nav-btns { display: none !important; }
        .screen { display: block !important; width: 100% !important; max-width: 100% !important; padding: 0; }
        .card { box-shadow: none; border: 1px solid #eee; margin-bottom: 10px; page-break-inside: avoid; }
        .pdf-header { display: flex !important; justify-content: space-between; align-items: center; border-bottom: 2px solid var(--primary); padding-bottom: 10px; margin-bottom: 20px; }
    }

    /* Elementos do Dashboard */
    .stats-container { display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 20px; }
    .stat-box { background: white; padding: 15px; border-radius: 8px; text-align: center; border-top: 4px solid var(--primary); }
    .stat-box h1 { margin: 5px 0; color: var(--primary); font-size: 32px; }
    
    table { width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 13px; }
    table th { background: #eee; text-align: left; padding: 10px; border-bottom: 2px solid #ddd; }
    table td { padding: 8px 10px; border-bottom: 1px solid #eee; }
    
    button { background: var(--primary); color: white; border: none; padding: 12px 20px; border-radius: 5px; cursor: pointer; font-weight: bold; }
    .btn-sync { background: #007bff; }
    select, input { padding: 10px; border-radius: 5px; border: 1px solid #ccc; width: 100%; margin-top: 5px; }
</style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - GESTÃO</header>

<div id="home" class="screen active no-print">
    <div class="card">
        <h2>Painel de Controle</h2>
        <p>Utilize os botões abaixo para gerenciar as auditorias.</p>
        <button class="btn-sync" onclick="puxarDaNuvem()">Sincronizar Celular/PC</button>
        <button onclick="window.location.href='auditoria5s.html'">Nova Auditoria</button>
        <button style="background:var(--secondary)" onclick="abrirDash()">Ver Relatório PDF</button>
    </div>
</div>

<div id="dashboard" class="screen">
    <div class="pdf-header" style="display:none;">
        <div>
            <h2 style="margin:0; color:var(--primary);">AUDITORIA 5S J2M</h2>
            <p style="margin:0; font-size:12px;">Relatório de Desempenho por Setor</p>
        </div>
        <div style="text-align:right">
            <p id="pdf-data" style="margin:0; font-size:12px; font-weight:bold;"></p>
        </div>
    </div>

    <div class="stats-container">
        <div class="stat-box">
            <p>TOTAL DE AUDITORIAS</p>
            <h1 id="count-audits">0</h1>
        </div>
        <div class="stat-box">
            <p>MÉDIA GERAL DA FÁBRICA</p>
            <h1 id="avg-factory">0.0</h1>
        </div>
    </div>

    <div class="card">
        <h3 style="margin-top:0">Nota Final por Setor</h3>
        <div style="height: 300px;">
            <canvas id="chartSetores"></canvas>
        </div>
    </div>

    <div class="card">
        <h3 style="margin-top:0">Histórico de Registros</h3>
        <table id="tabela-historico">
            <thead>
                <tr>
                    <th>Data</th>
                    <th>Setor</th>
                    <th>Responsável</th>
                    <th>Nota</th>
                </tr>
            </thead>
            <tbody id="lista-corpo"></tbody>
        </table>
    </div>

    <div class="no-print" style="margin-top:20px; text-align:center;">
        <button onclick="window.print()" class="btn-pdf" style="background: #10b981;">Gerar PDF Aprovado</button>
        <button onclick="location.reload()" style="background: #666;">Voltar</button>
    </div>
</div>

<script>
const API = "https://script.google.com/macros/s/AKfycbwPjqUwMg-YiCxlMhBujpMDUZZqZ_mZFFRIwyXUkojM40H_IaIjSr6utfevshUg5WFv/exec";
let db = JSON.parse(localStorage.getItem("j2m_db_v3") || "[]");
let chartSetores;

async function puxarDaNuvem() {
    alert("Buscando dados...");
    try {
        const r = await fetch(API);
        const data = await r.json();
        if(data && data.length > 1) {
            db = data.slice(1).map(row => ({
                id: row[10], setor: row[1], responsavel: row[2], auditor: row[3], data: row[9],
                respostas: JSON.parse(row[11])
            }));
            localStorage.setItem("j2m_db_v3", JSON.stringify(db));
            alert("Sincronizado!");
            abrirDash();
        }
    } catch(e) { alert("Erro na conexão."); }
}

function abrirDash() {
    document.getElementById("home").classList.remove("active");
    document.getElementById("dashboard").classList.add("active");
    document.getElementById("pdf-data").innerText = new Date().toLocaleDateString();
    renderizarDashboard();
}

function renderizarDashboard() {
    if(!db.length) return;

    // Estatísticas
    document.getElementById("count-audits").innerText = db.length;
    const somaGeral = db.reduce((acc, audit) => {
        const mediaAudit = audit.respostas.reduce((s, r) => s + r.media, 0) / 5;
        return acc + mediaAudit;
    }, 0);
    document.getElementById("avg-factory").innerText = (somaGeral / db.length).toFixed(1);

    // Preparar dados do gráfico (Média por setor)
    const setoresUnicos = [...new Set(db.map(a => a.setor))];
    const mediasSetores = setoresUnicos.map(s => {
        const auditsSetor = db.filter(a => a.setor === s);
        const soma = auditsSetor.reduce((acc, a) => acc + (a.respostas.reduce((s, r) => s + r.media, 0) / 5), 0);
        return (soma / auditsSetor.length).toFixed(1);
    });

    // Gráfico de Barras (Igual ao PDF aprovado)
    if(chartSetores) chartSetores.destroy();
    chartSetores = new Chart(document.getElementById("chartSetores"), {
        type: 'bar',
        data: {
            labels: setoresUnicos,
            datasets: [{
                label: 'Nota Final',
                data: mediasSetores,
                backgroundColor: '#f06639',
                borderRadius: 5
            }]
        },
        options: {
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: { y: { min: 0, max: 10 } }
        }
    });

    // Tabela de Histórico
    const corpo = document.getElementById("lista-corpo");
    corpo.innerHTML = "";
    db.slice().reverse().forEach(a => {
        const media = (a.respostas.reduce((s, r) => s + r.media, 0) / 5).toFixed(1);
        corpo.innerHTML += `<tr>
            <td>${a.data}</td>
            <td>${a.setor}</td>
            <td>${a.responsavel}</td>
            <td style="font-weight:bold; color:var(--primary)">${media}</td>
        </tr>`;
    });
}

function trocarTela(id) {
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active"));
    document.getElementById(id).classList.add("active");
}
</script>
</body>
</html>

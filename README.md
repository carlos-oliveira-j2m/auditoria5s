<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025 Enterprise</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>

<style>
:root { --primary: #f06639; --secondary: #3d3d3d; --meta: #28a745; --danger: #dc3545; }
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}

/* Fundo Dinâmico */
body { margin:0; min-height: 100vh; background-color: #f0f2f5; background-size: cover; background-position: center; background-attachment: fixed; transition: background-image 1.5s ease-in-out; position: relative; }
body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.9); z-index: -1; }

header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }

.screen { display:none; padding:20px; max-width: 1150px; margin: 10px auto; }
.active { display:block; animation: fadeIn 0.4s; }

.card { background: rgba(255,255,255,0.95); padding: 20px; border-radius: 8px; margin-bottom: 15px; border-left: 5px solid var(--primary); box-shadow: 0 4px 6px rgba(0,0,0,0.1); }

/* Filtros e KPIs */
.filter-bar { display: flex; gap: 10px; background: white; padding: 15px; border-radius: 8px; margin-bottom: 20px; flex-wrap: wrap; border: 1px solid #ddd; }
.kpi-container { display: flex; gap: 20px; margin-bottom: 20px; flex-wrap: wrap; }
.kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; min-width: 200px; text-align: center; border-bottom: 5px solid var(--primary); }
.kpi-card h2 { margin: 0; font-size: 35px; color: var(--primary); }

/* Botões */
button { border: none; padding: 10px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; transition: 0.2s; }
.btn-p { background: var(--primary); color: white; width: 100%; margin-bottom: 10px; }
.btn-s { background: var(--secondary); color: white; width: 100%; }
.btn-edit { background: #ffc107; color: black; padding: 5px 10px; font-size: 11px; }
.btn-del { background: var(--danger); color: white; padding: 5px 10px; font-size: 11px; }

/* Estilo Relatório para Impressão */
.folha-rosto { text-align:center; padding:40px; border:2px solid #333; min-height: 1000px; background:white; position: relative; display: none; }
.page-break { page-break-before: always; min-height: 1000px; background: white; border: 2px solid #333; padding: 20px; display: none; }
.table-checklist { width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 12px; }
.table-checklist th, .table-checklist td { border: 1px solid #333; padding: 6px; text-align: left; }
.senso-header { background: #444 !important; color: white; font-weight: bold; text-align: center !important; }

@media print {
    .no-print { display:none !important; }
    .screen { display: none !important; }
    .folha-rosto, .page-break { display: block; border: none; }
    body { background: white; }
    body::before { display: none; }
}
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body id="bgBody">

<header>AUDITORIA 5S J2M - GESTÃO INDUSTRIAL</header>

<div id="home" class="screen active">
    <div class="card">
        <h3 id="tituloHome">Identificação da Auditoria</h3>
        <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px;">
            <div>
                <label>Setor:</label>
                <select id="setor">
                    <option value="">-- SELECIONE (14 SETORES) --</option>
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
            </div>
            <div><label>Auditor:</label><input type="text" id="auditor"></div>
            <div><label>Data:</label><input type="date" id="data"></div>
        </div><br>
        <button class="btn-p" onclick="iniciar()">Iniciar Avaliação</button>
        <button class="btn-s" onclick="abrirDash()">Dashboard / Histórico</button>
    </div>

    <div class="card no-print">
        <h3>Auditorias Recentes (Dispositivo)</h3>
        <div id="lista_recentes"></div>
    </div>
</div>

<div id="senso" class="screen">
    <div id="box_perguntas"></div>
</div>

<div id="dashboard" class="screen">
    <div class="no-print">
        <div class="filter-bar">
            <div style="flex:1"><label>SETOR</label><select id="fSetor" onchange="renderRelatorio()"></select></div>
            <div style="flex:1"><label>MÊS</label>
                <select id="fMes" onchange="renderRelatorio()">
                    <option value="ALL">Todos</option>
                    <option value="0">Janeiro</option><option value="1">Fevereiro</option><option value="2">Março</option>
                    <option value="3">Abril</option><option value="4">Maio</option><option value="5">Junho</option>
                    <option value="6">Julho</option><option value="7">Agosto</option><option value="8">Setembro</option>
                    <option value="9">Outubro</option><option value="10">Novembro</option><option value="11">Dezembro</option>
                </select>
            </div>
            <div style="flex:1"><label>ANO</label><select id="fAno" onchange="renderRelatorio()"></select></div>
            <div style="display:flex; align-items:flex-end; gap:5px;">
                <button class="btn-p" style="margin:0" onclick="window.print()">Imprimir PDF</button>
                <button class="btn-s" style="margin:0" onclick="location.reload()">Voltar</button>
            </div>
        </div>
    </div>

    <div class="kpi-container no-print">
        <div class="kpi-card"><h2 id="mediaFiltro">0.0</h2><p>Média Selecionada</p></div>
        <div class="kpi-card"><h2 id="totalAuditorias">0</h2><p>Auditorias no Filtro</p></div>
    </div>

    <div class="no-print" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); gap: 20px; margin-bottom: 20px;">
        <div style="background:white; padding:10px; border-radius:8px;"><canvas id="cRadar"></canvas></div>
        <div style="background:white; padding:10px; border-radius:8px;"><canvas id="cBarra"></canvas></div>
    </div>

    <div id="area_impressao"></div>
</div>

<script>
const sensos = [
    { n: "1 - SELEÇÃO", q: ["Equipamentos necessários?", "Itens duplicados?", "Acondicionamento correto?", "Gestão visual atualizada?", "Informativos necessários?"] },
    { n: "2 - ORDENAÇÃO", q: ["Locais demarcados?", "Sinalização visível?", "Prateleiras identificadas?", "Retorno ao lugar?", "Arquivos organizados?"] },
    { n: "3 - LIMPEZA", q: ["Piso e corredores?", "Máquinas e vazamentos?", "Lixeiras identificadas?", "Iluminação?", "Fontes de sujeira?"] },
    { n: "4 - PADRONIZAÇÃO", q: ["Uso de EPIs?", "Quadros organizados?", "Extintores livres?", "Ambientes comuns?", "Padrão de cores?", "Ações anteriores?"] },
    { n: "5 - DISCIPLINA", q: ["Autoavaliação?", "Política qualidade?", "Manutenção padrões?", "Melhorias?", "Limpeza turnos?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, step = 0, editIdx = -1;
let chartRadar, chartBar;

// Fundo Animado
const imagens = ["https://images.unsplash.com/photo-1504917595217-d4dc5ebe6122?auto=format&fit=crop&q=80&w=1920", "https://images.unsplash.com/photo-1581092160562-40aa08e78837?auto=format&fit=crop&q=80&w=1920"];
let iImg = 0;
setInterval(() => { document.getElementById('bgBody').style.backgroundImage = `url('${imagens[iImg]}')`; iImg = (iImg + 1) % imagens.length; }, 10000);

function iniciar() {
    const s = document.getElementById('setor').value;
    if(!s) return alert("Selecione o setor");
    if(editIdx === -1) audit = { id: Date.now(), setor: s, auditor: document.getElementById('auditor').value, data: document.getElementById('data').value, respostas: [] };
    step = 0; renderPasso();
}

function renderPasso() {
    const s = sensos[step];
    let h = `<div class="card"><h2>${s.n}</h2>`;
    s.q.forEach((q, i) => {
        const val = (audit.respostas[step]?.notas) ? audit.respostas[step].notas[i] : "";
        h += `<div style="margin-bottom:12px"><strong>${q}</strong>
              <select class="nota-sel"><option value="">-- Nota --</option>
              ${[10,8,6,4,2].map(v => `<option value="${v}" ${val == v ? 'selected' : ''}>${v}</option>`).join('')}
              </select></div>`;
    });
    h += `<label>Plano de Ação Corretiva:</label><textarea id="plano" rows="3">${audit.respostas[step]?.plano || ""}</textarea>
          <button class="btn-p" onclick="salvarPasso()">Gravar Passo</button></div>`;
    document.getElementById('box_perguntas').innerHTML = h;
    show('senso');
}

function salvarPasso() {
    const notas = Array.from(document.querySelectorAll('.nota-sel')).map(s => Number(s.value));
    if(notas.includes(0)) return alert("Dê nota a todos os itens!");
    audit.respostas[step] = { media: (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1), plano: document.getElementById('plano').value.trim(), notas };
    step++;
    if(step < sensos.length) renderPasso();
    else {
        if(editIdx > -1) db[editIdx] = audit; else db.push(audit);
        localStorage.setItem("j2m_db", JSON.stringify(db));
        editIdx = -1; abrirDash();
    }
}

function abrirDash() {
    const fS = document.getElementById('fSetor');
    const fA = document.getElementById('fAno');
    fS.innerHTML = '<option value="ALL">Todos os Setores</option>';
    fA.innerHTML = '<option value="ALL">Ano (Todos)</option>';
    [...new Set(db.map(x=>x.setor))].sort().forEach(s => fS.innerHTML += `<option value="${s}">${s}</option>`);
    [...new Set(db.map(x => new Date(x.data + "T00:00:00").getFullYear()))].sort((a,b)=>b-a).forEach(a => fA.innerHTML += `<option value="${a}">${a}</option>`);
    renderRelatorio();
    renderListaHome();
    show('dashboard');
}

function renderRelatorio() {
    const selS = document.getElementById('fSetor').value;
    const selA = document.getElementById('fAno').value;
    const selM = document.getElementById('fMes').value;

    const filtrados = db.filter(a => {
        const dt = new Date(a.data + "T00:00:00");
        return (selS === "ALL" || a.setor === selS) && (selA === "ALL" || dt.getFullYear().toString() === selA) && (selM === "ALL" || dt.getMonth().toString() === selM);
    });

    if(!filtrados.length) { document.getElementById('area_impressao').innerHTML = ""; return; }
    
    const ult = filtrados[filtrados.length-1];
    const notaF = (ult.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);
    document.getElementById('mediaFiltro').innerText = notaF;
    document.getElementById('totalAuditorias').innerText = filtrados.length;

    // Gerar HTML para Impressão
    let hPrint = `
    <div class="folha-rosto">
        <h2 style="color:var(--primary)">J2M - EXCELÊNCIA OPERACIONAL</h2>
        <hr><h1>RELATÓRIO DE AUDITORIA 5S</h1>
        <div style="font-size:22px; margin:20px 0;">SETOR: <b>${ult.setor}</b></div>
        <div>Data: ${ult.data} | Auditor: ${ult.auditor}</div>
        <div style="font-size:100px; font-weight:bold; color:var(--primary); margin:20px 0;">${notaF}</div>
        <div style="display:grid; grid-template-columns: 1fr 1fr; gap:20px">
            <div><h4>Radar do Setor</h4><canvas id="radarPrint"></canvas></div>
            <div><h4>Comparativo Setores</h4><canvas id="barraPrint"></canvas></div>
        </div>
    </div>
    <div class="page-break">
        <h3>CHECKLIST E PLANOS DE AÇÃO</h3>
        <table class="table-checklist">
            <tr><th>Item Avaliado</th><th>Nota</th><th>Plano de Ação</th></tr>`;
    
    sensos.forEach((s, si) => {
        hPrint += `<tr><td colspan="3" class="senso-header">${s.n}</td></tr>`;
        s.q.forEach((q, qi) => {
            hPrint += `<tr><td>${q}</td><td>${ult.respostas[si].notas[qi]}</td>
            ${qi===0 ? `<td rowspan="${s.q.length}" style="vertical-align:top"><b>Ação:</b> ${ult.respostas[si].plano || "N/A"}</td>`:''}</tr>`;
        });
    });
    document.getElementById('area_impressao').innerHTML = hPrint + `</table></div>`;

    renderGraficos(filtrados, ult);
}

function renderGraficos(filtrados, ult) {
    if(chartRadar) chartRadar.destroy(); if(chartBar) chartBar.destroy();
    
    const configRadar = {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Saúde", "Disciplina"],
            datasets: [{ label: ult.setor, data: ult.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                       { label: 'Meta', data: [8,8,8,8,8], borderColor: '#28a745', borderDash:[5,5], fill:false }]
        },
        options: { scales: { r: { min:0, max:10 } } }
    };

    chartRadar = new Chart(document.getElementById('cRadar'), configRadar);
    new Chart(document.getElementById('radarPrint'), configRadar);

    const sts = [...new Set(db.map(d=>d.setor))];
    const nts = sts.map(s => {
        const d = db.filter(x=>x.setor===s);
        return (d.reduce((acc, a) => acc + (a.respostas.reduce((x,y)=>x+Number(y.media),0)/5), 0) / d.length).toFixed(1);
    });

    const configBar = {
        type: 'bar',
        data: { labels: sts, datasets: [{ label: 'Média Final', data: nts, backgroundColor: '#5d5a51' }] },
        options: { scales: { y: { min:0, max:10 } } }
    };

    chartBar = new Chart(document.getElementById('cBarra'), configBar);
    new Chart(document.getElementById('barraPrint'), configBar);
}

function renderListaHome() {
    let h = "";
    db.slice().reverse().forEach((a, idx) => {
        const rIdx = db.length - 1 - idx;
        h += `<div style="display:flex; justify-content:space-between; border-bottom:1px solid #eee; padding:8px 0;">
                <span>${a.data} - <b>${a.setor}</b></span>
                <div><button class="btn-edit" onclick="editarAuditoria(${rIdx})">✎</button> 
                <button class="btn-del" onclick="excluirAuditoria(${rIdx})">X</button></div></div>`;
    });
    document.getElementById('lista_recentes').innerHTML = h;
}

function editarAuditoria(idx) {
    editIdx = idx; audit = JSON.parse(JSON.stringify(db[idx]));
    document.getElementById('setor').value = audit.setor;
    document.getElementById('auditor').value = audit.auditor;
    document.getElementById('data').value = audit.data;
    iniciar();
}

function excluirAuditoria(idx) {
    if(confirm("Excluir registro?")) { db.splice(idx,1); localStorage.setItem("j2m_db", JSON.stringify(db)); renderListaHome(); }
}

function show(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }
renderListaHome();
</script>
</body>
</html>

<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025 Lean</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>

<style>
:root { --primary: #f06639; --secondary: #3d3d3d; --meta: #28a745; --danger: #dc3545; }
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}
body { margin:0; background:#f4f4f4; color:#333; }

/* Interface */
header { background:var(--secondary); color:white; padding:15px; text-align:center; border-bottom:5px solid var(--primary); font-weight:bold; }
.screen { display:none; padding:20px; max-width:1000px; margin:auto; }
.active { display:block; }
.card { background:white; padding:20px; border-radius:8px; margin-bottom:15px; box-shadow:0 2px 5px rgba(0,0,0,0.1); }

/* Form */
select, input, textarea { width:100%; padding:12px; border-radius:4px; border:1px solid #ccc; margin-bottom:10px; font-size:16px; }
.btn-p { background:var(--primary); color:white; border:none; padding:15px; width:100%; border-radius:4px; cursor:pointer; font-weight:bold; }
.btn-s { background:var(--secondary); color:white; border:none; padding:10px; width:100%; border-radius:4px; cursor:pointer; margin-top:5px; }

/* Relatório e Folha de Rosto */
.folha-rosto { text-align:center; padding:40px; border:2px solid #333; margin-bottom:30px; background:white; }
.nota-final { font-size: 72px; font-weight: bold; color: var(--primary); margin: 20px 0; }
.tabela-acoes { width:100%; border-collapse:collapse; margin-top:20px; }
.tabela-acoes th, .tabela-acoes td { border:1px solid #ddd; padding:12px; text-align:left; }
.tabela-acoes th { background:#f8f9fa; }

@media print {
    .no-print { display:none!important; }
    .screen { display:block!important; padding:0; }
    .page-break { page-break-before: always; }
    body { background:white; }
    .card { box-shadow:none; border:none; }
}
</style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - LEAN MANUFACTURING</header>

<div id="scr_home" class="screen active">
    <div class="card">
        <h3>Identificação da Auditoria</h3>
        <label>Setor:</label>
        <select id="setor">
            <option value="">-- SELECIONE --</option>
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="ALMOXARIFADO">ALMOXARIFADO</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-p" onclick="iniciar()">INICIAR AUDITORIA</button>
        <button class="btn-s" onclick="sincronizar()">SINCRONIZAR NUVEM</button>
        <button class="btn-s" onclick="abrirDash()">RELATÓRIOS E FILTROS</button>
    </div>
</div>

<div id="scr_audit" class="screen">
    <div id="box_perguntas"></div>
</div>

<div id="scr_dash" class="screen">
    <div class="no-print card">
        <h3>Filtros de Relatório</h3>
        <div style="display:flex; gap:10px">
            <select id="f_setor" onchange="renderRelatorio()"></select>
            <select id="f_mes" onchange="renderRelatorio()">
                <option value="ALL">Mês (Todos)</option>
                <option value="0">Janeiro</option><option value="1">Fevereiro</option><option value="2">Março</option>
                <option value="3">Abril</option><option value="4">Maio</option><option value="5">Junho</option>
            </select>
        </div>
        <button class="btn-p" onclick="window.print()">GERAR PDF / IMPRIMIR</button>
        <button class="btn-s" onclick="location.reload()">VOLTAR</button>
    </div>

    <div id="area_relatorio">
        </div>
</div>

<script>
const API = "https://script.google.com/macros/s/AKfycbzcntNB7ErwLkye0Y7kAoqneoeMAs_MMe7YvszVdHGrHJUGacpixxAYV9LDppBlUoNx/exec";

const sensos = [
    { n: "1 - SELEÇÃO", q: ["Itens necessários?", "Obsoletos descartados?", "Ferramentas em excesso?", "Gestão à vista?", "Avisos atuais?"] },
    { n: "2 - ORDENAÇÃO", q: ["Paletes demarcados?", "Sinalização visível?", "Prateleiras idêntificadas?", "Ferramentas no lugar?", "Documentos organizados?"] },
    { n: "3 - LIMPEZA", q: ["Piso limpo?", "Máquinas sem vazamentos?", "Lixeiras corretas?", "Iluminação limpa?", "Fontes de sujeira?"] },
    { n: "4 - PADRONIZAÇÃO", q: ["Uso de EPIs?", "Quadros organizados?", "Extintores livres?", "Áreas comuns?", "Cores padrões?", "Ações anteriores?"] },
    { n: "5 - DISCIPLINA", q: ["Checklist diário?", "Conhece Política?", "Mantém padrão?", "Melhorias feitas?", "Organização turnos?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, step = 0;

function iniciar() {
    if(!document.getElementById('setor').value) return alert("Selecione o setor");
    audit = { 
        id: Date.now(), 
        setor: document.getElementById('setor').value, 
        auditor: document.getElementById('auditor').value, 
        data: document.getElementById('data').value, 
        respostas: [] 
    };
    step = 0; renderPasso();
}

function renderPasso() {
    const s = sensos[step];
    let h = `<div class="card"><h2>${s.n}</h2>`;
    s.q.forEach((pergunta, i) => {
        h += `<div style="margin-bottom:15px"><strong>${pergunta}</strong>
              <select class="nota-sel"><option value="">-- NOTA (Evidências) --</option>
              <option value="10">10 (Excelente - 0 evidências)</option>
              <option value="8">8 (Bom - 1 evidência)</option>
              <option value="6">6 (Média - 2 evidências)</option>
              <option value="4">4 (Ruim - 3 evidências)</option>
              <option value="2">2 (Crítico - 4+ evidências)</option></select></div>`;
    });
    h += `<label>Plano de Ação (Obrigatório se Média < 6.0):</label>
          <textarea id="plano" placeholder="O que será feito?"></textarea>
          <button class="btn-p" onclick="salvarPasso()">PRÓXIMO</button></div>`;
    document.getElementById('box_perguntas').innerHTML = h;
    show('scr_audit');
}

function salvarPasso() {
    const notas = Array.from(document.querySelectorAll('.nota-sel')).map(s => Number(s.value));
    if(notas.includes(0)) return alert("Preencha todas as notas!");
    
    const media = (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1);
    const plano = document.getElementById('plano').value.trim();
    
    if(media < 6 && plano.length < 5) return alert("Média baixa exige Plano de Ação!");

    audit.respostas.push({ media, plano });
    step++;
    if(step < sensos.length) renderPasso();
    else {
        db.push(audit);
        localStorage.setItem("j2m_db", JSON.stringify(db));
        fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
        abrirDash();
    }
}

function abrirDash() {
    const f = document.getElementById('f_setor');
    f.innerHTML = '<option value="ALL">Setor (Todos)</option>';
    [...new Set(db.map(x=>x.setor))].forEach(s => f.innerHTML += `<option value="${s}">${s}</option>`);
    renderRelatorio();
    show('scr_dash');
}

function renderRelatorio() {
    const selS = document.getElementById('f_setor').value;
    const selM = document.getElementById('f_mes').value;
    
    const filtrados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return (selS === "ALL" || a.setor === selS) && (selM === "ALL" || d.getMonth() == selM);
    });

    const area = document.getElementById('area_relatorio');
    if(!filtrados.length) { area.innerHTML = "<h3>Sem dados para este filtro.</h3>"; return; }
    
    const ult = filtrados[filtrados.length-1];
    const notaFinal = (ult.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);

    // FOLHA DE ROSTO
    let html = `
    <div class="folha-rosto">
        <img src="https://j2m.com.br/wp-content/uploads/2021/08/logo-j2m.png" style="width:150px; margin-bottom:20px">
        <h1>RELATÓRIO DE AUDITORIA 5S</h1>
        <div style="font-size:20px; margin-bottom:10px">SETOR: <strong>${ult.setor}</strong></div>
        <div style="font-size:18px">DATA: ${ult.data} | AUDITOR: ${ult.auditor}</div>
        <div class="nota-final">${notaFinal}</div>
        <div style="width:100%; max-width:500px; margin:auto">
            <canvas id="radarPrint"></canvas>
        </div>
    </div>
    <div class="page-break card">
        <h3>PLANO DE AÇÃO POR SENSO</h3>
        <table class="tabela-acoes">
            <tr><th>Senso</th><th>Nota</th><th>Plano de Ação Corretiva</th></tr>`;

    ult.respostas.forEach((r, i) => {
        if(r.plano) {
            html += `<tr><td>${sensos[i].n}</td><td>${r.media}</td><td>${r.plano}</td></tr>`;
        }
    });

    area.innerHTML = html + `</table></div>`;

    // Gerar Radar do Relatório
    setTimeout(() => {
        new Chart(document.getElementById('radarPrint'), {
            type: 'radar',
            data: {
                labels: ["Seleção", "Ordenação", "Limpeza", "Saúde", "Disciplina"],
                datasets: [
                    { label: ult.setor, data: ult.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                    { label: 'Meta (8.0)', data: [8,8,8,8,8], borderColor: '#28a745', borderDash: [5,5], fill: false, pointRadius:0 }
                ]
            },
            options: { scales: { r: { min:0, max:10 } }, plugins: { legend: { position: 'bottom' } } }
        });
    }, 100);
}

function show(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }

async function sincronizar() {
    alert("Buscando dados na nuvem...");
    try {
        const r = await fetch(API);
        const data = await r.json();
        db = data.slice(1).map(row => ({ id: row[10], setor: row[1], auditor: row[3], data: row[9], respostas: JSON.parse(row[11]) }));
        localStorage.setItem("j2m_db", JSON.stringify(db));
        abrirDash();
    } catch(e) { alert("Erro de sincronização."); }
}
</script>
</body>
</html>

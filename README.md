<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - Executivo v4.0</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
:root { --primary: #f06639; --secondary: #3d3d3d; --meta: #28a745; --danger: #dc3545; }
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}
body { margin:0; background:#f4f4f4; color:#333; }

header { background:var(--secondary); color:white; padding:15px; text-align:center; border-bottom:5px solid var(--primary); font-weight:bold; }
.screen { display:none; padding:20px; max-width:1000px; margin:auto; }
.active { display:block; }
.card { background:white; padding:20px; border-radius:8px; margin-bottom:15px; box-shadow:0 2px 5px rgba(0,0,0,0.1); }

/* Form e Botões */
select, input, textarea { width:100%; padding:12px; border-radius:4px; border:1px solid #ccc; margin-bottom:10px; font-size:16px; }
.btn-p { background:var(--primary); color:white; border:none; padding:15px; width:100%; border-radius:4px; cursor:pointer; font-weight:bold; margin-bottom:5px; }
.btn-s { background:var(--secondary); color:white; border:none; padding:10px; width:100%; border-radius:4px; cursor:pointer; margin-bottom:5px; }
.btn-edit { background: #ffc107; color: black; padding: 5px 10px; border-radius: 4px; border: none; cursor: pointer; }
.btn-del { background: var(--danger); color: white; padding: 5px 10px; border-radius: 4px; border: none; cursor: pointer; }

/* Relatório Estilizado */
.folha-rosto { text-align:center; padding:40px; border:2px solid #333; height: 950px; background:white; position: relative; }
.nota-final { font-size: 90px; font-weight: bold; color: var(--primary); margin: 30px 0; }
.table-rep { width:100%; border-collapse:collapse; margin-top:15px; }
.table-rep th, .table-rep td { border:1px solid #ddd; padding:12px; text-align:left; font-size: 14px; }
.page-break { page-break-before: always; padding-top: 20px; min-height: 950px; background: white; border: 2px solid #333; padding: 20px; }

@media print {
    .no-print { display:none!important; }
    .screen { display:block!important; padding:0; }
    body { background:white; }
    .card { box-shadow:none; border:none; }
}
</style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - RELATÓRIO EXECUTIVO</header>

<div id="scr_home" class="screen active">
    <div class="card">
        <h3>Nova Auditoria / Editar</h3>
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
        <button class="btn-p" onclick="iniciar()">INICIAR AVALIAÇÃO</button>
        <button class="btn-s" onclick="abrirDash()">DASHBOARD E HISTÓRICO</button>
    </div>
    
    <div class="card no-print">
        <h3>Auditorias Recentes</h3>
        <div id="lista_recentes"></div>
    </div>
</div>

<div id="scr_audit" class="screen">
    <div id="box_perguntas"></div>
</div>

<div id="scr_dash" class="screen">
    <div class="no-print card">
        <h3>Filtros de Exportação</h3>
        <div style="display:flex; gap:10px">
            <select id="f_setor" onchange="renderRelatorio()"></select>
            <select id="f_mes" onchange="renderRelatorio()">
                <option value="ALL">Meses (Todos)</option>
                <option value="0">Janeiro</option><option value="1">Fevereiro</option><option value="2">Março</option>
            </select>
        </div>
        <button class="btn-p" onclick="window.print()">IMPRIMIR RELATÓRIO (2 PÁGS)</button>
        <button class="btn-s" onclick="location.reload()">VOLTAR</button>
    </div>

    <div id="area_impressao"></div>
</div>

<script>
const sensos = [
    { n: "1 - SELEÇÃO", q: ["Equipamentos necessários?", "Itens duplicados?", "Acondicionamento correto?", "Checklists disponíveis?", "Avisos atuais?"] },
    { n: "2 - ORDENAÇÃO", q: ["Demarcação paletes?", "Sinalização visível?", "Identificação prateleiras?", "Retorno ao lugar?", "Arquivos/Documentos?"] },
    { n: "3 - LIMPEZA", q: ["Piso/Lixo?", "Máquinas/Vazamentos?", "Lixeiras/Sacos?", "Iluminação?", "Fontes sujeira?"] },
    { n: "4 - PADRONIZAÇÃO", q: ["EPIs?", "Gestão Visual?", "Extintores?", "Higiene comum?", "Cores padrões?", "Ações anteriores?"] },
    { n: "5 - DISCIPLINA", q: ["Autoavaliação?", "Política Qualidade?", "Manutenção padrões?", "Melhorias?", "Limpeza turnos?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, step = 0, editIdx = -1;

function iniciar() {
    const s = document.getElementById('setor').value;
    if(!s) return alert("Selecione o setor");
    if(editIdx === -1) {
        audit = { id: Date.now(), setor: s, auditor: document.getElementById('auditor').value, data: document.getElementById('data').value, respostas: [] };
    }
    step = 0; renderPasso();
}

function renderPasso() {
    const s = sensos[step];
    let h = `<div class="card"><h2>${s.n}</h2>`;
    s.q.forEach((q, i) => {
        const valAnterior = (audit.respostas[step] && audit.respostas[step].notas) ? audit.respostas[step].notas[i] : "";
        h += `<div style="margin-bottom:12px"><strong>${q}</strong>
              <select class="nota-sel">
                <option value="">-- Nota --</option>
                ${[10,8,6,4,2].map(v => `<option value="${v}" ${valAnterior == v ? 'selected' : ''}>${v}</option>`).join('')}
              </select></div>`;
    });
    h += `<label>Plano de Ação Corretiva:</label>
          <textarea id="plano" rows="3">${audit.respostas[step]?.plano || ""}</textarea>
          <button class="btn-p" onclick="salvarPasso()">GRAVAR SENSO</button></div>`;
    document.getElementById('box_perguntas').innerHTML = h;
    show('scr_audit');
}

function salvarPasso() {
    const notas = Array.from(document.querySelectorAll('.nota-sel')).map(s => Number(s.value));
    if(notas.includes(0)) return alert("Dê nota a todos os itens!");
    
    const media = (notas.reduce((a,b)=>a+b,0)/notas.length).toFixed(1);
    const plano = document.getElementById('plano').value.trim();

    audit.respostas[step] = { media, plano, notas };
    step++;
    
    if(step < sensos.length) renderPasso();
    else {
        if(editIdx > -1) db[editIdx] = audit; else db.push(audit);
        localStorage.setItem("j2m_db", JSON.stringify(db));
        editIdx = -1;
        alert("Salvo com sucesso!");
        abrirDash();
    }
}

function abrirDash() {
    const f = document.getElementById('f_setor');
    f.innerHTML = '<option value="ALL">Todos os Setores</option>';
    [...new Set(db.map(x=>x.setor))].forEach(s => f.innerHTML += `<option value="${s}">${s}</option>`);
    renderRelatorio();
    renderListaHome();
    show('scr_dash');
}

function renderListaHome() {
    let h = "";
    db.slice().reverse().forEach((a, idx) => {
        const realIdx = db.length - 1 - idx;
        h += `<div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid #eee; padding:8px 0;">
                <span>${a.data} - ${a.setor}</span>
                <div>
                    <button class="btn-edit" onclick="editarAuditoria(${realIdx})">✎</button>
                    <button class="btn-del" onclick="excluirAuditoria(${realIdx})">X</button>
                </div>
              </div>`;
    });
    document.getElementById('lista_recentes').innerHTML = h;
}

function editarAuditoria(idx) {
    editIdx = idx;
    audit = JSON.parse(JSON.stringify(db[idx]));
    document.getElementById('setor').value = audit.setor;
    document.getElementById('auditor').value = audit.auditor;
    document.getElementById('data').value = audit.data;
    iniciar();
}

function excluirAuditoria(idx) {
    if(confirm("Excluir este relatório permanentemente?")) {
        db.splice(idx, 1);
        localStorage.setItem("j2m_db", JSON.stringify(db));
        renderListaHome();
        if(document.getElementById('scr_dash').classList.contains('active')) abrirDash();
    }
}

function renderRelatorio() {
    const selS = document.getElementById('f_setor').value;
    const selM = document.getElementById('f_mes').value;
    const filtrados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return (selS === "ALL" || a.setor === selS) && (selM === "ALL" || d.getMonth() == selM);
    });

    const area = document.getElementById('area_impressao');
    if(!filtrados.length) { area.innerHTML = "<h3>Nenhum relatório encontrado.</h3>"; return; }
    
    const ult = filtrados[filtrados.length-1];
    const notaFinal = (ult.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);

    area.innerHTML = `
    <div class="folha-rosto">
        <h2 style="color:var(--primary)">J2M - EXCELÊNCIA OPERACIONAL</h2>
        <hr>
        <h1>RELATÓRIO DE AUDITORIA 5S</h1>
        <div style="margin-top:40px; font-size:22px">SETOR: <strong>${ult.setor}</strong></div>
        <div style="font-size:18px">DATA: ${ult.data} | AUDITOR: ${ult.auditor}</div>
        <div class="nota-final">${notaFinal}</div>
        <div style="width:450px; margin:auto"><canvas id="radarRep"></canvas></div>
        <div style="position:absolute; bottom:40px; width:100%; text-align:center; font-size:12px">Meta de Pontuação: 8.0</div>
    </div>
    
    <div class="page-break">
        <h3>COMPARAÇÃO DE NOTAS ENTRE SETORES</h3>
        <div style="height:300px"><canvas id="barraRep"></canvas></div>
        
        <h3 style="margin-top:40px">PLANOS DE AÇÃO CORRETIVA</h3>
        <table class="table-rep">
            <tr style="background:#eee"><th>Senso</th><th>Nota</th><th>Ação Necessária</th></tr>
            ${ult.respostas.map((r, i) => r.plano ? `<tr><td>${sensos[i].n}</td><td>${r.media}</td><td>${r.plano}</td></tr>` : '').join('')}
        </table>
    </div>`;

    setTimeout(() => {
        new Chart(document.getElementById('radarRep'), {
            type: 'radar',
            data: {
                labels: ["Seleção", "Ordenação", "Limpeza", "Saúde", "Disciplina"],
                datasets: [{ label: ult.setor, data: ult.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                           { label: 'Meta', data: [8,8,8,8,8], borderColor: '#28a745', borderDash:[5,5], fill:false }]
            },
            options: { scales: { r: { min:0, max:10 } } }
        });

        const setores = [...new Set(db.map(d=>d.setor))];
        const notas = setores.map(s => {
            const d = db.filter(x=>x.setor===s);
            return (d.reduce((acc, a) => acc + (a.respostas.reduce((x,y)=>x+Number(y.media),0)/5), 0) / d.length).toFixed(1);
        });

        new Chart(document.getElementById('barraRep'), {
            type: 'bar',
            data: { labels: setores, datasets: [{ label: 'Média por Setor', data: notas, backgroundColor: '#5d5a51' }] },
            options: { maintainAspectRatio: false, scales: { y: { min:0, max:10 } } }
        });
    }, 150);
}

function show(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }
renderListaHome();
</script>

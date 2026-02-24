<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025 Multi-Ano</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
:root { --primary: #f06639; --secondary: #3d3d3d; --meta: #28a745; --danger: #dc3545; }
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}
body { margin:0; background:#f4f4f4; color:#333; }

header { background:var(--secondary); color:white; padding:15px; text-align:center; border-bottom:5px solid var(--primary); font-weight:bold; }
.screen { display:none; padding:20px; max-width:1100px; margin:auto; }
.active { display:block; }
.card { background:white; padding:20px; border-radius:8px; margin-bottom:15px; box-shadow:0 2px 5px rgba(0,0,0,0.1); }

select, input, textarea { width:100%; padding:12px; border-radius:4px; border:1px solid #ccc; margin-bottom:10px; font-size:16px; }
.btn-p { background:var(--primary); color:white; border:none; padding:15px; width:100%; border-radius:4px; cursor:pointer; font-weight:bold; margin-bottom:5px; }
.btn-s { background:var(--secondary); color:white; border:none; padding:10px; width:100%; border-radius:4px; cursor:pointer; margin-bottom:5px; }
.btn-edit { background: #ffc107; color: black; padding: 5px 10px; border: none; cursor: pointer; border-radius:4px; }
.btn-del { background: var(--danger); color: white; padding: 5px 10px; border: none; cursor: pointer; border-radius:4px; }

/* Estilo Relatório */
.folha-rosto { text-align:center; padding:30px; border:2px solid #333; min-height: 1000px; background:white; position: relative; }
.nota-final { font-size: 100px; font-weight: bold; color: var(--primary); margin: 20px 0; }
.table-checklist { width:100%; border-collapse:collapse; margin-top:10px; font-size: 12px; }
.table-checklist th, .table-checklist td { border:1px solid #333; padding:6px; text-align:left; }
.table-checklist th { background:#eee; text-align:center; }
.senso-header { background: #555 !important; color: white; font-weight: bold; text-align: center !important; font-size: 14px; }
.page-break { page-break-before: always; min-height: 1000px; background: white; border: 2px solid #333; padding: 20px; }

@media print {
    .no-print { display:none!important; }
    .screen { display:block!important; padding:0; }
    body { background:white; }
    .folha-rosto, .page-break { border: none; }
}
</style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - GESTÃO INDUSTRIAL</header>

<div id="scr_home" class="screen active">
    <div class="card">
        <h3>Nova Auditoria</h3>
        <label>Setor:</label>
        <select id="setor">
            <option value="">-- SELECIONE O SETOR --</option>
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
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data da Auditoria:</label><input type="date" id="data">
        <button class="btn-p" onclick="iniciar()">INICIAR AVALIAÇÃO</button>
        <button class="btn-s" onclick="abrirDash()">ACEDER RELATÓRIOS</button>
    </div>
    <div class="card no-print">
        <h3>Histórico Local (Editar/Excluir)</h3>
        <div id="lista_recentes"></div>
    </div>
</div>

<div id="scr_audit" class="screen">
    <div id="box_perguntas"></div>
</div>

<div id="scr_dash" class="screen">
    <div class="no-print card">
        <h3>Filtros de Relatório</h3>
        <div style="display:grid; grid-template-columns: 2fr 1fr 1fr; gap:10px">
            <select id="f_setor" onchange="renderRelatorio()"></select>
            <select id="f_ano" onchange="renderRelatorio()"></select>
            <select id="f_mes" onchange="renderRelatorio()">
                <option value="ALL">Mês (Todos)</option>
                <option value="0">Jan</option><option value="1">Fev</option><option value="2">Mar</option>
                <option value="3">Abr</option><option value="4">Mai</option><option value="5">Jun</option>
                <option value="6">Jul</option><option value="7">Ago</option><option value="8">Set</option>
                <option value="9">Out</option><option value="10">Nov</option><option value="11">Dez</option>
            </select>
        </div>
        <button class="btn-p" style="margin-top:10px" onclick="window.print()">IMPRIMIR RELATÓRIO PDF</button>
        <button class="btn-s" onclick="location.reload()">VOLTAR AO INÍCIO</button>
    </div>
    <div id="area_impressao"></div>
</div>

<script>
const sensos = [
    { n: "1 - SELEÇÃO", q: ["Equipamentos necessários?", "Itens duplicados na bancada?", "Ferramentas acondicionadas corretamente?", "Quadros de gestão atualizados?", "Avisos e informativos necessários?"] },
    { n: "2 - ORDENAÇÃO", q: ["Locais de paletes demarcados?", "Sinalização de caminhos visível?", "Prateleiras identificadas?", "Ferramentas retornam ao lugar?", "Documentos organizados?"] },
    { n: "3 - LIMPEZA", q: ["Piso e corredores limpos?", "Máquinas sem vazamentos?", "Lixeiras limpas/identificadas?", "Iluminação e janelas limpas?", "Fontes de sujeira eliminadas?"] },
    { n: "4 - PADRONIZAÇÃO", q: ["Uso correto de EPIs?", "Quadros organizados?", "Extintores livres?", "Ambientes comuns limpos?", "Padrão de cores respeitado?", "Ações anteriores resolvidas?"] },
    { n: "5 - AUTODISCIPLINA", q: ["Autoavaliação realizada?", "Conhecem a política da qualidade?", "Mantêm padrão sem cobrança?", "Melhorias implementadas?", "Organização mantida nos turnos?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, step = 0, editIdx = -1;

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
        const val = (audit.respostas[step] && audit.respostas[step].notas) ? audit.respostas[step].notas[i] : "";
        h += `<div style="margin-bottom:12px"><strong>${q}</strong>
              <select class="nota-sel"><option value="">-- Nota --</option>
              ${[10,8,6,4,2].map(v => `<option value="${v}" ${val == v ? 'selected' : ''}>${v}</option>`).join('')}
              </select></div>`;
    });
    h += `<label>Plano de Ação Corretiva:</label><textarea id="plano" rows="3">${audit.respostas[step]?.plano || ""}</textarea>
          <button class="btn-p" onclick="salvarPasso()">GRAVAR SENSO</button></div>`;
    document.getElementById('box_perguntas').innerHTML = h;
    show('scr_audit');
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
    const fS = document.getElementById('f_setor');
    const fA = document.getElementById('f_ano');
    fS.innerHTML = '<option value="ALL">Setor (Todos)</option>';
    fA.innerHTML = '<option value="ALL">Ano (Todos)</option>';
    
    [...new Set(db.map(x=>x.setor))].sort().forEach(s => fS.innerHTML += `<option value="${s}">${s}</option>`);
    
    const anos = [...new Set(db.map(x => new Date(x.data + "T00:00:00").getFullYear()))].sort((a,b)=>b-a);
    anos.forEach(a => fA.innerHTML += `<option value="${a}">${a}</option>`);
    
    renderRelatorio();
    renderListaHome();
    show('scr_dash');
}

function renderListaHome() {
    let h = "";
    db.slice().reverse().forEach((a, idx) => {
        const rIdx = db.length - 1 - idx;
        h += `<div style="display:flex; justify-content:space-between; border-bottom:1px solid #eee; padding:10px;">
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
    if(confirm("Deseja eliminar este registo?")) { db.splice(idx,1); localStorage.setItem("j2m_db", JSON.stringify(db)); renderListaHome(); abrirDash(); }
}

function renderRelatorio() {
    const selS = document.getElementById('f_setor').value;
    const selA = document.getElementById('f_ano').value;
    const selM = document.getElementById('f_mes').value;
    
    const filtrados = db.filter(a => {
        const dt = new Date(a.data + "T00:00:00");
        const matchS = (selS === "ALL" || a.setor === selS);
        const matchA = (selA === "ALL" || dt.getFullYear().toString() === selA);
        const matchM = (selM === "ALL" || dt.getMonth().toString() === selM);
        return matchS && matchA && matchM;
    });

    const area = document.getElementById('area_impressao');
    if(!filtrados.length) { area.innerHTML = "<h3 style='text-align:center;margin-top:50px'>Nenhuma auditoria encontrada para os filtros selecionados.</h3>"; return; }
    
    const ult = filtrados[filtrados.length-1];
    const notaFinal = (ult.respostas.reduce((a,b)=>a+Number(b.media),0)/5).toFixed(1);

    let h = `<div class="folha-rosto">
        <h2 style="color:var(--primary)">J2M - EXCELÊNCIA OPERACIONAL</h2>
        <hr><h1>RELATÓRIO DE AUDITORIA 5S</h1>
        <div style="font-size:22px; margin:20px 0;">SETOR: <b>${ult.setor}</b></div>
        <div>Data: ${ult.data} | Auditor: ${ult.auditor}</div>
        <div class="nota-final">${notaFinal}</div>
        <div style="display:grid; grid-template-columns: 1fr 1fr; gap:20px; margin-top:30px">
            <div class="card"><h4>Pontuação por Senso</h4><canvas id="radarRep"></canvas></div>
            <div class="card"><h4>Ranking Atual (Filtro)</h4><canvas id="barraRep"></canvas></div>
        </div>
        <div style="position:absolute; bottom:20px; width:100%; text-align:center; font-size:12px; color:#666">Meta estabelecida: 8.0 pontos</div>
    </div>
    <div class="page-break">
        <h3>RESPOSTAS DETALHADAS E PLANOS DE AÇÃO</h3>
        <table class="table-checklist">
            <tr><th style="width:70%">Item Avaliado</th><th style="width:10%">Nota</th><th>Plano de Ação Corretiva</th></tr>`;
    sensos.forEach((s, sIdx) => {
        h += `<tr><td colspan="3" class="senso-header">${s.n}</td></tr>`;
        s.q.forEach((q, qIdx) => {
            h += `<tr><td>${q}</td><td style="text-align:center; font-weight:bold;">${ult.respostas[sIdx].notas[qIdx]}</td>
            ${qIdx === 0 ? `<td rowspan="${s.q.length}" style="vertical-align:top; background:#fffdf5"><b>Ação:</b> ${ult.respostas[sIdx].plano || "N/A"}</td>` : ""}</tr>`;
        });
    });
    area.innerHTML = h + `</table></div>`;

    setTimeout(() => {
        new Chart(document.getElementById('radarRep'), { type: 'radar', data: { labels: ["Seleção", "Ordenação", "Limpeza", "Saúde", "Disciplina"], datasets: [{ label: ult.setor, data: ult.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' }, { label: 'Meta', data: [8,8,8,8,8], borderColor: '#28a745', borderDash:[5,5], fill:false }] }, options: { scales: { r: { min:0, max:10 } } } });
        
        const sts = [...new Set(filtrados.map(d=>d.setor))];
        const nts = sts.map(s => {
            const dSetor = filtrados.filter(x=>x.setor===s);
            return (dSetor.reduce((acc, a) => acc + (a.respostas.reduce((x,y)=>x+Number(y.media),0)/5), 0) / dSetor.length).toFixed(1);
        });
        new Chart(document.getElementById('barraRep'), { type: 'bar', data: { labels: sts, datasets: [{ label: 'Nota', data: nts, backgroundColor: '#5d5a51' }] }, options: { scales: { y: { min:0, max:10 } } } });
    }, 200);
}
function show(id) { document.querySelectorAll('.screen').forEach(s => s.classList.remove('active')); document.getElementById(id).classList.add('active'); }
renderListaHome();
</script>
</body>
</html>

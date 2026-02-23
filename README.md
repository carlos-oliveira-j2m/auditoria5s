<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --yellow: #ffc107; --green: #10b981; }
    *{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}
    body { margin:0; background-color: #f0f2f5; position: relative; min-height: 100vh; }
    body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.9); z-index: -1; }
    header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }
    .screen { display:none; padding:20px; max-width: 1200px; margin: auto; }
    .active { display:block; animation: fadeIn 0.4s; }
    .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); border-left: 6px solid var(--primary); margin-bottom: 20px; }
    .kpi-container { display: flex; gap: 20px; margin-bottom: 20px; flex-wrap: wrap; }
    .kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; text-align: center; border-bottom: 5px solid var(--primary); }
    .kpi-card h2 { margin: 0; font-size: 40px; color: var(--primary); }
    button { border: none; padding: 12px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; }
    .btn-primary { background: var(--primary); width: 100%; }
    .btn-pdf { background: var(--green); }
    .btn-edit { background: var(--yellow); color: #000; padding: 5px 10px; font-size: 11px; margin-right: 5px; }
    .btn-del { background: var(--red); padding: 5px 10px; font-size: 11px; }
    select, input, textarea { width: 100%; padding: 12px; border-radius: 4px; border: 1px solid #ccc; margin: 8px 0; font-size: 14px; }
    .teia-box { min-height: 650px; width: 100%; display: flex; justify-content: center; align-items: center; padding: 20px; background: white; border-radius: 8px; }
    .tabela-historico { width: 100%; border-collapse: collapse; margin-top: 20px; background: white; }
    .tabela-historico th, .tabela-historico td { border: 1px solid #ddd; padding: 12px; text-align: left; }
    .pergunta-label { font-weight: bold; color: var(--secondary); display: block; margin-top: 15px; }
    @media print { .no-print { display:none !important; } .screen { display:block; } body::before { background: white; } }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active">
    <div class="card">
        <h2 id="tituloHome">Nova Auditoria</h2>
        <label>Setor:</label>
        <select id="setor">
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="ENG. SOLVE">ENG. SOLVE</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
            <option value="SALA ELETRONICOS">SALA ELETRONICOS</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-primary" onclick="start()">Iniciar Avaliação</button>
        <button style="background:var(--secondary); width:100%; margin-top:10px" onclick="openDashboard()">Ver Dashboard / Histórico</button>
    </div>
</div>

<div id="senso_screen" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="no-print" style="margin-bottom:15px; display:flex; gap:10px">
        <button class="btn-pdf" onclick="window.print()">Salvar PDF / Imprimir</button>
        <button style="background:var(--secondary)" onclick="location.reload()">Início</button>
    </div>
    <div class="kpi-container">
        <div class="kpi-card"><h2 id="media_total">0.0</h2>Média Geral Fábrica</div>
        <div class="kpi-card"><h2 id="qtd_auditorias">0</h2>Total Auditorias</div>
    </div>
    <div class="card teia-box">
        <canvas id="graficoTeia"></canvas>
    </div>
    <div id="lista_auditorias"></div>
</div>

<script>
const checklist = [
    { s: "1 - SELEÇÃO", p: ["Ferramentas/equipamentos necessários para o trabalho diário?","Itens duplicados sobre a bancada?","Ferramentas usadas e acondicionadas corretamente?","Quadros de gestão/checklists atualizados e disponíveis?","Avisos e quadros informativos atuais são necessários?"]},
    { s: "2 - ORDENAÇÃO", p: ["Locais de armazenamento marcados?","Linhas e marcações visíveis?","Prateleiras e equipamentos etiquetados?","Armazenamento de limpeza organizado?","Identificação de departamentos/pessoal visível?","Objetos de uso pessoal em local apropriado?"]},
    { s: "3 - LIMPEZA", p: ["Local de trabalho limpo (bancadas, máquinas)?","Piso e calçadas livres de resíduos/líquidos?","Ferramentas em perfeitas condições técnicas?","Instalações de funcionários limpas/agradáveis?","Coleta seletiva de lixo conforme o padrão?"]},
    { s: "4 - PADRONIZAÇÃO", p: ["Documentação e indicadores atualizados no padrão?","Aspectos ergonômicos atendidos?","Lixeiras apropriadas e marcadas?","Segurança atendida (extintores, placas)?"]},
    { s: "5 - AUTODISCIPLINA", p: ["Consciência e manutenção de padrões?","Checklist de autoavaliação realizado?","Missão, visão e política conhecidos?","Funcionários usando EPI's?","Gestão à vista de limpeza/organização?","Ações corretivas anteriores atendidas?"]}
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let atual = {}, etapa = 0, editandoId = null;
let chartRadar;

function start() {
    if(!editandoId) {
        atual = { id: Date.now(), setor: setor.value, auditor: auditor.value, data: data.value, respostas: [] };
    }
    etapa = 0; mostrarSenso();
}

function mostrarSenso() {
    const senso = checklist[etapa];
    let html = `<div class="card"><h2>${senso.s}</h2>`;
    senso.p.forEach((pergunta, i) => {
        html += `<label class="pergunta-label">${pergunta}</label>
        <select id="n_${i}">
            <option value="10">Excelente: Sem evidências (10)</option>
            <option value="8">Bom: 1 evidência (8)</option>
            <option value="6">Média: 2 evidências (6)</option>
            <option value="4">Deve ser melhorado: 3 evidências (4)</option>
            <option value="2">Deve ser muito melhorado: 4 ou mais evidências (2)</option>
        </select>`;
    });
    html += `<button class="btn-primary" style="margin-top:20px" onclick="proximo()">${etapa < 4 ? 'Próximo' : 'Finalizar'}</button></div>`;
    document.getElementById("senso_screen").innerHTML = html;
    show("senso_screen");
}

function proximo() {
    let soma = 0, qtd = checklist[etapa].p.length;
    for(let i=0; i < qtd; i++) soma += Number(document.getElementById(`n_${i}`).value);
    let media = (soma / qtd).toFixed(1);
    if(!editandoId) atual.respostas.push(Number(media));
    else atual.respostas[etapa] = Number(media);
    etapa++;
    if(etapa < 5) mostrarSenso();
    else {
        if(editandoId) { db[db.findIndex(x => x.id === editandoId)] = atual; }
        else { db.push(atual); }
        localStorage.setItem("j2m_db", JSON.stringify(db));
        editandoId = null; openDashboard();
    }
}

function openDashboard() {
    renderizarDashboard();
    show("dashboard");
}

function renderizarDashboard() {
    if(db.length === 0) return;
    const medias = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + a.respostas[i], 0) / db.length).toFixed(1));
    const geral = (medias.reduce((a, b) => Number(a) + Number(b), 0) / 5).toFixed(1);
    
    media_total.innerText = geral;
    qtd_auditorias.innerText = db.length;

    if(chartRadar) chartRadar.destroy();
    chartRadar = new Chart(document.getElementById("graficoTeia"), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Autodisciplina"],
            datasets: [{ label: 'Média Atual', data: medias, borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.3)', pointRadius: 6 }]
        },
        options: { 
            maintainAspectRatio: false, 
            scales: { r: { min: 0, max: 10, ticks: { stepSize: 2 }, pointLabels: { font: { size: 16, weight: 'bold' } } } } 
        }
    });

    let html = "<h3>Histórico</h3><table class='tabela-historico'><tr><th>Data</th><th>Setor</th><th>Nota</th><th class='no-print'>Ações</th></tr>";
    db.slice().reverse().forEach(a => {
        const nf = (a.respostas.reduce((x,y)=>x+y,0)/5).toFixed(1);
        html += `<tr><td>${a.data}</td><td>${a.setor}</td><td><b>${nf}</b></td>
        <td class='no-print'><button class="btn-edit" onclick="prepararEdicao(${a.id})">Editar</button><button class="btn-del" onclick="excluir(${a.id})">Excluir</button></td></tr>`;
    });
    lista_auditorias.innerHTML = html + "</table>";
}

function excluir(id) { if(confirm("Excluir relatório?")) { db = db.filter(x => x.id !== id); localStorage.setItem("j2m_db", JSON.stringify(db)); renderizarDashboard(); } }

function prepararEdicao(id) {
    editandoId = id;
    atual = JSON.parse(JSON.stringify(db.find(x => x.id === id)));
    show('home');
    document.getElementById('tituloHome').innerText = "Editando Auditoria";
    document.getElementById('setor').value = atual.setor;
    document.getElementById('auditor').value = atual.auditor;
    document.getElementById('data').value = atual.data;
}

function show(id) { document.querySelectorAll(".screen").forEach(s => s.classList.remove("active")); document.getElementById(id).classList.add("active"); }
</script>
</body>
</html>

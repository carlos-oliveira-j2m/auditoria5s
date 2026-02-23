<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Auditoria 5S J2M - Enterprise v2.0</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
    <style>
        :root { --primary: #f06639; --secondary: #5d5a51; --green: #28a745; --blue: #007bff; --danger: #dc3545; --bg-card: #ffffff; }
        * { box-sizing: border-box; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
        body { margin: 0; background: #f4f7f6; color: #333; }
        header { background: var(--secondary); color: white; padding: 20px; text-align: center; border-bottom: 5px solid var(--primary); font-size: 24px; font-weight: bold; }
        .container { max-width: 1000px; margin: 20px auto; padding: 0 15px; }
        .screen { display: none; animation: fadeIn 0.4s; }
        .active { display: block; }
        .card { background: var(--bg-card); padding: 25px; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); margin-bottom: 20px; border-left: 6px solid var(--primary); }
        h2 { color: var(--secondary); margin-top: 0; border-bottom: 2px solid #eee; padding-bottom: 10px; }
        label { font-weight: bold; display: block; margin: 10px 0 5px; color: #444; }
        select, input, textarea { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 5px; font-size: 15px; margin-bottom: 10px; }
        .pergunta-box { background: #f9f9f9; padding: 15px; border-radius: 6px; margin-bottom: 15px; border: 1px solid #eee; }
        .pergunta-txt { font-weight: 600; color: #333; display: block; margin-bottom: 5px; }
        .pergunta-desc { font-size: 13px; color: #666; font-style: italic; display: block; margin-bottom: 10px; line-height: 1.4; }
        .btn-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 20px; }
        button { border: none; padding: 15px; border-radius: 5px; cursor: pointer; font-weight: bold; text-transform: uppercase; transition: 0.3s; color: white; }
        .btn-primary { background: var(--primary); width: 100%; }
        .btn-blue { background: var(--blue); width: 100%; }
        .btn-dark { background: var(--secondary); width: 100%; }
        .invalid { border: 2px solid var(--danger) !important; background: #fff5f5; }
        .kpi-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px; margin-bottom: 20px; }
        .kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; text-align: center; border-bottom: 5px solid var(--primary); }
        .kpi-card h3 { margin: 0; font-size: 32px; color: var(--primary); }
        .chart-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); gap: 20px; margin-bottom: 20px; }
        .plano-acao-relatorio { background: #fff8e1; border-left: 5px solid #ffc107; padding: 10px; margin-top: 10px; font-size: 14px; }
        @media print { .no-print { display: none !important; } .screen { display: block !important; } .card { box-shadow: none; border: 1px solid #ddd; page-break-inside: avoid; } }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div class="container">
    <div id="scr_home" class="screen active">
        <div class="card">
            <h2>Identificação</h2>
            <label>Setor:</label>
            <select id="setor">
                <option value="">-- SELECIONE --</option>
                <option value="ADM FILIAL">ADM FILIAL</option>
                <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
                <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
                <option value="INJEÇÃO">INJEÇÃO</option>
                <option value="MATRIZARIA">MATRIZARIA</option>
                <option value="MONTAGEM">MONTAGEM</option>
                <option value="QUALIDADE">QUALIDADE</option>
                <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
                <option value="MANUTENÇÃO">MANUTENÇÃO</option>
                <option value="ENGENHARIA SOLVE">ENGENHARIA SOLVE</option>
            </select>
            <label>Auditor:</label><input type="text" id="auditor" placeholder="Nome completo">
            <label>Data:</label><input type="date" id="data">
            <button class="btn-primary" onclick="iniciarAudit()">INICIAR NOVA AUDITORIA</button>
            <div class="btn-row">
                <button class="btn-blue" onclick="syncCloud()">🔄 SINCRONIZAR NUVEM</button>
                <button class="btn-dark" onclick="abrirDash()">DASHBOARD / HISTÓRICO</button>
            </div>
        </div>
    </div>

    <div id="scr_audit" class="screen">
        <div id="senso_container"></div>
    </div>

    <div id="scr_dash" class="screen">
        <div class="no-print card">
            <h2>Filtros de Relatório</h2>
            <div class="btn-row">
                <select id="f_setor" onchange="renderDash()"></select>
                <select id="f_mes" onchange="renderDash()">
                    <option value="ALL">Todos os Meses</option>
                    <option value="0">Janeiro</option><option value="1">Fevereiro</option><option value="2">Março</option>
                </select>
            </div>
            <div class="btn-row">
                <button class="btn-blue" onclick="window.print()">GERAR PDF / IMPRIMIR</button>
                <button class="btn-dark" onclick="location.reload()">VOLTAR AO INÍCIO</button>
            </div>
        </div>

        <div class="kpi-grid">
            <div class="kpi-card"><h3 id="k_media">0.0</h3>MÉDIA FÁBRICA</div>
            <div class="kpi-card"><h3 id="k_total">0</h3>TOTAL AUDITORIAS</div>
        </div>

        <div class="chart-grid">
            <div class="card"><canvas id="radarSetor"></canvas></div>
            <div class="card"><canvas id="barraRanking"></canvas></div>
        </div>

        <div id="relatorio_detalhado"></div>
    </div>
</div>

<script>
const URL_API = "https://script.google.com/macros/s/AKfycbzcntNB7ErwLkye0Y7kAoqneoeMAs_MMe7YvszVdHGrHJUGacpixxAYV9LDppBlUoNx/exec";

// AS 26 PERGUNTAS DETALHADAS
const checklist = [
    { s: "SELEÇÃO (UTILIZAÇÃO)", p: [
        {t: "Equipamentos e Ferramentas", d: "Existem itens desnecessários, quebrados ou sem uso no posto de trabalho?"},
        {t: "Materiais e Estoque", d: "Há excesso de matéria-prima ou caixas acumuladas sem necessidade?"},
        {t: "Documentos e Papéis", d: "Existem formulários, desenhos ou ordens de serviço obsoletas?"},
        {t: "Objetos Pessoais", d: "Armários e bancadas estão livres de itens pessoais em excesso?"},
        {t: "Área de Descarte", d: "Os itens para descarte estão em local sinalizado?"}
    ]},
    { s: "ORDENAÇÃO (ARRUMAÇÃO)", p: [
        {t: "Identificação de Itens", d: "Ferramentas e objetos possuem etiquetas de identificação clara?"},
        {t: "Locais Definidos", d: "Cada item tem um lugar fixo definido e demarcado?"},
        {t: "Demarcação de Piso", d: "Corredores, áreas de estoque e segurança estão pintados/marcados?"},
        {t: "Facilidade de Acesso", d: "Itens de uso frequente estão à mão? Leva-se menos de 30s para achar algo?"},
        {t: "Arquivamento", d: "Pastas e documentos digitais estão nomeados e organizados?"}
    ]},
    { s: "LIMPEZA", p: [
        {t: "Piso e Paredes", d: "O chão está livre de poeira, óleo ou lixo?"},
        {t: "Máquinas e Dispositivos", d: "Equipamentos estão limpos, sem vazamentos ou crostas de sujeira?"},
        {t: "Lixeiras", d: "As lixeiras estão limpas, com saco plástico e identificadas?"},
        {t: "Iluminação e Ventilação", d: "Lâmpadas estão limpas e funcionando? Janelas e ventiladores limpos?"},
        {t: "Responsabilidade", d: "A equipe mantém o hábito de limpar após o uso?"}
    ]},
    { s: "PADRONIZAÇÃO (SAÚDE)", p: [
        {t: "EPIs", d: "Todos os colaboradores utilizam os EPIs necessários para a área?"},
        {t: "Gestão Visual", d: "Quadros de avisos estão atualizados e limpos?"},
        {t: "Segurança", d: "Extintores, hidrantes e saídas de emergência estão desobstruídos?"},
        {t: "Condições de Higiene", d: "Bebedouros e áreas comuns estão em condições sanitárias adequadas?"},
        {t: "Uniformização", d: "Colaboradores estão com uniformes limpos e adequados?"},
        {t: "Padrão de Trabalho", d: "Existem POPs ou padrões visuais afixados?"}
    ]},
    { s: "DISCIPLINA", p: [
        {t: "Comprometimento", d: "A equipe demonstra conhecer as regras do 5S?"},
        {t: "Melhoria Contínua", d: "Existem evidências de melhorias feitas desde a última auditoria?"},
        {t: "Cumprimento de Horários", d: "Há pontualidade e respeito às rotinas da empresa?"},
        {t: "Educação e Respeito", d: "O clima organizacional e o trato entre colegas é profissional?"},
        {t: "Manutenção dos Sensos", d: "Os 4 sensos anteriores estão sendo mantidos consistentemente?"}
    ]}
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, sensoAtual = 0;
let radarObj, barraObj;

function iniciarAudit() {
    const s = document.getElementById('setor').value;
    const a = document.getElementById('auditor').value;
    const d = document.getElementById('data').value;
    if(!s || !a || !d) return alert("Preencha todos os campos de identificação!");
    
    audit = { id: Date.now(), setor: s, auditor: a, data: d, respostas: [] };
    sensoAtual = 0;
    renderSenso();
}

function renderSenso() {
    const senso = checklist[sensoAtual];
    let html = `<div class="card"><h2>${sensoAtual + 1}º Senso: ${senso.s}</h2>`;
    
    senso.p.forEach((item, i) => {
        html += `<div class="pergunta-box">
                    <span class="pergunta-txt">${item.t}</span>
                    <span class="pergunta-desc">${item.d}</span>
                    <select class="nota-sel">
                        <option value="">Selecione a Nota...</option>
                        <option value="10">10 - Excelente (Padrão Total)</option>
                        <option value="8">8 - Bom (Pequenos desvios)</option>
                        <option value="6">6 - Regular (Requer atenção)</option>
                        <option value="4">4 - Ruim (Desorganizado)</option>
                        <option value="2">2 - Crítico (Urgente)</option>
                    </select>
                 </div>`;
    });

    html += `<h3>Evidências e Ações do Senso</h3>
             <label>Descrição das Evidências (O que foi observado?):</label>
             <textarea id="evid_txt" rows="3" placeholder="Ex: Vazamento de óleo na prensa 02..."></textarea>
             <label>Plano de Ação (Obrigatório se Nota < 6):</label>
             <textarea id="plano_txt" rows="3" placeholder="O que será feito para corrigir?"></textarea>
             <button class="btn-primary" onclick="salvarSenso()">GRAVAR E CONTINUAR</button></div>`;
    
    document.getElementById('senso_container').innerHTML = html;
    window.scrollTo(0,0);
    show('scr_audit');
}

function salvarSenso() {
    const selects = document.querySelectorAll('.nota-sel');
    let notas = [];
    let incompleto = false;

    selects.forEach(s => {
        if(!s.value) { s.classList.add('invalid'); incompleto = true; }
        else { s.classList.remove('invalid'); notas.push(Number(s.value)); }
    });

    if(incompleto) return alert("Todas as notas deste senso devem ser preenchidas!");

    const media = (notas.reduce((a,b) => a+b, 0) / notas.length).toFixed(1);
    const plano = document.getElementById('plano_txt').value.trim();
    const evid = document.getElementById('evid_txt').value.trim();

    if(media < 6 && plano.length < 5) {
        document.getElementById('plano_txt').classList.add('invalid');
        return alert("Média " + media + ". Plano de Ação OBRIGATÓRIO para notas baixas!");
    }

    audit.respostas.push({ media, plano, evid });
    sensoAtual++;

    if(sensoAtual < checklist.length) renderSenso();
    else finalizarAudit();
}

function finalizarAudit() {
    db.push(audit);
    localStorage.setItem("j2m_db", JSON.stringify(db));
    
    // Enviar via POST
    fetch(URL_API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
    
    alert("Auditoria finalizada e salva com sucesso!");
    abrirDash();
}

async function syncCloud() {
    alert("Sincronizando com a nuvem... Aguarde.");
    try {
        const response = await fetch(URL_API);
        const cloudData = await response.json();
        
        // Transformar dados da planilha em JSON do App
        const parsed = cloudData.slice(1).map(row => ({
            id: row[10], setor: row[1], auditor: row[3], data: row[9],
            respostas: JSON.parse(row[11] || "[]")
        }));

        // Mesclar sem duplicar IDs
        const existingIds = new Set(db.map(a => a.id));
        parsed.forEach(item => {
            if(!existingIds.has(item.id)) db.push(item);
        });

        localStorage.setItem("j2m_db", JSON.stringify(db));
        alert("Sincronização concluída! Dados atualizados.");
        abrirDash();
    } catch(e) {
        alert("Erro na conexão. Verifique a internet e o link do Apps Script.");
    }
}

function abrirDash() {
    const fSetor = document.getElementById('f_setor');
    fSetor.innerHTML = '<option value="ALL">Todos os Setores</option>';
    const setores = [...new Set(db.map(a => a.setor))];
    setores.forEach(s => fSetor.innerHTML += `<option value="${s}">${s}</option>`);
    
    renderDash();
    show('scr_dash');
}

function renderDash() {
    const filterS = document.getElementById('f_setor').value;
    const filterM = document.getElementById('f_mes').value;
    
    const dados = db.filter(a => {
        const dataA = new Date(a.data + "T00:00:00");
        const matchS = filterS === "ALL" || a.setor === filterS;
        const matchM = filterM === "ALL" || dataA.getMonth() == filterM;
        return matchS && matchM;
    });

    if(dados.length === 0) return;
    const ult = dados[dados.length - 1];

    // Média Geral
    document.getElementById('k_total').innerText = db.length;
    let somaT = db.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById('k_media').innerText = (somaT / db.length || 0).toFixed(1);

    // RADAR
    if(radarObj) radarObj.destroy();
    const mediaGeralSensos = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + Number(a.respostas[i]?.media || 0), 0) / db.length).toFixed(1));

    radarObj = new Chart(document.getElementById('radarSetor'), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Saúde", "Disciplina"],
            datasets: [
                { label: ult.setor, data: ult.respostas.map(r=>r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Média Fábrica', data: mediaGeralSensos, borderColor: '#007bff', fill: false, borderDash: [5,5] },
                { label: 'Meta (8.0)', data: [8,8,8,8,8], borderColor: '#28a745', fill: false, borderDash: [2,2], pointRadius: 0 }
            ]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    // BARRA RANKING
    if(barraObj) barraObj.destroy();
    const setoresRanking = [...new Set(db.map(d=>d.setor))];
    const notasRanking = setoresRanking.map(s => {
        const dSetor = db.filter(x=>x.setor===s);
        return (dSetor.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0) / dSetor.length).toFixed(1);
    });

    barraObj = new Chart(document.getElementById('barraRanking'), {
        type: 'bar',
        data: { labels: setoresRanking, datasets: [{ label: 'Nota 5S', data: notasRanking, backgroundColor: '#5d5a51' }] },
        options: { plugins: { datalabels: { display: true, anchor: 'top', align: 'top' } } }
    });

    // LISTAGEM RELATÓRIO
    let html = `<h2>Relatório Detalhado: ${ult.setor}</h2><p>Data: ${ult.data} | Auditor: ${ult.auditor}</p>
                <table style="width:100%; border-collapse: collapse; margin-top:20px;">
                <tr style="background:#eee;"><th style="padding:10px; border:1px solid #ddd;">SENSO</th><th style="padding:10px; border:1px solid #ddd;">NOTA</th><th style="padding:10px; border:1px solid #ddd;">DETALHES</th></tr>`;
    
    ult.respostas.forEach((r, i) => {
        html += `<tr>
            <td style="padding:10px; border:1px solid #ddd;"><b>${checklist[i].s}</b></td>
            <td style="padding:10px; border:1px solid #ddd; text-align:center;">${r.media}</td>
            <td style="padding:10px; border:1px solid #ddd;">
                <i>Evidência:</i> ${r.evid || 'Sem observações.'}
                ${r.plano ? `<div class="plano-acao-relatorio"><b>PLANO DE AÇÃO:</b> ${r.plano}</div>` : ''}
            </td>
        </tr>`;
    });
    html += `</table>`;
    document.getElementById('relatorio_detalhado').innerHTML = html;
}

function show(id) {
    document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
}
</script>
</body>
</html>

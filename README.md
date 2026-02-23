<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - v1.2025 Enterprise</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>

<style>
:root { --primary: #f06639; --secondary: #5d5a51; --green: #28a745; --blue: #007bff; --danger: #dc3545; }
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}

/* Plano de Fundo Metalurgia - IDENTICO AO TEU */
body { 
    margin:0; min-height: 100vh; background-color: #f0f2f5; background-size: cover;
    background-position: center; background-attachment: fixed; transition: background-image 1.5s ease-in-out;
    position: relative;
}
body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.88); z-index: -1; }

header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }
.screen { display:none; padding:20px; max-width: 1150px; margin: 10px auto; }
.active { display:block; animation: fadeIn 0.4s; }

/* Cards do teu Layout Original */
.card { background: white; padding: 25px; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 15px; border-left: 5px solid var(--primary); box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.pergunta-item { margin-bottom: 18px; padding-bottom: 10px; border-bottom: 1px solid #eee; }
.pergunta-item label { font-weight: bold; color: var(--secondary); display: block; margin-bottom: 5px; }
.pergunta-desc { font-size: 13px; color: #666; font-style: italic; display: block; margin-bottom: 8px; line-height: 1.4; }

.kpi-container { display: flex; gap: 20px; margin-bottom: 20px; }
.kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; text-align: center; border-bottom: 5px solid var(--primary); }
.kpi-card h2 { margin: 0; font-size: 36px; color: var(--primary); }

button { border: none; padding: 12px 25px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.3s; }
.btn-primary { background: var(--primary); width: 100%; }
.btn-primary:hover { background: #d4562d; }
.btn-blue { background: var(--blue); width: 100%; margin-bottom: 10px; }

select, input, textarea { width: 100%; padding: 12px; border-radius: 4px; border: 1px solid #ccc; margin-bottom: 12px; font-size: 15px; }
.obrigatorio { border: 2px solid var(--danger) !important; background: #fff1f1; }

/* Tabela igual ao teu PDF */
.tabela-pdf { width: 100%; border-collapse: collapse; margin-top: 20px; background: white; }
.tabela-pdf th, .tabela-pdf td { border: 1px solid #999; padding: 12px; text-align: left; font-size: 13px; }
.tabela-pdf th { background: #f4f4f4; color: var(--secondary); }

@media print { 
    .no-print { display:none !important; } 
    .screen { display:block !important; }
    body::before { display:none; }
    .card { page-break-inside: avoid; border: 1px solid #ccc; box-shadow: none; }
}
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body id="bgBody">

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active">
    <div class="card">
        <h3>Identificação da Auditoria</h3>
        <label>Setor:</label>
        <select id="setor">
            <option value="">Selecione o Setor...</option>
            <option value="ADM FILIAL">ADM FILIAL</option>
            <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
            <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="ENGENHARIA SOLVE">ENGENHARIA SOLVE</option>
            <option value="INJEÇÃO">INJEÇÃO</option>
            <option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MANUTENÇÃO">MANUTENÇÃO</option>
            <option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option>
            <option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option>
            <option value="SALA DE ELETRONICOS">SALA DE ELETRONICOS</option>
            <option value="SALA DO IMPETUS">SALA DO IMPETUS</option>
        </select>
        <label>Auditor:</label><input type="text" id="auditor" placeholder="Seu nome">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-primary" onclick="startAudit()">INICIAR AVALIAÇÃO</button>
        <button class="btn-blue" style="margin-top:12px" onclick="syncCloud()">🔄 SINCRONIZAR DADOS</button>
        <button class="btn" style="background:var(--secondary); width:100%" onclick="openDash()">VER DASHBOARD</button>
    </div>
</div>

<div id="audit-screen" class="screen">
    <div id="senso-content"></div>
</div>

<div id="dash-screen" class="screen">
    <div class="no-print" style="margin-bottom: 20px;">
        <button class="btn-primary" onclick="location.reload()" style="width: auto;">NOVA AUDITORIA</button>
        <button style="background:var(--green); width: auto;" onclick="window.print()">SALVAR PDF</button>
    </div>

    <div class="kpi-container">
        <div class="kpi-card"><h2 id="media-geral">0.0</h2><p>MÉDIA GERAL DA FÁBRICA</p></div>
        <div class="kpi-card"><h2 id="total-audits">0</h2><p>TOTAL DE AUDITORIAS</p></div>
    </div>

    <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(450px, 1fr)); gap: 20px;">
        <div class="card"><h3>Setor Selecionado (Radar)</h3><canvas id="cRadar"></canvas></div>
        <div class="card"><h3>Comparativo entre Setores</h3><canvas id="cBarra"></canvas></div>
    </div>

    <div id="detalhes-tabela"></div>
</div>

<script>
const URL_API = "https://script.google.com/macros/s/AKfycbzcntNB7ErwLkye0Y7kAoqneoeMAs_MMe7YvszVdHGrHJUGacpixxAYV9LDppBlUoNx/exec";

// Perguntas Explicativas Reais
const checklist = [
    { 
        s: "SELEÇÃO (UTILIZAÇÃO)", 
        p: [
            {t: "Itens Necessários", d: "Apenas ferramentas e materiais de uso diário estão na área de trabalho?"},
            {t: "Descarte de Obsoletos", d: "Existem materiais, moldes ou caixas sem uso identificados para descarte?"},
            {t: "Organização de Papéis", d: "Ordens de produção e desenhos estão atualizados e sem folhas extras?"},
            {t: "Espaço Livre", d: "As bancadas e o topo das máquinas estão livres de objetos inúteis?"}
        ]
    },
    { 
        s: "ORDENAÇÃO (ARRUMAÇÃO)", 
        p: [
            {t: "Identificação de Locais", d: "Ferramentas e dispositivos possuem local fixo com etiqueta?"},
            {t: "Demarcação de Solo", d: "As áreas de trânsito e paletes estão devidamente pintadas/marcadas?"},
            {t: "Facilidade de Acesso", d: "Consegue-se encontrar qualquer item em menos de 30 segundos?"}
        ]
    },
    { 
        s: "LIMPEZA", 
        p: [
            {t: "Conservação de Máquinas", d: "Há vazamentos de óleo, água ou excesso de poeira nas máquinas?"},
            {t: "Piso e Cantos", d: "O chão está livre de sujidade, inclusive debaixo das estantes?"},
            {t: "Lixeiras", d: "As lixeiras estão limpas, com saco e no local correto?"}
        ]
    },
    { 
        s: "PADRONIZAÇÃO (SAÚDE)", 
        p: [
            {t: "Segurança e EPI", d: "O auditor observa o uso correto de todos os EPIs necessários?"},
            {t: "Quadro de Avisos", d: "As informações de gestão visual estão limpas e atualizadas?"}
        ]
    },
    { 
        s: "DISCIPLINA", 
        p: [
            {t: "Conformidade", d: "Os padrões definidos nas últimas auditorias estão a ser mantidos?"},
            {t: "Melhoria Contínua", d: "A equipa sugere ou executa melhorias sem precisar de ordens?"}
        ]
    }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let currentAudit = {}, currentSenso = 0;
let radarObj, barraObj;

// Fundo Dinâmico Metalurgia
const bgs = ["https://images.unsplash.com/photo-1504917595217-d4dc5ebe6122?w=1600", "https://images.unsplash.com/photo-1581092160562-40aa08e78837?w=1600"];
let bgIdx = 0;
setInterval(() => { 
    document.getElementById('bgBody').style.backgroundImage = `url('${bgs[bgIdx]}')`;
    bgIdx = (bgIdx + 1) % bgs.length;
}, 15000);

function startAudit() {
    const set = document.getElementById('setor').value;
    const aud = document.getElementById('auditor').value;
    if(!set || !aud) return alert("Preencha o Setor e o Auditor!");
    
    currentAudit = { id: Date.now(), setor: set, auditor: aud, data: document.getElementById('data').value, respostas: [] };
    currentSenso = 0;
    renderSenso();
}

function renderSenso() {
    const senso = checklist[currentSenso];
    let html = `<div class="card"><h2>${currentSenso + 1}º Senso: ${senso.s}</h2>`;
    
    senso.p.forEach((item, i) => {
        html += `<div class="pergunta-item">
                    <label>${item.t}</label>
                    <span class="pergunta-desc">${item.d}</span>
                    <select class="nota-sel">
                        <option value="10">10 - Excelente</option>
                        <option value="8">8 - Bom</option>
                        <option value="6">6 - Regular</option>
                        <option value="4">4 - Ruim (Alerta)</option>
                        <option value="2">2 - Crítico (Urgente)</option>
                    </select>
                 </div>`;
    });

    html += `<h3>Evidências e Ações</h3>
             <label>Observações / Evidências:</label>
             <textarea id="evid-text" placeholder="Descreva o que encontrou..."></textarea>
             <label id="label-plano">Plano de Ação (Obrigatório se Nota < 6):</label>
             <textarea id="plano-text" placeholder="O que será feito para corrigir?"></textarea>
             <button class="btn-primary" onclick="nextSenso()">GRAVAR E CONTINUAR</button></div>`;
    
    document.getElementById('senso-content').innerHTML = html;
    window.scrollTo(0,0);
    showScreen('audit-screen');
}

function nextSenso() {
    const notas = Array.from(document.querySelectorAll('.nota-sel')).map(s => Number(s.value));
    const media = (notas.reduce((a,b) => a+b, 0) / notas.length);
    const plano = document.getElementById('plano-text').value.trim();
    const evid = document.getElementById('evid-text').value.trim();

    // Regra: Plano de ação obrigatório abaixo de 6
    if(media < 6 && plano.length < 5) {
        document.getElementById('plano-text').classList.add('obrigatorio');
        return alert("Média do senso é " + media.toFixed(1) + ". O Plano de Ação é OBRIGATÓRIO!");
    }

    currentAudit.respostas.push({ media: media.toFixed(1), evidencia: evid, plano: plano });
    
    currentSenso++;
    if(currentSenso < checklist.length) renderSenso();
    else finishAudit();
}

function finishAudit() {
    db.push(currentAudit);
    localStorage.setItem("j2m_db", JSON.stringify(db));
    
    // Enviar para nuvem
    fetch(URL_API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(currentAudit) });
    
    alert("Auditoria Finalizada!");
    openDash();
}

async function syncCloud() {
    alert("Sincronizando com a Planilha...");
    try {
        const r = await fetch(URL_API);
        const data = await r.json();
        db = data.slice(1).map(row => ({ id: row[10], setor: row[1], auditor: row[3], data: row[9], respostas: JSON.parse(row[11]) }));
        localStorage.setItem("j2m_db", JSON.stringify(db));
        openDash();
    } catch(e) { alert("Erro ao sincronizar dados."); }
}

function openDash() {
    renderCharts();
    showScreen('dash-screen');
}

function renderCharts() {
    if(db.length === 0) return;
    const ult = db[db.length - 1];

    // KPIs
    document.getElementById('total-audits').innerText = db.length;
    let somaGeral = db.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById('media-geral').innerText = (somaGeral / db.length).toFixed(1);

    // Radar
    if(radarObj) radarObj.destroy();
    radarObj = new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Utilização", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets: [{ label: ult.setor, data: ult.respostas.map(r => r.media), backgroundColor: 'rgba(240,102,57,0.2)', borderColor: '#f06639' }]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    // Barra (Comparativo)
    const setores = [...new Set(db.map(d=>d.setor))];
    const mediasSetores = setores.map(s => {
        const d = db.filter(x=>x.setor===s);
        return (d.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0) / d.length).toFixed(1);
    });

    if(barraObj) barraObj.destroy();
    barraObj = new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: { labels: setores, datasets: [{ label: 'Nota Final', data: mediasSetores, backgroundColor: '#5d5a51' }] },
        options: { plugins: { datalabels: { display: true, anchor: 'end', align: 'top' } } }
    });

    // Tabela PDF
    let html = `<h3>Detalhamento da Auditoria - ${ult.setor}</h3><table class="tabela-pdf">
                <tr><th>SENSO</th><th>NOTA</th><th>EVIDÊNCIA / PLANO DE ACÇÃO</th></tr>`;
    ult.respostas.forEach((r, i) => {
        html += `<tr><td>${checklist[i].s}</td><td><b>${r.media}</b></td><td><i>Evidência:</i> ${r.evidencia || 'Sem obs.'}<br><b>Ação:</b> ${r.plano || 'N/A'}</td></tr>`;
    });
    html += "</table>";
    document.getElementById('detalhes-tabela').innerHTML = html;
}

function showScreen(id) {
    document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
}
</script>
</body>
</html>

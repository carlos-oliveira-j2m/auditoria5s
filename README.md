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

body { 
    margin:0; min-height: 100vh; background-color: #f0f2f5; background-size: cover;
    background-position: center; background-attachment: fixed; transition: background-image 1.5s ease-in-out;
    position: relative;
}
body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.9); z-index: -1; }

header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }
.screen { display:none; padding:20px; max-width: 1150px; margin: 10px auto; }
.active { display:block; animation: fadeIn 0.4s; }

.card { background: white; padding: 20px; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 15px; border-left: 5px solid var(--primary); box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
.pergunta-item { margin-bottom: 15px; padding-bottom: 10px; border-bottom: 1px dashed #ccc; }
.pergunta-item small { color: #666; display: block; margin-bottom: 5px; font-style: italic; }

.kpi-container { display: flex; gap: 20px; margin-bottom: 20px; }
.kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; text-align: center; border-bottom: 5px solid var(--primary); }

button { border: none; padding: 12px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; }
.btn-primary { background: var(--primary); width: 100%; }
.btn-blue { background: var(--blue); width: 100%; margin-bottom: 10px; }

select, input, textarea { width: 100%; padding: 10px; border-radius: 4px; border: 1px solid #ccc; margin-bottom: 10px; }
.obrigatorio { border: 2px solid var(--danger) !important; background: #fff1f1; }

.tabela-pdf { width: 100%; border-collapse: collapse; margin-top: 10px; background: white; }
.tabela-pdf th, .tabela-pdf td { border: 1px solid #999; padding: 8px; text-align: left; font-size: 12px; }

@media print { 
    .no-print { display:none !important; } 
    .screen { display:block !important; }
    body::before { display:none; }
    .card { page-break-inside: avoid; border: 1px solid #ccc; }
}
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body id="bgBody">

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active">
    <div class="card">
        <h3>Nova Auditoria</h3>
        <label>Setor:</label><input type="text" id="setor" placeholder="Ex: Matrizaria">
        <label>Auditor:</label><input type="text" id="auditor">
        <label>Data:</label><input type="date" id="data">
        <button class="btn-primary" onclick="start()">Iniciar Avaliação</button>
        <button class="btn-blue" style="margin-top:10px" onclick="puxarDados()">Sincronizar Celular / PC</button>
        <button class="btn" style="background:var(--secondary); width:100%" onclick="openDashboard()">Dashboard</button>
    </div>
</div>

<div id="senso" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="no-print">
        <button class="btn-primary" onclick="location.reload()">Novo Filtro / Voltar</button>
        <button style="background:var(--green); margin-top:10px" class="btn-primary" onclick="window.print()">Gerar Relatório PDF</button>
    </div>

    <div class="kpi-container">
        <div class="kpi-card"><h2 id="mediaGeralFabrica">0.0</h2><p>Média Geral</p></div>
        <div class="kpi-card"><h2 id="totalAuditorias">0</h2><p>Auditorias</p></div>
    </div>

    <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); gap: 20px;">
        <div class="card"><h3>Desempenho por Senso</h3><canvas id="cRadar"></canvas></div>
        <div class="card"><h3>Comparativo entre Setores</h3><canvas id="cBarra"></canvas></div>
    </div>

    <div id="relatorioView"></div>
</div>

<script>
const API_GOOGLE = "https://script.google.com/macros/s/AKfycbzcntNB7ErwLkye0Y7kAoqneoeMAs_MMe7YvszVdHGrHJUGacpixxAYV9LDppBlUoNx/exec";

const checklist = [
    { 
        s: "Utilização (Seleção)", 
        p: [
            {t: "Ferramentas e Equipamentos", d: "Verificar se apenas o necessário para a operação está na bancada/máquina."},
            {t: "Materiais Obsoletos", d: "Avaliar se há peças, caixas ou insumos sem uso há mais de 30 dias."},
            {t: "Documentação", d: "Documentos e ordens de produção estão organizados e são atuais?"},
            {t: "Sucata e Lixo", d: "Existem descartes acumulados fora dos locais previstos?"}
        ]
    },
    { 
        s: "Arrumação (Ordenação)", 
        p: [
            {t: "Identificação", d: "Tudo tem lugar definido e está devidamente identificado?"},
            {t: "Demarcação de Piso", d: "As faixas de pedestres e áreas de estoque estão visíveis e respeitadas?"},
            {t: "Acesso Rápido", d: "As ferramentas de uso constante estão ao alcance fácil (30 segundos)?"}
        ]
    },
    { 
        s: "Limpeza", 
        p: [
            {t: "Fontes de Sujidade", d: "Máquinas apresentam vazamentos de óleo ou acúmulo de cavacos?"},
            {t: "Conservação", d: "Pintura, lâmpadas e vidros estão limpos e conservados?"},
            {t: "Higiene Coletiva", d: "As áreas de uso comum do setor estão impecáveis?"}
        ]
    },
    { 
        s: "Padronização (Saúde)", 
        p: [
            {t: "Gestão Visual", d: "Os quadros de aviso estão atualizados e padronizados?"},
            {t: "Segurança e EPI", d: "Todos utilizam os EPIs e os dispositivos de segurança estão ativos?"}
        ]
    },
    { 
        s: "Disciplina", 
        p: [
            {t: "Rotina 5S", d: "A equipe demonstra conhecimento e prática dos conceitos sem supervisão?"},
            {t: "Melhoria Contínua", d: "Planos de ação de auditorias passadas foram executados?"}
        ]
    }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, sensoIndex = 0;

function start() {
    if(!document.getElementById('setor').value) return alert("Informe o Setor");
    audit = { 
        id: Date.now(), 
        setor: document.getElementById('setor').value.toUpperCase(), 
        auditor: document.getElementById('auditor').value, 
        data: document.getElementById('data').value, 
        respostas: [] 
    };
    sensoIndex = 0; renderSenso();
}

function renderSenso() {
    const s = checklist[sensoIndex];
    let h = `<div class="card"><h2>${sensoIndex+1}º Senso: ${s.s}</h2>`;
    
    s.p.forEach((item, i) => {
        h += `<div class="pergunta-item">
                <strong>${item.t}</strong>
                <small>${item.d}</small>
                <select class="nota-q">
                    <option value="10">10 - Excelente</option>
                    <option value="8">8 - Bom</option>
                    <option value="6">6 - Regular</option>
                    <option value="4">4 - Ruim (Ação Obrigatória)</option>
                    <option value="2">2 - Crítico (Ação Obrigatória)</option>
                </select>
              </div>`;
    });

    h += `<h3>Evidências e Plano de Ação</h3>
          <label>Descrição das Evidências (O que viu?):</label>
          <textarea id="txtEvidencia" placeholder="Ex: Encontrado vazamento de óleo na prensa 02..."></textarea>
          
          <label id="lblPlano">Plano de Ação (Obrigatório se nota < 6):</label>
          <textarea id="txtPlano" placeholder="O que será feito para corrigir?"></textarea>
          
          <button class="btn-primary" onclick="salvarSenso()">GRAVAR SENSO</button></div>`;
    
    document.getElementById('senso').innerHTML = h;
    window.scrollTo(0,0);
    show('senso');
}

function salvarSenso() {
    const notas = Array.from(document.querySelectorAll('.nota-q')).map(sel => Number(sel.value));
    const media = notas.reduce((a,b)=>a+b,0)/notas.length;
    const plano = document.getElementById('txtPlano').value.trim();
    const evidencia = document.getElementById('txtEvidencia').value.trim();

    // VALIDAÇÃO: Plano de ação obrigatório para notas baixas
    if(media < 6 && plano.length < 5) {
        document.getElementById('txtPlano').classList.add('obrigatorio');
        alert("ALERTA: Para notas abaixo de 6.0, o Plano de Ação é obrigatório!");
        return;
    }

    audit.respostas[sensoIndex] = { media: media.toFixed(1), evidencia: evidencia, plano: plano };
    
    sensoIndex++;
    if(sensoIndex < checklist.length) renderSenso();
    else finalizar();
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("j2m_db", JSON.stringify(db));
    fetch(API_GOOGLE, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
    alert("Auditoria Finalizada com Sucesso!");
    openDashboard();
}

async function puxarDados() {
    alert("Sincronizando...");
    try {
        const res = await fetch(API_GOOGLE);
        const data = await res.json();
        db = data.slice(1).map(row => ({ id: row[10], setor: row[1], auditor: row[3], data: row[9], respostas: JSON.parse(row[11]) }));
        localStorage.setItem("j2m_db", JSON.stringify(db));
        alert("Sincronizado!"); openDashboard();
    } catch(e) { alert("Erro na rede."); }
}

function openDashboard() {
    renderRelatorio();
    show("dashboard");
}

function renderRelatorio() {
    if(db.length === 0) return;
    const ult = db[db.length - 1];
    
    // KPIs
    document.getElementById("totalAuditorias").innerText = db.length;
    let somaT = db.reduce((acc, a) => acc + (a.respostas.reduce((s,r)=>s+Number(r.media),0)/5), 0);
    document.getElementById("mediaGeralFabrica").innerText = (somaT / db.length).toFixed(1);

    // Radar
    new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Utilização", "Arrumação", "Limpeza", "Saúde", "Auto-Disciplina"],
            datasets: [{ label: ult.setor, data: ult.respostas.map(r => r.media), backgroundColor: 'rgba(240,102,57,0.2)', borderColor: '#f06639' }]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    // Comparativo Setores (Barra)
    const setores = [...new Set(db.map(d=>d.setor))];
    const notasSetores = setores.map(s => {
        const d = db.filter(x=>x.setor===s);
        return (d.reduce((acc, a) => acc + (a.respostas.reduce((x,y)=>x+Number(y.media),0)/5), 0) / d.length).toFixed(1);
    });

    new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: { labels: setores, datasets: [{ label: 'Média Final', data: notasSetores, backgroundColor: '#5d5a51' }] },
        options: { plugins: { datalabels: { display: true, anchor: 'end', align: 'top' } } }
    });

    // Tabela Detalhada com Evidências
    let h = `<h3>Detalhamento da Última Auditoria: ${ult.setor}</h3><table class="tabela-pdf">
             <tr><th>SENSO</th><th>NOTA</th><th>EVIDÊNCIAS / PLANO DE AÇÃO</th></tr>`;
    ult.respostas.forEach((r, i) => {
        h += `<tr>
                <td>${checklist[i].s}</td>
                <td><b>${r.media}</b></td>
                <td><i>Evidência:</i> ${r.evidencia || 'N/A'}<br><b>Ação:</b> ${r.plano || 'N/A'}</td>
              </tr>`;
    });
    h += "</table>";
    document.getElementById("relatorioView").innerHTML = h;
}

function show(id) { 
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active")); 
    document.getElementById(id).classList.add("active"); 
}
</script>
</body>
</html>

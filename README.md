<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Auditoria 5S J2M - Fusion Enterprise</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>

    <style>
        :root { --primary: #f06639; --secondary: #3d3b35; --green: #28a745; --blue: #007bff; --light: #f8f9fa; }
        * { box-sizing: border-box; font-family: 'Segoe UI', Arial, sans-serif; }

        body { 
            margin: 0; min-height: 100vh; background-color: #f0f2f5; 
            background-size: cover; background-position: center; background-attachment: fixed;
            transition: background-image 1.5s ease-in-out; position: relative;
        }

        body::before {
            content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(255, 255, 255, 0.88); z-index: -1;
        }

        header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }

        .screen { display: none; padding: 20px; max-width: 1150px; margin: 10px auto; }
        .active { display: block; animation: fadeIn 0.4s; }

        .filter-bar { display: flex; gap: 10px; background: white; padding: 15px; border-radius: 8px; margin-bottom: 20px; flex-wrap: wrap; border: 1px solid #ddd; box-shadow: 0 4px 6px rgba(0,0,0,0.05); }
        .filter-item { flex: 1; min-width: 140px; }
        .filter-item label { font-size: 11px; font-weight: bold; display: block; margin-bottom: 4px; color: var(--secondary); }

        .kpi-container { display: flex; gap: 20px; margin-bottom: 20px; flex-wrap: wrap; }
        .kpi-card { background: var(--secondary); color: white; padding: 20px; border-radius: 8px; flex: 1; min-width: 200px; text-align: center; border-bottom: 5px solid var(--primary); }
        .kpi-card h2 { margin: 0; font-size: 32px; color: var(--primary); }

        .card { background: rgba(255,255,255,0.95); padding: 20px; border: 1px solid #ddd; border-radius: 8px; margin-bottom: 15px; border-left: 6px solid var(--primary); box-shadow: 0 2px 5px rgba(0,0,0,0.05); }
        .pergunta-texto { font-weight: bold; color: #222; display: block; margin-bottom: 4px; font-size: 16px; }
        .pergunta-ajuda { font-size: 13px; color: #666; font-style: italic; display: block; margin-bottom: 12px; background: #f1f1f1; padding: 8px; border-radius: 4px; }

        button { border: none; padding: 10px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.3s; }
        .btn-primary { background: var(--primary); }
        .btn-secondary { background: var(--secondary); }
        .btn-danger { background: #dc3545; font-size: 10px; }
        .btn-edit { background: #ffc107; color: #000; font-size: 10px; margin-right: 5px; }

        select, input, textarea { width: 100%; padding: 12px; border-radius: 4px; border: 1px solid #ccc; margin-top: 5px; }
        textarea { min-height: 80px; }

        .tabela-pdf { width: 100%; border-collapse: collapse; margin-top: 10px; background: white; }
        .tabela-pdf th, .tabela-pdf td { border: 1px solid #999; padding: 10px; text-align: left; font-size: 13px; }
        .tabela-pdf th { background: #eee; }

        @media print { .no-print { display:none !important; } body::before { background: white; } }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body id="bgBody">

<header>AUDITORIA 5S J2M - v1.2025 FUSION</header>

<div id="home" class="screen active">
    <div class="card">
        <h3 id="tituloHome">Identificação da Auditoria</h3>
        <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px;">
            <div><label>Setor:</label><input type="text" id="setor" placeholder="Ex: Almoxarifado"></div>
            <div><label>Auditor:</label><input type="text" id="auditor"></div>
            <div><label>Responsável:</label><input type="text" id="responsavel"></div>
            <div><label>Data:</label><input type="date" id="data"></div>
        </div><br>
        <button class="btn-primary" onclick="start()">Iniciar Avaliação</button>
        <button class="btn-secondary" onclick="openDashboard()">Dashboard / Histórico</button>
    </div>
</div>

<div id="senso" class="screen"></div>

<div id="dashboard" class="screen">
    <div class="no-print">
        <div class="filter-bar">
            <div class="filter-item"><label>SETOR</label><select id="fSetor" onchange="renderRelatorio()"></select></div>
            <div class="filter-item"><label>MÊS</label>
                <select id="fMes" onchange="renderRelatorio()">
                    <option value="GERAL">Todos</option>
                    <option value="0">Janeiro</option><option value="1">Fevereiro</option><option value="2">Março</option>
                    <option value="3">Abril</option><option value="4">Maio</option><option value="5">Junho</option>
                    <option value="6">Julho</option><option value="7">Agosto</option><option value="8">Setembro</option>
                    <option value="9">Outubro</option><option value="10">Novembro</option><option value="11">Dezembro</option>
                </select>
            </div>
            <div class="filter-item"><label>ANO</label>
                <select id="fAno" onchange="renderRelatorio()">
                    <option value="2025">2025</option><option value="2026">2026</option>
                </select>
            </div>
            <div style="display:flex; align-items:flex-end; gap:5px;">
                <button class="btn-secondary" onclick="resetForm()">Novo</button>
                <button style="background:#10b981" onclick="window.print()">Salvar PDF</button>
            </div>
        </div>
    </div>

    <div class="kpi-container">
        <div class="kpi-card"><h2 id="mediaGeralFabrica">0.0</h2><p>Média Geral Selecionada</p></div>
        <div class="kpi-card"><h2 id="totalAuditorias">0</h2><p>Auditorias no Filtro</p></div>
    </div>

    <div id="areaGraficos" style="display: grid; grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); gap: 20px; margin-bottom: 40px;">
        <div style="background:white; padding:10px; border-radius:8px; border:1px solid #ddd;"><canvas id="cRadar"></canvas></div>
        <div style="background:white; padding:10px; border-radius:8px; border:1px solid #ddd;"><canvas id="cBarra"></canvas></div>
    </div>

    <div id="relatorioView"></div>
</div>

<script>
Chart.register(ChartDataLabels);

// --- BANCO DE DADOS DE PERGUNTAS (V1 Completa) ---
const bancoPerguntas = {
    "1 - Seleção": [
        {p: "Todas as ferramentas e equipamentos são necessários para o trabalho?", d: "Diretriz: Observar itens inúteis, quebrados ou sem uso no posto de trabalho."},
        {p: "Existem itens duplicados sobre a bancada?", d: "Diretriz: Verificar excesso de ferramentas idênticas, estiletes ou material de escritório."},
        {p: "Ferramentas acondicionadas corretamente?", d: "Diretriz: Verificar se há um lugar definido para cada item e se estão nos suportes."},
        {p: "Quadros de gestão e documentos atualizados?", d: "Diretriz: Checklists e normas devem estar na versão vigente e preenchidos."},
        {p: "Os avisos e quadros informativos são necessários?", d: "Diretriz: Remover papéis antigos ou informações obsoletas."}
    ],
    "2 - Ordenação": [
        {p: "Os locais para paletes e caixas estão marcados?", d: "Diretriz: Verificar demarcações de solo para fluxo, estoque e áreas de segurança."},
        {p: "As linhas e marcações estão claramente visíveis?", d: "Diretriz: A pintura ou fita de demarcação deve estar conservada e nítida."},
        {p: "Prateleiras e locais de armazenamento estão identificados?", d: "Diretriz: Placas ou etiquetas indicando o que deve ser guardado."},
        {p: "As gavetas e armários estão identificados?", d: "Diretriz: Identificação externa facilitando a localização."},
        {p: "Ferramentas ordenadas por frequência de uso?", d: "Diretriz: Itens de uso constante devem estar mais próximos."}
    ],
    "3 - Limpeza": [
        {p: "O chão está limpo e livre de detritos?", d: "Diretriz: Ausência de poeira, óleo, papéis ou restos de material."},
        {p: "As máquinas e equipamentos estão conservados?", d: "Diretriz: Limpeza técnica do equipamento e bom estado geral."},
        {p: "As fontes de sujeira foram identificadas e tratadas?", d: "Diretriz: Verificar se há vazamentos de óleo, água ou ar comprimido."},
        {p: "Materiais de limpeza estão disponíveis?", d: "Diretriz: Vassouras e rodos em suportes próprios e em bom estado."},
        {p: "As lixeiras estão identificadas e limpas?", d: "Diretriz: Coleta seletiva respeitada e lixeiras sem transbordar."}
    ],
    "4 - Padronização": [
        {p: "Funcionários usam uniformes e EPIs corretamente?", d: "Diretriz: Uso integral de uniforme e equipamentos de segurança."},
        {p: "Placas de segurança e extintores em bom estado?", d: "Diretriz: Sinalização visível e extintores com acesso livre."},
        {p: "A iluminação e ventilação estão adequadas?", d: "Diretriz: Lâmpadas funcionando e boa circulação de ar."},
        {p: "Painéis elétricos estão fechados e identificados?", d: "Diretriz: Portas travadas e com adesivos de advertência."},
        {p: "Ambientes de uso comum estão organizados?", d: "Diretriz: Banheiros e copas seguem o padrão do setor."}
    ],
    "5 - Autodisciplina": [
        {p: "A gestão mantém os padrões?", d: "Diretriz: A liderança demonstra comprometimento e cobra o 5S."},
        {p: "Checklist de autoavaliação é realizado?", d: "Diretriz: A equipe realiza as próprias inspeções de rotina."},
        {p: "Missão, visão e valores são conhecidos?", d: "Diretriz: A equipe entende os objetivos da organização."},
        {p: "EPI's são usados como hábito?", d: "Diretriz: A segurança é praticada sem necessidade de cobrança."},
        {p: "Ações da Auditoria Anterior foram atendidas?", d: "Diretriz: Verificar se as falhas passadas foram corrigidas."}
    ]
};

const sensos = Object.keys(bancoPerguntas);
let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, sensoIndex = 0, editandoId = null;
let chartRadar, chartBar;

// --- LÓGICA DE FUNDO DINÂMICO ---
const imagensFundo = [
    "https://images.unsplash.com/photo-1504917595217-d4dc5ebe6122?auto=format&fit=crop&q=80&w=1920",
    "https://images.unsplash.com/photo-1581092160562-40aa08e78837?auto=format&fit=crop&q=80&w=1920",
    "https://images.unsplash.com/photo-1565608438257-fac3c27beb36?auto=format&fit=crop&q=80&w=1920"
];
let imgIdx = 0;
function mudarFundo() {
    document.getElementById('bgBody').style.backgroundImage = `url('${imagensFundo[imgIdx]}')`;
    imgIdx = (imgIdx + 1) % imagensFundo.length;
}
setInterval(mudarFundo, 12000); mudarFundo();

// --- FUNÇÕES DE NAVEGAÇÃO ---
function show(id) { document.querySelectorAll(".screen").forEach(s => s.classList.remove("active")); document.getElementById(id).classList.add("active"); }

function resetForm() {
    editandoId = null;
    document.getElementById('tituloHome').innerText = "Identificação da Auditoria";
    document.getElementById('setor').value = "";
    show('home');
}

function start() {
    if(!document.getElementById('setor').value || !document.getElementById('auditor').value) return alert("Preencha Setor e Auditor!");
    if(!editandoId) {
        audit = { 
            id: Date.now(), 
            setor: document.getElementById('setor').value, 
            auditor: document.getElementById('auditor').value,
            responsavel: document.getElementById('responsavel').value,
            data: document.getElementById('data').value || new Date().toISOString().split('T')[0],
            respostas: [] 
        };
    }
    sensoIndex = 0; renderSenso();
}

function renderSenso() {
    const nome = sensos[sensoIndex];
    let html = `<h2 style="color:var(--primary)">${nome}</h2>`;
    const progresso = audit.respostas[sensoIndex];

    bancoPerguntas[nome].forEach((item, i) => {
        const notaS = progresso ? progresso.detalhes[i].nota : "10";
        const obsS = progresso ? progresso.detalhes[i].obs : "";
        html += `<div class="card">
            <span class="pergunta-texto">${i+1}. ${item.p}</span>
            <span class="pergunta-ajuda">${item.d}</span>
            <select id="q${i}">
                <option value="10" ${notaS==10?'selected':''}>10 - Excelente - Sem evidências</option>
                <option value="8" ${notaS==8?'selected':''}>8 - Bom</option>
                <option value="6" ${notaS==6?'selected':''}>6 - Regular</option>
                <option value="4" ${notaS==4?'selected':''}>4 - Ruim</option>
                <option value="2" ${notaS==2?'selected':''}>2 - Crítico</option>
            </select>
            <textarea id="obs${i}" placeholder="Descreva a evidência ou observação...">${obsS}</textarea>
        </div>`;
    });

    html += `<div class="card"><label><b>PLANO DE AÇÃO DO SENSO:</b></label><textarea id="plan">${progresso?.plano || ""}</textarea></div>
    <div style="display:flex; gap:10px;">
        ${sensoIndex > 0 ? `<button class="btn-secondary" onclick="sensoIndex--; renderSenso()">Anterior</button>` : ''}
        <button class="btn-primary" onclick="salvarSenso()">${sensoIndex < 4 ? 'Próximo Senso' : 'Finalizar Auditoria'}</button>
    </div>`;
    
    const container = document.getElementById('senso');
    container.innerHTML = html; show("senso"); window.scrollTo(0,0);
}

function salvarSenso() {
    let detalhes = []; let soma = 0;
    for(let i=0; i<5; i++) {
        const n = Number(document.getElementById("q"+i).value);
        soma += n;
        detalhes.push({ pergunta: bancoPerguntas[sensos[sensoIndex]][i].p, nota: n, obs: document.getElementById("obs"+i).value });
    }
    audit.respostas[sensoIndex] = { senso: sensos[sensoIndex], media: (soma/5).toFixed(1), detalhes, plano: document.getElementById("plan").value };
    
    if(sensoIndex < 4) { sensoIndex++; renderSenso(); } 
    else {
        let dbLocal = JSON.parse(localStorage.getItem("j2m_db") || "[]");
        if(editandoId) { dbLocal[dbLocal.findIndex(a => a.id === editandoId)] = audit; } 
        else { dbLocal.push(audit); }
        localStorage.setItem("j2m_db", JSON.stringify(dbLocal));
        openDashboard();
    }
}

// --- DASHBOARD E RELATÓRIOS ---
function openDashboard() {
    db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
    const sel = document.getElementById("fSetor");
    const atual = sel.value || "GERAL";
    sel.innerHTML = '<option value="GERAL">Visão Geral (Fábrica)</option>';
    [...new Set(db.map(a => a.setor))].sort().forEach(s => sel.innerHTML += `<option value="${s}">${s}</option>`);
    sel.value = atual;
    renderRelatorio(); show("dashboard");
}

function renderRelatorio() {
    const fS = document.getElementById("fSetor").value;
    const fM = document.getElementById("fMes").value;
    const fA = document.getElementById("fAno").value;

    const filtrados = db.filter(a => {
        const d = new Date(a.data + "T00:00:00");
        return (fS === "GERAL" || a.setor === fS) && (fM === "GERAL" || d.getMonth() == fM) && (fA === "GERAL" || d.getFullYear() == fA);
    });

    let somaMedias = 0;
    filtrados.forEach(a => somaMedias += a.respostas.reduce((acc, r) => acc + Number(r.media), 0) / 5);
    document.getElementById("mediaGeralFabrica").innerText = filtrados.length > 0 ? (somaMedias / filtrados.length).toFixed(1) : "0.0";
    document.getElementById("totalAuditorias").innerText = filtrados.length;

    let html = "";
    if(fS === "GERAL") {
        html = "<h3>Histórico de Auditorias</h3><table class='tabela-pdf'><tr><th>Data</th><th>Setor</th><th>Nota</th><th class='no-print'>Ações</th></tr>";
        filtrados.slice().reverse().forEach(a => {
            const notaF = (a.respostas.reduce((acc,r)=>acc+Number(r.media),0)/5).toFixed(1);
            html += `<tr><td>${a.data}</td><td>${a.setor}</td><td><b>${notaF}</b></td>
            <td class='no-print'>
                <button class='btn-edit' onclick='prepararEdicao(${a.id})'>Editar</button>
                <button class='btn-danger' onclick='excluirAuditoria(${a.id})'>Excluir</button>
            </td></tr>`;
        });
        html += "</table>";
    } else {
        filtrados.forEach(a => {
            const notaF = (a.respostas.reduce((acc,r)=>acc+Number(r.media),0)/5).toFixed(1);
            html += `<div style="border-bottom: 4px solid var(--primary); margin-top:40px;">
                <h1 style="color:var(--primary); margin:0;">RELATÓRIO: ${a.setor}</h1>
                <p><b>AUDITOR:</b> ${a.auditor} | <b>DATA:</b> ${a.data} | <b>NOTA FINAL:</b> ${notaF}</p>
            </div>`;
            a.respostas.forEach(r => {
                html += `<h4 style="background:var(--secondary); color:white; padding:8px; margin:15px 0 0 0;">${r.senso} (Média: ${r.media})</h4>
                <table class="tabela-pdf"><tr><th style="width:75%">Critério</th><th style="width:25%">Nota</th></tr>
                ${r.detalhes.map(d=> `<tr><td>${d.pergunta}<br><small style="color:#666">${d.obs || ''}</small></td><td><b>${d.nota}</b></td></tr>`).join('')}
                </table><div style="background:#f9f9f9; padding:10px; border:1px solid #ddd; font-size:13px;"><b>PLANO DE AÇÃO:</b> ${r.plano || 'N/A'}</div>`;
            });
        });
    }
    document.getElementById("relatorioView").innerHTML = html;
    atualizarGraficos(filtrados);
}

function atualizarGraficos(filtrados) {
    if(chartRadar) chartRadar.destroy(); if(chartBar) chartBar.destroy();
    if(filtrados.length === 0) return;

    const mFiltro = [0,1,2,3,4].map(i => (filtrados.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / filtrados.length).toFixed(1));
    const mGeral = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + Number(a.respostas[i].media), 0) / db.length).toFixed(1));

    chartRadar = new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets: [
                { label: 'Filtro Atual', data: mFiltro, backgroundColor: 'rgba(240,102,57,0.2)', borderColor: '#f06639', pointRadius: 4 },
                { label: 'Média Fábrica', data: mGeral, borderColor: '#007bff', borderDash: [5,5], fill: false },
                { label: 'Meta (8.0)', data: [8,8,8,8,8], borderColor: '#28a745', borderDash: [2,2], fill: false, pointRadius: 0 }
            ]
        },
        options: { scales: { r: { min: 0, max: 10 } } }
    });

    const setores = [...new Set(filtrados.map(d=>d.setor))];
    const notas = setores.map(s => {
        const d = filtrados.filter(x=>x.setor===s);
        return (d.reduce((acc, a) => acc + (a.respostas.reduce((x,y)=>x+Number(y.media),0)/5), 0) / d.length).toFixed(1);
    });

    chartBar = new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: { labels: setores, datasets: [{ label: 'Nota Final por Setor', data: notas, backgroundColor: '#3d3b35' }] },
        options: { scales: { y: { min: 0, max: 10 } }, plugins: { datalabels: { display: true, anchor: 'end', align: 'top' } } }
    });
}

function prepararEdicao(id) {
    const a = db.find(x => x.id === id);
    if(a) {
        editandoId = id; audit = JSON.parse(JSON.stringify(a));
        document.getElementById('setor').value = a.setor;
        document.getElementById('auditor').value = a.auditor;
        document.getElementById('responsavel').value = a.responsavel;
        document.getElementById('data').value = a.data;
        show('home');
        document.getElementById('tituloHome').innerText = "Editando Auditoria";
    }
}

function excluirAuditoria(id) {
    if(confirm("Excluir permanentemente?")) {
        let dbLocal = JSON.parse(localStorage.getItem("j2m_db") || "[]").filter(a => a.id !== id);
        localStorage.setItem("j2m_db", JSON.stringify(dbLocal));
        openDashboard();
    }
}
</script>
</body>
</html>

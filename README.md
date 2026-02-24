<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>J2M 5S Enterprise - v1.2025</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --p: #f06639; --s: #3d3d3d; --bg: #f4f7f6; --err: #dc3545; }
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { margin: 0; background: var(--bg); font-size: 14px; }
        header { background: var(--s); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--p); font-weight: bold; }
        .container { max-width: 900px; margin: auto; padding: 15px; }
        .screen { display: none; }
        .active { display: block; }
        .card { background: white; padding: 15px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 15px; border-left: 5px solid var(--p); }
        label { display: block; font-weight: bold; margin: 10px 0 5px; }
        .req::after { content: " *"; color: var(--err); }
        input, select, textarea { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; margin-bottom: 5px; }
        button { cursor: pointer; border: none; padding: 15px; border-radius: 5px; font-weight: bold; width: 100%; margin-bottom: 10px; }
        .btn-p { background: var(--p); color: white; }
        .btn-sync { background: #007bff; color: white; }
        .q-row { border-bottom: 1px solid #eee; padding: 10px 0; }
        .q-text { display: block; margin-bottom: 5px; font-weight: 600; color: #444; }
        .obs-input { font-size: 12px; border-color: #eee; background: #fafafa; }
    </style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div class="container">
    <div id="scr_home" class="screen active">
        <div class="card">
            <h3>Identificação Obrigatória</h3>
            <label class="req">Setor:</label>
            <select id="setor" required>
                <option value="">-- SELECIONE --</option>
                <option value="ADM Filial">ADM Filial</option>
                <option value="ALMOXARIFADO/ESTOQUE">ALMOXARIFADO/ESTOQUE</option>
                <option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
                <option value="INJEÇÃO">INJEÇÃO</option>
                <option value="MATRIZARIA">MATRIZARIA</option>
                <option value="MONTAGEM">MONTAGEM</option>
                <option value="PPCP">PPCP</option>
                <option value="QUALIDADE">QUALIDADE</option>
                <option value="SALA DE ELETRONICOS">SALA DE ELETRONICOS</option>
            </select>
            <label class="req">Responsável:</label><input type="text" id="resp">
            <label class="req">Auditor:</label><input type="text" id="aud">
            <label class="req">Data:</label><input type="date" id="dt">
            
            <button class="btn-p" onclick="iniciar()">INICIAR AVALIAÇÃO</button>
            <button class="btn-sync" onclick="sincronizar()">🔄 SINCRONIZAR AUTOMÁTICO</button>
        </div>
    </div>

    <div id="scr_form" class="screen"><div id="form_content"></div></div>
</div>

<script>
const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwYTDUTtW_tA8PgDQ3L5j6NFKlC2soH722nR_7lk_Kazuus8kMwn4-SLk9pG9ANwZ0/exec"; 

const perguntas = [
    { s: "1- Seleção", q: ["Todas as ferramentas e equipamentos são necessários para o trabalho?","Existem itens duplicados sobre a bancada?","Ferramentas acondicionadas corretamente?","Quadros de gestão e documentos atualizados?","Os avisos e quadros informativos são necessários?"]},
    { s: "2- Ordenação", q: ["Os locais para paletes e caixas estão marcados?","As linhas e marcações estão claramente visíveis?","Prateleiras e locais de armazenamento estão identificados?","As gavetas e armários estão identificados?","Ferramentas ordenadas por frequência de uso?"]},
    { s: "3- Limpeza", q: ["O chão está limpo e livre de detritos?","As máquinas e equipamentos estão conservados?","As fontes de sujeira foram identificadas e tratadas?","Materiais de limpeza estão disponíveis e organizados?","As lixeiras estão identificadas e limpas?"]},
    { s: "4- Padronização", q: ["Funcionários usam uniformes e EPIs corretamente?","Placas de segurança e extintores em bom estado?","A iluminação e ventilação estão adequadas?","Painéis elétricos estão fechados e identificados?","Ambientes de uso comum estão organizados?"]},
    { s: "5- Autodisciplina", q: ["A gestão mantém os padrões?","Checklist de autoavaliação é realizado no setor?","Missão, visão e valores são conhecidos?","EPI's são usados constantemente como hábito?","Existe padrão de limpeza em gestão à vista?","Ações da Auditoria Anterior foram atendidas?"]}
];

let db = JSON.parse(localStorage.getItem("db_j2m_5s") || "[]");
let audit = {};
let etapa = 0;

function iniciar() {
    const s = document.getElementById('setor').value;
    const r = document.getElementById('resp').value;
    const a = document.getElementById('aud').value;
    const d = document.getElementById('dt').value;
    if(!s || !r || !a || !d) return alert("Preencha todos os campos obrigatórios!");
    audit = { setor: s, auditor: a, data: d, responsavel: r, detalhes: [] };
    etapa = 0;
    renderEtapa();
}

function renderEtapa() {
    const senso = perguntas[etapa];
    let html = `<div class="card"><h2>${senso.s}</h2>`;
    senso.q.forEach((q, i) => {
        html += `<div class="q-row">
            <span class="q-text">${q}</span>
            <select id="n_${i}" required>
                <option value="">-- NOTA --</option>
                <option value="10">10 - Excelente</option>
                <option value="6">6 - Parcial</option>
                <option value="2">2 - Crítico</option>
            </select>
            <input type="text" id="obs_${i}" class="obs-input" placeholder="Observação/Evidência (Obrigatório se nota < 10)">
        </div>`;
    });
    html += `<label class="req">Plano de Ação Geral do Senso:</label>
             <textarea id="acao_geral"></textarea>
             <button class="btn-p" onclick="salvarEtapa()">PRÓXIMO</button></div>`;
    
    document.getElementById('form_content').innerHTML = html;
    document.getElementById('scr_home').classList.remove('active');
    document.getElementById('scr_form').classList.add('active');
    window.scrollTo(0,0);
}

function salvarEtapa() {
    const itens = [];
    for(let i=0; i<perguntas[etapa].q.length; i++) {
        const nota = document.getElementById(`n_${i}`).value;
        const obs = document.getElementById(`obs_${i}`).value;
        if(!nota) return alert("Responda todas as notas!");
        if(nota < 10 && !obs) return alert("Descreva a evidência para as notas menores que 10!");
        itens.push({ nota: Number(nota), obs });
    }
    const acao = document.getElementById('acao_geral').value;
    if(!acao) return alert("O Plano de Ação Geral é obrigatório!");

    audit.detalhes[etapa] = { senso: perguntas[etapa].s, itens, acao };
    if(etapa < 4) { etapa++; renderEtapa(); } 
    else { 
        db.push(audit); 
        localStorage.setItem("db_j2m_5s", JSON.stringify(db)); 
        alert("Salvo! Sincronize na tela inicial.");
        location.reload();
    }
}

async function sincronizar() {
    const btn = event.target;
    btn.innerText = "⏳ ENVIANDO...";
    try {
        await fetch(WEB_APP_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(db) });
        alert("✅ Sincronizado!");
    } catch (e) { alert("Erro de rede."); }
    btn.innerText = "🔄 SINCRONIZAR AUTOMÁTICO";
}
</script>
</body>
</html>

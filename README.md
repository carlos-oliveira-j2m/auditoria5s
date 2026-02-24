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
        body { margin: 0; background: var(--bg); font-size: 16px; }
        header { background: var(--s); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--p); font-weight: bold; }
        .container { max-width: 900px; margin: auto; padding: 15px; }
        .screen { display: none; }
        .active { display: block; }
        .card { background: white; padding: 20px; border-radius: 8px; box-shadow: 0 2px 8px rgba(0,0,0,0.1); margin-bottom: 15px; border-left: 5px solid var(--p); }
        label { display: block; font-weight: bold; margin: 10px 0 5px; }
        .req::after { content: " *"; color: var(--err); }
        input, select, textarea { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 5px; font-size: 16px; margin-bottom: 10px; }
        button { cursor: pointer; border: none; padding: 15px; border-radius: 5px; font-weight: bold; width: 100%; margin-bottom: 10px; transition: 0.2s; }
        .btn-p { background: var(--p); color: white; }
        .btn-s { background: var(--s); color: white; }
        .btn-sync { background: #007bff; color: white; }
        .q-row { border-bottom: 1px solid #eee; padding: 15px 0; }
        .q-text { display: block; margin-bottom: 8px; font-weight: 600; color: #444; }
        select.nota-box { border: 2px solid var(--p); background: #fff9f7; }
        @media print { .no-print { display: none !important; } .screen { display: block !important; } }
    </style>
</head>
<body>

<header class="no-print">AUDITORIA 5S J2M - v1.2025</header>

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
            <label class="req">Responsável da Área:</label><input type="text" id="resp" placeholder="Nome do Responsável">
            <label class="req">Auditor:</label><input type="text" id="aud" placeholder="Nome do Auditor">
            <label class="req">Data:</label><input type="date" id="dt">
            
            <button class="btn-p" onclick="iniciar()">INICIAR AVALIAÇÃO</button>
            <button class="btn-s" onclick="alert('Funcionalidade de Relatórios em desenvolvimento local.')">DASHBOARD / RELATÓRIOS</button>
            <button class="btn-sync" onclick="sincronizar()">🔄 SINCRONIZAR (CELULAR/PC)</button>
        </div>
    </div>

    <div id="scr_form" class="screen">
        <div id="form_content"></div>
    </div>
</div>

<script>
const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwYTDUTtW_tA8PgDQ3L5j6NFKlC2soH722nR_7lk_Kazuus8kMwn4-SLk9pG9ANwZ0/exec"; 

const perguntas = [
    { s: "1- Seleção", q: ["Todas as ferramentas e equipamentos são necessários?","Existem itens duplicados sobre a bancada?","Ferramentas acondicionadas corretamente?","Quadros de gestão e documentos atualizados?","Os avisos e quadros informativos são necessários?"]},
    { s: "2- Ordenação", q: ["Os locais para paletes e caixas estão marcados?","As linhas e marcações estão claramente visíveis?","Prateleiras e locais de armazenamento identificados?","As gavetas e armários estão identificados?","Ferramentas ordenadas por frequência de uso?"]},
    { s: "3- Limpeza", q: ["O chão está limpo e livre de detritos?","As máquinas e equipamentos estão conservados?","As fontes de sujeira foram identificadas e tratadas?","Materiais de limpeza estão disponíveis?","As lixeiras estão identificadas e limpas?"]},
    { s: "4- Padronização", q: ["Funcionários usam uniformes e EPIs corretamente?","Placas de segurança e extintores em bom estado?","A iluminação e ventilação estão adequadas?","Painéis elétricos estão fechados e identificados?","Ambientes de uso comum estão organizados?"]},
    { s: "5- Autodisciplina", q: ["A gestão mantém os padrões?","Checklist de autoavaliação é realizado?","Missão, visão e valores são conhecidos?","EPI's são usados constantemente como hábito?","Existe padrão de limpeza em gestão à vista?","Ações da Auditoria Anterior foram atendidas?"]}
];

let db = JSON.parse(localStorage.getItem("db_j2m_5s") || "[]");
let audit = {};
let etapa = 0;

function iniciar() {
    const fields = ['setor', 'resp', 'aud', 'dt'];
    for (let f of fields) if (!document.getElementById(f).value) return alert("Preencha todos os campos obrigatórios!");
    
    audit = { 
        setor: document.getElementById('setor').value, 
        auditor: document.getElementById('aud').value, 
        responsavel: document.getElementById('resp').value,
        data: document.getElementById('dt').value, 
        respostas: [] 
    };
    etapa = 0;
    renderEtapa();
}

function renderEtapa() {
    const senso = perguntas[etapa];
    let html = `<div class="card"><h2>${senso.s}</h2>`;
    senso.q.forEach((q, i) => {
        html += `<div class="q-row"><span class="q-text">${q}</span>
            <select class="nota-box" id="n_${i}" required>
                <option value="">-- NOTA --</option>
                <option value="10">10 - Excelente</option>
                <option value="6">6 - Regular</option>
                <option value="2">2 - Crítico</option>
            </select></div>`;
    });
    html += `<label class="req">Plano de Ação Corretiva / Observações:</label>
             <textarea id="acao" placeholder="Descreva as melhorias necessárias para este senso..."></textarea>
             <button class="btn-p" onclick="salvarEtapa()">PRÓXIMO</button></div>`;
    
    document.getElementById('form_content').innerHTML = html;
    document.getElementById('scr_home').classList.remove('active');
    document.getElementById('scr_form').classList.add('active');
    window.scrollTo(0,0);
}

function salvarEtapa() {
    const notas = [];
    for(let i=0; i<perguntas[etapa].q.length; i++) {
        const v = document.getElementById(`n_${i}`).value;
        if(!v) return alert("Por favor, atribua uma nota para todos os itens!");
        notas.push(Number(v));
    }
    const acao = document.getElementById('acao').value;
    if(!acao || acao.length < 5) return alert("O Plano de Ação/Observação é obrigatório para garantir a melhoria contínua!");

    audit.respostas[etapa] = { notas, acao };
    if(etapa < 4) { etapa++; renderEtapa(); } 
    else { 
        db.push(audit); 
        localStorage.setItem("db_j2m_5s", JSON.stringify(db)); 
        alert("Auditoria salva localmente! Não esqueça de sincronizar.");
        location.reload();
    }
}

async function sincronizar() {
    const btn = event.target;
    const originalText = btn.innerText;
    btn.innerText = "⏳ ENVIANDO DADOS...";
    btn.disabled = true;

    try {
        await fetch(WEB_APP_URL, {
            method: 'POST',
            mode: 'no-cors',
            body: JSON.stringify(db)
        });
        alert("✅ Sincronização automática realizada com sucesso!");
    } catch (e) {
        alert("Erro na conexão. Verifique sua internet.");
    } finally {
        btn.innerText = originalText;
        btn.disabled = false;
    }
}
</script>
</body>
</html>

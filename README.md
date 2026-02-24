<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <title>Auditoria 5S J2M - v1.2025 Enterprise</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2.0.0"></script>
    <style>
        :root { --primary: #f06639; --secondary: #5d5a51; --green: #28a745; --blue: #007bff; }
        *{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}
        body { margin:0; min-height: 100vh; background-color: #f0f2f5; background-size: cover; background-position: center; background-attachment: fixed; transition: background-image 1.5s ease-in-out; position: relative; }
        body::before { content: ""; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(255, 255, 255, 0.85); z-index: -1; }
        header { background: var(--secondary); padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); color: white; font-weight: bold; font-size: 22px; }
        .screen { display:none; padding:20px; max-width: 1150px; margin: 10px auto; }
        .active { display:block; animation: fadeIn 0.4s; }
        .card { background: rgba(255,255,255,0.95); padding: 20px; border: 1px solid #ddd; border-radius: 6px; margin-bottom: 15px; border-left: 5px solid var(--primary); box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .q-row { border-bottom: 1px solid #eee; padding: 15px 0; }
        .q-text { font-weight: bold; display: block; margin-bottom: 5px; color: #444; }
        .obs-input { font-size: 12px; background: #fff9f7; margin-top: 5px; border: 1px solid #f0663944; }
        button { border: none; padding: 12px 20px; border-radius: 4px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; transition: 0.2s; margin-top: 5px; }
        .btn-primary { background: var(--primary); width: 100%; }
        .btn-sync { background: var(--blue); width: 100%; margin-top: 10px; }
        select, input, textarea { width: 100%; padding: 10px; border-radius: 4px; border: 1px solid #ccc; margin-bottom: 5px; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    </style>
</head>
<body id="bgBody">

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="home" class="screen active">
    <div class="card">
        <h3>Identificação da Auditoria</h3>
        <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 15px;">
            <div><label>Setor:</label>
                <select id="setor">
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
            </div>
            <div><label>Auditor:</label><input type="text" id="auditor"></div>
            <div><label>Responsável:</label><input type="text" id="responsavel"></div>
            <div><label>Data:</label><input type="date" id="data"></div>
        </div><br>
        <button class="btn-primary" onclick="start()">Iniciar Avaliação</button>
        <button class="btn-sync" onclick="sincronizar()">🔄 Sincronização Automática</button>
    </div>
</div>

<div id="senso" class="screen"></div>

<script>
const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbwYTDUTtW_tA8PgDQ3L5j6NFKlC2soH722nR_7lk_Kazuus8kMwn4-SLk9pG9ANwZ0/exec";

const perguntas = [
    { s: "1- Seleção", q: ["Todas as ferramentas e equipamentos são necessários para o trabalho?","Existem itens duplicados sobre a bancada?","Ferramentas acondicionadas corretamente?","Quadros de gestão e documentos atualizados?","Os avisos e quadros informativos são necessários?"]},
    { s: "2- Ordenação", q: ["Os locais para paletes e caixas estão marcados?","As linhas e marcações estão claramente visíveis?","Prateleiras e locais de armazenamento identificados?","As gavetas e armários estão identificados?","Ferramentas ordenadas por frequência de uso?"]},
    { s: "3- Limpeza", q: ["O chão está limpo e livre de detritos?","As máquinas e equipamentos estão conservados?","As fontes de sujeira foram identificadas e tratadas?","Materiais de limpeza estão disponíveis?","As lixeiras estão identificadas e limpas?"]},
    { s: "4- Padronização", q: ["Funcionários usam uniformes e EPIs corretamente?","Placas de segurança e extintores em bom estado?","A iluminação e ventilação estão adequadas?","Painéis elétricos estão fechados e identificados?","Ambientes de uso comum estão organizados?"]},
    { s: "5- Autodisciplina", q: ["A gestão mantém os padrões?","Checklist de autoavaliação é realizado?","Missão, visão e valores são conhecidos?","EPI's são usados constantemente como hábito?","Existe padrão de limpeza em gestão à vista?","Ações da Auditoria Anterior foram atendidas?"]}
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, sensoIndex = 0;

function start() {
    if(!document.getElementById('setor').value || !document.getElementById('data').value) return alert("Preencha Setor e Data!");
    audit = { 
        id: Date.now(), 
        setor: document.getElementById('setor').value, 
        auditor: document.getElementById('auditor').value, 
        responsavel: document.getElementById('responsavel').value, 
        data: document.getElementById('data').value, 
        respostas: [] 
    };
    sensoIndex = 0; 
    renderSenso();
}

function renderSenso() {
    const s = perguntas[sensoIndex];
    let html = `<div class="card"><h2>${s.s}</h2>`;
    s.q.forEach((pergunta, i) => {
        html += `<div class="q-row">
            <span class="q-text">${pergunta}</span>
            <select id="n_${i}">
                <option value="">-- NOTA --</option>
                <option value="10">10 - Excelente</option>
                <option value="6">6 - Regular/Parcial</option>
                <option value="2">2 - Crítico</option>
            </select>
            <input type="text" id="obs_${i}" class="obs-input" placeholder="Evidência/Observação (Ex: Itens em excesso)">
        </div>`;
    });
    html += `<h3>Ação Corretiva do Senso</h3>
             <textarea id="acao_txt" placeholder="Descreva a ação obrigatória..."></textarea>
             <button class="btn-primary" onclick="salvarSenso()">Próximo / Finalizar</button></div>`;
    
    document.getElementById('senso').innerHTML = html;
    show('senso');
}

function salvarSenso() {
    const notas = [];
    for(let i=0; i<perguntas[sensoIndex].q.length; i++) {
        const n = document.getElementById(`n_${i}`).value;
        const o = document.getElementById(`obs_${i}`).value;
        if(!n) return alert("Responda todas as perguntas!");
        if(n < 10 && !o) return alert("Notas menores que 10 exigem uma observação/evidência!");
        notas.push({ nota: Number(n), obs: o });
    }
    const acao = document.getElementById('acao_txt').value;
    if(!acao) return alert("O Plano de Ação para o senso é obrigatório!");

    audit.respostas.push({ senso: perguntas[sensoIndex].s, notas, acao, media: (notas.reduce((a,b)=>a+b.nota,0)/notas.length).toFixed(1) });

    if(sensoIndex < 4) {
        sensoIndex++;
        renderSenso();
    } else {
        db.push(audit);
        localStorage.setItem("j2m_db", JSON.stringify(db));
        alert("Auditoria salva localmente!");
        location.reload();
    }
}

async function sincronizar() {
    const btn = event.target;
    btn.innerText = "⏳ ENVIANDO PARA PLANILHA...";
    try {
        await fetch(WEB_APP_URL, { method: 'POST', mode: 'no-cors', body: JSON.stringify(db) });
        alert("✅ Sincronização automática concluída!");
    } catch (e) {
        alert("Erro na conexão. Verifique a internet.");
    } finally {
        btn.innerText = "🔄 Sincronização Automática";
    }
}

function show(id) { 
    document.querySelectorAll(".screen").forEach(s => s.classList.remove("active")); 
    document.getElementById(id).classList.add("active"); 
}
</script>
</body>
</html>

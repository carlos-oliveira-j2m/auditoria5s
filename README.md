<!DOCTYPE html>
<html lang="pt-br">
<head>
<meta charset="UTF-8">
<title>Auditoria 5S J2M - Oficial v4.0</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-datalabels@2"></script>
<style>
    :root { --primary: #f06639; --secondary: #5d5a51; --red: #dc3545; --green: #10b981; --blue: #007bff; }
    *{box-sizing:border-box; font-family: 'Segoe UI', sans-serif;}
    body { margin:0; background: #f4f7f6; color: #333; }
    header { background: var(--secondary); color: white; padding: 15px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; font-size: 22px; }
    .screen { display:none; padding:15px; max-width: 1000px; margin: auto; }
    .active { display:block; animation: fadeIn 0.4s; }
    .card { background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); border-left: 8px solid var(--primary); margin-bottom: 20px; }
    button { border: none; padding: 12px; border-radius: 6px; cursor: pointer; font-weight: bold; text-transform: uppercase; color: white; }
    .btn-next { background: var(--primary); width: 100%; margin-top:15px; font-size: 16px;}
    .btn-sync { background: var(--blue); width: 100%; margin-top: 10px; }
    .btn-pdf { background: var(--green); }
    label { font-weight: bold; display: block; margin-top: 12px; font-size: 14px; color: #555; }
    select, input, textarea { width: 100%; padding: 12px; border: 2px solid #ddd; border-radius: 6px; margin-top: 5px; font-size: 15px; }
    .teia-box { height: 450px; width: 100%; background: white; }
    .plano-item { padding: 10px; border-bottom: 1px solid #eee; font-size: 14px; }
    .plano-item b { color: var(--primary); display: block; text-transform: uppercase; font-size: 12px; }
    @media print { .no-print { display:none !important; } .screen { display:block !important; } .card { border: 1px solid #eee; box-shadow: none; } }
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>
</head>
<body>

<header>AUDITORIA 5S - J2M</header>

<div id="home" class="screen active">
    <div class="card">
        <h2>Nova Avaliação</h2>
        <label>Setor:</label>
        <select id="setor">
            <option value="">Selecione o Setor...</option>
            <option value="ADM FILIAL">ADM FILIAL</option><option value="CONFORMAÇÃO">CONFORMAÇÃO</option>
            <option value="ENG. SOLVE">ENG. SOLVE</option><option value="INJEÇÃO">INJEÇÃO</option>
            <option value="LOGISTICA FILIAL">LOGISTICA FILIAL</option><option value="LOGISTICA MATRIZ">LOGISTICA MATRIZ</option>
            <option value="MANUTENÇÃO">MANUTENÇÃO</option><option value="MATRIZARIA">MATRIZARIA</option>
            <option value="MONTAGEM">MONTAGEM</option><option value="PPCP">PPCP</option>
            <option value="QUALIDADE">QUALIDADE</option><option value="EXPEDIÇÃO">EXPEDIÇÃO</option>
            <option value="SALA ELETRONICOS">SALA ELETRONICOS</option><option value="SALA IMPETUS">SALA IMPETUS</option>
        </select>
        <label>Responsável:</label><input type="text" id="responsavel" placeholder="Quem responde pelo setor?">
        <label>Auditor:</label><input type="text" id="auditor" placeholder="Seu nome">
        <label>Data:</label><input type="date" id="data_audit">
        <button class="btn-next" onclick="iniciar()">Iniciar Auditoria</button>
        <button id="btnSync" class="btn-sync" onclick="puxarDaNuvem()">Sincronizar Celular/PC</button>
        <button style="background:var(--secondary); width:100%; margin-top:10px" onclick="abrirDash()">Ver Histórico/Dashboard</button>
    </div>
</div>

<div id="senso_screen" class="screen"></div>

<div id="dashboard" class="screen">
    <div id="pdf_header" style="display:none; text-align:center; margin-bottom:20px; border-bottom:2px solid var(--primary);">
        <h2 style="color:var(--primary); margin-bottom:5px;">RELATÓRIO DE AUDITORIA 5S</h2>
        <p id="pdf_dados_top" style="font-weight:bold;"></p>
    </div>
    <div class="card no-print">
        <div style="display:flex; gap:10px">
            <select id="fSetor" onchange="filtrar()"></select>
            <button class="btn-pdf" onclick="window.print()">Gerar PDF</button>
            <button style="background:#444" onclick="location.reload()">Início</button>
        </div>
    </div>
    <div class="card"><div class="teia-box"><canvas id="chartRadar"></canvas></div></div>
    <div class="card"><h3>Ações Corretivas do Setor</h3><div id="lista_acoes"></div></div>
    <div class="card no-print"><h3>Histórico de Registros</h3><div id="lista_hist"></div></div>
</div>

<script>
Chart.register(ChartDataLabels);

// SEU LINK GOOGLE (MANTIDO)
const API = "https://script.google.com/macros/s/AKfycbwPjqUwMg-YiCxlMhBujpMDUZZqZ_mZFFRIwyXUkojM40H_IaIjSr6utfevshUg5WFv/exec";

const nomesSensos = ["Seleção", "Ordenação", "Limpeza", "Padronização", "Autodisciplina"];
const perguntas5S = [
    { s: "SELEÇÃO", p: ["Ferramentas e equipamentos são necessários?", "Existem itens desnecessários na área?", "Ferramentas acondicionadas corretamente?", "Checklists e documentos atualizados?", "Avisos e quadros são úteis e atuais?"] },
    { s: "ORDENAÇÃO", p: ["Locais de estoque (paletes/caixas) demarcados?", "Corredores e marcações de piso visíveis?", "Prateleiras e armários identificados?", "Material de limpeza organizado?", "Identificação de pessoal e setores clara?", "Objetos pessoais em local apropriado?"] },
    { s: "LIMPEZA", p: ["Máquinas e bancadas estão limpas?", "Piso livre de óleo e resíduos?", "Equipamentos em boas condições técnicas?", "Áreas comuns e salas limpas?", "Coleta seletiva sendo respeitada?"] },
    { s: "PADRONIZAÇÃO", p: ["Documentos e indicadores seguem o padrão?", "Aspectos ergonômicos e iluminação OK?", "Lixeiras identificadas e no local correto?", "Equipamentos de segurança desobstruídos?"] },
    { s: "AUTODISCIPLINA", p: ["Padrões 5S são mantidos no dia a dia?", "Autoavaliação periódica é realizada?", "Missão e Valores são conhecidos?", "Uso correto de EPIs pela equipe?", "Gestão à vista atualizada?", "Ações da última auditoria resolvidas?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db_v3") || "[]");
let audit = {}, etapa = 0;
let myChart;

function iniciar() {
    if(!setor.value || !auditor.value || !data_audit.value) return alert("Por favor, preencha todos os campos iniciais.");
    audit = { id: Date.now(), setor: setor.value, responsavel: responsavel.value, auditor: auditor.value, data: data_audit.value, respostas: [] };
    etapa = 0; mostrarSenso();
}

function mostrarSenso() {
    const s = perguntas5S[etapa];
    let h = `<div class="card"><h2>${etapa+1}º Senso: ${s.s}</h2>`;
    s.p.forEach((p, i) => {
        h += `<label>${p}</label><select class="p-item" id="p_${i}">
            <option value="">Selecione a Nota...</option>
            <option value="10">10 - Excelente (Sem evidências)</option><option value="8">08 - Bom (1 evidência)</option>
            <option value="6">06 - Médio (2 evidências)</option><option value="4">04 - Melhorar (3 evidências)</option><option value="2">02 - Crítico (4+ evidências)</option>
        </select>`;
    });
    h += `<label>Plano de Ação (Obrigatório para médias < 6):</label><textarea id="txt_plano" rows="3" placeholder="O que será feito para melhorar?"></textarea>
          <button class="btn-next" onclick="proximo()">Salvar e Próximo</button></div>`;
    document.getElementById("senso_screen").innerHTML = h;
    trocarTela("senso_screen");
}

function proximo() {
    const sels = document.querySelectorAll(".p-item");
    let soma = 0, ok = true;
    sels.forEach(s => { if(!s.value) ok = false; soma += Number(s.value); });
    if(!ok) return alert("Responda todas as perguntas antes de continuar.");
    
    const media = soma / sels.length;
    const plano = document.getElementById("txt_plano").value.trim();
    if(media < 6 && !plano) return alert("Para notas baixas, o Plano de Ação é obrigatório!");

    audit.respostas.push({ media, plano });
    etapa++;
    if(etapa < 5) mostrarSenso();
    else finalizar();
}

function finalizar() {
    db.push(audit);
    localStorage.setItem("j2m_db_v3", JSON.stringify(db));
    fetch(API, { method: 'POST', mode: 'no-cors', body: JSON.stringify(audit) });
    alert("Auditoria finalizada com sucesso!");
    abrirDash();
}

async function puxarDaNuvem() {
    const btn = document.getElementById("btnSync");
    btn.innerText = "Sincronizando...";
    try {
        const r = await fetch(API);
        const data = await r.json();
        if(data && data.length > 1) {
            db = data.slice(1).map(row => ({
                id: row[10], setor: row[1], responsavel: row[2], auditor: row[3], data: row[9],
                respostas: JSON.parse(row[11])
            }));
            localStorage.setItem("j2m_db_v3", JSON.stringify(db));
            alert("Dados recuperados da nuvem!");
            abrirDash();
        } else { alert("A planilha está vazia."); }
    } catch(e) { alert("Erro ao conectar com a planilha. Verifique a internet."); }
    btn.innerText = "Sincronizar Celular/PC";
}

function abrirDash() {
    const fs = document.getElementById("fSetor");
    fs.innerHTML = '<option value="TODOS">Filtrar Setor</option>';
    [...new Set(db.map(x => x.setor))].forEach(s => fs.innerHTML += `<option value="${s}">${s}</option>`);
    filtrar();
    trocarTela("dashboard");
}

function filtrar() {
    const sel = document.getElementById("fSetor").value;
    render(sel === "TODOS" ? db : db.filter(x => x.setor === sel));
}

function render(dados) {
    if(!dados.length) { document.getElementById("lista_hist").innerHTML = "Nenhum dado encontrado."; return; }
    const ult = dados[dados.length - 1];
    const mSetor = [0,1,2,3,4].map(i => (dados.reduce((acc, a) => acc + a.respostas[i].media, 0) / dados.length).toFixed(1));
    const mGeral = [0,1,2,3,4].map(i => (db.reduce((acc, a) => acc + a.respostas[i].media, 0) / db.length).toFixed(1));

    document.getElementById("pdf_dados_top").innerText = `Setor: ${ult.setor} | Resp: ${ult.responsavel} | Data: ${ult.data}`;

    if(myChart) myChart.destroy();
    myChart = new Chart(document.getElementById("chartRadar"), {
        type: 'radar',
        data: {
            labels: nomesSensos,
            datasets: [
                { label: 'Nota Atual', data: mSetor, borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.3)', datalabels: { display: true } },
                { label: 'Média Fábrica', data: mGeral, borderColor: '#5d5a51', borderDash: [5,5], fill: false },
                { label: 'Meta 8.0', data: [8,8,8,8,8], borderColor: '#10b981', fill: false }
            ]
        },
        options: { maintainAspectRatio: false, scales: { r: { min: 0, max: 10, ticks: { display: false } } } }
    });

    let hAcoes = "";
    ult.respostas.forEach((r, i) => { if(r.plano) hAcoes += `<div class='plano-item'><b>${nomesSensos[i]}</b> ${r.plano}</div>`; });
    document.getElementById("lista_acoes").innerHTML = hAcoes || "Setor em conformidade (Sem ações pendentes).";
    
    let hHist = "<table style='width:100%; border-collapse:collapse; font-size:13px;'>";
    dados.slice().reverse().forEach(a => {
        hHist += `<tr style='border-bottom:1px solid #eee; height:35px'><td>${a.data}</td><td>${a.setor}</td><td style='text-align:right'><button onclick="excluir(${a.id})" style='background:red;padding:2px 5px; font-size:10px'>X</button></td></tr>`;
    });
    document.getElementById("lista_hist").innerHTML = hHist + "</table>";
}

function excluir(id) { if(confirm("Deseja apagar este registro?")) { db = db.filter(x => x.id !== id); localStorage.setItem("j2m_db_v3", JSON.stringify(db)); filtrar(); } }
function trocarTela(id) { document.querySelectorAll(".screen").forEach(s => s.classList.remove("active")); document.getElementById(id).classList.add("active"); }
window.onbeforeprint = () => { document.getElementById("pdf_header").style.display = "block"; };
window.onafterprint = () => { document.getElementById("pdf_header").style.display = "none"; };
</script>
</body>
</html>

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

body { 
    margin:0; 
    min-height: 100vh;
    background-color: #f0f2f5;
    background-size: cover;
    background-position: center;
    background-attachment: fixed;
    transition: background-image 1.5s ease-in-out;
    position: relative;
}

body::before {
    content: "";
    position: fixed;
    top: 0; left: 0; width: 100%; height: 100%;
    background: rgba(255, 255, 255, 0.85);
    z-index: -1;
}

header { 
    background: var(--secondary); 
    color: white; 
    padding: 20px; 
    text-align: center; 
    border-bottom: 6px solid var(--primary);
    font-size: 24px;
    font-weight: bold;
    letter-spacing: 1px;
}

.container { max-width: 1100px; margin: 30px auto; padding: 0 20px; }

.card { 
    background: white; 
    padding: 30px; 
    border-radius: 12px; 
    box-shadow: 0 10px 25px rgba(0,0,0,0.1); 
    margin-bottom: 30px;
    border-top: 5px solid var(--primary);
}

.stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; margin-bottom: 25px; }
.stat-card { 
    background: white; 
    padding: 20px; 
    border-radius: 10px; 
    text-align: center; 
    box-shadow: 0 4px 6px rgba(0,0,0,0.05);
    border: 1px solid #eee;
}
.stat-card h1 { margin: 10px 0 0 0; font-size: 42px; color: var(--secondary); }
.stat-card p { margin: 0; color: #666; font-weight: bold; text-transform: uppercase; font-size: 13px; }

.grid-charts { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }

/* Estilos de Botões */
.btn { 
    padding: 12px 25px; 
    border: none; 
    border-radius: 6px; 
    cursor: pointer; 
    font-weight: bold; 
    text-transform: uppercase; 
    transition: 0.3s;
    color: white;
}
.btn-primary { background: var(--primary); width: 100%; font-size: 16px; }
.btn-sync { background: var(--blue); margin-bottom: 10px; width: 100%; }

.screen { display: none; }
.active { display: block; animation: fadeIn 0.5s; }

@keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

@media print {
    .no-print { display: none !important; }
    body::before { display: none; }
    .screen { display: block !important; }
    .container { max-width: 100%; margin: 0; }
    .card { box-shadow: none; border: 1px solid #ccc; }
}

/* Tabelas e Formulários (Mantidos do seu original) */
table { width: 100%; border-collapse: collapse; margin-top: 15px; }
th { background: #f8f9fa; padding: 12px; text-align: left; border-bottom: 2px solid #dee2e6; }
td { padding: 12px; border-bottom: 1px solid #eee; }
select, input, textarea { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd; border-radius: 6px; }
</style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div class="container">
    
    <div id="screen-home" class="screen active no-print">
        <div class="card">
            <h2>Bem-vindo, Auditor</h2>
            <div style="margin-top: 20px;">
                <button class="btn btn-sync" onclick="sincronizarComGoogle()">Sincronizar Dados da Nuvem</button>
                <button class="btn btn-primary" onclick="irPara('screen-setup')">Iniciar Nova Auditoria</button>
                <button class="btn" style="background: var(--secondary); width: 100%; margin-top: 10px;" onclick="irPara('screen-relatorio')">Visualizar Relatórios</button>
            </div>
        </div>
    </div>

    <div id="screen-setup" class="screen no-print">
        <div class="card">
            <h3>Configurações Iniciais</h3>
            <label>Setor:</label>
            <select id="setor">
                <option value="ADM Filial">ADM Filial</option>
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
            <label>Auditor:</label>
            <input type="text" id="auditor" placeholder="Seu nome">
            <button class="btn btn-primary" style="margin-top: 20px;" onclick="iniciarAuditoria()">Começar</button>
            <button class="btn" style="background:#666; width:100%; margin-top:10px" onclick="irPara('screen-home')">Voltar</button>
        </div>
    </div>

    <div id="screen-audit" class="screen no-print">
        <div id="pergunta-container"></div>
    </div>

    <div id="screen-relatorio" class="screen">
        <div class="stats-grid">
            <div class="stat-card">
                <p>Média Geral da Fábrica</p>
                <h1 id="avg-factory">0.0</h1>
            </div>
            <div class="stat-card">
                <p>Total de Auditorias</p>
                <h1 id="total-audits">0</h1>
            </div>
        </div>

        <div class="grid-charts">
            <div class="card">
                <h3>Setor Selecionado</h3>
                <div style="height: 350px;"><canvas id="cRadar"></canvas></div>
            </div>
            <div class="card">
                <h3>Nota Final por Setor</h3>
                <div style="height: 350px;"><canvas id="cBarra"></canvas></div>
            </div>
        </div>

        <div class="card">
            <h3>Histórico de Auditorias</h3>
            <table id="tabela-hist">
                <thead>
                    <tr><th>Data</th><th>Setor</th><th>Auditor</th><th>Nota</th><th>Ações</th></tr>
                </thead>
                <tbody id="lista-hist"></tbody>
            </table>
        </div>

        <div class="no-print" style="text-align: center;">
            <button class="btn" style="background:var(--green); margin-right:10px" onclick="window.print()">Imprimir PDF</button>
            <button class="btn" style="background:#666" onclick="irPara('screen-home')">Menu Inicial</button>
        </div>
    </div>
</div>

<script>
Chart.register(ChartDataLabels);

// CONFIGURAÇÃO DO GOOGLE SHEETS
const URL_GOOGLE = "https://script.google.com/macros/s/AKfycbwPjqUwMg-YiCxlMhBujpMDUZZqZ_mZFFRIwyXUkojM40H_IaIjSr6utfevshUg5WFv/exec";

const imagens = [
    'https://images.unsplash.com/photo-1581091226825-a6a2a5aee158?auto=format&fit=crop&q=80&w=1920',
    'https://images.unsplash.com/photo-1504328332780-bebf217e0653?auto=format&fit=crop&q=80&w=1920',
    'https://images.unsplash.com/photo-1581092580497-e0d23cbdf1dc?auto=format&fit=crop&q=80&w=1920'
];

const checklist = [
    { s: "Seleção", p: ["Ferramentas e equipamentos são necessários na área?", "Existem itens desnecessários, duplicados ou obsoletos?", "Materiais de uso frequente estão próximos?", "Documentos e informativos estão atualizados?", "Área livre de sucatas e lixos?"] },
    { s: "Ordenação", p: ["Locais de estoque e paletes demarcados?", "Corredores e linhas de fluxo visíveis?", "Prateleiras e gavetas identificadas?", "Material de limpeza em local próprio?", "Objetos pessoais guardados corretamente?"] },
    { s: "Limpeza", p: ["Piso, máquinas e bancadas estão limpos?", "Ausência de vazamentos de óleo/ar/água?", "Equipamentos em boas condições de uso?", "Coleta seletiva está sendo respeitada?", "Higiene pessoal e uniformes adequados?"] },
    { s: "Padronização", p: ["Utilização correta de cores e sinais?", "Documentação técnica acessível?", "Lixeiras padronizadas e limpas?", "Quadros de avisos organizados?"] },
    { s: "Disciplina", p: ["Normas de segurança e EPIs respeitados?", "Melhorias propostas foram executadas?", "Padrões 5S mantidos no dia a dia?", "Equipe engajada com a organização?"] }
];

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]");
let audit = {}, etapa = 0, chartRadar, chartBar;

function mudarFundo() {
    const img = imagens[Math.floor(Math.random() * imagens.length)];
    document.body.style.backgroundImage = `url('${img}')`;
}
setInterval(mudarFundo, 10000);
mudarFundo();

function irPara(id) {
    document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
    document.getElementById(id).classList.add('active');
    if(id === 'screen-relatorio') renderRelatorio();
}

function iniciarAuditoria() {
    if(!document.getElementById('auditor').value) return alert("Informe seu nome!");
    audit = {
        id: Date.now(),
        setor: document.getElementById('setor').value,
        auditor: document.getElementById('auditor').value,
        data: new Date().toLocaleDateString(),
        respostas: []
    };
    etapa = 0;
    gerarPergunta();
    irPara('screen-audit');
}

function gerarPergunta() {
    const s = checklist[etapa];
    let h = `<div class="card"><h2>Senso de ${s.s}</h2>`;
    s.p.forEach((p, i) => {
        h += `<label>${p}</label>
              <select class="q-nota" data-idx="${i}">
                <option value="10">10 - Excelente</option>
                <option value="8">8 - Bom</option>
                <option value="6">6 - Regular</option>
                <option value="4">4 - Ruim</option>
                <option value="2">2 - Crítico</option>
              </select>`;
    });
    h += `<label>Plano de Ação (se necessário):</label><textarea id="plano-txt"></textarea>`;
    h += `<button class="btn btn-primary" onclick="proximoSenso()">Próximo</button></div>`;
    document.getElementById('pergunta-container').innerHTML = h;
}

function proximoSenso() {
    const notas = Array.from(document.querySelectorAll('.q-nota')).map(sel => Number(sel.value));
    const media = notas.reduce((a,b)=>a+b,0) / notas.length;
    audit.respostas.push({ media: media, plano: document.getElementById('plano-txt').value });
    
    etapa++;
    if(etapa < checklist.length) gerarPergunta();
    else finalizarAuditoria();
}

function finalizarAuditoria() {
    db.push(audit);
    localStorage.setItem("j2m_db", JSON.stringify(db));
    
    // SINCRONIZAÇÃO AUTOMÁTICA AO FINALIZAR
    fetch(URL_GOOGLE, {
        method: 'POST',
        mode: 'no-cors',
        body: JSON.stringify(audit)
    });

    alert("Auditoria salva e enviada para a nuvem!");
    irPara('screen-relatorio');
}

async function sincronizarComGoogle() {
    const btn = document.querySelector('.btn-sync');
    btn.innerText = "A Sincronizar...";
    try {
        const r = await fetch(URL_GOOGLE);
        const data = await r.json();
        if(data && data.length > 1) {
            // Reconstrói o DB local com os dados da Planilha
            db = data.slice(1).map(row => ({
                id: row[10], setor: row[1], auditor: row[3], data: row[9],
                respostas: JSON.parse(row[11])
            }));
            localStorage.setItem("j2m_db", JSON.stringify(db));
            alert("Sincronização concluída com sucesso!");
            renderRelatorio();
        }
    } catch(e) { alert("Erro ao sincronizar. Verifique a conexão."); }
    btn.innerText = "Sincronizar Dados da Nuvem";
}

function renderRelatorio() {
    if(db.length === 0) return;
    const ult = db[db.length - 1];
    
    // Indicadores
    document.getElementById('total-audits').innerText = db.length;
    const totalSoma = db.reduce((acc, a) => acc + (a.respostas.reduce((x,y)=>x+Number(y.media),0)/5), 0);
    document.getElementById('avg-factory').innerText = (totalSoma / db.length).toFixed(1);

    // Radar
    if(chartRadar) chartRadar.destroy();
    chartRadar = new Chart(document.getElementById('cRadar'), {
        type: 'radar',
        data: {
            labels: ["Seleção", "Ordenação", "Limpeza", "Padronização", "Disciplina"],
            datasets: [
                { label: ult.setor, data: ult.respostas.map(r => r.media), borderColor: '#f06639', backgroundColor: 'rgba(240,102,57,0.2)' },
                { label: 'Meta', data: [8,8,8,8,8], borderColor: '#28a745', borderDash: [5,5], fill: false }
            ]
        },
        options: { scales: { r: { min:0, max:10 } } }
    });

    // Barras
    const setores = [...new Set(db.map(a => a.setor))];
    const notas = setores.map(s => {
        const d = db.filter(x => x.setor === s);
        return (d.reduce((acc, a) => acc + (a.respostas.reduce((x,y)=>x+Number(y.media),0)/5), 0) / d.length).toFixed(1);
    });

    if(chartBar) chartBar.destroy();
    chartBar = new Chart(document.getElementById('cBarra'), {
        type: 'bar',
        data: { labels: setores, datasets: [{ label: 'Nota Final', data: notas, backgroundColor: '#5d5a51' }] },
        options: { scales: { y: { min: 0, max: 10 } }, plugins: { datalabels: { display: true } } }
    });

    // Tabela
    const lista = document.getElementById('lista-hist');
    lista.innerHTML = "";
    db.slice().reverse().forEach(a => {
        const nota = (a.respostas.reduce((x,y)=>x+Number(y.media),0)/5).toFixed(1);
        lista.innerHTML += `<tr>
            <td>${a.data}</td><td>${a.setor}</td><td>${a.auditor}</td><td><b>${nota}</b></td>
            <td><button onclick="excluir(${a.id})" style="background:red; color:white; border:none; border-radius:4px; padding:4px 8px; cursor:pointer">X</button></td>
        </tr>`;
    });
}

function excluir(id) {
    if(confirm("Excluir auditoria?")) {
        db = db.filter(a => a.id !== id);
        localStorage.setItem("j2m_db", JSON.stringify(db));
        renderRelatorio();
    }
}
</script>
</body>
</html>

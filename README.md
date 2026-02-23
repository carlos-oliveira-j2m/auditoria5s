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

let db = JSON.parse(localStorage.getItem("j2m_db") || "[]

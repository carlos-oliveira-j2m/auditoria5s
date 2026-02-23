<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Auditoria 5S J2M - v1.2025</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root { --primary: #f06639; --secondary: #5d5a51; }
        body { font-family: 'Segoe UI', sans-serif; margin: 0; background: #f4f4f4; }
        header { background: var(--secondary); color: white; padding: 20px; text-align: center; border-bottom: 5px solid var(--primary); font-weight: bold; font-size: 20px; }
        .container { max-width: 800px; margin: 20px auto; background: white; padding: 25px; border-radius: 8px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); border-left: 8px solid var(--primary); }
        h2 { color: var(--secondary); border-bottom: 2px solid #eee; padding-bottom: 10px; }
        .pergunta-bloco { margin-bottom: 20px; padding: 15px; background: #fafafa; border-radius: 5px; border: 1px solid #eee; }
        label { display: block; font-weight: bold; margin-bottom: 8px; color: #333; }
        select, textarea, input { width: 100%; padding: 12px; border: 1px solid #ccc; border-radius: 4px; box-sizing: border-box; font-size: 14px; }
        button { background: var(--primary); color: white; border: none; padding: 15px 30px; border-radius: 4px; cursor: pointer; font-weight: bold; width: 100%; text-transform: uppercase; }
        button:hover { background: #d9532d; }
        .foto-preview { margin-top: 10px; max-width: 100%; border-radius: 4px; }
    </style>
</head>
<body>

<header>AUDITORIA 5S J2M - v1.2025</header>

<div id="app" class="container">
    </div>

<script>
    const setores = ["ADM FILIAL", "CONFORMAÇÃO", "ENG. SOLVE", "INJEÇÃO", "LOGISTICA FILIAL", "LOGISTICA MATRIZ", "MANUTENÇÃO", "MATRIZARIA", "MONTAGEM", "PPCP", "QUALIDADE", "SALA ELETRONICOS", "SALA IMPETUS"];

    const checklist = [
        { senso: "SEIRI - UTILIZAÇÃO", perguntas: [
            "Existem itens desnecessários (ferramentas, papéis, peças) na área?",
            "Há estoque excessivo ou materiais obsoletos no setor?",
            "Os quadros de avisos e documentos estão atualizados?",
            "As ferramentas de uso constante estão acessíveis?",
            "O descarte de itens inúteis é feito corretamente?"
        ]},
        { senso: "SEITON - ORGANIZAÇÃO", perguntas: [
            "As ferramentas e materiais possuem local fixo e identificado?",
            "O piso possui demarcação de corredores e áreas de estoque?",
            "É fácil localizar qualquer item em menos de 30 segundos?",
            "Armários e gavetas estão organizados internamente?",
            "Os equipamentos estão posicionados para facilitar o fluxo?"
        ]}
        // Adicione os outros 3 sensos aqui seguindo o mesmo padrão
    ];

    let currentStep = 0;
    let auditoriaData = { setor: "", auditor: "", data: "", respostas: [] };

    function renderHome() {
        let html = `<h2>Identificação</h2>
            <label>Setor:</label>
            <select id="setor">${setores.map(s => `<option value="${s}">${s}</option>`).join('')}</select>
            <br><br><label>Auditor:</label>
            <input type="text" id="auditor" placeholder="Seu nome">
            <br><br><button onclick="startAudit()">Iniciar Auditoria</button>`;
        document.getElementById('app').innerHTML = html;
    }

    function startAudit() {
        auditoriaData.setor = document.getElementById('setor').value;
        auditoriaData.auditor = document.getElementById('auditor').value;
        auditoriaData.data = new Date().toLocaleDateString();
        renderStep();
    }

    function renderStep() {
        const step = checklist[currentStep];
        let html = `<h2>${step.senso}</h2>`;
        step.perguntas.forEach((p, i) => {
            html += `<div class="pergunta-bloco">
                <label>${i+1}. ${p}</label>
                <select id="nota_${i}">
                    <option value="10">Excelente (Tudo OK)</option>
                    <option value="8">Bom (Pequenos desvios)</option>
                    <option value="5">Regular (Atenção necessária)</option>
                    <option value="0">Péssimo (Grave/Inexistente)</option>
                </select>
            </div>`;
        });
        html += `<label>Plano de Ação / Observações:</label>
                 <textarea id="obs" rows="3" placeholder="O que precisa ser melhorado?"></textarea>
                 <br><br><label>Foto da Evidência:</label>
                 <input type="file" accept="image/*" capture="environment">
                 <br><br><button onclick="nextStep()">${currentStep === checklist.length - 1 ? 'Finalizar' : 'Próximo Senso'}</button>`;
        document.getElementById('app').innerHTML = html;
    }

    function nextStep() {
        // Lógica para salvar e avançar
        currentStep++;
        if(currentStep < checklist.length) renderStep();
        else alert("Auditoria Concluída! (Simulação de salvamento)");
    }

    renderHome();
</script>
</body>
</html>

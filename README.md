<!doctype html>
<html lang="pt-br">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>Avaliação 5S v1.2025</title>
  <meta name="description" content="Auditoria 5S - Avaliação v1.2025 (GitHub Pages)"/>

  <!-- Materialize CSS + Icons -->
  <link rel="preconnect" href="https://fonts.googleapis.com" crossorigin>
  <link rel="dns-prefetch" href="https://fonts.googleapis.com">
  <link rel="dns-prefetch" href="https://cdnjs.cloudflare.com">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/css/materialize.min.css"/>
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">

  <!-- Estilos do app -->
  <link rel="stylesheet" href="./styles.css">
</head>
<body>
  <nav class="blue-grey darken-3">
    <div class="nav-wrapper container">
      <a href="#inicio" class="brand-logo">Avaliação 5S</a>
      <ul class="right hide-on-med-and-down">
        <li><a href="#inicio">Início</a></li>
        <li><a href="#form">Formulário</a></li>
        <li><a href="#dash">Dashboard</a></li>
      </ul>
    </div>
  </nav>

  <main class="container" style="margin-top:24px;">
    <!-- INÍCIO -->
    <section id="tela-inicio" class="page">
      <h5>Dados Iniciais (preenchimento obrigatório)</h5>
      <div class="row">
        <div class="input-field col s12 m6">
          <select id="setor"></select>
          <label>Setor</label>
        </div>
        <div class="input-field col s12 m6">
          <input id="responsavel" type="text" required>
          <label for="responsavel">Responsável da área</label>
        </div>
        <div class="input-field col s12 m6">
          <input id="auditor" type="text" required>
          <label for="auditor">Auditor</label>
        </div>
        <div class="input-field col s12 m6">
          <input id="dataISO" type="date" required>
          <label class="active" for="dataISO">Data</label>
        </div>
      </div>

      <div class="row btn-group">
        <a class="btn waves-effect" id="btnIniciar"><i class="material-icons left">play_arrow</i>Iniciar</a>
        <a class="btn grey darken-2 waves-effect" href="#dash"><i class="material-icons left">dashboard</i>Dashboard</a>
        <a class="btn teal waves-effect" id="btnSync"><i class="material-icons left">sync</i>Sincronizar</a>
      </div>
    </section>

    <!-- FORM -->
    <section id="tela-form" class="page" style="display:none;">
      <div class="card-panel">
        <span class="title">Formulário – Avaliação 5S v1.2025</span>
        <p class="grey-text">
          Responda as 26 perguntas. Notas: <b>10, 8, 6, 4, 2</b>. Para <b>nota &lt; 6</b>, a <b>Ação corretiva</b> é obrigatória.
        </p>
        <div id="formPerguntas"></div>

        <div class="row" style="margin-top:24px;">
          <a class="btn grey waves-effect" id="btnVoltar"><i class="material-icons left">arrow_back</i>Voltar (salva automaticamente)</a>
          <a class="btn teal waves-effect" id="btnSalvarRascunho"><i class="material-icons left">save</i>Salvar rascunho</a>
          <a class="btn green waves-effect" id="btnEnviar"><i class="material-icons left">cloud_upload</i>Finalizar</a>
        </div>
      </div>
    </section>

    <!-- DASHBOARD -->
    <section id="tela-dash" class="page" style="display:none;">
      <div class="row">
        <div class="input-field col s12 m4">
          <select id="filtroSetor"></select>
          <label>Filtro: Setor</label>
        </div>
        <div class="input-field col s12 m4">
          <select id="filtroMes">
            <option value="" selected>Todos</option>
            <option value="1">01</option><option value="2">02</option><option value="3">03</option>
            <option value="4">04</option><option value="5">05</option><option value="6">06</option>
            <option value="7">07</option><option value="8">08</option><option value="9">09</option>
            <option value="10">10</option><option value="11">11</option><option value="12">12</option>
          </select>
          <label>Filtro: Mês</label>
        </div>
        <div class="input-field col s12 m4">
          <input id="filtroAno" type="number" min="2000" max="2100">
          <label for="filtroAno">Filtro: Ano</label>
        </div>
      </div>
      <div class="row btn-group">
        <a class="btn waves-effect" id="btnAplicarFiltros"><i class="material-icons left">filter_list</i>Aplicar filtros</a>
        <a class="btn teal waves-effect" id="btnSync2"><i class="material-icons left">sync</i>Sincronizar formulários</a>
        <a class="btn red waves-effect" id="btnPDF"><i class="material-icons left">picture_as_pdf</i>Salvar Relatório em PDF</a>
      </div>

      <div class="row">
        <div class="col s12 m6">
          <canvas id="chartRadar" height="300"></canvas>
        </div>
        <div class="col s12 m6">
          <canvas id="chartBarras" height="300"></canvas>
        </div>
      </div>

      <h6>Registros</h6>
      <table class="striped responsive-table">
        <thead>
          <tr>
            <th>Data</th><th>Setor</th><th>Resp.</th><th>Auditor</th><th>Nota</th><th>Ações</th>
          </tr>
        </thead>
        <tbody id="tbRegistros"></tbody>
      </table>

      <h6 style="margin-top:24px;">Planos de Ação por Senso</h6>
      <div class="row">
        <div class="input-field col s12 m6">
          <select id="filtroSensoAcoes">
            <option value="">Todos</option>
            <option value="Seleção">Seleção</option>
            <option value="Ordenação">Ordenação</option>
            <option value="Limpeza">Limpeza</option>
            <option value="Padronização">Padronização</option>
            <option value="Autodisciplina">Autodisciplina</option>
          </select>
          <label>Filtrar por Senso</label>
        </div>
        <div class="input-field col s12 m6">
          <select id="ordenarAcoes">
            <option value="PrazoISO">Ordenar por Prazo</option>
            <option value="Status">Ordenar por Status</option>
            <option value="Setor">Ordenar por Setor</option>
          </select>
          <label>Ordenação</label>
        </div>
      </div>
      <table class="striped responsive-table">
        <thead>
          <tr>
            <th>Senso</th><th>Pergunta</th><th>Nota</th><th>Ação</th><th>Responsável</th><th>Prazo</th><th>Status</th>
          </tr>
        </thead>
        <tbody id="tbAcoes"></tbody>
      </table>
    </section>
  </main>

  <!-- Libs -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/materialize/1.0.0/js/materialize.min.js" defer></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js" defer></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js" defer></script>

  <!-- App -->
  <script src="./questions.js" defer></script>
  <script src="./app.js" defer></script>
</body>
</html>

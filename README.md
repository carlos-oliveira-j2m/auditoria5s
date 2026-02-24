/* global M, Chart, jsPDF, QUESTOES_5S, EXPLICACOES_NOTA, SCORES_VALIDOS, LISTA_SETORES, MEDIA_DESEJADA */

// ================== CONFIG ==================
// URL do seu Web App (Apps Script). Pré-configurada com a que você enviou.
// Você pode alterar no modal "Sincronização" e o valor fica salvo em localStorage.
const API_BASE_DEFAULT = 'https://script.google.com/macros/s/AKfycbyJ_kHIIJC3_7mbJTLbZEnEBtnyU1DpxH5aS42HL-VRXAG1KI4AmDUH0ZGIPgkEXXWX/exec';

// Keys de LocalStorage
const LS_KEYS = {
  drafts: 'aud5s_drafts',
  submissions: 'aud5s_submissions',
  apiBase: 'aud5s_api_base'
};
const getApiBase = () => (localStorage.getItem(LS_KEYS.apiBase) || API_BASE_DEFAULT || '').trim();

// ================== STATE ==================
let radarChart = null;
let barrasChart = null;
let CURRENT_DRAFT_META = null; // {id,setor,responsavel,auditor,dataISO}

// ================== UTILS ==================
const Utils = {
  byId: (id) => document.getElementById(id),
  toast: (msg) => M.toast({ html: msg }),
  fmtDate: (iso) => (iso || '').split('T')[0] || iso,
  ensureArray: (x) => Array.isArray(x) ? x : [],
  loadSubmissions() { return JSON.parse(localStorage.getItem(LS_KEYS.submissions) || '[]'); },
  saveSubmissions(arr) { localStorage.setItem(LS_KEYS.submissions, JSON.stringify(arr || [])); },
  loadDrafts() { return JSON.parse(localStorage.getItem(LS_KEYS.drafts) || '[]'); },
  saveDrafts(arr) { localStorage.setItem(LS_KEYS.drafts, JSON.stringify(arr || [])); },
  round2(n) { return Math.round(n * 100) / 100; }
};

// ================== NAV ==================
const Nav = {
  go(hash) {
    document.querySelectorAll('.page').forEach(p => p.style.display = 'none');
    if (hash === '#form') Utils.byId('tela-form').style.display = 'block';
    else if (hash === '#dash') Utils.byId('tela-dash').style.display = 'block';
    else Utils.byId('tela-inicio').style.display = 'block';
    setTimeout(()=> M.updateTextFields(), 50);
  },
  init() {
    window.addEventListener('hashchange', () => Nav.go(location.hash));
    Nav.go(location.hash || '#inicio');
  }
};

// ================== APP (Página Inicial) ==================
const App = {
  initInicio() {
    // Popula setores e filtros
    const selSetor = Utils.byId('setor');
    selSetor.innerHTML = '<option value="" disabled selected>Selecione</option>' + LISTA_SETORES.map(s => `<option value="${s}">${s}</option>`).join('');
    const filtroSetor = Utils.byId('filtroSetor');
    filtroSetor.innerHTML = '<option value="" selected>Todos</option>' + LISTA_SETORES.map(s => `<option value="${s}">${s}</option>`).join('');
    M.FormSelect.init(selSetor); M.FormSelect.init(filtroSetor);

    // Botões
    Utils.byId('btnIniciar').addEventListener('click', () => {
      const meta = App.getMeta();
      if (!App.validarMeta(meta)) { Utils.toast('Preencha todos os campos obrigatórios!'); return; }
      meta.id = 'draft_' + Date.now();
      CURRENT_DRAFT_META = meta;
      Form.renderPerguntas(null); // novo
      location.hash = '#form';
    });

    // Modais
    M.Modal.init(document.querySelectorAll('.modal'));
    // URL API
    Utils.byId('apiBase').value = getApiBase();
    Utils.byId('btnSalvarApi').addEventListener('click', () => {
      const url = (Utils.byId('apiBase').value || '').trim();
      localStorage.setItem(LS_KEYS.apiBase, url);
      Utils.toast(url ? 'URL salva.' : 'URL removida. Modo local.');
    });

    // Sync
    Utils.byId('btnSyncEnviar').addEventListener('click', Sync.sincronizarUp);
    Utils.byId('btnSyncBaixar').addEventListener('click', Sync.sincronizarDown);
  },

  getMeta() {
    return {
      setor: Utils.byId('setor').value?.trim(),
      responsavel: Utils.byId('responsavel').value?.trim(),
      auditor: Utils.byId('auditor').value?.trim(),
      dataISO: Utils.byId('dataISO').value?.trim()
    };
  },

  validarMeta(m) {
    return m.setor && m.responsavel && m.auditor && m.dataISO;
  }
};

// ================== FORM ==================
const Form = {
  // Se idEdit for fornecido, carrega submissão e permite editar
  renderPerguntas(idEdit) {
    const container = Utils.byId('formPerguntas');
    container.innerHTML = '';

    // Se edição: carrega respostas
    let editData = null;
    if (idEdit) {
      const subs = Utils.loadSubmissions();
      editData = subs.find(x => x.id === idEdit) || null;
      if (editData) CURRENT_DRAFT_META = editData.meta;
    }

    QUESTOES_5S.forEach(q => {
      const card = document.createElement('div'); card.className = 'card pergunta-card';
      const inner = document.createElement('div'); inner.className = 'card-content';

      const title = document.createElement('span'); title.className = 'card-title'; title.textContent = `${q.id} – ${q.senso}`;
      inner.appendChild(title);

      const pTxt = document.createElement('p'); pTxt.textContent = q.texto; inner.appendChild(pTxt);

      // notas
      const notasDiv = document.createElement('div'); notasDiv.style.marginTop = '6px';
      SCORES_VALIDOS.forEach(sc => {
        const checked = editData && editData.respostas?.[q.id]?.nota === sc ? 'checked' : '';
        notasDiv.innerHTML += `
          <label class="nota-chip">
            <input name="${q.id}" type="radio" value="${sc}" ${checked} onchange="Form._onNotaChange('${q.id}', ${sc})" />
            <span>${sc}</span>
          </label>
          <span class="obs-text">(${EXPLICACOES_NOTA[sc] || ''})</span><br/>
        `;
      });
      inner.appendChild(notasDiv);

      // evidências
      const evVal = editData?.respostas?.[q.id]?.evidencia || '';
      const ev = document.createElement('div'); ev.className = 'input-field';
      ev.innerHTML = `
        <textarea id="ev_${q.id}" class="materialize-textarea" placeholder="Descreva evidências (observações, fotos, registros)...">${evVal}</textarea>
        <label class="active" for="ev_${q.id}">Evidências</label>
      `;
      inner.appendChild(ev);

      // ação corretiva
      const notaVal = editData?.respostas?.[q.id]?.nota || '';
      const precisa = notaVal !== '' && Number(notaVal) < 6;
      const acData = editData?.respostas?.[q.id]?.acao || {};
      const ac = document.createElement('div'); ac.id = `acao_${q.id}`; ac.className = 'acao-box';
      ac.style.display = precisa ? 'block' : 'none';
      ac.innerHTML = `
        <strong>Ação corretiva (obrigatória pois a nota &lt; 6):</strong>
        <div class="row" style="margin-bottom:0;">
          <div class="input-field col s12">
            <input id="ac_desc_${q.id}" type="text" value="${acData.descricao || ''}">
            <label class="active" for="ac_desc_${q.id}">Descrição da ação</label>
          </div>
          <div class="input-field col s12 m6">
            <input id="ac_resp_${q.id}" type="text" value="${acData.responsavel || ''}">
            <label class="active" for="ac_resp_${q.id}">Responsável</label>
          </div>
          <div class="input-field col s12 m4">
            <input id="ac_prazo_${q.id}" type="date" value="${acData.prazoISO || ''}">
            <label class="active" for="ac_prazo_${q.id}">Prazo</label>
          </div>
          <div class="input-field col s12 m2">
            <select id="ac_status_${q.id}">
              <option value="Aberta" ${acData.status==='Aberta'?'selected':''}>Aberta</option>
              <option value="Em andamento" ${acData.status==='Em andamento'?'selected':''}>Em andamento</option>
              <option value="Concluída" ${acData.status==='Concluída'?'selected':''}>Concluída</option>
            </select>
            <label>Status</label>
          </div>
        </div>
      `;
      inner.appendChild(ac);

      card.appendChild(inner);
      container.appendChild(card);
    });

    // Inicializa selects (status)
    setTimeout(() => M.AutoInit(), 50);
  },

  _onNotaChange(qid, val) {
    const ac = Utils.byId(`acao_${qid}`);
    if (Number(val) < 6) ac.style.display = 'block'; else ac.style.display = 'none';
  },

  getRespostas() {
    const respostas = {};
    QUESTOES_5S.forEach(q => {
      const sel = document.querySelector(`input[name="${q.id}"]:checked`);
      const nota = sel ? Number(sel.value) : '';
      const evidencia = (Utils.byId(`ev_${q.id}`)?.value || '').trim();
      const precisaAcao = nota !== '' && nota < 6;
      let acao = null;
      if (precisaAcao) {
        acao = {
          descricao: (Utils.byId(`ac_desc_${q.id}`)?.value || '').trim(),
          responsavel: (Utils.byId(`ac_resp_${q.id}`)?.value || '').trim(),
          prazoISO: (Utils.byId(`ac_prazo_${q.id}`)?.value || '').trim(),
          status: (Utils.byId(`ac_status_${q.id}`)?.value || 'Aberta')
        };
      }
      respostas[q.id] = { nota, evidencia, acao };
    });
    return respostas;
  },

  validarRespostas(resp) {
    for (const q of QUESTOES_5S) {
      const r = resp[q.id];
      if (r.nota === '' || isNaN(r.nota)) { Utils.toast(`Responda a ${q.id}`); return false; }
      if (Number(r.nota) < 6) {
        const a = r.acao || {};
        if (!a.descricao || !a.responsavel || !a.prazoISO) {
          Utils.toast(`Ação corretiva obrigatória em ${q.id} (nota < 6)`); return false;
        }
      }
    }
    return true;
  },

  salvarRascunho() {
    const draft = { meta: CURRENT_DRAFT_META, respostas: Form.getRespostas() };
    const drafts = Utils.loadDrafts();
    const idx = drafts.findIndex(d => d.meta.id === draft.meta.id);
    if (idx >= 0) drafts[idx] = draft; else drafts.push(draft);
    Utils.saveDrafts(drafts);
    Utils.toast('Rascunho salvo localmente');
  },

  finalizar() {
    const respostas = Form.getRespostas();
    if (!Form.validarRespostas(respostas)) return;

    // Médias por senso + nota total
    const soma = { Seleção:0, Ordenação:0, Limpeza:0, Padronização:0, Autodisciplina:0 };
    const cnt  = { Seleção:0, Ordenação:0, Limpeza:0, Padronização:0, Autodisciplina:0 };
    QUESTOES_5S.forEach(q => { const n = Number(respostas[q.id].nota || 0); soma[q.senso]+=n; cnt[q.senso]+=1; });
    const medias = {}; Object.keys(soma).forEach(s => medias[s] = Utils.round2(soma[s] / Math.max(1, cnt[s])));
    const notaTotal = Utils.round2((medias['Seleção'] + medias['Ordenação'] + medias['Limpeza'] + medias['Padronização'] + medias['Autodisciplina']) / 5);

    // Grava localmente
    const subs = Utils.loadSubmissions();
    const isEdit = !!(CURRENT_DRAFT_META && CURRENT_DRAFT_META.editId);
    const id = isEdit ? CURRENT_DRAFT_META.editId : ('R' + Date.now());
    const d = new Date(CURRENT_DRAFT_META.dataISO);
    const rec = {
      id,
      meta: { ...CURRENT_DRAFT_META },
      respostas,
      calculos: { medias, notaTotal },
      Ano: d.getFullYear(),
      Mes: d.getMonth() + 1
    };

    if (isEdit) {
      const i = subs.findIndex(x => x.id === id);
      if (i >= 0) subs[i] = rec; else subs.push(rec);
    } else {
      subs.push(rec);
    }
    Utils.saveSubmissions(subs);

    // remove rascunho
    const drafts = Utils.loadDrafts().filter(d => d.meta.id !== CURRENT_DRAFT_META.id);
    Utils.saveDrafts(drafts);

    Utils.toast('Registro salvo.');
    location.hash = '#dash';
    Dash.atualizar();
  }
};

// ================== DASHBOARD ==================
const Dash = {
  atualizar() {
    const filtro = {
      setor: Utils.byId('filtroSetor').value || '',
      mes: Utils.byId('filtroMes').value || '',
      ano: Utils.byId('filtroAno').value || ''
    };
    const registros = Dash.filtrarRegistros(filtro);
    Dash.renderBarras(registros);
    Dash.renderRadar(registros, filtro.setor);
    Dash.listarRegistros(registros);
    Dash.listarAcoes(registros);
  },

  filtrarRegistros(f) {
    const subs = Utils.loadSubmissions();
    return subs.filter(r => {
      if (f.setor && r.meta.setor !== f.setor) return false;
      if (f.ano && Number(r.Ano) !== Number(f.ano)) return false;
      if (f.mes && Number(r.Mes) !== Number(f.mes)) return false;
      return true;
    });
  },

  renderBarras(registros) {
    const ctx = Utils.byId('chartBarras').getContext('2d');
    if (barrasChart) barrasChart.destroy();

    // média por setor
    const map = {};
    registros.forEach(r => { (map[r.meta.setor] ||= []).push(r.calculos.notaTotal); });
    const barras = Object.keys(map).map(s => ({
      setor: s,
      nota: Utils.round2(map[s].reduce((a,b)=>a+b,0)/map[s].length)
    })).sort((a,b)=> b.nota - a.nota);

    barrasChart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: barras.map(b => b.setor),
        datasets: [{
          label: 'Nota média por Setor',
          data: barras.map(b => b.nota),
          backgroundColor: 'rgba(33, 150, 243, 0.6)'
        }]
      },
      options: {
        responsive: true,
        scales: { y: { beginAtZero: true, suggestedMax: 10 } }
      }
    });
  },

  renderRadar(registros, setorSel) {
    const ctx = Utils.byId('chartRadar').getContext('2d');
    if (radarChart) radarChart.destroy();

    // média geral por senso
    const somaG = { Seleção:0, Ordenação:0, Limpeza:0, Padronização:0, Autodisciplina:0 };
    let countG = 0;
    registros.forEach(r => { Object.keys(somaG).forEach(k => somaG[k] += Number(r.calculos.medias[k] || 0)); countG++; });
    const geral = countG ? Object.keys(somaG).map(k => Utils.round2(somaG[k]/countG)) : [];

    // média do setor selecionado (se houver filtro)
    let setor = [];
    if (setorSel) {
      const rSetor = registros.filter(r => r.meta.setor === setorSel);
      if (rSetor.length) {
        const s = { Seleção:0, Ordenação:0, Limpeza:0, Padronização:0, Autodisciplina:0 };
        rSetor.forEach(r => Object.keys(s).forEach(k => s[k]+=Number(r.calculos.medias[k]||0)));
        setor = Object.keys(s).map(k => Utils.round2(s[k]/rSetor.length));
      }
    }

    const desejada = [MEDIA_DESEJADA,MEDIA_DESEJADA,MEDIA_DESEJADA,MEDIA_DESEJADA,MEDIA_DESEJADA];
    const labels = ['Seleção','Ordenação','Limpeza','Padronização','Autodisciplina'];


// questions.js — Avaliação 5S v1.2025 (transcrito do seu arquivo)

// Explicações oficiais das notas
const EXPLICACOES_NOTA = {
  10: 'Totalmente conforme ao requisito do senso.',
  8:  'Quase conforme, pequenas oportunidades de melhoria.',
  6:  'Atende parcialmente, há desvios importantes a corrigir.',
  4:  'Longe do esperado, requer ação imediata.',
  2:  'Não conforme/ausente, requer ação corretiva abrangente.'
};

// Lista de setores (aba “Notas gerais”)
const LISTA_SETORES = [
  'ADM FILIAL',
  'AMOXARIFADO',
  'CONFORMAÇÃO',
  'ESPAÇO MAKER',
  'ESTOQUE',
  'INJEÇÃO',
  'MANUTENÇÃO',
  'MATRIZARIA',
  'MONTAGEM',
  'MONTAGEM 2 (IMPETUS E CM GERADOR)',
  'PPCP',
  'QUALIDADE',
  'RECEBIMENTO',
  'SALA DE ELETRONICOS'
];

// Meta desejada (aba “Avaliação” → Meta 8,0)
const MEDIA_DESEJADA = 8;

// Notas válidas
const SCORES_VALIDOS = [10, 8, 6, 4, 2];

// 26 perguntas oficiais (aba “Avaliação”)
const QUESTOES_5S = [
  // 1 - Seleção
  { id:'Q1',  senso:'Seleção', texto:'Todas as ferramentas, dispositivos de medição e outros equipamentos são necessários para o trabalho diário' },
  { id:'Q2',  senso:'Seleção', texto:'Existem itens duplicados sobre a bancada de trabalho? ( ex.: ferramentas de trabalho, estiletes, estopas, etc...)' },
  { id:'Q3',  senso:'Seleção', texto:'As ferramentas de trabalho são usadas e acondicionadas corretamente? "Um lugar para cada coisa, cada coisa no seu lugar".' },
  { id:'Q4',  senso:'Seleção', texto:'Os quadros de gestão a vista / check list comunicação entre turnos / documentos necessários para a execução das atividades da área estão disponíveis, atualizados e em locais apropriados?' },
  { id:'Q5',  senso:'Seleção', texto:'Todos os avisos e quadros informativos atuais são necessários? (catálogos, armários, folhas de ajuste, pedidos, cartões de ponto,...)' },

  // 2 - Ordenação
  { id:'Q6',  senso:'Ordenação', texto:'Os locais de armazenamento para paletes, caixas, carrinhos de transporte, etc...estão marcados?' },
  { id:'Q7',  senso:'Ordenação', texto:'As linhas e marcações necessárias existem e são claramente visíveis? Caminhos - locais de armazenamento - área restrita' },
  { id:'Q8',  senso:'Ordenação', texto:'Todas as prateleiras, armários e equipamentos nesta área estão marcados com etiquetas?' },
  { id:'Q9',  senso:'Ordenação', texto:'O armazenamento de aparelhos, ferramentas e materiais de limpeza é bem organizado? É visível onde as ferramentas, gabaritos e acessórios são colocados?' },
  { id:'Q10', senso:'Ordenação', texto:'Existem marcas de identificação para identificar departamentos, pessoal responsável, etc.?' },
  { id:'Q11', senso:'Ordenação', texto:'Os objetos de uso pessoal estão guardados em local apropriado?' },

  // 3 - Limpeza
  { id:'Q12', senso:'Limpeza', texto:'O local de trabalho é limpo? Bancadas, máquinas, etc...' },
  { id:'Q13', senso:'Limpeza', texto:'Piso e calçadas estão livres de lascas, líquidos, materiais, resíduos?' },
  { id:'Q14', senso:'Limpeza', texto:'Equipamentos e ferramentas de trabalho, gabaritos e acessórios em uso estão limpos e em perfeitas condições técnicas? por ex. cabos, cárteres, dispositivos de segurança de carga, ….' },
  { id:'Q15', senso:'Limpeza', texto:'As instalações dos funcionários (cantos de informação, salas de descanso) são arrumadas, limpas e agradáveis' },
  { id:'Q16', senso:'Limpeza', texto:'A coleta seletiva do lixo está de acordo com o padrão? Existem coletores identificados e em bom estado?' },

  // 4 - Padronização
  { id:'Q17', senso:'Padronização', texto:'A documentação e os indicadores da área estão atualizados? No local as identificações, etiquetas, estão no padrão estabelicido?' },
  { id:'Q18', senso:'Padronização', texto:'Aspectos ergonômicos relevantes são atendidos, excesso de peso, postura, esforço físico, bem como a iluminação e os ambiente frequentados são favoráveis à saúde?' },

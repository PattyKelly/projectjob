# ✅ CHECKLIST COMPLETO POR MENU — Sherlock Holmes Monitor de Acessos
> **Versão:** v6.4 Alpha | **Data:** 09/05/2026 | **Mantido por:** Patty

---

## 🔗 LINKS DE ACESSO ESSENCIAIS

| Recurso | Link | Observação |
|---|---|---|
| **AppScript (Backend)** | [Abrir Editor GAS](https://script.google.com/u/0/home/projects/18ekpNGuXhBpFnZ62k8I6dM4iGCr3BlIpTYOK8RHyvxsy0NCGHSrTikk4/edit) | Editor atual do projeto |
| **Gerenciador de Acessos Franquia** | [Abrir App](https://script.google.com/a/macros/grupoboticario.com.br/s/AKfycbxcvKo-M2tq1-kS4YtHwdys6nls-ObcKFhY-4ykRwBaIHd6zm6CaS9vC_rCnp8WDP30pg/exec) | Web App publicado |
| **Pasta Resumo Drive (shard data)** | ID: `1tIqgxus9VmC3rEaCj9z7JB1rmsahfJ_a` | Fonte dos JSON/CSV |
| **Planilha Auditoria (BQ CI)** | ID: `1cJjAOlnVn1sUxq36Bhvay_WCCPF4jKo7KNFtQBYXNNs` | Sheets principal |

---

## 🗂️ ESTRUTURA DE MENUS

| # | Menu | ID da Tab | Acesso |
|---|---|---|---|
| 1 | Dashboard | `tabOverview` | Padrão (todos) |
| 2 | Analytics | `tabAnalytics` | Padrão (todos) |
| 3 | Comprovante PDF | `tabOnePage` | Padrão (todos) |
| 4 | FAQ & Ajuda | `tabFaq` | Padrão (todos) |
| 5 | Tech & Diagnóstico | `tabTech` | Padrão (todos) |
| 6 | Gerenciador de Acessos Franquia | — (abre nova janela) | Padrão (todos) |
| 7 | Logs Admin | `tabAdmin` | Oculto — só admins |

---

---

## 1️⃣ MENU: DASHBOARD (`tabOverview`)

> **Propósito:** Visão executiva em tempo real. É a home do sistema.

---

### 1.1 — TELA DE LOGIN / LANDING PAGE

| Campo / Elemento | O que deve ter | Status |
|---|---|---|
| Logo animado (SVG pulsante) | Exibido com animação `pulse` | ☐ |
| Título | `Torre Controle _ Monitor de Acessos` | ☐ |
| Subtítulo rosa | `Sherlock H. _ Pegadas de franquias nos Reports` | ☐ |
| Bloco de autenticação | Mostra `Usuário Identificado:` + e-mail da sessão Google | ☐ |
| Spinner de autenticação | Exibido enquanto autentica (`Autenticando...`) | ☐ |
| Botão `Acessar Plataforma` | Só aparece quando e-mail for carregado (classe `.ready`) | ☐ |
| Background animado (chuva) | `#loginRain` com animação `rainAni` infinita | ☐ |
| Efeito blur no fundo | `backdrop-filter: blur(20px)` sobre a chuva | ☐ |

---

### 1.2 — HEADER GLOBAL (visível em todas as abas)

| Campo / Elemento | O que deve ter | Status |
|---|---|---|
| Logo no canto esquerdo | Ícone gradiente azul→rosa + texto "Sherlock Holmes" + subtítulo "Monitor de Acessos" | ☐ |
| Nav central | 6 botões de menu (Dashboard, Analytics, Comprovante PDF, FAQ, Tech, Gerenciador) | ☐ |
| Botão ativo | Classe `.active` = fundo branco, texto escuro | ☐ |
| Sino de notificação 🔔 | Com bolinha rosa indicando notificação | ☐ |
| Perfil do usuário | Avatar roxo + nome do usuário (`id="userProfileName"`) | ☐ |
| Botão Logs Admin | Oculto por padrão; aparece via `display:none` → JS habilita para admins | ☐ |

---

### 1.3 — BRIEFING EXECUTIVO (Topo da Dashboard)

| Campo / Elemento | Cálculo / Fonte | Status |
|---|---|---|
| Título "Briefing executivo Sherlock" | Estático | ☐ |
| Data hoje (`id="overviewToday"`) | Formatada pelo JS no carregamento | ☐ |
| **Acessos/ View - Acumulado** (`id="welcomeAuditTotal"`) | `stats.historyTotal` — total bruto de linhas auditadas desde implantação | ☐ |
| **Disponibilizações** (`id="welcomeDispoTotal"`) | `stats.dispoTotal` — regra teórica: `dias do período × 2 reports × total CPs da carteira FVC vigente` (2025/2026); fallback para contagem de linhas com RESPONSÁVEL = "wojciechowski" fora desse escopo | ☐ |
| **CPs impactados** (`id="welcomeCpTotal"`) | `stats.uniqueCps` — contagem de CPs distintos com pelo menos 1 acesso | ☐ |
| **Identificação Sherlock** (`id="welcomeQualityTotal"`) | `71,55%` — calculado no ETL Python: `(título reconhecido + cache + Organizze) / total eventos` | ☐ |
| Bloco "Última Carga DB" (`id="lastSync"`) | Timestamp do snapshot mais recente do Drive | ☐ |
| Botão "Carga do Sistema" (expansível) | Toggle mostra/oculta detalhes da carga (`id="lastUpdate"`) | ☐ |
| Nota sobre carga manual | Texto fixo explicando omissões por janela temporal | ☐ |

---

### 1.4 — CARDS DE DECISÃO (KPI Grid — 4 cards)

| Card | ID | Label | Cálculo | Cor |
|---|---|---|---|---|
| Card 1 | `bigTotal` | Acessos Validados | Total de linhas filtradas e validadas no período atual | Azul |
| Card 2 | `k1` | Acessos no Período | Recorte do filtro aplicado (mês/data selecionada) | Rosa |
| Card 3 | `k2` | Franquias Engajadas | Contagem de CPs distintos com ≥ 1 acesso no período | Verde |
| Card 4 | `k3` | Disponibilizações | Mesma lógica de `welcomeDispoTotal` — regra teórica por período | Laranja |

**Regras de cálculo detalhadas:**

```
bigTotal    = COUNT(registros com e-mail válido de franquia, excluindo sistema/admin)
              ⚠️ BUG REPORTADO: valor de ~34k sumiu — investigar se o multiplicador
              foi removido ou se o filtro de exclusão de e-mail está bloqueando registros
k1          = COUNT(registros dentro do filtro de período ativo)
k2          = COUNT(DISTINCT cod_cp WHERE acessos >= 1 no período)
k3 (2026)   = dias_no_período × multiplicador_temporal × total_cps_carteira_fvc_vigente
              ⚠️ Multiplicador: ×2 até 04/05/2026, ×4 a partir de 05/05/2026 (ver Bloco 1)
k3 (2025)   = mesma fórmula estendida para 2025 (multiplicador ×2 fixo para todo 2025)
k3 (outros) = COUNT(linhas WHERE responsável CONTÉM "wojciechowski")  ← fallback legado
```

---

> ### 🔴 BUG 1 — `bigTotal` não exibe mais ~34k de acessos
>
> **Sintoma:** O card `bigTotal` (Acessos Validados) parou de exibir o total acumulado esperado (~34.000). Suspeita: o multiplicador foi removido acidentalmente, ou a lógica de exclusão de e-mail está filtrando registros legítimos.
>
> **O que investigar no backend (`Code.gs`):**
> - [ ] Confirmar que a função que alimenta `stats.audit` / `bigTotal` **ainda aplica o multiplicador** (×2 por e-mail único por CP/dia)
> - [ ] Verificar se houve mudança recente no filtro de exclusão de e-mails que possa estar descartando registros válidos de franquia
> - [ ] Logar o count antes e depois do multiplicador para comparar: `Logger.log('antes: ' + rawCount + ' | depois: ' + auditCount)`
> - [ ] Confirmar que `renderData()` no frontend ainda atribui `stats.audit` ao elemento `#bigTotal`
> - [ ] Valor esperado de referência: **~34.000** acessos acumulados

---

> ### 🔴 BUG 2 — E-mail `patricia.wojciechowski@grupoboticario.com.br` sumiu do campo Disponibilizações
>
> **Regra EXATA de uso deste e-mail:**
>
> | Campo | Deve aparecer? | Regra |
> |---|---|---|
> | **`bigTotal` / `k1` / Audit Log Explorer** | ❌ **NÃO** | Este e-mail é interno/admin — deve ser **excluído** de toda contagem de acessos de franquias e da tabela |
> | **`k3` / `welcomeDispoTotal` / `anTotalDispo`** — Disponibilizações | ✅ **SIM** | As linhas deste e-mail representam os **envios do relatório** (disponibilizações reais) — devem ser **contadas** como disponibilizações |
>
> **Lógica exata para Disponibilizações (fallback / outros anos):**
> ```
> k3 (fallback) = COUNT(linhas WHERE e-mail = 'patricia.wojciechowski@grupoboticario.com.br')
> ```
> Esta contagem representa a quantidade de arquivos/relatórios disponibilizados no Drive para as franquias.
> A soma de 2025 + 2026 forma o total de disponibilizações históricas.
>
> **O que implementar/corrigir no backend (`Code.gs`):**
> - [ ] Garantir que `patricia.wojciechowski@grupoboticario.com.br` está na lista de **exclusão** do `bigTotal`, `k1`, `k2` e da tabela `Audit Log Explorer`
> - [ ] Garantir que o mesmo e-mail está na lógica de **inclusão** do cálculo de `k3`/`stats.dispoTotal` (Disponibilizações)
> - [ ] A contagem de disponibilizações é por **linha no arquivo** (cada linha = 1 disponibilização), não por dia ou CP distinto
> - [ ] Os dados vêm do backend (`getConsolidatedData()`) — o frontend não recalcula, apenas exibe `stats.dispoTotal`
> - [ ] Após correção, validar que `k3` volta a exibir o total correto de disponibilizações (2025 + 2026)

---

| Checklist | Status |
|---|---|
| Todos os 4 cards renderizando com valores numéricos | ☐ |
| `bigTotal` exibe ~34k (verificar multiplicador e filtro de exclusão) | ☐ |
| `k3` inclui linhas do e-mail `patricia.wojciechowski@...` (disponibilizações) | ☐ |
| `patricia.wojciechowski@...` **ausente** da tabela Audit Log Explorer | ☐ |
| Fallback para `0` quando dado não existir (não exibir `-` ou `NaN`) | ☐ |
| Formatação pt-BR (separador de milhar: ponto, decimal: vírgula) | ☐ |
| Tags de status ("Live", "Período", "Stable", "GB") visíveis | ☐ |

---

### 1.5 — GRÁFICO DE TENDÊNCIA TEMPORAL

| Campo / Elemento | Detalhe | Status |
|---|---|---|
| Canvas `id="trendChart"` | Gráfico de linha (Chart.js) | ☐ |
| Eixo X | Datas/períodos agrupados | ☐ |
| Eixo Y | Contagem de acessos | ☐ |
| Cor da linha | Gradiente azul `#264FEC` → rosa `#FF5EC9` | ☐ |
| Botão "Force Refresh" (`id="btnSync"`) | Oculto por padrão; aparece para admins | ☐ |
| Altura do container | `310px` fixo | ☐ |

---

### 1.6 — BARRA DE FILTROS

| Campo | ID | Tipo | Comportamento |
|---|---|---|---|
| Seletor de Mês | `periodFilter` | `<select>` | Filtra por mês/ano; options de 2025 e 2026 disponíveis | 
| Campo busca CP | `cpSearch` | `<input>` + `<datalist id="cpList">` | Autocomplete com CPs; aciona `runFiltersDebounced()` |
| Filtro Tipo | `typeFilter` | `<select>` (oculto) | Oculto por padrão; aparece se houver tipos distintos |
| Data Inicial | `dateFrom` | `<input type="date">` | Filtra a partir desta data |
| Data Final | `dateTo` | `<input type="date">` | Filtra até esta data |
| Botão Localizar | `btnRefresh` | `<button>` | Aciona `runFilters()` |
| Botão Limpar | — | `<button>` | Aciona `clearFilters()` — reseta todos os campos |

**Checklist de filtros:**

| Item | Status |
|---|---|
| Seletor de mês com optgroup separado por ano (2025 / 2026) | ☐ |
| Meses de 2025 disponíveis no dropdown | ☐ |
| Autocomplete de CP funciona com nome ou código | ☐ |
| Filtro por data "De/Até" funciona isolado ou combinado com mês | ☐ |
| Botão Limpar reseta tabela para estado original | ☐ |
| Label de filtro ativo exibido ao lado de "Audit Log Explorer" (`id="filterLabel"`) | ☐ |

---

### 1.7 — TABELA PRINCIPAL (Audit Log Explorer)

| Campo | Coluna | Detalhe |
|---|---|---|
| DATA | 1ª coluna | Formato `dd/mm/aaaa` |
| TIPO | 2ª coluna | CI, CA ou Consolidado (forma como foi auditado) |
| COD CP | 3ª coluna | Código numérico da franquia |
| NOME CP | 4ª coluna | Nome da cidade/franquia |

**Paginação:**

| Elemento | ID | Comportamento |
|---|---|---|
| Info paginação | `tablePaginationInfo` | "Exibindo X–Y de Z registros" |
| Botão Anterior | `pagePrevBtn` | `setTablePage(page - 1)` |
| Indicador de página | `pageIndicator` | "Página X de Y" |
| Botão Próxima | `pageNextBtn` | `setTablePage(page + 1)` |
| Tamanho da página | `TABLE_PAGE_SIZE` | 50 registros por página |

**Funcionalidades da tabela:**

| Item | Status |
|---|---|
| Header fixo (sticky) ao rolar | ☐ |
| Hover suave nas linhas | ☐ |
| Clique na linha abre Modal do CP | ☐ |
| Botão "Download CSV" exporta dados filtrados | ☐ |
| Estado vazio exibe mensagem de carregamento | ☐ |
| Paginação aparece somente quando há resultados (`display:none` inicial) | ☐ |

---

### 1.8 — RANKINGS LATERAIS

#### Engajados 🏆 (CPs com maior acesso)

| Campo | ID | Detalhe |
|---|---|---|
| Lista de ranking | `topList` | Itens `.top-item` com posição, nome e contagem |
| Página anterior | `topPrevBtn` | `setRankPage('top', page - 1)` |
| Indicador | `topPageIndicator` | "Página X de Y" |
| Próxima página | `topNextBtn` | `setRankPage('top', page + 1)` |
| Tamanho da página | `RANK_PAGE_SIZE` | 5 CPs por página |

#### Baixo Acesso ⚠️ (CPs com menor acesso)

| Campo | ID | Detalhe |
|---|---|---|
| Lista de ranking | `bottomList` | Mesma estrutura visual |
| Paginação | `bottomPrevBtn` / `bottomNextBtn` / `bottomPageIndicator` | Igual ao ranking de engajados |
| Critério | — | CPs com menor frequência de acessos consolidados |

**Nota de cálculo:** Ambos os rankings usam **Acessos Totais Consolidado** (CI + CA somados por CP).

| Checklist Rankings | Status |
|---|---|
| Exibe posição (#1, #2...) com bolinha numerada | ☐ |
| Exibe nome do CP e contagem total | ☐ |
| Paginação funcional com estado disabled nos extremos | ☐ |
| Clique no item abre modal do CP | ☐ |

---

### 1.9 — MODAL DO CP (ao clicar na tabela ou ranking)

| Campo | ID | Detalhe |
|---|---|---|
| Nome do CP | `modalNome` | Nome completo da franquia |
| Código do CP | `modalCod` | Código numérico |
| Views Totais | `modalTotal` | Total de acessos do CP |
| Frequência Semanal | `modalDatas` | Quantidade de semanas distintas com acesso |
| Gráfico de linha | `modalChart` | Histórico de acessos do CP (Canvas 200px) |
| Tabela detalhe | `modalBody` | DATA / TIPO / REGISTRO (e-mail ofuscado 25% para não-admins) |
| Botão Imprimir PDF | — | Aciona `printComprovante()` |
| Aviso de redirecionamento | — | Texto em amarelo/vermelho orientando a usar o menu "Comprovante PDF" |

| Checklist Modal | Status |
|---|---|
| Abre sem erros de script | ☐ |
| E-mail ofuscado para usuários não-admin | ☐ |
| Gráfico renderiza com histórico real do CP | ☐ |
| Botão fechar (✕) funcional | ☐ |
| Botão "Imprimir Comprovante PDF" funcional | ☐ |

---

---

## 2️⃣ MENU: ANALYTICS (`tabAnalytics`)

> **Propósito:** Análise aprofundada por tipo, franquia e tendência.

---

### 2.1 — HEADER E SUBTABS

| Sub-aba | Função |
|---|---|
| **Resumo** | Visão geral dos dados carregados |
| **Por Tipo** | Distribuição CI vs CA |
| **Franquias** | Top CPs por volume |
| **Tendência** | Evolução temporal + projeção |

| Botão de período | Comportamento |
|---|---|
| Semanal | Filtra e agrupa por semana |
| Mensal | Agrupa por mês (padrão) |
| Anual | Agrupa por ano |

---

### 2.2 — GRÁFICO PRINCIPAL (`anMainChart`)

| Campo | ID | Detalhe |
|---|---|---|
| Título dinâmico | `anChartTitle` | Ex: "Views por Mês — 2026" |
| Legenda Report CA | Ponto azul `#264FEC` | |
| Legenda Report CI | Ponto rosa `#FF5EC9` | |
| Canvas | `anMainChart` | Gráfico de barras empilhadas, altura `260px` |
| Estado vazio | `anChartEmpty` | Mensagem quando não há dados carregados |

---

### 2.3 — KPIs LATERAIS (Analytics)

| KPI | ID | Cálculo | Badge |
|---|---|---|---|
| **Total de Views** | `anTotalAcessos` | Total de acessos no período selecionado | `anBadgeAudit` — exibe % auditados |
| **CPs Engajados** | `anTotalCPs` | Distinct CPs com acesso | `anBadgeCps` — "Franquias" |
| **Disponibilizações (GB)** | `anTotalDispo` | Mesma regra do Dashboard (teórico ou real) | `anBadgeDispo` — "Entregas" |

---

### 2.4 — CARDS INFERIORES (3 cartões)

#### Cartão 1 — Visão por Franquia

| Campo | ID | Detalhe |
|---|---|---|
| Big Number total | `anBigKpiTotal` | Total de views no período |
| Label "views totais no período" | — | Fixo |
| Meta (Top X CPs) | `anFranchiseMeta` | "Top 5 CPs" dinâmico |
| Barras horizontais | `anFranchiseBars` | CP → barra de progresso proporcional + valor |

**Cálculo das barras:** `(acessos_cp / total_acessos) × 100%` para largura proporcional.

#### Cartão 2 — Análise por Tipo

| Campo | ID | Detalhe |
|---|---|---|
| Gráfico Donut | `anDonutChart` | CI vs CA em porcentagem |
| Total central | `anDonutTotal` | Número total no centro do donut |
| Meta total | `anTypeTotal` | "X total" |
| Legenda de tipos | `anTypeLegend` | Listagem CI / CA com % |

**Cálculo do donut:**
```
CI% = COUNT(tipo = "CI") / COUNT(total) × 100
CA% = COUNT(tipo = "CA") / COUNT(total) × 100
```

#### Cartão 3 — Previsão de Tendência

| Campo | ID | Detalhe |
|---|---|---|
| Gráfico de linha | `anForecastChart` | Linha histórica + projeção futura ponderada |
| Caixa de insight | `anInsightBox` | Texto gerado pelo JS com análise da tendência |
| Texto do insight | `anInsightText` | Ex: "Tendência de crescimento X% ao mês" |

**Cálculo da projeção:** Média ponderada dos últimos N períodos com peso crescente para dados mais recentes.

| Checklist Analytics | Status |
|---|---|
| Sub-abas alternam sem quebrar gráficos | ☐ |
| Gráfico principal exibe CA (azul) e CI (rosa) separados | ☐ |
| Botões de período (Semanal/Mensal/Anual) funcionam | ☐ |
| KPIs laterais atualizam junto com o gráfico | ☐ |
| Barras de franquia calculam proporção corretamente | ☐ |
| Donut exibe percentuais coerentes com total | ☐ |
| Projeção de tendência exibe linha futura | ☐ |
| Insight box exibe texto interpretativo | ☐ |
| Estado vazio (`anChartEmpty`) aparece sem dados | ☐ |

---

---

## 3️⃣ MENU: COMPROVANTE PDF (`tabOnePage`)

> **Propósito:** Geração de evidência de acesso por CP para uso em auditorias.

---

### 3.1 — FORMULÁRIO DE GERAÇÃO

| Campo | ID | Tipo | Detalhe |
|---|---|---|---|
| Código do CP | `opSearch` | `<input type="text">` | Ex: `53123` — obrigatório |
| Data inicial | `opDateFrom` | `<input type="date">` | Período inicial da evidência |
| Data final | `opDateTo` | `<input type="date">` | Período final da evidência |
| Botão gerar | — | `<button>` | Aciona `generateDirectOnePage()` |

### 3.2 — DOCUMENTO PDF GERADO

| Seção do PDF | Conteúdo |
|---|---|
| Header com logo | Título "Comprovante de Acesso" + nome do CP + período |
| KPIs do comprovante | Total de Acessos + Quantidade de Datas distintas |
| Tabela de registros | DATA / TIPO / REGISTRO (e-mail ofuscado 25% conforme LGPD) |
| Rodapé | Data/hora de geração + aviso de uso restrito |

### 3.3 — REGRAS DO COMPROVANTE

| Regra | Detalhe |
|---|---|
| E-mails ofuscados | 25% dos caracteres mascarados → conformidade LGPD + evitar DLP |
| 1 Acesso = 1 arquivo | Report CI **ou** Report CA separados |
| 1 Visita Drive = 2 acessos | Cada Drive tem CI + CA |
| Retenção de dados | Precisão garantida nas últimas 24 semanas |

| Checklist Comprovante PDF | Status |
|---|---|
| Campo CP obrigatório valida antes de gerar | ☐ |
| Loading overlay aparece durante busca | ☐ |
| Alert se CP não tiver registros no período | ☐ |
| PDF gerado com formatação limpa (Arial, A4) | ☐ |
| E-mails ofuscados no PDF | ☐ |
| Período exibido corretamente no PDF | ☐ |
| Data/hora de geração no rodapé | ☐ |
| Função `window.print()` abre diálogo de impressão | ☐ |

---

---

## 4️⃣ MENU: FAQ & AJUDA (`tabFaq`)

> **Propósito:** Documentação inline — respostas para dúvidas das franquias e gestores.

---

### 4.1 — PERGUNTAS DOCUMENTADAS

| # | Pergunta | Resposta Resumida |
|---|---|---|
| Q1 | Política de Retenção de Dados | Logs precisos das **últimas 24 semanas**; anteriores podem estar fragmentados |
| Q2 | Privacidade e LGPD | E-mails ofuscados em **25%** nos comprovantes |
| Q3 | O que é o Volume Histórico? | Bruto acumulado de todos engajamentos desde 2024 |
| Q4 | O que significa "Franquias Engajadas"? | CPs distintos com ≥ 1 acesso (conta 1 vez independente da frequência) |
| Q5 | O que é 1 Acesso? | 1 arquivo (CI ou CA); 1 visita ao Drive = **2 acessos** |
| Q6 | Temos tabela produtiva? | Não. Consumo apenas pela camada da ferramenta (acesso restrito) |
| Q7 | Histórico vs Auditoria | Histórico = todas linhas brutas; Auditoria = subconjunto validado |
| Q8 | Como filtrar por período ou CP? | Filtros no Dashboard (De/Até, busca CP) |
| Q9 | Como os KPIs são calculados? | (detalhado abaixo) |

### 4.2 — DETALHAMENTO DOS CÁLCULOS (Q9)

| KPI | Fórmula |
|---|---|
| **Disponibilizações históricas** | `COUNT(linhas WHERE RESPONSÁVEL CONTÉM "wojciechowski")` — sem somar QUANTIDADE DE ACESSOS |
| **Acumulado Estratégico (2025/2026)** | Soma por faixa de datas no trend diário; para CI+CA aplica fator **×2** |
| **Resumo anual (linhas CI+CA)** | Soma das linhas auditadas no período anual, **sem multiplicador** |
| **Big Number 2026 (franquias)** | Recorte 2026, excluindo usuário logado + e-mails de sistema + administrativos; validação por cadastro de e-mails da franquia quando disponível |

| Checklist FAQ | Status |
|---|---|
| Todos os cards expandem com texto completo | ☐ |
| Formatação clara com negrito nos termos técnicos | ☐ |
| Sem texto cortado ou overflow | ☐ |
| Link para Comprovante PDF mencionado no Q8 | ☐ |

---

---

## 5️⃣ MENU: TECH & DIAGNÓSTICO (`tabTech`)

> **Propósito:** Painel interno de qualidade do Sherlock — restrito operacionalmente, mas acessível na nav.

---

### 5.1 — PULSO SHERLOCK (Minigráfico)

| Campo | ID | Detalhe |
|---|---|---|
| Canvas mini tendência | `miniTrendChart` | Últimos 7 dias de acessos, altura `120px` |
| Saúde dos dados | — | Fixo: `99%` |
| Carga atual | — | Fixo: `v6.4 Alpha` |
| Latência | — | Fixo: `12ms` |
| Fonte | — | Fixo: `Drive + Cache` |

---

### 5.2 — QUALIDADE DA LEITURA SHERLOCK

**Base de cálculo:** 1.186.958 eventos processados na última execução Python (ETL v4.5)

| Métrica | ID | Valor Atual | Eventos |
|---|---|---|---|
| Título reconhecido | — | **61,03%** | 724.366 |
| Cache interno | — | **10,53%** | 124.926 |
| Organizze | — | **0,00%** | 14 |
| Sem identificação | — | **28,45%** | 337.652 |
| **TOTAL IDENTIFICADO** | — | **71,55%** | 849.306 |

**Barra visual (Quality Meter):**
```
[==========================][====][  ][============]
      61,03%               10,53% 0%    28,45%
      Azul                 Rosa   Verde  Laranja
```

**Fórmula:**
```
Taxa de identificação = (título_reconhecido + cache_interno + organizze) / total_eventos × 100
= (724.366 + 124.926 + 14) / 1.186.958 × 100
= 71,55%
```

---

### 5.3 — PRÓXIMO FOCO RECOMENDADO

| # | Foco | Detalhe |
|---|---|---|
| 1 | Atacar zona sem identificação | Priorizar eventos sem ID dos meses mais recentes |
| 2 | Reforçar cache interno | Transformar títulos recorrentes em novas regras/cache |
| 3 | Validar concentração por CP | Usar rankings para separar alta demanda real de lacunas de cadastro |

---

### 5.4 — DIAGNÓSTICO DE CONEXÃO GAS

| Botão | Função |
|---|---|
| **Testar GAS** | Verifica comunicação com o AppScript Backend |
| **Testar Dados** | Valida retorno de dados da função `getConsolidatedData()` |
| **Copiar Log** | Copia log de diagnóstico para clipboard |

| Checklist Tech & Diagnóstico | Status |
|---|---|
| Minigráfico renderiza sem erro | ☐ |
| Métricas de qualidade com valores corretos | ☐ |
| Barra visual proporcional às porcentagens | ☐ |
| Botões de diagnóstico funcionais | ☐ |
| Texto "Pronto para diagnóstico" aparece sem erros | ☐ |

---

---

## 6️⃣ MENU: GERENCIADOR DE ACESSOS FRANQUIA (janela externa)

> **Propósito:** Gestão dos e-mails e acessos das franquias — app separado.

| Campo | Detalhe |
|---|---|
| Como abre | `window.open(url, '_blank', 'noopener,noreferrer')` |
| URL | `https://script.google.com/a/macros/grupoboticario.com.br/s/AKfycbxcvKo-M2tq1-kS4YtHwdys6nls-ObcKFhY-4ykRwBaIHd6zm6CaS9vC_rCnp8WDP30pg/exec` |
| Segurança | `noopener,noreferrer` — sem acesso ao contexto pai |
| Escopo | App publicado separado do Sherlock principal |

| Checklist Gerenciador | Status |
|---|---|
| Abre em nova aba (não substitui a atual) | ☐ |
| Link funcional (app publicado ativo) | ☐ |
| Segurança `noopener,noreferrer` mantida | ☐ |

---

---

## 7️⃣ MENU: LOGS ADMIN (`tabAdmin`) — RESTRITO

> **Propósito:** Trace logs operacionais do servidor AppScript para debug interno (`[OK]`, `[WARN]`, `[INFO]`, `[ERRO]`). Ferramenta dev exclusiva — **não remover**.

> **✅ VALIDADO 09/05/2026 — Regra de acesso coerente e duplo bloqueio confirmado:**
> - **Frontend:** `btnAdminLogs` é `display:none` por padrão. JS exibe apenas para `devList` = `['patricia.wojciechowski@grupoboticario.com.br', 'opatriciakelly@gmail.com']`
> - **Backend:** `getAdminLogs()` possui bloqueio absoluto de servidor — valida `Session.getActiveUser()` contra o mesmo array `adminEmails`. Qualquer outro e-mail, mesmo via URL direta, recebe `{ error: "Acesso Negado..." }` antes de qualquer leitura.
> - Os dois arrays batem exatamente — **nenhuma brecha de acesso**.
> - Os logs são trace completo (não só erros): lê aba `Logs` da planilha central `1DZmRVe2KxMELsVrxQY1zN4iQ6_BkdsF96H-bmUXrdWE`, últimas 500 linhas, ordem decrescente.
> - **Decisão: MANTER o menu.** Nenhuma alteração necessária na lógica de acesso.

| Campo | ID | Detalhe |
|---|---|---|
| Visibilidade | `btnAdminLogs` | `display:none` por padrão — JS ativa para admins |
| Botão Refresh | — | Aciona `loadAdminLogs()` |
| Tabela de logs | `adminTable` + `adminBody` | TIMESTAMP / EXECUTOR / DETAILS / (col 4) |
| Scroll da tabela | — | `max-height: 500px; overflow-y: auto` |
| Proteção backend | — | Bloqueio em `getAdminLogs()` antes de qualquer leitura |

| Checklist Admin Logs | Status |
|---|---|
| Menu oculto para não-admins | ✅ validado |
| Botão visível apenas após verificação de permissão | ✅ validado |
| Duplo bloqueio frontend + backend com arrays idênticos | ✅ validado |
| Tabela carrega logs ao clicar "Refresh Trace" | ☐ |
| Colunas: TIMESTAMP / EXECUTOR / DETAILS | ☐ |

---

---

---

## 🔴 AJUSTES CRÍTICOS — 08/05/2026 (para a IA corrigir no código)

> **ATENÇÃO IA:** Esta seção contém as mudanças de lógica solicitadas em 08/05/2026 que precisam ser implementadas/validadas nos arquivos `Code.gs` (backend AppScript) e `index.html` (frontend). Os arquivos de referência com o código mais atual estão em `0805 Ajustes/`.  
> **Não altere o layout, CSS ou fluxo de login.** Apenas os pontos listados abaixo.

---

### 🔧 BLOCO 1 — Multiplicador Temporal de Disponibilizações (BACKEND `Code.gs`)

**O que mudar:** A fórmula de cálculo do `stats.dispo` (Disponibilizações) precisa respeitar um **ponto de corte temporal em 05/05/2026**.

| Regra | Período | Multiplicador |
|---|---|---|
| **Regra antiga** | Até 04/05/2026 | `dias_no_período × 2 × total_CPs_FVC` |
| **Regra nova** | A partir de 05/05/2026 | `dias_no_período × 4 × total_CPs_FVC` |
| **Regra híbrida (quando o filtro cobre os dois lados)** | Período que cruza 05/05/2026 | `(dias_antes × 2 + dias_depois × 4) × total_CPs_FVC` |

**Checklist de implementação:**

- [ ] Definir constante `THRESHOLD_DATE = '2026-05-05'` no backend
- [ ] Na função que calcula `dispo`, verificar se o período do filtro é:
  - Inteiramente antes de 05/05/2026 → usa multiplicador `2`
  - Inteiramente após 05/05/2026 → usa multiplicador `4`
  - Cruza 05/05/2026 → calcula proporcional: `diasAntes × 2 + diasDepois × 4`
- [ ] O resultado `dispo` deve refletir a regra correta no payload retornado por `getConsolidatedData()`
- [ ] O campo `stats.dispoRule` (metadado já existente) deve indicar qual regra foi aplicada: `"x2"`, `"x4"` ou `"hibrido_x2_x4"`
- [ ] Validar que o card `k3` (Disponibilizações) no Dashboard exibe o valor recalculado corretamente

---

### 🔧 BLOCO 2 — Unicidade por E-mail no Audit (BACKEND `Code.gs`)

**O que mudar:** O campo `stats.audit` (usado em `bigTotal`, `k1`, `welcomeAuditTotal`) deve contar **e-mails únicos por CP por dia**, multiplicado por `2` (representando CI + CA).

**Regra detalhada:**

```
Para cada (cp, data):
  emailsUnicos = COUNT(DISTINCT email WHERE cp = X AND data = Y)
  
auditTotal = SUM(emailsUnicos × 2)  ← multiplicador fixo x2 para o Big Number
```

**Importante:**
- O `×2` é **fixo** no Big Number principal (`stats.audit`) — não depende do threshold de 05/05
- Nos **detalhes por CP** (modal, tabela): manter lógica CI+CA real (contagem real de linhas)
- **Não** contar a mesma linha de log duplicada mais de uma vez por e-mail/CP/dia

**Checklist de implementação:**

- [ ] Na função `getConsolidatedData()`, substituir contagem bruta de linhas por contagem de e-mails únicos por (CP, dia)
- [ ] Após contar e-mails únicos, aplicar `× 2` antes de retornar `stats.audit`
- [ ] Garantir que `stats.audit2025` e `stats.audit2026` usem a mesma lógica de unicidade + x2
- [ ] O campo `stats.cps` (franquias engajadas) continua contando CPs distintos com ≥ 1 e-mail único (sem o x2)
- [ ] Os objetos `stats.top10` e `stats.bottom10` devem usar a contagem por e-mails únicos (propriedade `count` ou `v` no item)
- [ ] A propriedade `v` em cada item da `res.list` deve refletir os acessos validados com a nova contagem

---

### 🔧 BLOCO 3 — CPs SEM FVC no Bloco Welcome (BACKEND `Code.gs`)

**O que mudar:** O campo `stats.semFvcCount` deve ser preenchido com um valor **real**, replicando exatamente o filtro já aplicado manualmente na planilha de referência.

**Fonte de dados EXATA:**

| Campo | Valor |
|---|---|
| **Planilha de referência** | [CPs SEM FVC — Sheets](https://docs.google.com/spreadsheets/d/1m-FxjYDAUlAx4dyvKNqlL9xWtip1i9nTNVvp2DTJ5vU/edit?gid=0#gid=0) |
| **Spreadsheet ID** | `1m-FxjYDAUlAx4dyvKNqlL9xWtip1i9nTNVvp2DTJ5vU` |
| **Aba (gid=0)** | Primeira aba (índice 0) |
| **Coluna a filtrar** | **Coluna C** (índice 2 no array, base 0) |
| **Valor do filtro** | Célula contém `"SEM FVC"` (case-insensitive) |
| **Operação** | Contar linhas onde coluna C = `"SEM FVC"` |
| **Resultado esperado atual** | **40 linhas** (valor de referência em 09/05/2026) |

> ⚠️ **IMPORTANTE:** Esta planilha é a **fonte da verdade**. O backend deve ler esta planilha diretamente (não inferir o valor). O ID acima já é o Spreadsheet ID correto — não confundir com `PRODUCT_MAPPING_SS_ID` (planilha diferente).

**Lógica de leitura no Code.gs:**
```javascript
// Constante a adicionar no Code.gs:
const SEM_FVC_SS_ID = '1m-FxjYDAUlAx4dyvKNqlL9xWtip1i9nTNVvp2DTJ5vU';

// Função helper:
function getSemFvcCount() {
  try {
    var ss = SpreadsheetApp.openById(SEM_FVC_SS_ID);
    var sheet = ss.getSheets()[0]; // aba índice 0 (gid=0)
    var colC = sheet.getRange(2, 3, sheet.getLastRow() - 1, 1).getValues(); // coluna C, a partir linha 2
    return colC.filter(function(row) {
      return row[0].toString().toUpperCase().indexOf('SEM FVC') !== -1;
    }).length;
  } catch(e) {
    Logger.log('Erro getSemFvcCount: ' + e.message);
    return 0;
  }
}
```

**Checklist de implementação:**

- [ ] Adicionar constante `SEM_FVC_SS_ID = '1m-FxjYDAUlAx4dyvKNqlL9xWtip1i9nTNVvp2DTJ5vU'` no Code.gs
- [ ] Implementar função `getSemFvcCount()` conforme modelo acima
- [ ] Chamar `getSemFvcCount()` dentro de `getConsolidatedData()` e incluir no payload: `stats.semFvcCount`
- [ ] Validar que o resultado retorna **40** (valor de referência atual) — se retornar diferente, revisar a coluna/aba
- [ ] Se a leitura falhar, retornar `semFvcCount: 0` com log de erro (não quebrar o carregamento)
- [ ] No frontend (`index.html`), o ID `welcomeSemFvc` já existe e já lê `stats.semFvcCount` — **não alterar o frontend neste ponto**
- [ ] Verificar se a cor do elemento muda para vermelho quando `semFvcCount === 0` mas `stats.cps > 0` (lógica já existe no COD2 — confirmar que está preservada)
- [ ] Garantir que o escopo OAuth do Apps Script inclua permissão de leitura desta planilha (adicionar no `appsscript.json` se necessário)

---

### 🔧 BLOCO 4 — Identificação Sherlock Dinâmica (BACKEND `Code.gs`)

**O que mudar:** O campo `stats.qualityTotal` (exibido em `welcomeQualityTotal`) não deve ser mais **hardcoded** como `71,55%`. Deve ser calculado dinamicamente.

**Fórmula:**
```
qualityTotal = (CPs identificados com e-mail válido de franquia / total de CPs da base FVC) × 100
```

**Checklist de implementação:**

- [ ] Calcular dinamicamente `qualityTotal` no backend baseado na relação entre CPs detectados no período e a base FVC
- [ ] Retornar no payload: `stats.qualityTotal = número_decimal` (ex.: `71.55`)
- [ ] No frontend, o campo `welcomeQualityTotal` já usa `stats.qualityTotal` com `toFixed(2)` + `%` — **não alterar o frontend**
- [ ] Se `qualityTotal` vier nulo/zero, exibir `'—'` em vez de `0,00%` (adicionar fallback no frontend apenas se necessário)

---

### 🔧 BLOCO 5 — Validação do Multiplicador x2 por PRODUCT_MAPPING (BACKEND `Code.gs`)

**O que validar:** O multiplicador `×2` no audit deve ser aplicado **somente** para CPs que possuem ambos os produtos (CI **e** CA) registrados no `PRODUCT_MAPPING_SS_ID`. CPs com apenas 1 produto não devem receber o dobro.

**Checklist de validação:**

- [ ] Na lógica de unicidade (Bloco 2), antes de aplicar `×2`, verificar se o CP tem CI **E** CA no mapeamento de produtos
- [ ] CPs com apenas CI ou apenas CA → não multiplicar por 2 (contar apenas e-mails únicos sem multiplicador)
- [ ] CPs com ambos (CI + CA) → aplicar `×2`
- [ ] Documentar no `stats.dispoRule` ou novo campo `stats.productMapping` se o mapeamento foi consultado com sucesso

---

### 🔧 BLOCO 6 — Integridade do Comprovante PDF e Analytics (FRONTEND `index.html`)

**O que validar:** O valor "Audit" exibido no Comprovante PDF e na aba Analytics deve usar a mesma lógica de unicidade por e-mail + x2.

**Checklist de validação:**

- [ ] No `generateDirectOnePage()`: o `total` exibido no PDF deve vir de `res.list.length` com e-mails únicos por dia (não contagem bruta de linhas)
- [ ] Na aba Analytics (`anTotalAcessos`): deve exibir `stats.audit` (que já vem com a nova lógica do backend)
- [ ] A propriedade `v` nos itens de `res.list` deve conter a contagem validada (para ranking e modal)
- [ ] Confirmar que `exportCSV()` exporta os dados filtrados corretamente sem duplicatas

---

### 🔧 BLOCO 7 — Novos Campos no Frontend (FRONTEND `index.html`) — diferenças do COD2 vs Code GS

Comparando os dois arquivos em `0805 Ajustes/`, o **Code GS.txt** (versão mais nova) já contém as seguintes diferenças que devem estar no `index.html` atual:

| Elemento | COD2 (antigo) | Code GS (novo) | Ação |
|---|---|---|---|
| `welcome-stats` | 4 cards | **5 cards** (adicionou `welcomeSemFvc`) | Confirmar que o 5º card existe no HTML |
| `grid-template-columns` de `.welcome-stats` | `repeat(4, ...)` | `repeat(5, minmax(140px, 1fr))` | Atualizar CSS |
| Card 5 (amarelo) | Não existe | `welcomeSemFvc` — "CPs SEM FVC" | Adicionar card se não existir |
| `.welcome-stat:nth-child(4)` | cor laranja | cor **amarela** (`rgba(234,179,8,...)`) | Atualizar cor |
| `.welcome-stat:nth-child(5)` | — | cor **laranja** | Adicionar estilo |
| `renderData()` | Não popula `bigTotal2025`/`bigTotal2026` | Popula ambos com `stats.audit2025` e `stats.audit2026` | Confirmar se IDs existem no HTML |
| `renderData()` | Não lê `stats.qualityTotal` | Lê `stats.qualityTotal` dinamicamente | Confirmar se está no `renderData()` |
| `renderTrendSummary()` | Calcula local y2025/y2026 | Prioriza `stats.audit2025`/`stats.audit2026` do servidor | Atualizar função |
| `forceSync()` | Chama `admin_forceRefreshSnapshot()` | Chama `clearAllSherlockCache()` | Atualizar chamada |
| `.nav-btn.analytics-nav-btn` | Não existe | Estilo especial para botão Analytics | Adicionar estilo se ausente |
| Sistema de Toast (`showToast`) | Não existe | Existe com CSS `.custom-toast` | Adicionar se ausente |
| `normCpName()` | Não existe | Existe — limpa sufixos "Nº378" dos nomes | Adicionar função |
| `refreshQuickInsights()` | Não existe | Existe — calcula delta % entre 2025 e 2026 | Confirmar presença |

**Checklist de validação dos 5 cards welcome:**

- [ ] Card 1 (azul) — `welcomeAuditTotal` — "Acessos/ View - Acumulado"
- [ ] Card 2 (rosa) — `welcomeDispoTotal` — "Disponibilizações"
- [ ] Card 3 (verde) — `welcomeCpTotal` — "CPs impactados"
- [ ] Card 4 (amarelo) — `welcomeSemFvc` — "CPs SEM FVC" ← **NOVO**
- [ ] Card 5 (laranja) — `welcomeQualityTotal` — "Identificação Sherlock"
- [ ] CSS `grid-template-columns: repeat(5, minmax(140px, 1fr))` aplicado em `.welcome-stats`

---

### 🔧 BLOCO 8 — Performance: Monitorar `Set` de Unicidade Global (BACKEND `Code.gs`)

**O que validar:**

- [ ] O `Set` de e-mails únicos usado no cálculo de audit não deve crescer indefinidamente em memória durante execução do GAS
- [ ] Usar `Set` **por escopo de função** (não global/persistente entre chamadas)
- [ ] Se o volume for muito alto, considerar usar `Object` com chaves compostas `"CP|DATA|EMAIL"` para reduzir overhead
- [ ] Adicionar log de tamanho do Set no início e fim do processamento para monitoramento

---

### 📋 RESUMO DO QUE PRECISA SER FEITO (ordem de prioridade)

| # | Bloco | Arquivo | Prioridade | Status |
|---|---|---|---|---|
| 1 | Multiplicador Temporal x2/x4 (threshold 05/05/2026) | `Code.gs` | 🔴 Crítica | ☐ |
| 2 | Contagem por e-mails únicos por CP/dia + x2 | `Code.gs` | 🔴 Crítica | ☐ |
| 3 | CPs SEM FVC — leitura real da aba `CPS 062026` | `Code.gs` | 🔴 Crítica | ☐ |
| 4 | `qualityTotal` dinâmico (não hardcoded) | `Code.gs` | 🟡 Alta | ☐ |
| 5 | Validar x2 apenas para CPs com CI+CA no mapeamento | `Code.gs` | 🟡 Alta | ☐ |
| 6 | Comprovante PDF e Analytics com nova lógica | `index.html` | 🟡 Alta | ☐ |
| 7 | 5 cards no welcome, CSS e novos campos frontend | `index.html` | 🟡 Alta | ☐ |
| 8 | Performance do Set de unicidade global | `Code.gs` | 🟢 Média | ☐ |

---

---

## ⚙️ CHECKLIST TÉCNICO GLOBAL

### Permissões AppScript (`appsscript.json`)

| Scope | Finalidade | Status |
|---|---|---|
| `https://www.googleapis.com/auth/drive` | Ler/escrever arquivos e pastas no Drive | ☐ |
| `https://www.googleapis.com/auth/spreadsheets` | Ler dados do Sheets (Organizze CI + FVC) | ☐ |
| `https://www.googleapis.com/auth/userinfo.email` | Identificar usuário logado | ☐ |
| `https://www.googleapis.com/auth/script.external_request` | Requisições HTTP externas | ☐ |

### Funções Backend (Code.gs)

| Função | Finalidade | Status |
|---|---|---|
| `doGet()` | Serve o `index.html` | ☐ |
| `getConsolidatedData()` | Orquestra leitura e retorna JSON completo | ☐ |
| `internal_processSheetsData_()` | Lê dados do Sheets | ☐ |
| `fetchDbEmails_Unified_()` | Busca e-mails no Drive | ☐ |
| `testDriveAccess()` | Valida permissão ao Drive (ID: `1tIqgxus9VmC3rEaCj9z7JB1rmsahfJ_a`) | ☐ |

### IDs críticos do DOM (não remover/renomear)

| ID | Usado em |
|---|---|
| `bigTotal`, `k1`, `k2`, `k3` | `renderData()` — KPI cards |
| `trendChart` | `renderTrend()` — gráfico de tendência |
| `mainTable` | `renderTable()` — tabela principal |
| `topList`, `bottomList` | Rankings de CPs |
| `periodFilter`, `cpSearch`, `dateFrom`, `dateTo` | Filtros — `runFilters()` |
| `anMainChart`, `anDonutChart`, `anForecastChart` | Analytics charts |
| `loadingOverlay`, `loadingMsg`, `loadingBarFill` | Loading state |
| `userEmailLp`, `userProfileName` | Autenticação |

### Smoke Test de Validação Pós-Deploy

| Teste | Status |
|---|---|
| Dashboard carrega sem trava (< 10s) | ☐ |
| E-mail do usuário exibido na landing page | ☐ |
| Botão "Acessar Plataforma" aparece após autenticação | ☐ |
| KPI cards exibem números (não `-` ou `undefined`) | ☐ |
| Gráfico de tendência renderiza sem erro no console | ☐ |
| Filtro por mês retorna dados coerentes | ☐ |
| Filtro por CP filtra tabela corretamente | ☐ |
| Filtro de data "De/Até" funciona | ☐ |
| Botão Limpar reseta todos os filtros | ☐ |
| Rankings Engajados e Baixo Acesso carregam | ☐ |
| Paginação da tabela (Anterior/Próxima) funciona | ☐ |
| Clique em linha da tabela abre modal do CP | ☐ |
| Modal exibe dados corretos do CP | ☐ |
| Aba Analytics renderiza gráficos sem erro | ☐ |
| Comprovante PDF gera documento após busca por CP | ☐ |
| FAQ exibe todos os cards sem overflow | ☐ |
| Tech & Diagnóstico exibe métricas de qualidade | ☐ |
| Gerenciador de Acessos abre em nova aba | ☐ |
| Logs Admin oculto para usuário comum | ☐ |
| Nenhum erro JS no console após navegação completa | ☐ |

---

## 📋 PENDÊNCIAS ABERTAS (consolidadas)

| # | Pendência | Origem | Prioridade |
|---|---|---|---|
| P01 | Validar em produção valores do Big Number 2026 (amostra x painel) | CHECKLIST_PENDENCIAS | 🔴 Alta |
| P02 | Validar em produção se meses de 2025 retornam dados esperados | Seção 7 pendências | 🔴 Alta |
| P03 | Validar com negócio se regra teórica 2026 e base FVC estão aderentes | Seção 8 pendências | 🔴 Alta |
| P04 | Validar com negócio o comparativo final Views CI+CA x Disponibilizações | Seção 5 pendências | 🔴 Alta |
| P05 | Revisar microcopy final (texto curto, padrão e clareza) | CHECKLIST_PENDENCIAS | 🟡 Média |
| P06 | Validar layout em resolução 1366x768 e mobile | CHECKLIST_PENDENCIAS | 🟡 Média |
| P07 | Registrar aceite final da área dona do dashboard | CHECKLIST_PENDENCIAS | 🟡 Média |
| P08 | Validar com negócio exemplos reais de CPs sem código | Seção 10 pendências | 🟡 Média |
| P09 | Botões de diagnóstico GAS ("Testar GAS", "Testar Dados") precisam de implementação | Tech tab | 🟡 Média |
| P10 | Imagens de melhoria pendentes na pasta `solicitacoes_melhoria_imagens/` | CHECKLIST_PENDENCIAS | 🟢 Baixa |

---

*Documento gerado em 09/05/2026 — baseado no código atual de `frontend/index.html` e histórico de checklists do projeto.*

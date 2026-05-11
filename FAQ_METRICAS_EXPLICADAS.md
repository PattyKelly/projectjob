# 📊 FAQ - Métricas Explicadas
## Sistema Sherlock Holmes v6.0 - Guia Ilustrativo

---

## 📌 Índice Rápido

1. [O Radar Já Consolidou (Disponibilizações)](#o-radar-já-consolidou-a-disponibilização-total)
2. [Volume do Período (audit)](#volume-do-período)
3. [Franquias Engajadas (cps)](#franquias-engajadas)
4. [1 PTS - O que Significa?](#1-pts---o-que-significa)
5. [Disponibilização Admin (dispo)](#disponibilização)
6. [Exemplos Práticos](#exemplos-práticos)
7. [Evento DLP Unknown em 2025](#evento-dlp-unknown-em-2025)

---

## Evento DLP Unknown em 2025

Mensagem como "Outro Unknown Evento Não Registrado ou Perdido" (ou em inglês,
"Another Unknown Unlogged or Lost Event") geralmente indica limitação de
detalhamento no log de segurança do Google Drive quando uma regra de DLP é
acionada sem uma ação humana direta no exato momento do evento.

### Principais motivos

1. Varredura em segundo plano: após criação/ajuste de regra DLP, o Google pode reavaliar arquivos antigos e gerar evento sem ator explícito.
2. Drive para Desktop/Sync: ações do cliente de sincronização local podem aparecer com menos contexto que ações no navegador.
3. Regra em modo auditoria: políticas audit-only podem registrar impacto potencial de forma genérica.
4. Limitação da API/log: alguns tipos de operação (view, shared drive, fluxos automáticos) chegam sem detalhes completos de executor.

### Como investigar

1. Cruzar data/hora no Admin Console (Drive Log + DLP Rules) para confirmar varredura automática.
2. Revisar histórico de mudanças da política DLP (escopo, condição, modo auditoria/bloqueio).
3. Verificar origem do evento (web, app desktop, integração automatizada).
4. Validar tipo de ação do evento (view, move, share, scan) e metadados disponíveis.
5. Para 2025, considerar retenção: o volume pode permanecer válido, mas parte do metadado de ator pode não estar mais íntegra.

---

## O Radar Já Consolidou a Disponibilização Total

### **Número Exibido: 718 (exemplo no período de teste)**

```
┌────────────────────────────────────────────────────────────┐
│  "O radar já consolidou a disponibilização total de 718    │
│   no período de teste (exemplo: envios de arquivos/        │
│   disponibilâncias para toda rede de franquias que         │
│   continha arquivo válido no Drive)"                       │
└────────────────────────────────────────────────────────────┘
```

### **O que é?**

**DISPONIBILIZAÇÕES VALIDADAS** = Quantidade de dados/arquivos **efetivamente validados e disponibilizados** para toda rede de franquias (período específico)

### **Por que 718 e não 3.447.130?**

```
3.447.130  ← Dados BRUTOS (todas as linhas, inclui erros/perdidos)
           ├─ Contém eventos perdidos
           ├─ Contém dados de sistema
           ├─ Contém duplicatas
           └─ Números "sujos" (backend interno)

718        ← Dados VALIDADOS (apenas o que realmente funciona)
           ├─ Removidos eventos inválidos
           ├─ Removidos dados de sistema
           ├─ Dados limpos e normalizados
           ├─ Realmente "disponibilizado" para franquias
           └─ Números "limpos" (visível para usuário)
```

### **Como é Calculado?**

```javascript
// Code.gs (calculateStatsHybrid):
// Contagem = Linhas que PASSARAM na validação
stats.audit = 718;  // Apenas dados válidos

// Fórmula:
Disponibilizações Total = Linhas com:
  ✅ COD válido (não "Unknown")
  ✅ NOME válido (não "Evento Não Registrado")
  ✅ Data capturada
  ✅ Não é evento de sistema/admin
```

### **Exemplo Visual:**

```
CENÁRIO: Período de Teste (Janeiro - Abril 2025)

Dados Brutos Importados:
┌────────────────────────────────┐
│ Arquivo Original (raw)         │ ~850.000 linhas
│                                │ (inclui lixo, erros, teste)
└────────────────────────────────┘
                ↓
        Python ETL v4.5
        (limpeza + normalização)
                ↓
┌────────────────────────────────┐
│ Dados Processados              │ 718 linhas VÁLIDAS
│ (após motor de decisão)        │ (prontas para uso)
│                                │
│ ✅ Nomes limpos (removido N.°) │
│ ✅ CPs identificados           │
│ ✅ Datas normalizadas          │
│ ✅ Eventos duplicados resolvidos
│ ✅ Pronto para distribuição    │
└────────────────────────────────┘
                ↓
        Disponibilizado para Franquias
        (718 envios/acessos válidos)
```

### **Significado Prático:**

```
"O radar consolidou 718 disponibilizações"

Significa:
├─ 718 acessos/envios foram validados ✅
├─ 718 transações sem erro
├─ 718 eventos que realmente ocorreram
├─ 718 dados que as franquias PODEM confiar
└─ 718 eventos prontos para auditoria
```

### **Quando Muda?**

```
✅ Aumenta quando:
   ├─ Mais acessos válidos são registrados
   ├─ Python ETL processa novos dados
   └─ Período é expandido

✅ Diminui quando:
   ├─ Filtro por data específica
   ├─ Filtro por CP específico
   └─ Período é reduzido
```

### **Diferença Entre Números Mostrados:**

```
DASHBOARD (o que VEEM):
"O radar já consolidou 718 disponibilizações"
└─ Número limpo, significativo, confiável

BACKEND INTERNO (o que PROCESSAMOS):
historyTotal = 3.447.130 (linhas brutas)
audit = 718 (linhas validadas)
└─ Números técnicos, informação completa

PROPORÇÃO:
718 / 3.447.130 = 0,020 = ~2% do total (alta qualidade!)
79,1% do restante = eventos perdidos/sistema/erro
```

**Em resumo:** O número exibido (718) é o que **realmente importa** - dados que foram validados e podem ser usados com confiança.

---

## Volume do Período

### **Número Exibido: 718.000 (ou variável conforme filtros)**

```
┌──────────────────────────────────────────────┐
│  VOLUME DO PERÍODO                           │
│  718.000                                     │
│  (no painel principal - "Big Numbers")       │
└──────────────────────────────────────────────┘
```

### **O que é?**

**ACESSOS VÁLIDOS FILTRADOS** = Quantidade de acessos que passaram na validação (excludes "Evento Não Registrado ou Perdido")

### **Como é Calculado?**

```javascript
// Code.gs (calculateStatsHybrid):
for (let row of data) {
  // ...validações...
  if (cod !== 'Unknown' && nom !== 'Evento Não Registrado') {
    stats.audit += qtd;  // ← Incrementa aqui
  }
}
```

**Algoritmo:**

```
Para CADA linha do JSON:
  ├─ Valida: "Esta linha tem dados válidos?"
  ├─ Se passou na validação:
  │  └─ audit += QUANTIDADE_DE_ACESSOS (coluna H)
  └─ Se não passou:
     └─ Ignora (não conta)

Resultado = stats.audit = VOLUME DO PERÍODO
```

### **Exemplo Prático:**

```
Dados Brutos (16 linhas totais):
┌─────┬──────┬──────────┬────────┬─────────┐
│ ID  │ Tipo │ CP       │ Data   │ Qtd     │
├─────┼──────┼──────────┼────────┼─────────┤
│ 1   │ CA   │ SP       │ 01/04  │ 5       │ ✅ Válido
│ 2   │ CA   │ RJ       │ 01/04  │ 3       │ ✅ Válido
│ 3   │ CI   │ MG       │ 02/04  │ 7       │ ✅ Válido
│ 4   │ CA   │ Unknown  │ 02/04  │ 1       │ ❌ Invalid (Unknown)
│ 5   │ Unknown │ —     │ 02/04  │ 2       │ ❌ Invalid (evento perdido)
│ 6   │ CA   │ BA       │ 03/04  │ 4       │ ✅ Válido
│ 7   │ CI   │ SP       │ 03/04  │ 6       │ ✅ Válido
│ 8   │ DLP  │ System   │ 03/04  │ 1       │ ❌ Invalid (sistema)
│ ...                                        │
└─────┴──────┴──────────┴────────┴─────────┘

CÁLCULO:
audit = 5(row1) + 3(row2) + 7(row3) + 0(row4❌) + 0(row5❌) + 4(row6) + 6(row7) + 0(row8❌) + ...
      = 25 + ...outros
      = 718.000 (total final)
```

### **Fatores que Afetam o Volume:**

```
Aumenta:
├─ Mais acessos registrados
├─ Mais linhas válidas processadas
└─ Período maior selecionado

Diminui:
├─ Filtro por CP específico
├─ Filtro por tipo (só CA ou só CI)
├─ Período reduzido (ex: semana vs. mês)
└─ Linhas inválidas/perdidas
```

### **Relação com o Número de Linhas:**

```
historyTotal (3.447.130)  = Total bruto (inclui inválidas)
            ↓ (remove inválidas)
audit (718.000)           = Volume válido do período

Inválidas = 3.447.130 - 718.000 = 2.729.130 eventos perdidos/sistema
```

---

## Franquias Engajadas

### **Número Exibido: 142 (ou variável conforme filtros)**

```
┌──────────────────────────────────────────────┐
│  FRANQUIAS ENGAJADAS                         │
│  142                                         │
│  (CPs únicos com pelo menos 1 acesso)        │
└──────────────────────────────────────────────┘
```

### **O que é?**

**CONTAGEM DE CPs ÚNICOS** = Quantos CPs diferentes tiveram pelo menos 1 acesso válido registrado

### **Como é Calculado? (Passo-a-Passo)**

```javascript
// Code.gs (calculateStatsHybrid):
const cpAggregation = {};  // Dicionário para armazenar CPs únicos

for (let i = 0; i < data.length; i++) {
  const row = data[i];
  const cod = String(row['COD DO CP'] || 'Unknown').trim();
  const nom = String(row['NOME DO CP'] || '').trim();
  const qtd = parseFloat(row['QUANTIDADE DE ACESSOS']) || 1;
  
  // REGRA: Só conta se COD é válido E NOME não é "Evento Não Registrado"
  if (cod !== 'Unknown' && nom !== 'Evento Não Registrado ou Perdido') {
    // Incrementa acessos para este CP
    cpAggregation[cod] = (cpAggregation[cod] || 0) + qtd;
  }
}

// RESULTADO: Conta apenas as CHAVES únicas (não importa quantas vezes apareceu)
stats.cps = Object.keys(cpAggregation).length;
```

### **Visualização do Algoritmo:**

```
ENTRADA DE DADOS (16 linhas):
┌─────┬──────┬──────────┬─────┐
│ ID  │ Tipo │ CP       │ Qtd │
├─────┼──────┼──────────┼─────┤
│ 1   │ CA   │ SP       │ 5   │ ◀─ 1ª vez SP
│ 2   │ CA   │ RJ       │ 3   │ ◀─ 1ª vez RJ
│ 3   │ CI   │ MG       │ 7   │ ◀─ 1ª vez MG
│ 4   │ CA   │ Unknown  │ 1   │ ❌ Inválido
│ 5   │ CI   │ SP       │ 2   │ ◀─ 2ª vez SP (mesma chave!)
│ 6   │ CA   │ BA       │ 4   │ ◀─ 1ª vez BA
│ 7   │ CA   │ SC       │ 6   │ ◀─ 1ª vez SC
│ 8   │ Unknown │ —     │ 1   │ ❌ Inválido
│ ...                          │
└─────┴──────┴──────────┴─────┘

PROCESSAMENTO (Dicionário):
cpAggregation = {
  "SP": 5 + 2 = 7        ← SP apareceu 2x, mas conta como 1 CP
  "RJ": 3                ← RJ apareceu 1x
  "MG": 7                ← MG apareceu 1x
  "BA": 4                ← BA apareceu 1x
  "SC": 6                ← SC apareceu 1x
  ...
  (+ 137 CPs)            ← Mais 137 CPs diferentes
}

RESULTADO:
stats.cps = Object.keys(cpAggregation).length = 142 CPs únicos ✅
```

### **Regras de Contagem (Critérios de Engajamento):**

```
Um CP é CONTADO como "Engajado" se:

✅ COD DO CP ≠ "Unknown"
   └─ CP precisa estar identificado

✅ NOME DO CP ≠ "Evento Não Registrado ou Perdido"
   └─ CP precisa ter nome válido (não sido perdido)

✅ Teve PELO MENOS 1 acesso
   └─ Não precisa ser muito, apenas 1 já conta

✅ Apareceu em QUALQUER número de linhas
   └─ Se aparecer 1x ou 1.000x = mesmo resultado (1 CP)
```

### **Exemplo Prático Detalhado:**

```
CENÁRIO: Período de Abril 2026

LINHA POR LINHA (o que acontece):

Linha 1: CP=SP, Qtd=5    → cpAggregation["SP"] = 5
Linha 2: CP=RJ, Qtd=3    → cpAggregation["RJ"] = 3
Linha 3: CP=SP, Qtd=2    → cpAggregation["SP"] = 5 + 2 = 7 (SP continua sendo 1!)
Linha 4: CP=MG, Qtd=8    → cpAggregation["MG"] = 8
...continued...

CONTAGEM FINAL:
cpAggregation = {
  "SP": 7,    ← Aparecer múltiplas vezes não importa
  "RJ": 3,
  "MG": 8,
  "BA": 4,
  "SC": 6,
  "DF": 12,
  ...
  (142 chaves no total)
}

RESULTADO:
Franquias Engajadas = 142 CPs únicos ✅

Interpretação:
├─ 142 CPs diferentes participaram/acessaram
├─ Total de acessos = 718.000 PTS
├─ Média por CP = 718.000 / 142 = 5.056 PTS/CP
└─ Alguns CPs somam muito mais que a média (SP = 7 vs média = 5.056)
```

### **O Dicionário JavaScript (Chave-Valor):**

```javascript
// Estrutura interna que o Code.gs cria:
const cpAggregation = {
  "SP": 7,           // CP código SP tem 7 PTS totais
  "RJ": 3,           // CP código RJ tem 3 PTS totais
  "MG": 8,           // CP código MG tem 8 PTS totais
  // ...
  "AC": 1            // CP código AC tem 1 PTS total
};

// Contagem final:
const numCPs = Object.keys(cpAggregation).length;  // = 142

// Cada "chave" única = 1 CP engajado
// Não importa o valor (PTS), apenas a existência da chave
```

### **Por que "Engajadas"?**

```
"Engajada" = CP que teve pelo menos 1 evidência de uso
           = CP "participou" do sistema  
           = CP "mostrou movimento"
           = CP entrou em ação

Exemplos de engajamento:
├─ CP SP: 7 PTS (participou)
├─ CP RJ: 3 PTS (participou)
├─ CP MG: 8 PTS (participou)
└─ CP AC: 1 PTS (participou, mas bem pouco)

NÃO conta como engajado:
├─ CP que nunca acessou
├─ CP com "Unknown" no código
└─ CP marcado como "Evento Não Registrado ou Perdido"
```

### **Fórmula Simples:**

```
Franquias Engajadas = COUNT(CPs ÚNICOS com audit válido)
                    = Número de CHAVES no dicionário cpAggregation
                    = Object.keys(cpAggregation).length
```

### **Diferença Chave: CPs vs PTS**

```
┌──────────────────────────────────────────────────┐
│ CPs (Contagem) vs PTS (Valor)                   │
├──────────────────────────────────────────────────┤
│                                                  │
│ CPs Engajados = 142                             │
│ └─ "Quantas franquias participaram?"            │
│ └─ Resposta: 142 franquias diferentes           │
│                                                  │
│ Volume (PTS) = 718.000                          │
│ └─ "Quantos acessos total?"                     │
│ └─ Resposta: 718.000 eventos auditados          │
│                                                  │
│ Relação:                                         │
│ └─ 142 CPs × variação de atividade = 718.000    │
│ └─ SP sozinho: 7 PTS (1 CP)                     │
│ └─ Mas 718.000 / 142 = 5.056 PTS/CP (média)   │
│                                                  │
└──────────────────────────────────────────────────┘
```

### **Quando Muda (CPs Engajados)?**

```
✅ AUMENTA quando:
   ├─ Um novo CP faz primeiro acesso
   ├─ Período selecionado cresce
   └─ Python ETL processa novos dados

✅ DIMINUI quando:
   ├─ Filtro por CP específico (ex: só SP = 1)
   ├─ Filtro por período reduzido
   └─ Período muito curto (nenhum CP participou)

⚠️ NÃO MUDA se:
   ├─ CP SP faz múltiplos acessos (continua sendo 1)
   ├─ CP SP sobe de 1 para 1.000 PTS (continua sendo 1)
   └─ Apenas importa "Participou?" não "Quanto?"
```

Exemplo:
Total de CPs no Brasil = 2.000
CPs com dados válidos = 142
Taxa de engajamento = 142/2000 = 7,1%
```

---

## 1 PTS - O que Significa?

### **Contexto**

"PTS" = **P**oin**TS** = Pontos de Auditoria

### **O que é 1 PTS?**

```
1 PTS = 1 evento de auditoria (1 linha do JSON)
      = 1 acesso registrado
      = 1 trilha de auditoria
      = Peso unitário
```

### **Como é Usado?**

```javascript
// No JSON de entrada:
{
  "DATA": "2025-01-15",
  "TIPO DO REPORT": "Report CA",
  "QUANTIDADE DE ACESSOS": 5  ← Isso é "5 PTS"
}

// Ou padrão (quando não especificado):
const qtd = parseFloat(row['QUANTIDADE DE ACESSOS']) || 1;  // Default = 1 PTS
```

### **Exemplo Visual:**

```
Timeline de Acessos (5 PTS para SP no dia 01/04):

Dia 01/04:
├─ 09:15 - User A acessou Report CA (SP) → 1 PTS
├─ 10:32 - User B acessou Report CA (SP) → 1 PTS
├─ 11:45 - User C acessou Report CA (SP) → 1 PTS
├─ 14:20 - User A acessou Report CI (SP) → 1 PTS
└─ 16:05 - User D acessou Report CA (SP) → 1 PTS
                                         ─────────
                        Total do dia = 5 PTS para SP
```

### **Top 10 Explicado (em PTS):**

```
┌─────┬──────────┬────────┐
│ #   │ CP       │ PTS    │
├─────┼──────────┼────────┤
│ 1   │ SP       │ 1.245  │ ← SP teve 1.245 acessos (PTS)
│ 2   │ RJ       │ 983    │ ← RJ teve 983 acessos (PTS)
│ 3   │ MG       │ 847    │
│ 4   │ BA       │ 723    │
│ 5   │ SC       │ 612    │
│ ...                      │
│ 10  │ RS       │ 425    │
└─────┴──────────┴────────┘

Interpretação:
SP liderou com 1.245 eventos de auditoria (1.245 PTS)
```

### **PTS vs Outras Métricas:**

```
┌──────────────────────────────────────────────────┐
│ PTS (Pontos) vs Outras Métricas                 │
├──────────────────────────────────────────────────┤
│                                                  │
│ 1 PTS       = 1 evento (o que contamos)        │
│ 5 PTS       = 5 eventos (mais atividade)       │
│ 1.245 PTS   = 1.245 eventos (muito ativo)      │
│                                                  │
│ Relação com CPs:                                │
│ • 142 CPs (franquias)                          │
│ • 718.000 PTS totais (acessos)                 │
│ • Média = 718.000 / 142 = 5.056 PTS/CP        │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## Disponibilização

### **Número Exibido: 0 (ou variável)**

```
┌──────────────────────────────────────────────┐
│  DISPONIBILIZAÇÃO                            │
│  0 (ou número variável)                      │
│  (eventos de teste/adm)                      │
└──────────────────────────────────────────────┘
```

### **O que é?**

**EVENTOS ADMINISTRATIVOS** = Acessos gerados por administradores (Patricia Wojciechowski) durante testes/manutenção

### **Como é Calculado?**

```javascript
// Code.gs (calculateStatsHybrid ou internal_processSheetsData):
const isWoj = email.includes('wojciechowski') || 
              nom.includes('Wojciechowski') ||
              adminEmails.includes(email);

if (isWoj) {
  stats.dispo += (parseFloat(row[7]) || 1);  // Conta separado
}
```

### **Por que Separado?**

```
audit = acessos reais de usuários normais
dispo  = acessos de testes/admin (não conta pro volume real)

Exemplo:
audit = 718.000 (usuários reais)
dispo = 2.500 (admin testando sistema)

Total bruto = 720.500
Número reportado = 718.000 (excluindo testes)
```

---

## Exemplos Práticos

### **Cenário 1: Análise Mensal (Abril 2026)**

```
DADOS BRUTOS:
Total de eventos em dados coletados = 3.447.130 linhas

APÓS FILTRO PELO PERÍODO (01-30 de Abril):
historyTotal (apenas abril) = 300.000 linhas
audit (válidas em abril) = 287.000 (válidas)
cps = 98 (CPs que acessaram em abril)
dispo = 1.500 (testes do admin em abril)

INSIGHTS:
├─ 287.000 PTS em abril
├─ 98 franquias participaram
├─ Média: 287.000 / 98 = 2.929 PTS/CP em abril
└─ Alguns CPs são mais ativos que outros
```

**Top 5 de Abril:**

```
CP       PTS      Posição
────────────────────────
SP     15.200    🥇 1º lugar
RJ      8.400    🥈 2º lugar
MG      6.100    🥉 3º lugar
BA      4.800    4º
SC      3.500    5º
```

---

### **Cenário 2: Filtro por CP (Apenas SP)**

```
Selecionou: CP SP

ANTES (Todos os CPs):
historyTotal = 3.447.130
audit = 718.000
cps = 142

DEPOIS (Apenas SP):
historyTotal = 45.000 (linhas de SP)
audit = 42.350 (acessos válidos de SP)
cps = 1 (apenas SP)

INSIGHTS:
├─ SP tem 42.350 acessos no período
├─ Representa 42.350 / 718.000 = 5,9% do volume total
└─ SP é o 1º lugar em engajamento
```

---

### **Cenário 3: Contabilização de 1 PTS**

```
DATA: 2025-01-15 10:30
CP: SP
Evento: "Usuário patricia acessou Report CA"
QUANTIDADE: 1

Efeito:
├─ historyTotal += 1
├─ audit += 1 (se válido)
├─ cpAggregation["SP"] += 1
├─ trend["2025-01"] += 1
└─ 1 PTS computado ✅
```

---

### **Cenário 4: Interpretação de Top 10 vs Bottom 10**

```
TOP 10 (Franquias Mais Ativas):
CP       PTS       Participação
────────────────────────────────
1. SP   45.230    6,3%
2. RJ   38.450    5,4%
3. MG   32.100    4,5%
...
10. RS  12.450    1,7%

Interpretação:
├─ SP é mais de 3,6x mais ativa que RS
├─ Top 10 concentra 23.5% do volume total
└─ Grande dispersão de engajamento

BOTTOM 10 (Franquias Menos Ativas):
CP       PTS       Status
─────────────────────────────
Top anterior: 12.450 (RS)

Bottom 10:
...
142. AM  15       Muito inativo
143. AC  22       Muito inativo
144. RO  18       Muito inativo

Interpretação:
├─ Grande variação: de 45.000 PTS (SP) a 18 PTS (RO)
├─ Necessário acompanhamento das franquias baixas
└─ Possíveis motivos: localização, tamanho, recursos
```

---

## Quadro Comparativo Final

```
┌─────────────────┬──────────────┬──────────────────────────────┐
│ Métrica         │ Tipo         │ Interpretação                │
├─────────────────┼──────────────┼──────────────────────────────┤
│ historyTotal    │ Contagem     │ Total bruto de dados         │
│ 3.447.130       │ de Linhas    │ desde início do Sherlock     │
├─────────────────┼──────────────┼──────────────────────────────┤
│ audit           │ PTS          │ Acessos válidos do período   │
│ 718.000         │ (Pontos)     │ (exclui inválidos/admin)     │
├─────────────────┼──────────────┼──────────────────────────────┤
│ cps             │ Contagem     │ Quantos CPs tiveram pelo     │
│ 142             │ de CPs       │ menos 1 acesso              │
├─────────────────┼──────────────┼──────────────────────────────┤
│ dispo           │ PTS          │ Acessos de testes/admin      │
│ 0-2.500         │ (Pontos)     │ (não contam no volume real)  │
├─────────────────┼──────────────┼──────────────────────────────┤
│ 1 PTS           │ Unidade      │ 1 evento auditado            │
│                 │ Atômica      │ = 1 acesso registrado        │
└─────────────────┴──────────────┴──────────────────────────────┘
```

---

## Fórmulas Rápidas

```
┌─────────────────────────────────────────────────────┐
│ FÓRMULAS ÚTEIS                                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│ Taxa de Cobertura:                                 │
│   = (cps / Total de CPs Brasil) × 100              │
│   = (142 / 2000) × 100 = 7,1%                      │
│                                                     │
│ Média de PTS por CP:                               │
│   = audit / cps                                    │
│   = 718.000 / 142 = 5.056 PTS/CP                  │
│                                                     │
│ Taxa de Dados Válidos:                             │
│   = (audit / historyTotal) × 100                   │
│   = (718.000 / 3.447.130) × 100 = 20,8% válidos   │
│                                                     │
│ Participação de CP:                                │
│   = (ptos_do_cp / audit_total) × 100              │
│   = (45.000 / 718.000) × 100 = 6,3% (SP)         │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Troubleshooting - "Por que o número mudou?"

```
┌────────────────────────────────────────────────────┐
│ Métrica Mudou? Aqui estão as Razões Comuns:        │
├────────────────────────────────────────────────────┤
│                                                    │
│ historyTotal aumentou:                             │
│ └─ Python ETL rodou e gerou novos JSONs 📊        │
│                                                    │
│ audit diminuiu:                                    │
│ └─ Aplicou filtro por período/CP específico ⏰    │
│                                                    │
│ cps diminuiu:                                      │
│ └─ Selecionou CP singular ou período reduzido 🎯 │
│                                                    │
│ dispo aumentou:                                    │
│ └─ Admin fez testes/manutenção 👨‍💼              │
│                                                    │
│ Número ficou = 0:                                  │
│ └─ Sem dados para esse período/filtro ❌          │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## Contatos & Suporte

**Documentação Relacionada:**
- [FLUXO_LEITURA_DADOS.md](./FLUXO_LEITURA_DADOS.md) - Como os dados fluem
- [MIGRACAO_V6_HIBRIDO.md](./MIGRACAO_V6_HIBRIDO.md) - Arquitetura técnica v6.0
- [Code.js](./frontend/Code.js) - Código fonte completo

**Responsável:** Patricia Wojciechowski  
**Versão:** 1.0 - Abril 6, 2026  
**Status:** ✅ Pronto para Uso

---

## Resumo em Uma Página

```
┌────────────────────────────────────────────────────────────┐
│  SHERLOCK HOLMES - MÉTRICAS EM RESUMO                     │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  718             = Disponibilização total validada        │
│                    (o que o usuário VÊ - números limpos)  │
│                                                            │
│  3.447.130       = Total histórico bruto (backend)        │
│                    (dados técnicos, inclui erros)         │
│                                                            │
│  142             = Franquias engajadas (CPs únicos)       │
│                                                            │
│  1 PTS           = 1 evento auditado (unidade)            │
│                    1 acesso = 1 linha = 1 evento          │
│                                                            │
│  0 (admin)       = Eventos administrativos (testes)       │
│                                                            │
│  Lei de Ouro:                                             │
│  audit (718) = dados validados (confiáveis)              │
│  historyTotal (3.447.130) = dados brutos + erros         │
│  audit ≤ historyTotal (sempre)                           │
│  718 é o que REALMENTE foi disponibilizado               │
│                                                            │
└────────────────────────────────────────────────────────────┘
```


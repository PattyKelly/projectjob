# 📦 MODELO DE DADOS — Sherlock Monitor de Acessos
> **Para IA / Colaborador externo:** Os arquivos reais (JSON/CSV/Sheets) estão no Google Drive privado da Patty.
> O acesso é restrito. Este documento é uma **ilustração fiel** da estrutura de dados para que qualquer IA
> consiga entender e corrigir o backend sem precisar acessar os arquivos reais.

---

## 🔒 AVISO DE ACESSO RESTRITO

```
┌─────────────────────────────────────────────────────────────────┐
│  ⛔  OS DADOS REAIS ESTÃO NO GOOGLE DRIVE PRIVADO DA PATTY      │
│                                                                  │
│  Pasta Drive ID : 1tIqgxus9VmC3rEaCj9z7JB1rmsahfJ_a            │
│  Planilha Audit : 1cJjAOlnVn1sUxq36Bhvay_WCCPF4jKo7KNFtQBYXNNs │
│  Planilha FVC   : 1m-FxjYDAUlAx4dyvKNqlL9xWtip1i9nTNVvp2DTJ5vU │
│                                                                  │
│  ✅  Use APENAS este arquivo como referência de estrutura.       │
│  ✅  Não tente acessar nem recriar os dados reais.              │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🗂️ FONTES DE DADOS (em ordem de prioridade de leitura)

> Pasta Drive: **"Pasta Resumo Atualizada"** dentro de **"Report Drive - Acessos"**
> ID: `1tIqgxus9VmC3rEaCj9z7JB1rmsahfJ_a`

```
getConsolidatedData()
       │
       ├── 1️⃣  CACHE (Drive)
       │         sherlock_v59_snapshot.json  ◄── 291 KB  (snapshot rápido)
       │         v6.json                     ◄── 320 KB  (cache v6)
       │
       ├── 2️⃣  JSON MENSAIS (Drive) ◄── FONTE PRIMÁRIA
       │         sherlock_logs_2026-07.json  ◄── 3,8 MB
       │         sherlock_logs_2026-08.json  ◄── 5,6 MB
       │         sherlock_logs_2026-09.json  ◄── 3,3 MB
       │         sherlock_logs_2026-10.json  ◄── 5,1 MB
       │         sherlock_logs_2026-11.json  ◄── 5,5 MB
       │         sherlock_logs_2026-12.json  ◄── 4,5 MB
       │         (padrão de nome: sherlock_logs_YYYY-MM.json)
       │
       ├── 3️⃣  FALLBACK: Google Sheets (quando JSONs falham)
       │         Aba: resumo_2025  ◄── 4,5 MB  (modificado 07/mai/2026)
       │         Aba: resumo_2026  ◄── 1,6 MB  (modificado 29/abr/2026)
       │
       └── 4️⃣  FALLBACK BRUTO: CSV (última instância)
                 resumo_2025.csv  ◄── 90,1 MB  (carga completa 2025)
                 resumo_2026.csv  ◄── 52,4 MB  (carga completa 2026)
```

> ⚠️ **Por que 2025 pesa 90 MB e 2026 pesa 52 MB?**
> 2025 tem o ano inteiro consolidado. 2026 ainda está em andamento (até mai/2026).
> Os JSONs mensais (`sherlock_logs_YYYY-MM.json`) são o recorte já processado — mais leves e rápidos.
> Os CSVs são o dado bruto completo — usados apenas como último recurso.

---

## 📄 ESTRUTURA DE UM REGISTRO — JSON MENSAL

> Colunas exatas (confirmadas 09/05/2026): `TIPO DO REPORT | COD DO CP | NOME DO CP | DATA | RESPONSÁVEL (ATOR) | ARQUIVO_LOG | QUANTIDADE DE ACESSOS`

Cada arquivo JSON da pasta shard é um **array de objetos**. Exemplo com dados ilustrativos (estrutura real):

```json
{
  "TIPO DO REPORT"         : "Report",
  "COD DO CP"              : "19825",
  "NOME DO CP"             : "CP BTSG",
  "DATA"                   : "2025-04-24 20:4",
  "RESPONSÁVEL (ATOR)"     : "greyse.aguiar@x...",
  "ARQUIVO_LOG"            : "Logs - Drive Fra...",
  "QUANTIDADE DE ACESSOS"  : 1
}
```

> ⚠️ **ATENÇÃO IA:** Os valores de `TIPO DO REPORT` chegam BRUTOS do Sheets/JSON.
> O backend normaliza para `Report CI` / `Report CA` durante o processamento.

| Campo | Tipo | Valores brutos reais | Como o backend processa |
|---|---|---|---|
| `TIPO DO REPORT` | string | `"Report"` `"CI"` `"Outro"` | Normaliza: `CI` → `Report CI`; `Outro` com log CA → `Report CA`; etc. |
| `COD DO CP` | string | `"19825"` `"14459"` `"11621"` | Usado como chave de agrupamento; `"Unknown"` se vazio |
| `NOME DO CP` | string | `"CP BTSG"` `"CP LEAO"` `"CP BEM ESTAR"` | Remove sufixo `Nº123` via regex |
| `DATA` | string | `"2025-04-24 20:4"` (com timestamp) | `getNativeDateContext()` extrai só a data `YYYY-MM-DD` |
| `RESPONSÁVEL (ATOR)` | string | `"greyse.aguiar@x..."` `"cp.leao.19340@..."` `"esterf.cairo@gm..."` | Minúsculo + trim; verificado contra adminEmails |
| `ARQUIVO_LOG` | string | `"Logs - OABo-zRI..."` `"Logs - Drive Fra..."` | Detecta CI/CA quando TIPO não é claro |
| `QUANTIDADE DE ACESSOS` | number | `1` (sempre 1 por linha) | Multiplicador base; backend aplica ×2 se CP tem CI+CA |

---

## 📋 ESTRUTURA SHEETS — FALLBACK (quando JSON falha)

Aba real confirmada: **`resumo_2025`** (Google Sheets) — 7 colunas visíveis + colunas extras vazias:

```
col[0]  →  TIPO DO REPORT         (ex: "Report", "CI", "Outro")
col[1]  →  COD DO CP              (ex: "19825", "11621", "14459")
col[2]  →  NOME DO CP             (ex: "CP BTSG", "CP LEAO", "CP BEM ESTAR")
col[3]  →  DATA                   (ex: "2025-04-24 20:4" — com timestamp)
col[4]  →  RESPONSÁVEL (ATOR)     (ex: "greyse.aguiar@x...", "cp.leao.19340@...")
col[5]  →  ARQUIVO_LOG            (ex: "Logs - Drive Fra...", "Logs - OABo-zRI...")
col[6]  →  QUANTIDADE DE ACESSOS  (sempre 1 por linha)
col[7]  →  (vazio / reservado)
```

> 📸 **Screenshot confirmado por Patty em 09/05/2026** — estrutura idêntica entre Sheets e JSON.

**Como o backend detecta o header:**
```javascript
// Varre as 10 primeiras linhas procurando a linha de cabeçalho
const txt = data[r].join('|').toUpperCase();
if (txt.includes('DATA') || txt.includes('COD')) { startIdx = r + 1; break; }
```

---

## 🗂️ PLANILHA FVC — Mapeamento de Produtos por CP

**ID:** `1m-FxjYDAUlAx4dyvKNqlL9xWtip1i9nTNVvp2DTJ5vU`
**Aba:** `CPS 062026` (coluna C = PRODUTOS)

Estrutura da aba (3 colunas):

```
col[0]  →  COD CP     (ex: "53123")
col[1]  →  NOME CP    (ex: "CP CIDADE EXEMPLO")
col[2]  →  PRODUTOS   (ex: "CI+CA" | "CI" | "CA" | "SEM FVC")
```

Exemplo ilustrativo:

| COD CP | NOME CP | PRODUTOS |
|---|---|---|
| 53123 | CP CIDADE EXEMPLO | CI+CA |
| 53456 | CP OUTRO EXEMPLO | CI |
| 53789 | CP TERCEIRO EXEMPLO | SEM FVC |
| 53999 | CP QUARTO EXEMPLO | CA |

**Contagens reais (referência 09/05/2026):**

| Categoria | Quantidade |
|---|---|
| CI + CA | 166 |
| CI | 160 |
| CA | 10 |
| SEM FVC | 41 |
| **TOTAL** | **377** |

---

## 📤 PAYLOAD RETORNADO — `getConsolidatedData()` → Frontend

```json
{
  "success": true,
  "canAudit": false,
  "sourceMethod": "JSON_MONTHLY",
  "isLimited": false,

  "stats": {
    "audit"         : 34000,
    "dispo"         : 12800,
    "dispoRule"     : "x4",
    "historyTotal"  : 189000,
    "uniqueCps"     : 312,
    "semFvcCount"   : 40,
    "qualityTotal"  : "71.55",

    "trend": {
      "2026-04-01": 142,
      "2026-04-02": 98,
      "2026-04-03": 205
    },

    "top10": [
      { "k": "53123", "n": "CP CIDADE EXEMPLO", "val": 87 },
      { "k": "53456", "n": "CP OUTRO EXEMPLO",  "val": 65 }
    ],
    "bottom10": [
      { "k": "53999", "n": "CP BAIXO ACESSO",   "val": 2 }
    ],

    "cpAggregation": {
      "53123": 87,
      "53456": 65
    },

    "suggestions": ["53123", "53456", "53789"],

    "lastUpdate": "09/05/2026 16:14"
  },

  "list": [
    { "t": "Report CI", "c": "53123", "n": "CP CIDADE EXEMPLO", "d": "15/04/2026", "e": null, "v": 1 },
    { "t": "Report CA", "c": "53123", "n": "CP CIDADE EXEMPLO", "d": "15/04/2026", "e": null, "v": 1 }
  ]
}
```

**Legenda do objeto `list[]`:**

| Chave | Significado | Tipo |
|---|---|---|
| `t` | Tipo: `"Report CI"` / `"Report CA"` / `"Acesso ao Drive"` | string |
| `c` | COD CP | string |
| `n` | NOME CP (normalizado, sem sufixo Nº) | string |
| `d` | Data formatada `"dd/mm/aaaa"` | string |
| `e` | E-mail do ator — `null` se `canAudit=false` | string\|null |
| `v` | Multiplicador (1 ou 2) | number |

---

## 🔄 FLUXO COMPLETO DE LEITURA (visão em diagrama)

```
Google Drive (Privado Patty)
  └── Pasta Resumo Atualizada (ID: 1tIqgxus9VmC3rEaCj9z7JB1rmsahfJ_a)
        ├── sherlock_logs_2026-07.json  ←─┐
        ├── sherlock_logs_2026-08.json    │  loadAllJsonMonthlyData_()
        ├── sherlock_logs_2026-09.json    │  lê todos os arquivos
        ├── sherlock_logs_2026-10.json    │  com padrão sherlock_logs_*.json
        ├── sherlock_logs_2026-11.json    │
        ├── sherlock_logs_2026-12.json  ←─┘
        │
        ├── resumo_2025  (Sheets)  ←── fallback 2025 (4,5 MB)
        ├── resumo_2026  (Sheets)  ←── fallback 2026 (1,6 MB)
        ├── resumo_2025.csv        ←── fallback bruto (90,1 MB)
        └── resumo_2026.csv        ←── fallback bruto (52,4 MB)
                │
                ▼
        buildResponseFromJson_()
                │
                ├── Exclui: DLP, system, admin emails
                ├── Unicidade: cod|ator|data (dedup por dia)
                ├── Multiplicador: ×2 se CP tem CI+CA no FVC map
                ├── Mascara: email → e: null (se canAudit=false)
                └── Monta: stats{} + list[]
                │
                ▼
        saveDataToCacheFile_()  →  sherlock_cache.json (Drive)
                │
                ▼
        Frontend recebe JSON → renderData() → renderTable() → renderRankings()
```

---

## ⚠️ REGRAS QUE A IA NUNCA DEVE IGNORAR

```
┌──────────────────────────────────────────────────────────────────────┐
│  🔴 NUNCA FAZER SEM AUTORIZAÇÃO EXPLÍCITA DA PATTY:                 │
│                                                                      │
│  1. Alterar tela de login / fluxo de boot / overlay de loading      │
│  2. Remover ou renomear IDs do DOM (bigTotal, k1, k2, k3, etc.)     │
│  3. Apagar blocos de cálculo — apenas CORRIGIR ou ADICIONAR         │
│  4. Mudar arrays adminEmails sem confirmação                         │
│  5. Alterar lógica de canAudit (exposição de e-mails)               │
│  6. Remover qualquer menu sem confirmação explícita                  │
│  7. Fazer push/deploy em produção sem aprovação                      │
└──────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────┐
│  ✅ PODE FAZER LIVREMENTE:                                           │
│                                                                      │
│  1. Corrigir lógica de cálculo (audit, dispo, multiplier)           │
│  2. Adicionar novos campos ao payload sem remover existentes        │
│  3. Corrigir bugs de filtro / data / paginação                      │
│  4. Melhorar CSS/visual sem alterar IDs ou estrutura HTML           │
│  5. Adicionar logs de debug (Logger.log) no backend                 │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 📁 ARQUIVOS DE REFERÊNCIA NESTE REPOSITÓRIO

| Arquivo | O que é | Usar para |
|---|---|---|
| `CHECKLIST_POR_MENU_COMPLETO.md` | Spec completa de bugs e funcionalidades | Entender o que corrigir e na que ordem |
| `0805 Ajustes/Code GS.txt` | Frontend HTML+JS atual (08/05/2026) | Corrigir interface, charts, filtros |
| `0805 Ajustes/code.back and 0805` | Backend AppScript atual (08/05/2026) | Corrigir lógica de dados |
| `_versions/` | Snapshots datados | Reverter se necessário |
| `.github/copilot-instructions.md` | Regras de proteção para IA | Leia ANTES de qualquer edição |

---

*Documento criado em 09/05/2026 — estrutura extraída do código real do backend.*

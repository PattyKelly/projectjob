/**
 * Dashboard de Auditoria de Acessos - Backend (v5.1 Super-Performance)
 * RESOLUÃ‡ÃƒO: Processamento de 350k+ linhas via AgregaÃ§Ã£o Total
 * 
 * STATUS: VERSAO CORRIGIDA COM TODOS OS AJUSTES
 * - SHARD_FOLDER_ID: 1tIqgxus9VmC3rEaCj9z7JB1rmsahfJ_a
 * - BQ_CI_SSID: 1cJjAOlnVn1sUxq36Bhvay_WCCPF4jKo7KNFtQBYXNNs
 * - fetchDbEmails_Unified_: deduplicaÃ§Ã£o sem Set
 * - testDriveAccess: funÃ§Ã£o de teste incluÃ­da
 */

function doGet() {
  return HtmlService.createTemplateFromFile('index')
    .evaluate()
    .setTitle('Auditoria v5.1 - Big Data')
    .addMetaTag('viewport', 'width=device-width, initial-scale=1')
    .setXFrameOptionsMode(HtmlService.XFrameOptionsMode.ALLOWALL);
}

function getUserEmail() {
  try {
    return Session.getActiveUser().getEmail() || "Colaborador Autorizado";
  } catch(e) {
    return "Colaborador Autorizado";
  }
}

/**
 * GravaÃ§Ã£o silenciosa de logs de acesso (Sistema Sherlock)
 */
function internal_logAccess_(user) {
    try {
       const centralSS = SpreadsheetApp.openById('1DZmRVe2KxMELsVrxQY1zN4iQ6_BkdsF96H-bmUXrdWE');
       let logSheet = centralSS.getSheetByName('Logs');
       if (!logSheet) {
          logSheet = centralSS.insertSheet('Logs');
          logSheet.appendRow(["Data/Hora", "UsuÃ¡rio", "AÃ§Ã£o", "Plataforma"]);
       }
       const d = new Date();
       const fDate = `${String(d.getDate()).padStart(2,'0')}/${String(d.getMonth()+1).padStart(2,'0')}/${d.getFullYear()} ${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}`;
       logSheet.appendRow([fDate, user, "Acesso a Plataforma Torre de Controle", "Sherlock Holmes"]);
    } catch(e) { console.error("Falha no Log Central:", e); }
}

// ═══════════════════════════════════════════════════════════
// ADMIN / TECH ACCESS — Sherlock
// Alterado Patty - 10/05/2026
// Motivo: garantir que Tech & Diagnóstico só apareça e só execute para admins.
// ═══════════════════════════════════════════════════════════

const SHERLOCK_ADMIN_EMAILS = [
  'patricia.wojciechowski@grupoboticario.com.br',
  'opatriciakelly@gmail.com',
  'opatricia.kelly@gmail.com'
];

// ═══════════════════════════════════════════════════════════
// LGPD — OCULTAR SOMENTE E-MAILS DEV/ADMIN ESPECÍFICOS
// Regra: mantém todos os outros e-mails de franquia/usuário.
// Apenas retorna null para os e-mails da Patty (dev/admin).
// ═══════════════════════════════════════════════════════════

const SHERLOCK_DEV_EMAILS_HIDE_ONLY = [
  'patricia.wojciechowski@grupoboticario.com.br',
  'opatriciakelly@gmail.com',
  'opatricia.kelly@gmail.com'
];

function normalizeAuditEmailOnly_(value) {
  const raw = String(value || '').toLowerCase().trim();
  if (!raw) return '';
  const match = raw.match(/[a-z0-9._%+\-]+@[a-z0-9.\-]+\.[a-z]{2,}/i);
  return match ? String(match[0]).toLowerCase().trim() : raw;
}

function isDevAdminEmailToHideOnly_(value) {
  const email = normalizeAuditEmailOnly_(value);
  if (!email) return false;
  if (SHERLOCK_DEV_EMAILS_HIDE_ONLY.includes(email)) return true;
  if (/^patricia\.wojciechowsk?i@grupoboticario\.com\.br$/i.test(email)) return true;
  if (/^opatricia\.?kelly@gmail\.com$/i.test(email)) return true;
  return false;
}

function auditEmailForUiOnly_(value) {
  const email = normalizeAuditEmailOnly_(value);
  if (!email) return null;
  if (isDevAdminEmailToHideOnly_(email)) return null;
  return email;
}

function sanitizeAuditListEmailOnly_(list) {
  if (!Array.isArray(list)) return [];
  return list.map(function(row) {
    const clean = Object.assign({}, row);
    clean.e = auditEmailForUiOnly_(clean.e);
    return clean;
  });
}

function getActiveUserEmailSafe_() {
  try {
    return String(Session.getActiveUser().getEmail() || '').toLowerCase().trim();
  } catch (e) {
    return '';
  }
}

function isSherlockAdmin_() {
  return SHERLOCK_ADMIN_EMAILS.includes(getActiveUserEmailSafe_());
}

function requireSherlockAdmin_() {
  const email = getActiveUserEmailSafe_();
  if (!SHERLOCK_ADMIN_EMAILS.includes(email)) {
    throw new Error('Acesso negado. Área restrita Tech & Diagnóstico.');
  }
  return email;
}

/**
 * Usado pelo front para decidir se exibe o menu Tech.
 * Não joga erro para usuário comum; só retorna isAdmin=false.
 */
function getTechAccessContext() {
  const email = getActiveUserEmailSafe_();
  return {
    success: true,
    email: email,
    isAdmin: SHERLOCK_ADMIN_EMAILS.includes(email)
  };
}

// ==== AREA RESTRITA DE ADMIN / DEV ==== 
function getAdminLogs() {
    const adminEmails = ['patricia.wojciechowski@grupoboticario.com.br', 'opatriciakelly@gmail.com'];
    const activeUser = Session.getActiveUser().getEmail().toLowerCase();
    
    // BLOQUEIO ABSOLUTO DE SERVIDOR: Nem mesmo com hack via URL alguÃ©m passa dessa linha
    if (!adminEmails.includes(activeUser)) {
        return { error: "Acesso Negado. Credenciais Dev invÃ¡lidas para o e-mail: " + activeUser };
    }
    
    try {
        const centralSS = SpreadsheetApp.openById('1DZmRVe2KxMELsVrxQY1zN4iQ6_BkdsF96H-bmUXrdWE');
        const logSheet = centralSS.getSheetByName('Logs');
        if (!logSheet) {
            return { error: "Aba de log 'Logs' nÃ£o encontrada na planilha central." };
        }
        const lastRow = logSheet.getLastRow();
        if (lastRow < 2) return { logs: [] };
        // Retorna as 500 Ãºltimas ocorrÃªncias do log (4 colunas)
        const startR = Math.max(2, lastRow - 500);
        const data = logSheet.getRange(startR, 1, lastRow - startR + 1, 4).getValues();
        return { logs: data.reverse() }; // Mais recentes primeiro
    } catch(err) {
        return { error: "Erro ao ler banco de dados de log central (Check Permissions)." };
    }
}

function isAdminUser_(email) {
  const adminEmails = ['patricia.wojciechowski@grupoboticario.com.br', 'opatriciakelly@gmail.com'];
  const active = String(email || '').toLowerCase().trim();
  return adminEmails.includes(active);
}

function buildJsonEventSignature_(row, dateKey) {
  const tipo = String(row['TIPO DO REPORT'] || row['TIPO'] || '').trim().toUpperCase();
  const cod = String(row['COD DO CP'] || row['COD'] || '').trim().toUpperCase();
  const nom = String(row['NOME DO CP'] || row['NOME'] || '').trim().toUpperCase();
  const ator = String(row['RESPONSÃVEL (ATOR)'] || row['RESPONSAVEL (ATOR)'] || row['ATOR'] || '').trim().toLowerCase();
  const qtd = String(parseFloat(row['QUANTIDADE DE ACESSOS']) || 1);
  const logRef = String(row['ARQUIVO_LOG'] || row['ARQUIVO'] || '').trim().toLowerCase();
  return [dateKey, tipo, cod, nom, ator, qtd, logRef].join('|');
}

function admin_checkDateDuplicationRisk(targetDateIso) {
  try {
    const activeUser = Session.getActiveUser().getEmail().toLowerCase();
    if (!isAdminUser_(activeUser)) {
      return { error: 'Acesso negado para diagnÃ³stico de duplicidade.' };
    }

    const target = /^\d{4}-\d{2}-\d{2}$/.test(String(targetDateIso || '').trim())
      ? String(targetDateIso).trim()
      : '2026-03-16';

    const folder = DriveApp.getFolderById(SHARD_FOLDER_ID);
    const files = folder.getFiles();
    const monthFilePattern = /^sherlock_logs_\d{4}-\d{2}\.json$/i;

    let fileCount = 0;
    let rawRows = 0;
    let rawQty = 0;
    let uniqueQty = 0;
    const signatures = {};
    const perFile = [];

    while (files.hasNext()) {
      const file = files.next();
      const fileName = file.getName();
      if (!monthFilePattern.test(fileName)) continue;

      fileCount++;
      let rowsInDate = 0;
      let qtyInDate = 0;

      try {
        const monthData = JSON.parse(file.getBlob().getDataAsString());
        if (!Array.isArray(monthData)) continue;

        for (let i = 0; i < monthData.length; i++) {
          const row = monthData[i] || {};
          const dCtx = getNativeDateContext(row['DATA'] || '');
          if (!dCtx || dCtx.k !== target) continue;

          const qty = parseFloat(row['QUANTIDADE DE ACESSOS']) || 1;
          const sig = buildJsonEventSignature_(row, dCtx.k);

          rawRows += 1;
          rawQty += qty;
          rowsInDate += 1;
          qtyInDate += qty;

          if (!signatures[sig]) {
            signatures[sig] = {
              count: 1,
              qty: qty,
              sample: {
                tipo: String(row['TIPO DO REPORT'] || row['TIPO'] || '').trim(),
                cod: String(row['COD DO CP'] || row['COD'] || '').trim(),
                nome: String(row['NOME DO CP'] || row['NOME'] || '').trim(),
                ator: String(row['RESPONSÃVEL (ATOR)'] || row['RESPONSAVEL (ATOR)'] || row['ATOR'] || '').trim(),
                arquivo: String(row['ARQUIVO_LOG'] || row['ARQUIVO'] || '').trim()
              }
            };
            uniqueQty += qty;
          } else {
            signatures[sig].count += 1;
          }
        }
      } catch (e) {
        Logger.log('[WARN] Diagnostico duplicidade: falha no arquivo ' + fileName + ' -> ' + e);
      }

      if (rowsInDate > 0) {
        perFile.push({ fileName: fileName, rows: rowsInDate, qty: qtyInDate });
      }
    }

    const uniqueRows = Object.keys(signatures).length;
    const duplicateRows = Math.max(0, rawRows - uniqueRows);
    const duplicateQty = Math.max(0, rawQty - uniqueQty);
    const factorRows = uniqueRows > 0 ? rawRows / uniqueRows : 1;
    const factorQty = uniqueQty > 0 ? rawQty / uniqueQty : 1;

    const topDuplicatedSignatures = Object.keys(signatures)
      .map(sig => ({ sig: sig, count: signatures[sig].count, qty: signatures[sig].qty, sample: signatures[sig].sample }))
      .filter(x => x.count > 1)
      .sort((a, b) => b.count - a.count)
      .slice(0, 10);

    perFile.sort((a, b) => b.rows - a.rows);

    return {
      success: true,
      source: 'JSON_MENSAL',
      date: target,
      scannedFiles: fileCount,
      filesWithTargetDate: perFile.length,
      rawRows: rawRows,
      rawQty: rawQty,
      uniqueRows: uniqueRows,
      uniqueQty: uniqueQty,
      duplicateRows: duplicateRows,
      duplicateQty: duplicateQty,
      duplicationFactorRows: Number(factorRows.toFixed(4)),
      duplicationFactorQty: Number(factorQty.toFixed(4)),
      perFileTop: perFile.slice(0, 15),
      duplicatedSignatureTop: topDuplicatedSignatures,
      risk: duplicateRows > 0 || duplicateQty > 0 ? 'POSSIVEL_DUPLICIDADE' : 'SEM_INDICIOS_DE_DUPLICIDADE'
    };
  } catch (e) {
    return { error: 'Erro no diagnÃ³stico de duplicidade: ' + e.toString() };
  }
}

/**
 * Converte uma data de auditoria em chave ordenável.
 * Alterado Patty - 10/05/2026
 * Motivo: garantir Audit Log Explorer do mais atual para o mais antigo.
 */
function getAuditDateSortKey_(value) {
  const raw = String(value || '').trim();
  if (!raw) return '';

  // Formato ISO vindo do Python: YYYY-MM-DD HH:mm:ss ou YYYY-MM-DDTHH:mm:ss
  let m = raw.match(/^(\d{4})-(\d{2})-(\d{2})(?:[T\s](\d{1,2}):(\d{2})(?::(\d{2}))?)?/);
  if (m) {
    return [
      m[1], m[2], m[3],
      String(m[4] || '00').padStart(2, '0'),
      String(m[5] || '00').padStart(2, '0'),
      String(m[6] || '00').padStart(2, '0')
    ].join('');
  }

  // Formato BR: DD/MM/YYYY HH:mm:ss
  m = raw.match(/^(\d{2})[\/\-](\d{2})[\/\-](\d{4})(?:\s+(\d{1,2}):(\d{2})(?::(\d{2}))?)?/);
  if (m) {
    return [
      m[3], m[2], m[1],
      String(m[4] || '00').padStart(2, '0'),
      String(m[5] || '00').padStart(2, '0'),
      String(m[6] || '00').padStart(2, '0')
    ].join('');
  }

  // Fallback usando parser já existente do Sherlock
  const ctx = getNativeDateContext(raw);
  if (!ctx || !ctx.k) return '';
  return String(ctx.k).replace(/\D/g, '') + String(ctx.tm || '00:00').replace(/\D/g, '');
}

/**
 * Ordena linhas brutas do JSON mensal pela DATA desc.
 * 2026 primeiro, depois 2025.
 */
function sortJsonRowsByAuditDateDesc_(rows) {
  return (Array.isArray(rows) ? rows.slice() : []).sort((a, b) => {
    const ak = getAuditDateSortKey_(a && a['DATA']);
    const bk = getAuditDateSortKey_(b && b['DATA']);
    if (bk !== ak) return bk.localeCompare(ak);
    const acp = String(a && a['COD DO CP'] || '');
    const bcp = String(b && b['COD DO CP'] || '');
    return acp.localeCompare(bcp);
  });
}

function getNativeDateContext(dVal) {
    if (!dVal) return null;
    let d;
    
    // Se for string, tentamos identificar o padrÃ£o brasileiro DD/MM/YYYY hh:mm:ss
    if (typeof dVal === 'string') {
        const str = dVal.trim();
      // Regex para capturar YYYY-MM-DD (com ou sem hora, separador T ou espaÃ§o)
      const isoPattern = /^(\d{4})-(\d{2})-(\d{2})(?:[T\s](\d{1,2}):(\d{2})(?::(\d{2}))?)?/;
      const mIso = str.match(isoPattern);
        // Regex para capturar DD/MM/YYYY ou DD-MM-YYYY (com ou sem hora)
        const brPattern = /^(\d{2})[\/\-](\d{2})[\/\-](\d{4})(?:\s+(\d{1,2}):(\d{2})(?::\d{2})?)?/;
        const m = str.match(brPattern);
      if (mIso) {
        const yr = parseInt(mIso[1], 10);
        const mo = parseInt(mIso[2], 10) - 1;
        const da = parseInt(mIso[3], 10);
        const hh = mIso[4] ? parseInt(mIso[4], 10) : 0;
        const mm = mIso[5] ? parseInt(mIso[5], 10) : 0;
        const ss = mIso[6] ? parseInt(mIso[6], 10) : 0;
        d = new Date(yr, mo, da, hh, mm, ss);
      } else if (m) {
            const da = parseInt(m[1], 10);
            const mo = parseInt(m[2], 10) - 1;
            const yr = parseInt(m[3], 10);
            const hh = m[4] ? parseInt(m[4], 10) : 0;
            const mm = m[5] ? parseInt(m[5], 10) : 0;
            d = new Date(yr, mo, da, hh, mm, 0);
        } else {
            d = new Date(str);
        }
    } else {
        d = dVal instanceof Date ? dVal : new Date(dVal);
    }
    
    if (isNaN(d.getTime())) return null;

    const yr = d.getFullYear();
    const mo = String(d.getMonth() + 1).padStart(2, '0');
    const da = String(d.getDate()).padStart(2, '0');
    const hh = String(d.getHours()).padStart(2, '0');
    const mm = String(d.getMinutes()).padStart(2, '0');
    const timeStr = `${hh}:${mm}`;
    
    const target = new Date(d.valueOf());
    const dayNr = (d.getDay() + 6) % 7;
    target.setDate(target.getDate() - dayNr + 3);
    const firstThurs = target.valueOf();
    target.setMonth(0, 1);
    if (target.getDay() !== 4) target.setMonth(0, 1 + ((4 - target.getDay()) + 7) % 7);
    const wk = 1 + Math.ceil((firstThurs - target) / 604800000);

    return { k: `${yr}-${mo}-${da}`, f: `${da}/${mo}/${yr}`, w: `${yr}-W${String(wk).padStart(2, '0')}`, tm: timeStr };
}

function shouldHideFromRanking_(label) {
  const txt = String(label || '')
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '')
    .trim();

  if (!txt) return true;

  const blacklist = [
    'cp sem log apurado',
    'sem log apurado',
    'evento nao registrado ou perdido',
    'unknown',
    'desconhecido'
  ];

  return blacklist.some(term => txt.includes(term));
}

const FVC_CARTEIRA_SPREADSHEET_ID = '1yJYVJ1-mW0UxngqC07aCaff8_CaD46SuqJgfBFswIGw';
const FVC_CARTEIRA_SHEET_NAME = "CARTEIRA FVC 3.0 -Ciclo 06'26";
// Planilha FVC com mapeamento CP → PRODUTO (coluna C = PRODUTOS, SEM FVC, etc.)
const PRODUCT_MAPPING_SS_ID = '1m-FxjYDAUlAx4dyvKNqlL9xWtip1i9nTNVvp2DTJ5vU';
const FVC_CARTEIRA_URL = 'https://docs.google.com/spreadsheets/d/1yJYVJ1-mW0UxngqC07aCaff8_CaD46SuqJgfBFswIGw/edit?gid=258254121#gid=258254121';
const FVC_MISSING_EMAIL_CACHE_KEY = 'fvc_missing_email_c06_v1';

function normalizeCpCode_(value) {
  const raw = String(value || '').trim();
  if (!raw) return '';
  const onlyDigits = raw.replace(/\D/g, '');
  if (!onlyDigits) return '';
  return onlyDigits.replace(/^0+(?=\d)/, '');
}

// ═══════════════════════════════════════════════════════
// BUSCA POR COD CP — BACKEND
// Alterado Patty - 10/05/2026
// Motivo: priorizar COD CP; nome/ator só como fallback.
// Exemplo: "10817 — CP TAMAI..." busca somente COD = 10817.
// ═══════════════════════════════════════════════════════

function normalizeSearchTextSherlock_(value) {
  return String(value || '')
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '')
    .replace(/[^a-z0-9\s]/g, ' ')
    .replace(/\s+/g, ' ')
    .trim();
}

function extractCpCodeFromSearch_(value) {
  const raw = String(value || '').trim();
  const match = raw.match(/\d{3,}/);
  if (!match) return '';
  return normalizeCpCode_(match[0]);
}

function matchesCpSearchByCodeFirst_(term, cod, nome, ator) {
  const rawTerm = String(term || '').trim();
  if (!rawTerm) return true;

  const queryCode = extractCpCodeFromSearch_(rawTerm);
  const rowCode   = normalizeCpCode_(cod);

  // Prioridade: se veio código, compara somente COD CP.
  if (queryCode) {
    return rowCode === queryCode;
  }

  // Fallback textual por nome/ator apenas quando não veio código.
  const hay = normalizeSearchTextSherlock_([cod, nome, ator].join(' '));
  const words = normalizeSearchTextSherlock_(rawTerm).split(/\s+/).filter(Boolean);
  if (words.length === 0) return true;
  return words.every(function(w) { return hay.indexOf(w) > -1; });
}

function getCurrentFvcCycleCatalog_() {
  try {
    const ss = SpreadsheetApp.openById(FVC_CARTEIRA_SPREADSHEET_ID);
    let sh = ss.getSheetByName(FVC_CARTEIRA_SHEET_NAME);

    if (!sh) {
      const targetPrefix = 'CARTEIRA FVC 3.0 -Ciclo 06';
      const allSheets = ss.getSheets();
      for (let i = 0; i < allSheets.length; i++) {
        const nm = String(allSheets[i].getName() || '');
        if (nm.indexOf(targetPrefix) === 0) {
          sh = allSheets[i];
          break;
        }
      }
    }

    if (!sh) {
      return {
        success: false,
        error: 'Aba da carteira FVC nao encontrada.',
        sourceUrl: FVC_CARTEIRA_URL,
        sheetName: FVC_CARTEIRA_SHEET_NAME,
        totalCp: 0,
        byCode: {},
        list: []
      };
    }

    const lastRow = sh.getLastRow();
    if (lastRow < 1) {
      return {
        success: true,
        sourceUrl: FVC_CARTEIRA_URL,
        sheetName: sh.getName(),
        totalCp: 0,
        byCode: {},
        list: []
      };
    }

    const values = sh.getRange(1, 1, lastRow, 3).getValues();
    const byCode = {};
    const list = [];

    for (let i = 0; i < values.length; i++) {
      const row = values[i] || [];
      const cod = normalizeCpCode_(row[0]);
      if (!cod) continue;
      if (byCode[cod]) continue;

      const nome = String(row[1] || '').trim();
      const produto = String(row[2] || '').trim();
      const item = { cod: cod, nome: nome, produto: produto };
      byCode[cod] = item;
      list.push(item);
    }

    return {
      success: true,
      sourceUrl: FVC_CARTEIRA_URL,
      sheetName: sh.getName(),
      totalCp: list.length,
      byCode: byCode,
      list: list
    };
  } catch (e) {
    return {
      success: false,
      error: 'Falha ao ler carteira FVC: ' + e.toString(),
      sourceUrl: FVC_CARTEIRA_URL,
      sheetName: FVC_CARTEIRA_SHEET_NAME,
      totalCp: 0,
      byCode: {},
      list: []
    };
  }
}

function getCurrentFvcMissingEmailInsight_(catalog) {
  const fallback = {
    success: false,
    totalCpFvc: 0,
    totalCpSemEmailCadastro: 0,
    cpSemEmailCadastro: [],
    cpSemEmailCadastroTruncado: false,
    error: 'Falha ao analisar CPs sem e-mail cadastrado.'
  };

  try {
    const cache = CacheService.getScriptCache();
    const cached = cache.get(FVC_MISSING_EMAIL_CACHE_KEY);
    if (cached) {
      const parsed = JSON.parse(cached);
      if (parsed && typeof parsed === 'object') return parsed;
    }
  } catch (_ignoreCacheErr) {}

  try {
    const fvcCatalog = catalog && typeof catalog === 'object' ? catalog : getCurrentFvcCycleCatalog_();
    if (!fvcCatalog || !fvcCatalog.success) {
      const failed = Object.assign({}, fallback, {
        error: String((fvcCatalog && fvcCatalog.error) || fallback.error)
      });
      return failed;
    }

    const missing = [];
    const items = Array.isArray(fvcCatalog.list) ? fvcCatalog.list : [];
    for (let i = 0; i < items.length; i++) {
      const item = items[i] || {};
      const cod = normalizeCpCode_(item.cod);
      if (!cod) continue;

      let emails = [];
      try {
        emails = fetchDbEmails_Unified_(cod);
      } catch (_ignoreEmailErr) {
        emails = [];
      }

      const validEmails = Array.isArray(emails)
        ? emails.filter(e => String(e || '').trim().indexOf('@') > 0)
        : [];

      if (validEmails.length > 0) continue;
      missing.push({ cod: cod, nome: String(item.nome || '').trim() });
    }

    missing.sort((a, b) => String(a.cod || '').localeCompare(String(b.cod || ''), 'pt-BR'));

    const result = {
      success: true,
      totalCpFvc: Number(fvcCatalog.totalCp) || 0,
      totalCpSemEmailCadastro: missing.length,
      cpSemEmailCadastro: missing.slice(0, 200),
      cpSemEmailCadastroTruncado: missing.length > 200,
      error: ''
    };

    try {
      const cache = CacheService.getScriptCache();
      cache.put(FVC_MISSING_EMAIL_CACHE_KEY, JSON.stringify(result), 21600);
    } catch (_ignoreCacheWriteErr) {}

    return result;
  } catch (e) {
    return Object.assign({}, fallback, {
      error: 'Falha ao analisar CPs sem e-mail cadastrado: ' + e.toString()
    });
  }
}

function buildFvcOutsideInsight_(cpUniverseCodes, cpNameByCode, cpUnresolvedLabels) {
  const catalog = getCurrentFvcCycleCatalog_();
  const missingEmailInsight = getCurrentFvcMissingEmailInsight_(catalog);
  const sourceMap = cpNameByCode || {};
  const uniq = {};

  const codes = Array.isArray(cpUniverseCodes) ? cpUniverseCodes : [];
  for (let i = 0; i < codes.length; i++) {
    const cod = normalizeCpCode_(codes[i]);
    if (!cod) continue;
    uniq[cod] = true;
  }

  const outside = [];
  if (catalog.success) {
    const allCodes = Object.keys(uniq);
    for (let i = 0; i < allCodes.length; i++) {
      const cod = allCodes[i];
      if (catalog.byCode[cod]) continue;
      outside.push({
        cod: cod,
        nome: String(sourceMap[cod] || '').trim()
      });
    }

    // Registra CPs sem codigo util (nome livre vindo dos logs/abas).
    const unresolved = {};
    const labels = Array.isArray(cpUnresolvedLabels) ? cpUnresolvedLabels : [];
    for (let i = 0; i < labels.length; i++) {
      const label = String(labels[i] || '').trim();
      if (!label) continue;
      if (shouldHideFromRanking_(label)) continue;
      const k = label.toLowerCase();
      if (unresolved[k]) continue;
      unresolved[k] = true;
      outside.push({
        cod: 'SEM_CODIGO',
        nome: label
      });
    }

    outside.sort((a, b) => a.cod.localeCompare(b.cod, 'pt-BR'));
  }

  // PENDING 1 — "Nunca Acessou": CPs da carteira FVC que NÃO aparecem nos logs
  const neverAccessed = [];
  if (catalog.success && catalog.list) {
    for (let i = 0; i < catalog.list.length; i++) {
      const cp = catalog.list[i];
      if (!cp || !cp.cod) continue;
      if (uniq[cp.cod]) continue; // já apareceu nos logs
      neverAccessed.push({ cod: cp.cod, nome: cp.nome || '' });
    }
    neverAccessed.sort((a, b) => a.cod.localeCompare(b.cod, 'pt-BR'));
  }

  return {
    success: !!catalog.success,
    sourceUrl: catalog.sourceUrl,
    sheetName: catalog.sheetName,
    totalCpFvc: Number(catalog.totalCp) || 0,
    totalCpSemEmailCadastro: Number(missingEmailInsight.totalCpSemEmailCadastro) || 0,
    cpSemEmailCadastro: Array.isArray(missingEmailInsight.cpSemEmailCadastro) ? missingEmailInsight.cpSemEmailCadastro : [],
    cpSemEmailCadastroTruncado: !!missingEmailInsight.cpSemEmailCadastroTruncado,
    totalCpForaFvc: outside.length,
    cpForaFvc: outside.slice(0, 200),
    cpForaFvcTruncado: outside.length > 200,
    neverAccessedCount: neverAccessed.length,
    neverAccessed: neverAccessed.slice(0, 300),
    neverAccessedTruncado: neverAccessed.length > 300,
    error: catalog.success ? '' : String(catalog.error || 'Erro ao ler carteira FVC.')
  };
}

function getCurrentFvcCycleSnapshot() {
  const catalog = getCurrentFvcCycleCatalog_();
  return {
    success: !!catalog.success,
    sourceUrl: catalog.sourceUrl,
    sheetName: catalog.sheetName,
    totalCp: Number(catalog.totalCp) || 0,
    sample: Array.isArray(catalog.list) ? catalog.list.slice(0, 200) : [],
    error: catalog.success ? '' : String(catalog.error || 'Erro ao ler carteira FVC.')
  };
}

function toIsoDateLocal_(d) {
  const dt = d instanceof Date ? d : new Date(d);
  if (isNaN(dt.getTime())) return '';
  const yyyy = dt.getFullYear();
  const mm = String(dt.getMonth() + 1).padStart(2, '0');
  const dd = String(dt.getDate()).padStart(2, '0');
  return `${yyyy}-${mm}-${dd}`;
}

function diffDaysInclusive_(startIso, endIso) {
  const s = new Date(String(startIso) + 'T00:00:00');
  const e = new Date(String(endIso) + 'T00:00:00');
  if (isNaN(s.getTime()) || isNaN(e.getTime()) || s > e) return 0;
  const msPerDay = 24 * 60 * 60 * 1000;
  return Math.floor((e - s) / msPerDay) + 1;
}

function computeDisponibilizacaoRegra2025e2026_(filterParams, fallbackValue) {
  try {
    const fallback = Number(fallbackValue) || 0;
    const p = filterParams || {};
    const SCOPE_START = '2025-01-01';
    const SCOPE_END = '2026-12-31';

    const catalog = getCurrentFvcCycleCatalog_();
    const cpBase = Number(catalog && catalog.totalCp) || 0;
    if (!catalog || !catalog.success || cpBase <= 0) {
      return {
        applied: false,
        total: fallback,
        reason: 'carteira_fvc_indisponivel',
        cpBase: 0,
        days: 0,
        start: '',
        end: ''
      };
    }

    const todayIso = toIsoDateLocal_(new Date());
    let start = String(p.dateFrom || '').trim();
    let end = String(p.dateTo || '').trim();

    // Sem filtro de data explicito: preserva comportamento atual (2026 acumulado ate hoje).
    if (!start) start = '2026-01-01';
    if (!end) end = todayIso;

    // Limita a janela ao escopo 2025-2026 e ao dia atual.
    if (end < SCOPE_START || start > SCOPE_END) {
      return {
        applied: false,
        total: fallback,
        reason: 'fora_do_escopo_2025_2026',
        cpBase: cpBase,
        days: 0,
        start: '',
        end: ''
      };
    }

    const startClamped = start < SCOPE_START ? SCOPE_START : start;
    const endScopeClamped = end > SCOPE_END ? SCOPE_END : end;
    const maxEnd = todayIso < SCOPE_END ? todayIso : SCOPE_END;
    const endClamped = endScopeClamped > maxEnd ? maxEnd : endScopeClamped;

    const days = diffDaysInclusive_(startClamped, endClamped);
    if (days <= 0) {
      return {
        applied: false,
        total: fallback,
        reason: 'janela_invalida',
        cpBase: cpBase,
        days: 0,
        start: startClamped,
        end: endClamped
      };
    }

    const REPORTS_PER_DAY = 2;
    const total = cpBase * REPORTS_PER_DAY * days;
    return {
      applied: true,
      total: total,
      reason: 'regra_teorica_2025_2026',
      cpBase: cpBase,
      days: days,
      start: startClamped,
      end: endClamped,
      sourceUrl: catalog.sourceUrl,
      sheetName: catalog.sheetName,
      reportsPerDay: REPORTS_PER_DAY
    };
  } catch (e) {
    return {
      applied: false,
      total: Number(fallbackValue) || 0,
      reason: 'erro_regra_2025_2026: ' + e.toString(),
      cpBase: 0,
      days: 0,
      start: '',
      end: ''
    };
  }
}

function internal_processSheetsData_(filterParams) {
    const adminEmails = ['patricia.wojciechowski@grupoboticario.com.br', 'opatriciakelly@gmail.com'];
    const cpProductMap = getCpProductMap_(); // Mapeamento dinâmico de produtos

  // Controle de auditoria: sÃ³ esses usuÃ¡rios podem ver o campo 'Registro Auditado'
  const AUDIT_ALLOWED_EMAILS = [
    'patricia.wojciechowski@grupoboticario.com.br',
    'opatriciakelly@gmail.com'
  ];
  let canAudit = false;
  try {
    const activeUser = Session.getActiveUser().getEmail().toLowerCase();
    canAudit = AUDIT_ALLOWED_EMAILS.some(ae => activeUser === ae.toLowerCase());
  } catch(e) { canAudit = false; }
  
  try {
    const SPREADSHEET_ID = '1cJjAOlnVn1sUxq36Bhvay_WCCPF4jKo7KNFtQBYXNNs'; 
    const ss = SpreadsheetApp.openById(SPREADSHEET_ID);
    const sheets = ss.getSheets();
    
    let resultData = [];
    let stats = { audit: 0, dispo: 0, cps: 0, cpAggregation: {}, types: new Set(), trend: {}, sugs: new Set(), historyTotal: 0 };
    let auditSeen = new Set(); 
    let weeklySeen = new Set(); 

    // CÃ¡lculo Bruto de HistÃ³rico (Massa Total de Dados Disponibilizados pelo Radar)
    for (let s of sheets) {
      const lr = s.getLastRow();
      if (lr > 1) stats.historyTotal += (lr - 1) * 10;
    }

    for (let s of sheets) {
      const sName = s.getName();
    // FILTRO DE SEGURANÃ‡A v5.9.7: Pula abas de logs de sistema, mÃ©tricas internas ou configuraÃ§Ãµes
    if (sName.startsWith('_') || sName.toLowerCase().includes('log') || sName.toLowerCase().includes('conf')) continue;

      const lastRow = s.getLastRow();
      if (lastRow < 2) continue;
      
      const data = s.getRange(1, 1, lastRow, 8).getValues();
      
      let startIdx = 0;
      for (let r=0; r<20; r++) {
         const rs = data[r].join('').toUpperCase();
         if (rs.includes('DATA') || rs.includes('TIPO') || rs.includes('COD')) { startIdx = r + 1; break; }
      }

      // --- PASSO 1: PROCV REVERSO SEGURO ---
      let emailAssoc = {};
      for (let i = startIdx; i < data.length; i++) {
        const row = data[i];
        let cod = String(row[1] || 'Unknown').trim();
        let nom = String(row[2] || '').trim();
        let email = String(row[4] || '').toLowerCase().trim();
        let tipo = String(row[0] || '').toLowerCase();
        
        if (tipo.includes('dlp')) continue; // ignora logs de sistema DLP Rule
        
        if (cod !== 'Unknown' && cod !== '' && email) {
            if (!emailAssoc[email]) {
                emailAssoc[email] = { cods: new Set(), lastNom: nom };
            }
            emailAssoc[email].cods.add(cod);
            emailAssoc[email].lastNom = nom;
        }
      }

      let safeEmailToCP = {};
      for (let e in emailAssoc) {
          // Mapeia se, e SOMENTE SE, acessou apenas 1 CP em toda a vida na base de logs!
          if (emailAssoc[e].cods.size === 1) {
              safeEmailToCP[e] = { cod: Array.from(emailAssoc[e].cods)[0], nom: emailAssoc[e].lastNom };
          }
      }

      // --- PASSO 2: AGREGAÃ‡ÃƒO & LÃ“GICA DE VIEW ---
      const limitList = 3000;
      
      for (let i = data.length - 1; i >= startIdx; i--) {
        const row = data[i];
        const email = String(row[4] || '').toLowerCase().trim();
        const isAdmin = adminEmails.some(ae => email.includes(ae.toLowerCase()));
        
        let type = String(row[0] || 'Report').trim().replace(/_/g, ' ');

        // â”€â”€ DECLARAÃ‡ÃƒO ANTECIPADA DE nom (evita uso antes da declaraÃ§Ã£o) â”€â”€
        let cod = String(row[1] || 'Unknown').trim();
        // Normaliza nome: remove sufixo de nÃºmero de loja (ex: "NÂº378", "NÂ°12", "N 5")
        // Garante que "CP FULANO NÂº378" e "CP FULANO" sejam o mesmo CP na agregaÃ§Ã£o
        let nom = String(row[2] || '').trim().replace(/\s+N[Â°Âº]?\s*\d+\s*$/i, '').trim();

        // FILTRO ANTI-RUÃDO: Exclui rastros de auditoria mecÃ¢nica/robÃ´s
        if (type.toLowerCase().includes('dlp') || email.includes('dlp') || email.includes('rule') || nom.includes('Acesso a Plataforma')) continue;

        // DisponibilizaÃ§Ã£o: conta apenas linhas cujo RESPONSÃVEL (ATOR) / coluna E contÃ©m Wojciechowski.
        const isWoj = email.includes('wojciechowski');

        // â”€â”€ NORMALIZAÃ‡ÃƒO DO TIPO DE REPORT â”€â”€
        // Inclui: Report CA, Report_CA, Outro (com CA no log), etc.
        const typeUpper = type.toUpperCase();
        const logName   = String(row[6] || '').toUpperCase(); // coluna ARQUIVO_LOG

        if (typeUpper.includes('CA') || logName.includes('REPORT_CA') || logName.includes('REPORT CA')) {
          type = 'Report CA';
        } else if (typeUpper.includes('CI') || logName.includes('REPORT_CI') || logName.includes('REPORT CI')) {
          type = 'Report CI';
        } else if (typeUpper === 'OUTRO' || typeUpper === 'OTHER') {
          // "Outro" com COD vÃ¡lido: inferir tipo pelo log ou manter como Report CI (default)
          type = logName.includes('CA') ? 'Report CA' : 'Report CI';
        } else {
          type = 'Report'; // genÃ©rico (sem CI/CA identificÃ¡vel)
        }


        // APLICA O PROCV REVERSO SEGURO
        if ((cod === 'Unknown' || cod === '') && safeEmailToCP[email]) {
            cod = safeEmailToCP[email].cod;
            nom = safeEmailToCP[email].nom;
        }
        
        // Expurgo LGPD & Lixo Base: Linhas que nÃ£o identificam CP nem por Nome nem por CÃ³digo nÃ£o tÃªm valor analÃ­tico
        if ((cod === 'Unknown' || cod === '') && nom === '') continue;

        // SEMPRE COLETA TODAS AS CHAVES GLOBAIS PARA OS MENUS
        if (!isAdmin) {
          if (cod !== 'Unknown' && cod !== '') {
            stats.sugs.add(cod);
          }
        }

        let dateVal = row[3];
        let dCtx = getNativeDateContext(dateVal);
        let dateKey = dCtx ? dCtx.k : "";
        let fDate = dCtx ? dCtx.f : String(dateVal);
        let yw = dCtx ? dCtx.w : "Unknown";

        // VERIFICA SE ESTA LINHA PASSA NOS FILTROS ESCOLHIDOS PELO USUÃRIO NO FRONTEND
        let matchType = true;
        let matchTerm = true;
        let matchDate = true;

        if (filterParams) {
           if (filterParams.type && filterParams.type !== "") {
               if (type !== filterParams.type) matchType = false;
           }
           if (filterParams.term && filterParams.term !== "") {
               if (!matchesCpSearchByCodeFirst_(filterParams.term, cod, nom, ator)) matchTerm = false;
           }
            // Filtro de periodo: sem restriÃ§Ã£o (comeÃ§a do primeiro dia disponÃ­vel)
            if (dateKey !== '') {
                if (filterParams && filterParams.dateFrom && dateKey < filterParams.dateFrom) matchDate = false;
                if (filterParams && filterParams.dateTo && dateKey > filterParams.dateTo) matchDate = false;
             }
         }
        // AGREGAÃ‡Ã•ES DA INTELIGÃŠNCIA APENAS SE A LINHA PASSOU PELO FILTRO
        if (matchType && matchTerm && matchDate) {
            // Se o e-mail no log veio oculto/Unknown, tenta adivinhar cruzando o nome do cara (nom) com os e-mails homologados do CP (cod)
            if (!email || email === 'unknown' || email === '---') {
                if (cod && cod !== 'Unknown') {
                    // cache local para nÃ£o buscar no Drive centenas de vezes
                    if (!stats.cpEmailCache) stats.cpEmailCache = {};
                    if (stats.cpEmailCache[cod] === undefined) {
                        try { stats.cpEmailCache[cod] = fetchDbEmails_Unified_(cod); } catch(e) { stats.cpEmailCache[cod] = []; }
                    }
                    const permitidos = stats.cpEmailCache[cod];
                    if (permitidos && permitidos.length > 0) {
                        // Tenta achar um email que contenha o primeiro nome
                        const p1 = (nom || '').split(' ')[0].toLowerCase().normalize("NFD").replace(/[\u0300-\u036f]/g, "");
                        let matched = permitidos.find(e => p1.length > 2 && e.includes(p1));
                        if (matched) {
                            email = matched;
                        } else if (permitidos.length === 1) {
                            email = permitidos[0]; // Se sÃ³ tem 1 cara no CP, deve ser ele
                        }
                    }
                }
            }

            if (isWoj) {
              stats.dispo += 1;
            } else {
              // Volumetria Real: Regra de Unicidade por E-mail por CP por Dia (Checklist Item 2)
              const uniqAuditKey = `${cod}|${email}|${dateKey}`;
              if (!auditSeen.has(uniqAuditKey) && email && email !== 'unknown') {
                auditSeen.add(uniqAuditKey);
                
                // Multiplicador Dinâmico baseada em Produtos (Checklist Item 4)
                const normCod = normalizeCpCode_(cod);
                const prods = cpProductMap[normCod] || 'CI'; 
                const multiplier = (prods.includes('CI') && prods.includes('CA')) ? 2 : 1;
                const auditValue = 1 * multiplier; 

                stats.audit += auditValue;
                
                let aggKey = cod && cod !== 'Unknown' ? cod : nom;
                if (cod && cod !== 'Unknown') {
                    if (!stats.cpNames) stats.cpNames = {};
                    stats.cpNames[cod] = nom;
                }
                stats.cpAggregation[aggKey] = (stats.cpAggregation[aggKey] || 0) + auditValue;
                
                if (dateKey !== "") {
                    stats.trend[dateKey] = (stats.trend[dateKey] || 0) + auditValue;
                }
              }

              // Registro Auditado (LGPD) — mascara intercalada estilo maskEmail25 (25% central)
              let registroAuditado = null;
              if (canAudit) {
                if (email && email.includes('@')) {
                    const [localPart, domainFull] = email.split('@');
                    const isServiceAcc = /^(noreply|no-reply|bot|googleapis|admin|postmaster|mailer|dlp)/i.test(localPart);
                    if (isServiceAcc) {
                        registroAuditado = 'â€” sistema â€”';
                    } else {
                        // maskCenter_: mascara pct% do centro da string
                        const maskCenter = (s, pct) => {
                          if (s.length <= 3) { if(s.length<=1) return s; if(s.length===2) return s[0]+'*'; return s[0]+'*'+s[2]; }
                          const k = Math.max(1, Math.round(s.length * pct));
                          const start = Math.floor((s.length - k) / 2);
                          return s.slice(0, start) + '*'.repeat(k) + s.slice(start + k);
                        };
                        const userMasked = maskCenter(localPart, 0.25);
                        const parts = domainFull.split('.');
                        const head = parts.shift() || '';
                        const headMasked = head.length >= 4 ? maskCenter(head, 0.20) : head;
                        const domMasked = [headMasked, ...parts].join('.');
                        registroAuditado = userMasked + '@' + domMasked;
                    }
                } else if (email) {
                    const k = Math.max(1, Math.round(email.length * 0.25));
                    const start = Math.floor((email.length - k) / 2);
                    registroAuditado = email.slice(0, start) + '*'.repeat(k) + email.slice(start + k);
                } else {
                    registroAuditado = '---';
                }
              }

              // Preenche na tela os logs duplicados metodologicamente (1 para CA, 1 para CI)
              if (resultData.length < limitList - 1) {
                resultData.push({ t: 'Report CA', c: cod, n: nom, d: fDate, e: registroAuditado, i: String(row[5]||''), v: auditValue / 2 });
                resultData.push({ t: 'Report CI', c: cod, n: nom, d: fDate, e: registroAuditado, i: String(row[5]||''), v: auditValue / 2 });
              }
            }
        }
      }
      // Continua para prÃ³ximas abas (nÃ£o usa break, pois dados podem estar em mÃºltiplas abas)
    }

    const rankingEntries = Object.entries(stats.cpAggregation)
      .filter(e => e[1] > 0)
      .map(e => {
        const theCod = e[0];
        const theNom = (stats.cpNames && stats.cpNames[theCod]) ? stats.cpNames[theCod] : theCod;
        const dispName = theCod !== theNom ? `${theNom}` : theCod;
        return { cod: theCod, name: dispName, val: e[1] };
      })
      .filter(e => !shouldHideFromRanking_(e.name) && !shouldHideFromRanking_(e.cod));

    const top10 = rankingEntries
      .slice()
      .sort((a,b) => b.val - a.val)
      .slice(0, 10)
      .map(e => ({ name: e.name, val: e.val, cod: e.cod }));

    // Ranking Inferior (Filtra quem pelo menos abriu 1 vez, mas ficou com mÃ©trica baixa)
    const bottom10 = rankingEntries
      .slice()
      .sort((a,b) => a.val - b.val)
      .slice(0, 10)
      .map(e => ({ name: e.name, val: e.val, cod: e.cod }));

    const cpUniverseCodes = [];
    const cpNameByCode = {};
    const cpUnresolvedLabels = [];
    const cpKeys = Object.keys(stats.cpAggregation || {});
    for (let i = 0; i < cpKeys.length; i++) {
      const rawCode = cpKeys[i];
      const normCode = normalizeCpCode_(rawCode);
      const cpName = (stats.cpNames && stats.cpNames[rawCode]) ? stats.cpNames[rawCode] : rawCode;
      if (!normCode) {
        cpUnresolvedLabels.push(String(cpName || rawCode || '').trim());
        continue;
      }
      cpUniverseCodes.push(normCode);
      if (!cpNameByCode[normCode]) cpNameByCode[normCode] = String(cpName || '').trim();
    }
    const fvcOutsideCurrentCycle = buildFvcOutsideInsight_(cpUniverseCodes, cpNameByCode, cpUnresolvedLabels);
    const pendingFranchiseDrives = Array.isArray(fvcOutsideCurrentCycle.cpSemEmailCadastro)
      ? fvcOutsideCurrentCycle.cpSemEmailCadastro.map(cp => {
          const cod = String(cp && cp.cod ? cp.cod : '').trim();
          const nome = String(cp && cp.nome ? cp.nome : '').trim();
          return nome ? `${cod} - ${nome}` : cod;
        }).filter(Boolean)
      : [];
    const pendingFranchiseDriveCount = Number(fvcOutsideCurrentCycle.totalCpSemEmailCadastro) || pendingFranchiseDrives.length;
    const dispoRule = computeDisponibilizacaoRegra2025e2026_(filterParams, stats.dispo);
    const fvcMetadata = getFvcMetadata_();
    const cpProductMap = fvcMetadata.map;
    const semFvcCount = fvcMetadata.semFvcCount;
    
    const totalCps = Object.keys(cpAgg).length;
    const auditTotal = auditSeen.size * 2; 
    
    // Separação por ano (Melhoria 09/05 19:10)
    let audit2025 = 0;
    let audit2026 = 0;
    auditSeen.forEach(key => {
        const parts = key.split('|');
        const dateKey = parts[2];
        if (dateKey && dateKey.startsWith('2025')) audit2025 += 2;
        else if (dateKey && dateKey.startsWith('2026')) audit2026 += 2;
    });

    // BLOCO 4 FIX: qualityTotal dinâmico — CPs com e-mail válido / total carteira FVC
    const fvcCatalogSheets = getCurrentFvcCycleCatalog_();
    const fvcCatalogTotalSheets = (fvcCatalogSheets && fvcCatalogSheets.success && fvcCatalogSheets.totalCp > 0) ? fvcCatalogSheets.totalCp : totalCps;
    const cpsComEmailSheets = Math.max(0, totalCps - pendingFranchiseDriveCount);
    const qualityTotal = fvcCatalogTotalSheets > 0 ? Math.min(100, (cpsComEmailSheets / fvcCatalogTotalSheets) * 100) : 0;

    const response = {
      success: true,
      canAudit: canAudit,
      stats: {
        audit: auditTotal,
        audit2025: audit2025,
        audit2026: audit2026,
        dispo: Number(dispoRule.total) || 0,
        dispoRule: dispoRule,
        cps: totalCps,
        semFvcCount: semFvcCount,
        qualityTotal: qualityTotal,
        top10: top10,
        bottom10: bottom10,
        types: Array.from(stats.types).sort(),
        trend: trend,
        historyTotal: audit,
        cpUniverseCodes: cpUniverseCodes,
        fvcOutsideCurrentCycle: fvcOutsideCurrentCycle,
        pendingFranchiseDrives: pendingFranchiseDrives,
        pendingFranchiseDriveCount: pendingFranchiseDriveCount,
        suggestions: Array.from(sugs)
          .map(function(s) { return normalizeCpCode_(s); })
          .filter(function(s) { return s && s !== 'Unknown'; })
          .sort(function(a, b) { return a.localeCompare(b, 'pt-BR', { numeric: true }); }),
        cpNames: stats.cpNames || {},
        lastUpdate: new Date().toISOString()
      },
      list: resultData
    };

    return response;

  } catch (e) {
    return { error: e.toString() };
  }
}

// ==== MOTOR DE PERFORMANCE JSON (v5.9.7) ====
const CACHE_FILE_NAME = 'sherlock_v59_snapshot.json';

// ═══════════════════════════════════════════════════════════
// MÉTRICAS OFICIAIS — SHERLOCK VIEWS
// Alterado Patty - 10/05/2026
// Motivo:
// - Separar views auditadas de views potenciais/capacidade.
// - Regra auditada: CP + e-mail/ator + dia × 2 reports.
// - Regra potencial: CPs no cesto × e-mails esperados × 2 reports × dias.
// - A partir de 05/2026, considerar 4 e-mails por CP.
// ═══════════════════════════════════════════════════════════

const SHERLOCK_OFFICIAL_2025_VIEWS          = 56000; // ajuste para o número exato validado, se necessário
const SHERLOCK_CP_CESTO_OFICIAL             = 378;
const SHERLOCK_REPORTS_POR_CP               = 2;     // CI + CA
const SHERLOCK_EMAILS_POR_CP_ATE_202604     = 2;
const SHERLOCK_EMAILS_POR_CP_APOS_202605    = 4;
const SHERLOCK_ABERTURAS_POR_REPORT_DIA     = 1;     // troque para 2 se quiser considerar 2 aberturas por report/dia
const SHERLOCK_OFFICIAL_SEM_FVC             = 40;    // CPs não ativos (fora da carteira FVC ativa)

// Modo do card principal:
// AUDITED_PLUS_HISTORICAL = card usa 2025 oficial + 2026 auditado pelo Sherlock
// POTENTIAL_CAPACITY      = card usa 2025 oficial + capacidade/potencial 2026
const SHERLOCK_CARD_METRIC_MODE = 'AUDITED_PLUS_HISTORICAL';

function sherlockToIsoDateLocal_(d) {
  const dt = d instanceof Date ? d : new Date(d);
  if (isNaN(dt.getTime())) return '';
  const yyyy = dt.getFullYear();
  const mm = String(dt.getMonth() + 1).padStart(2, '0');
  const dd = String(dt.getDate()).padStart(2, '0');
  return `${yyyy}-${mm}-${dd}`;
}

function sherlockDiffDaysInclusive_(startIso, endIso) {
  const s = new Date(String(startIso) + 'T00:00:00');
  const e = new Date(String(endIso)   + 'T00:00:00');
  if (isNaN(s.getTime()) || isNaN(e.getTime()) || s > e) return 0;
  return Math.floor((e - s) / (24 * 60 * 60 * 1000)) + 1;
}

function sherlockIntersectionDays_(rangeStart, rangeEnd, blockStart, blockEnd) {
  const start = String(rangeStart) > String(blockStart) ? String(rangeStart) : String(blockStart);
  const end   = String(rangeEnd)   < String(blockEnd)   ? String(rangeEnd)   : String(blockEnd);
  return sherlockDiffDaysInclusive_(start, end);
}

function calculateSherlockPotentialViews2026_(dateFrom, dateTo) {
  const todayIso = sherlockToIsoDateLocal_(new Date());
  const start    = String(dateFrom || '2026-01-01');
  const endRaw   = String(dateTo   || todayIso);
  const end      = endRaw > todayIso ? todayIso : endRaw;

  const daysBeforeMay = sherlockIntersectionDays_(start, end, '2026-01-01', '2026-04-30');
  const daysFromMay   = sherlockIntersectionDays_(start, end, '2026-05-01', '2026-12-31');

  const viewsPerCpDayBeforeMay = SHERLOCK_EMAILS_POR_CP_ATE_202604  * SHERLOCK_REPORTS_POR_CP * SHERLOCK_ABERTURAS_POR_REPORT_DIA;
  const viewsPerCpDayFromMay   = SHERLOCK_EMAILS_POR_CP_APOS_202605 * SHERLOCK_REPORTS_POR_CP * SHERLOCK_ABERTURAS_POR_REPORT_DIA;

  const totalBeforeMay = SHERLOCK_CP_CESTO_OFICIAL * viewsPerCpDayBeforeMay * daysBeforeMay;
  const totalFromMay   = SHERLOCK_CP_CESTO_OFICIAL * viewsPerCpDayFromMay   * daysFromMay;

  return {
    success: true,
    start: start,
    end: end,
    cpBase: SHERLOCK_CP_CESTO_OFICIAL,
    reportsPerCp: SHERLOCK_REPORTS_POR_CP,
    openingsPerReportDay: SHERLOCK_ABERTURAS_POR_REPORT_DIA,
    emailsPerCpBeforeMay: SHERLOCK_EMAILS_POR_CP_ATE_202604,
    emailsPerCpFromMay:   SHERLOCK_EMAILS_POR_CP_APOS_202605,
    daysBeforeMay: daysBeforeMay,
    daysFromMay:   daysFromMay,
    viewsPerCpDayBeforeMay:   viewsPerCpDayBeforeMay,
    viewsPerCpDayFromMay:     viewsPerCpDayFromMay,
    viewsAllCpsDayBeforeMay:  SHERLOCK_CP_CESTO_OFICIAL * viewsPerCpDayBeforeMay,
    viewsAllCpsDayFromMay:    SHERLOCK_CP_CESTO_OFICIAL * viewsPerCpDayFromMay,
    totalBeforeMay: totalBeforeMay,
    totalFromMay:   totalFromMay,
    total: totalBeforeMay + totalFromMay,
    rule: 'CPs × e-mails por CP × 2 reports × aberturas por report/dia'
  };
}

function applySherlockOfficialMetrics_(data) {
  if (!data || !data.stats) return data;
  const stats = data.stats;

  // ═══════════════════════════════════════════════════════
  // BLOQUEIO DE REAUDITORIA
  // Alterado Patty - 11/05/2026
  // Motivo: se o Python v6.1 já enviou stats.cards, esta função
  // não pode sobrescrever os números oficiais com regra antiga
  // CP + e-mail/ator + dia ×2.
  // ═══════════════════════════════════════════════════════
  if (stats.cards && Number(stats.cards.viewsOficiais || 0) > 0) {
    const cards = stats.cards;

    stats.audit2025 = Number(cards.views2025 || 0);
    stats.audit2026 = Number(cards.views2026 || 0);
    stats.audit = Number(cards.viewsOficiais || (stats.audit2025 + stats.audit2026));

    stats.viewsRaw2025 = stats.audit2025;
    stats.viewsRaw2026 = stats.audit2026;
    stats.viewsRaw = stats.audit;

    stats.viewsOfficial2025 = stats.audit2025;
    stats.viewsOfficial2026 = stats.audit2026;
    stats.viewsOficiais = stats.audit;
    stats.viewsEmailMaioPlus = Number(cards.viewsEmailMaioPlus || 0);

    stats.dispo = Number(cards.disponibilizacoes || stats.dispo || 0);
    stats.cps = Number(cards.cpsImpactados || stats.cps || 0);
    stats.semFvcCount = Number(cards.cpsSemFvc || stats.semFvcCount || 40);
    stats.qualityTotal = Number(cards.identificacaoSherlock || stats.qualityTotal || 74.84);

    stats.businessMetricMode = 'PYTHON_STATS_CARDS_OFFICIAL';
    stats.businessMetricDescription =
      'Views oficiais vindas do Python/Bloco 2 v6.1 via stats.cards. O Apps Script não recalcula CP+e-mail+dia×2.';

    return data;
  }

  const rawAudit  = Number(stats.audit)                            || 0;
  const raw2025   = Number(stats.viewsRaw2025  || stats.audit2025) || 0;
  const raw2026   = Number(stats.viewsRaw2026  || stats.audit2026) || 0;
  const unique2025 = Number(stats.auditUnique2025)                 || 0;
  const unique2026 = Number(stats.auditUnique2026)                 || 0;

  const official2025   = Math.max(SHERLOCK_OFFICIAL_2025_VIEWS, raw2025, unique2025);
  const audited2026    = unique2026 > 0 ? unique2026 : raw2026;
  const potential2026  = calculateSherlockPotentialViews2026_('2026-01-01', null);

  // Alterado Patty - 10/05/2026
  // Métrica: Views após ajuste do campo e-mail (maio em diante, 4 e-mails/CP)
  const todayIsoForEmailAdjustment = sherlockToIsoDateLocal_(new Date());
  const daysFromMayForEmailAdjustment = sherlockIntersectionDays_(
    '2026-05-01', todayIsoForEmailAdjustment, '2026-05-01', '2026-12-31'
  );
  const distinctCpCodes = {};
  if (Array.isArray(stats.cpUniverseCodes)) {
    stats.cpUniverseCodes.forEach(function(codRaw) {
      const cod = normalizeCpCode_(codRaw);
      if (cod) distinctCpCodes[cod] = true;
    });
  }
  const cpBaseDistinct = Object.keys(distinctCpCodes).length;
  const cpBaseAfterEmailAdjustment = cpBaseDistinct > 0
    ? cpBaseDistinct
    : Math.max(0, SHERLOCK_CP_CESTO_OFICIAL - SHERLOCK_OFFICIAL_SEM_FVC);
  const viewsAfterEmailAdjustmentFromMay =
    cpBaseAfterEmailAdjustment *
    SHERLOCK_EMAILS_POR_CP_APOS_202605 *
    SHERLOCK_REPORTS_POR_CP *
    SHERLOCK_ABERTURAS_POR_REPORT_DIA *
    daysFromMayForEmailAdjustment;

  // Rastreabilidade
  stats.auditRaw           = rawAudit;
  stats.audit2025Raw       = raw2025;
  stats.audit2026Raw       = raw2026;
  stats.auditUnique2025Raw = unique2025;
  stats.auditUnique2026Raw = unique2026;

  stats.viewsOfficial2025        = official2025;
  stats.viewsAudited2026         = audited2026;
  stats.viewsPotential2026       = potential2026.total;
  stats.viewsPotential2026Details = potential2026;
  stats.viewsPerCpDayFromMay2026  = potential2026.viewsPerCpDayFromMay;
  stats.viewsAllCpsDayFromMay2026 = potential2026.viewsAllCpsDayFromMay;
  stats.viewsAfterEmailAdjustmentFromMay = viewsAfterEmailAdjustmentFromMay;
  stats.viewsAfterEmailAdjustmentDetails = {
    start: '2026-05-01',
    end: todayIsoForEmailAdjustment,
    cpBase: cpBaseAfterEmailAdjustment,
    emailsPerCp: SHERLOCK_EMAILS_POR_CP_APOS_202605,
    reportsPerCp: SHERLOCK_REPORTS_POR_CP,
    openingsPerReportDay: SHERLOCK_ABERTURAS_POR_REPORT_DIA,
    days: daysFromMayForEmailAdjustment,
    rule: 'COD CP distinto × 4 e-mails × 2 reports × dias desde maio/2026'
  };

  // Fallback: stats.cards não existe ou viewsOficiais = 0 → usa regra CP+e-mail+dia×2
  if (SHERLOCK_CARD_METRIC_MODE === 'POTENTIAL_CAPACITY') {
    stats.audit2025 = official2025;
    stats.audit2026 = potential2026.total;
    stats.audit     = official2025 + potential2026.total;
    stats.businessMetricMode = 'POTENTIAL_CAPACITY_378_CP_EMAILS_REPORTS';
  } else {
    stats.audit2025 = official2025;
    stats.audit2026 = audited2026;
    stats.audit     = official2025 + audited2026;
    stats.businessMetricMode = 'AUDITED_CP_EMAIL_DAY_X2_PLUS_2025_OFFICIAL';
  }

  stats.businessMetricDescription =
    'Views = CP + e-mail/ator + dia × 2 reports. Capacidade = CPs × e-mails esperados × 2 reports × dias.';

  stats.cpsRaw = Number(stats.cps) || 0;
  stats.cps    = Math.max(stats.cpsRaw, SHERLOCK_CP_CESTO_OFICIAL);

  stats.semFvcCount  = SHERLOCK_OFFICIAL_SEM_FVC;
  stats.semFvcSource = 'FIXO_MANUAL_VALIDADO_CPS_NAO_ATIVOS';

  stats.fvcOutsideCurrentCycle = {
    success: true,
    sourceMethod: 'SNAPSHOT_ONLY_MANUAL',
    totalCpFvc: 0,
    totalCpForaFvc: SHERLOCK_OFFICIAL_SEM_FVC,
    cpForaFvc: [],
    cpForaFvcTruncado: false,
    error: ''
  };

  return data;
}

function normalizeFilterParams_(filterParams) {
  const p = Object.assign({}, filterParams || {});
  const period = String(p.period || '').trim();

  if (period && /^\d{4}-\d{2}$/.test(period)) {
    const parts = period.split('-');
    const y = Number(parts[0]);
    const m = Number(parts[1]);
    const lastDay = new Date(y, m, 0).getDate();
    if (!p.dateFrom) p.dateFrom = `${parts[0]}-${parts[1]}-01`;
    if (!p.dateTo) p.dateTo = `${parts[0]}-${parts[1]}-${String(lastDay).padStart(2, '0')}`;
  }

  return p;
}

function hasActiveUserFilters_(filterParams) {
  if (!filterParams) return false;
  return Boolean(
    (filterParams.type && String(filterParams.type).trim() !== '') ||
    (filterParams.term && String(filterParams.term).trim() !== '') ||
    (filterParams.dateFrom && String(filterParams.dateFrom).trim() !== '') ||
    (filterParams.dateTo && String(filterParams.dateTo).trim() !== '') ||
    (filterParams.period && String(filterParams.period).trim() !== '')
  );
}

/************************************************************
 * SHERLOCK — LEITURA OFICIAL DOS ARQUIVOS RESUMO
 * Patty — 10/05/2026
 * Motivo: cards 2025/2026 estavam usando snapshot/unique (194 views).
 * Correção: ler resumo_2025 e resumo_2026 diretamente das Sheets.
 ************************************************************/

var SH_RESUMO_2025_SS_ID  = '1xsP5ggBCmt5YjUlGocOs45Zic4JFdGBrz85wVs-_VOQ';
var SH_RESUMO_2026_SS_ID  = '1UK0mI2-Jc-qSChSuf0BoZ7Owvl-GSorkS9bJsQH959w';
var SH_RESUMO_2025_SHEET  = 'resumo_2025';
var SH_RESUMO_2026_SHEET  = 'resumo_2026';
var SH_RESUMO_CHUNK_SIZE  = 10000;   // linhas por leitura para não estourar memória
var SH_RESUMO_MAX_LIST    = 25000;   // 500 páginas × 50 linhas para o Explorer

// ═══════════════════════════════════════════════════════════
// ADMIN — EXCLUIR SOMENTE E-MAILS DEV/ADMIN ESPECÍFICOS
// Alterado Patty - 11/05/2026
// Motivo: o Python já tratou a base oficial.
// Não remover domínio inteiro como redeboticario.com.br.
// Não tratar e-mail vazio como admin automaticamente.
// ═══════════════════════════════════════════════════════════

var SH_RESUMO_ADMIN_EMAILS_EXATOS = [
  'patricia.wojciechowski@grupoboticario.com.br',
  'opatriciakelly@gmail.com',
  'opatricia.kelly@gmail.com'
];

function SH_isResumoAdminEmail_(email) {
  var e = String(email || '').toLowerCase().trim();

  if (!e) return false;

  e = e.replace(/^mailto:/i, '').replace(/[<>"]/g, '');

  if (SH_RESUMO_ADMIN_EMAILS_EXATOS.indexOf(e) >= 0) return true;

  if (/^patricia\.wojciechowsk?i@grupoboticario\.com\.br$/i.test(e)) return true;
  if (/^opatricia\.?kelly@gmail\.com$/i.test(e)) return true;

  return false;
}

// ═══════════════════════════════════════════════════════
// SHERLOCK — QUALIDADE DO RESUMO OFICIAL
// Alterado Patty - 11/05/2026
// Motivo: impedir que ruídos e datas futuras contaminem Explorer, Analytics e cards.
// ═══════════════════════════════════════════════════════

// Enquanto a última carga manual oficial for 30/04/2026.
// Quando houver nova carga manual validada, atualizar esta constante.
var SH_RESUMO_MAX_ACCESS_DATE = '2026-04-30';

function SH_isResumoNoiseCp_(codCp, nomeCp) {
  var txt = String([codCp, nomeCp].join(' ') || '')
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '')
    .trim();

  if (!txt) return true;

  // Pega tanto texto normal quanto variações com problema de encoding,
  // pois "Evento N\u00c3\u00a3o Registrado ou Perdido" ainda contém evento/registrado/perdido.
  if (txt.includes('unknown'))                        return true;
  if (txt.includes('desconhecido'))                   return true;
  if (txt.includes('sem log'))                        return true;
  if (txt.includes('sem registro'))                   return true;
  if (txt.includes('capturado mas sem'))              return true;
  if (txt.includes('evento') && txt.includes('registrado')) return true;
  if (txt.includes('evento') && txt.includes('perdido'))    return true;

  return false;
}

function SH_getResumoMaxDate_() {
  var todayIso = Utilities.formatDate(
    new Date(),
    Session.getScriptTimeZone(),
    'yyyy-MM-dd'
  );
  // Usa a menor data entre hoje e a última carga oficial conhecida.
  // Evita exibir maio/dezembro quando a última carga validada foi 30/04.
  if (SH_RESUMO_MAX_ACCESS_DATE && SH_RESUMO_MAX_ACCESS_DATE < todayIso) {
    return SH_RESUMO_MAX_ACCESS_DATE;
  }
  return todayIso;
}

function SH_normalizeTipoReport_(tipo) {
  var t = String(tipo || '').trim().toUpperCase();
  if (t === 'CI') return 'Report CI';
  if (t === 'CA') return 'Report CA';
  if (t === 'REPORT') return 'Report';
  if (t === 'OUTRO' || t === 'OUTROS') return 'Acesso ao Drive';
  if (t === 'ACESSO AO DRIVE') return 'Acesso ao Drive';
  return String(tipo || 'Report').trim() || 'Report';
}

function SH_parseResumoDate_(value) {
  if (!value) return null;
  if (Object.prototype.toString.call(value) === '[object Date]' && !isNaN(value.getTime())) {
    return value;
  }
  var txt = String(value).trim();
  var iso = txt.match(/^(\d{4})-(\d{2})-(\d{2})/);
  if (iso) return new Date(Number(iso[1]), Number(iso[2]) - 1, Number(iso[3]));
  var br = txt.match(/^(\d{2})\/(\d{2})\/(\d{4})/);
  if (br) return new Date(Number(br[3]), Number(br[2]) - 1, Number(br[1]));
  return null;
}

function SH_buildResumoIdx_(headers) {
  var norm = {};
  headers.forEach(function(h, i) {
    var k = String(h || '').normalize('NFD').replace(/[\u0300-\u036f]/g, '').replace(/[^A-Z0-9]+/gi, ' ').trim().toUpperCase();
    norm[k] = i;
  });
  function pick(names) {
    for (var ni = 0; ni < names.length; ni++) {
      var k = names[ni].toUpperCase().replace(/[^A-Z0-9]+/g, ' ').trim();
      if (norm[k] !== undefined) return norm[k];
    }
    return -1;
  }
  return {
    tipo:        pick(['TIPO DO REPORT', 'TIPO DO RE', 'TIPO']),
    codCp:       pick(['COD DO CP', 'COD CP', 'CODIGO CP', 'COD']),
    nomeCp:      pick(['NOME DO CP', 'NOME CP', 'NOME']),
    data:        pick(['DATA', 'DT']),
    responsavel: pick(['RESPONSAVEL ATOR', 'RESPONSAVEL', 'RESPONSÁVEL']),
    quantidade:  pick(['QUANTIDADE DE ACESSOS', 'QTD ACESSOS', 'QUANTIDADE', 'QTD', 'VIEWS', 'ACESSOS'])
  };
}

function SH_openResumoSheet_(ssId, sheetName) {
  var ss = SpreadsheetApp.openById(ssId);
  var sh = ss.getSheetByName(sheetName);
  if (!sh) {
    var sheets = ss.getSheets();
    if (sheets.length > 0) sh = sheets[0];
  }
  return sh;
}

/**
 * Lê uma planilha resumo inteira em chunks e retorna stats + list capped.
 * O total (views) conta TODAS as linhas válidas, mesmo que list esteja capped.
 */
function SH_readResumoFull_(ssId, sheetName, canAudit, payload) {
  var empty = { views: 0, dispo: 0, list: [], trend: {}, types: {}, cpAgg: {}, cpDistinct: {}, totalRows: 0 };
  var sh = SH_openResumoSheet_(ssId, sheetName);
  if (!sh) return empty;

  var lastRow = sh.getLastRow();
  var lastCol = Math.min(sh.getLastColumn(), 7);
  if (lastRow < 2) return empty;

  var headers = sh.getRange(1, 1, 1, lastCol).getValues()[0];
  var idx = SH_buildResumoIdx_(headers);

  var filterPeriod = String((payload && payload.period)   || '').trim();
  var filterFrom   = String((payload && payload.dateFrom) || '').trim();
  var filterTo     = String((payload && payload.dateTo)   || '').trim();
  var filterTerm   = String((payload && payload.term)     || '').toLowerCase().trim();

  var out = { views: 0, dispo: 0, list: [], trend: {}, types: {}, cpAgg: {}, cpDistinct: {}, totalRows: 0 };
  var tz  = Session.getScriptTimeZone();

  var startRow = 2;
  while (startRow <= lastRow) {
    var rowsToRead = Math.min(SH_RESUMO_CHUNK_SIZE, lastRow - startRow + 1);
    var values = sh.getRange(startRow, 1, rowsToRead, lastCol).getValues();

    for (var ri = 0; ri < values.length; ri++) {
      var row = values[ri];

      // COD CP obrigatório
      var codCpRaw = idx.codCp >= 0 ? String(row[idx.codCp] || '').trim() : '';
      var codCp    = codCpRaw.replace(/\D/g, '') || codCpRaw;
      if (!codCp) continue;

      // DATA obrigatória
      var dataRaw = idx.data >= 0 ? row[idx.data] : '';
      var dateObj = SH_parseResumoDate_(dataRaw);
      if (!dateObj) continue;

      var isoDate  = Utilities.formatDate(dateObj, tz, 'yyyy-MM-dd');
      var dispDate = Utilities.formatDate(dateObj, tz, 'dd/MM/yyyy');

      // Alterado Patty - 11/05/2026
      // Motivo: impedir que datas futuras ou posteriores à última carga manual oficial
      // contaminem Explorer, Analytics, trend e totais.
      var maxAllowedDate = SH_getResumoMaxDate_();
      if (isoDate > maxAllowedDate) continue;

      // Filtros de período/data
      if (filterPeriod && isoDate.substring(0, 7) !== filterPeriod) continue;
      if (filterFrom   && isoDate < filterFrom) continue;
      if (filterTo     && isoDate > filterTo)   continue;

      var emailRaw = idx.responsavel >= 0 ? String(row[idx.responsavel] || '').trim().toLowerCase() : '';
      var isAdmin  = SH_isResumoAdminEmail_(emailRaw);
      var nomeCpRaw = idx.nomeCp >= 0 ? String(row[idx.nomeCp] || '').trim() : '';
      var nomeCp    = SH_normalizeCpName_(nomeCpRaw);

      // Alterado Patty - 11/05/2026
      // Motivo: remover registros de ruído antes da agregação.
      // Importante: precisa ser antes de out.views/out.trend/out.cpAgg.
      if (SH_isResumoNoiseCp_(codCp, nomeCp)) continue;

      var tipoRaw   = idx.tipo >= 0 ? String(row[idx.tipo] || '').trim() : '';
      var tipo      = SH_normalizeTipoReport_(tipoRaw);
      var qtdRaw    = idx.quantidade >= 0 ? Number(row[idx.quantidade]) : 1;
      var qtd       = (Number.isFinite(qtdRaw) && qtdRaw > 0) ? qtdRaw : 1;

      if (isAdmin) {
        out.dispo += qtd;
        continue;
      }

      // Filtro de termo (cod, nome, e-mail)
      if (filterTerm) {
        var termDigits = filterTerm.replace(/\D/g, '');
        var match = codCp.toLowerCase().includes(filterTerm)
          || nomeCp.toLowerCase().includes(filterTerm)
          || emailRaw.includes(filterTerm)
          || (termDigits && codCp.includes(termDigits));
        if (!match) continue;
      }

      out.views += qtd;
      out.totalRows += 1;
      out.trend[isoDate]   = (out.trend[isoDate]  || 0) + qtd;
      out.types[tipo]      = (out.types[tipo]      || 0) + qtd;
      out.cpDistinct[codCp] = true;

      if (!out.cpAgg[codCp]) out.cpAgg[codCp] = { cod: codCp, name: nomeCp, val: 0 };
      out.cpAgg[codCp].val += qtd;

      if (out.list.length < SH_RESUMO_MAX_LIST) {
        out.list.push({
          d: dispDate,
          t: tipo,
          c: codCp,
          n: nomeCp,
          e: canAudit ? SH_maskEmailForAudit_(emailRaw) : '',
          i: '',      // arquivo_log não existe nos resumos — campo mantido para compatibilidade
          v: qtd,     // 'v' = quantidade, padrão compact do sistema (não usar 'q')
          q: qtd      // alias mantido para retrocompatibilidade
        });
      }
    }

    startRow += rowsToRead;
  }

  return out;
}

/**
 * Ponto de entrada para o Dashboard — usa os arquivos resumo oficiais.
 * Chamado pelo front em loadHeavyDashboard e fetchAuditTotalByRange.
 */
function getResumoAuditData(payload) {
  payload = payload || {};
  var canAudit = canCurrentUserAudit_();

  var r2025 = SH_readResumoFull_(SH_RESUMO_2025_SS_ID, SH_RESUMO_2025_SHEET, canAudit, payload);
  var r2026 = SH_readResumoFull_(SH_RESUMO_2026_SS_ID, SH_RESUMO_2026_SHEET, canAudit, payload);

  // Merge list: 2026 primeiro (mais recente), depois 2025
  var allList = r2026.list.concat(r2025.list).slice(0, SH_RESUMO_MAX_LIST);

  // Merge ranking
  var cpAggAll = {};
  [r2025.cpAgg, r2026.cpAgg].forEach(function(agg) {
    Object.keys(agg).forEach(function(k) {
      if (!cpAggAll[k]) cpAggAll[k] = { cod: agg[k].cod, name: agg[k].name, val: 0 };
      cpAggAll[k].val += agg[k].val;
    });
  });
  var ranking = Object.values(cpAggAll).sort(function(a, b) { return b.val - a.val; });

  var trendAll = {};
  [r2025.trend, r2026.trend].forEach(function(t) {
    Object.keys(t).forEach(function(k) { trendAll[k] = (trendAll[k] || 0) + t[k]; });
  });
  var typesAll = {};
  [r2025.types, r2026.types].forEach(function(t) {
    Object.keys(t).forEach(function(k) { typesAll[k] = (typesAll[k] || 0) + t[k]; });
  });
  var cpsAll = Object.assign({}, r2025.cpDistinct, r2026.cpDistinct);

  var statsObj = {
    audit:        r2025.views + r2026.views,
    audit2025:    r2025.views,
    audit2026:    r2026.views,
    views2025:    r2025.views,
    views2026:    r2026.views,
    viewsRaw2025: r2025.views,
    viewsRaw2026: r2026.views,
    dispo:        r2025.dispo  + r2026.dispo,
    cps:          Object.keys(cpsAll).length,
    cps2025:      Object.keys(r2025.cpDistinct).length,
    cps2026:      Object.keys(r2026.cpDistinct).length,
    trend:        trendAll,
    types:        typesAll,
    top10:        ranking.slice(0, 10),
    bottom10:     ranking.slice().sort(function(a, b) { return a.val - b.val; }).slice(0, 10),
    suggestions:  ranking.map(function(x) { return x.cod + ' - ' + x.name; }),
    listMeta: {
      pageSize:            50,
      maxPages:            500,
      totalRowsAvailable:  r2025.totalRows + r2026.totalRows,
      capped:              (r2025.totalRows + r2026.totalRows) > SH_RESUMO_MAX_LIST
    }
  };

  // Cards executivos — objeto próprio para o front não adivinhar campo
  statsObj.cards = SH_buildExecutiveCards_(statsObj);

  return {
    success: true,
    canAudit: canAudit,
    sourceMethod: 'SHEETS_RESUMO_OFICIAL',
    list: allList,
    stats: statsObj
  };
}

/************************************************************
 * SHERLOCK v8.4 — CARDS EXECUTIVOS OFICIAIS
 * Patty — 10/05/2026
 * Motivo: cards estavam lendo snapshot antigo (7.952).
 * Cards oficiais lêem resumo_2025 e resumo_2026.
 ************************************************************/
function SH_buildExecutiveCards_(stats) {
  stats = stats || {};

  var views2025 = Number(stats.views2025 || 0);
  var views2026 = Number(stats.views2026 || 0);
  var viewsEmailMaio = Number(stats.viewsEmailMaioPlus || 0);

  var cpsTotal = Number(stats.cps || 0);
  var cpsSemFvc = 40;
  var cpsImpactadosFvc = Math.max(0, cpsTotal - cpsSemFvc);

  return {
    views2025:           views2025,
    views2026:           views2026,
    viewsEmailMaioPlus:  viewsEmailMaio,
    viewsOficiais:       views2025 + views2026,
    disponibilizacoes:   Number(stats.dispo || 0),
    cpsTotalEncontrados: cpsTotal,
    cpsImpactados:       cpsImpactadosFvc,
    cpsSemFvc:           cpsSemFvc,
    identificacaoSherlock: Number(stats.qualityTotal || 74.84)
  };
}

/**
 * Explorer paginado direto dos arquivos resumo.
 * Substitui api_getAuditExplorerPage para trazer dados reais das Sheets resumo.
 * Limite: 500 páginas máx.
 */
function api_getResumoExplorerPage(params) {
  params = params || {};
  var canAudit = canCurrentUserAudit_();

  var pageSize = Math.min(Math.max(10, Number(params.pageSize || 50)), 100);
  var page     = Math.max(1, Number(params.page || 1));
  var period   = String(params.period   || '').trim();
  var term     = String(params.term     || '').trim();
  var dateFrom = String(params.dateFrom || '').trim();
  var dateTo   = String(params.dateTo   || '').trim();

  // Determina fonte(s) pelo ano do período
  var sources = [];
  var yearSrc = (period || dateFrom || '').match(/^(\d{4})/);
  var year    = yearSrc ? Number(yearSrc[1]) : 0;
  if (year === 2025) {
    sources = [{ ssId: SH_RESUMO_2025_SS_ID, sheet: SH_RESUMO_2025_SHEET }];
  } else if (year === 2026) {
    sources = [{ ssId: SH_RESUMO_2026_SS_ID, sheet: SH_RESUMO_2026_SHEET }];
  } else {
    // sem filtro de ano: lê ambas (2026 primeiro)
    sources = [
      { ssId: SH_RESUMO_2026_SS_ID, sheet: SH_RESUMO_2026_SHEET },
      { ssId: SH_RESUMO_2025_SS_ID, sheet: SH_RESUMO_2025_SHEET }
    ];
  }

  var payload  = { period: period, dateFrom: dateFrom, dateTo: dateTo, term: term };
  var allItems = [];

  sources.forEach(function(src) {
    var r = SH_readResumoFull_(src.ssId, src.sheet, canAudit, payload);
    allItems = allItems.concat(r.list);
  });

  // Sort: mais recente primeiro
  allItems.sort(function(a, b) {
    var toIso = function(d) {
      if (!d) return '';
      var p = String(d).split('/');
      return p.length === 3 ? p[2] + '-' + p[1] + '-' + p[0] : String(d);
    };
    return toIso(b.d).localeCompare(toIso(a.d));
  });

  var total      = allItems.length;
  var maxPages   = 500;
  var totalPages = Math.min(maxPages, Math.max(1, Math.ceil(total / pageSize)));
  page           = Math.min(page, totalPages);
  var start      = (page - 1) * pageSize;
  var pageItems  = allItems.slice(start, start + pageSize);

  return {
    success:    true,
    period:     period || dateFrom || 'todos',
    page:       page,
    pageSize:   pageSize,
    total:      total,
    totalPages: totalPages,
    canAudit:   canAudit,
    list:       pageItems
  };
}

/**
 * Ponto de entrada otimizado para o Dashboard.
 * Tenta ler o JSON do Drive (ms) antes de processar o Sheets (minutos).
 */
function getConsolidatedData(filterParams, forceRefresh = false) {
  try {
    const normalizedFilters = normalizeFilterParams_(filterParams);
    const hasActiveFilters = hasActiveUserFilters_(normalizedFilters);
    const activeUser = Session.getActiveUser().getEmail().toLowerCase();
    
    // Registra o log em segundo plano (nÃ£o trava o retorno do JSON)
    if (!normalizedFilters || forceRefresh) {
        internal_logAccess_(activeUser);
    }
    const folder = DriveApp.getFolderById(SHARD_FOLDER_ID);
    const files = folder.searchFiles("title = '" + CACHE_FILE_NAME + "'");
    
    // Se nÃ£o for um "force refresh" e o arquivo existir, lÃª o cache
    if (!forceRefresh && !hasActiveFilters && files.hasNext()) {
      const file = files.next();
      const content = file.getBlob().getDataAsString();
      let cachedData = JSON.parse(content);
      const cachedStats = cachedData.stats || {};
      const cachedAudit = Number(cachedStats.audit) || 0;
      const cachedTrendKeys = Object.keys(cachedStats.trend || {});
      const cachedTrendSize = cachedTrendKeys.length;

      // Detecta mÃªs mais recente no trend do cache (formato YYYY-MM para comparaÃ§Ã£o precisa)
      const latestCachedYM = cachedTrendKeys.reduce((max, k) => {
        const ym = String(k).substring(0, 7); // YYYY-MM
        return ym > max ? ym : max;
      }, '0000-00');
      const now = new Date();
      const currentYM = now.getFullYear() + '-' + String(now.getMonth() + 1).padStart(2, '0');
      // MÃªs anterior ao atual (aceita que o mÃªs atual ainda pode estar incompleto)
      const prevMonth = new Date(now.getFullYear(), now.getMonth() - 1, 1);
      const prevYM = prevMonth.getFullYear() + '-' + String(prevMonth.getMonth() + 1).padStart(2, '0');

      if (cachedAudit > 0 && cachedTrendSize === 0) {
        // Cache inconsistente: trend vazio.
        cachedData = null;
      } else if (latestCachedYM !== '0000-00' && latestCachedYM < prevYM) {
        // Cache desatualizado: o mÃªs mais recente no cache Ã© anterior ao mÃªs passado.
        Logger.log(`[WARN] Cache desatualizado (ultimo mes no cache: ${latestCachedYM}, mes anterior atual: ${prevYM}). Reprocessando...`);
        cachedData = null;
      }

      if (cachedData) {
      
      // Verifica permissÃµes de auditoria dinamicamente (pois depende do usuÃ¡rio logado agora)
      const AUDIT_ALLOWED_EMAILS = [
        'patricia.wojciechowski@grupoboticario.com.br',
        'opatriciakelly@gmail.com'
      ];
      const activeUser = Session.getActiveUser().getEmail().toLowerCase();
      cachedData.canAudit = AUDIT_ALLOWED_EMAILS.some(ae => activeUser === ae.toLowerCase());

      if (cachedData.stats && !cachedData.stats.fvcOutsideCurrentCycle) {
        let cpUniverseCodes = Array.isArray(cachedData.stats.cpUniverseCodes)
          ? cachedData.stats.cpUniverseCodes.slice()
          : [];

        if (cpUniverseCodes.length === 0 && Array.isArray(cachedData.list)) {
          for (let i = 0; i < cachedData.list.length; i++) {
            const row = cachedData.list[i] || {};
            const norm = normalizeCpCode_(row.c);
            if (norm) cpUniverseCodes.push(norm);
          }
        }

        cachedData.stats.cpUniverseCodes = cpUniverseCodes;
        cachedData.stats.fvcOutsideCurrentCycle = buildFvcOutsideInsight_(cpUniverseCodes, {});
      }

      if (cachedData.stats && (!Array.isArray(cachedData.stats.pendingFranchiseDrives) || cachedData.stats.pendingFranchiseDrives.length === 0)) {
        const fvcInfo = cachedData.stats.fvcOutsideCurrentCycle || {};
        const pendingFromFvc = Array.isArray(fvcInfo.cpSemEmailCadastro)
          ? fvcInfo.cpSemEmailCadastro.map(cp => {
              const cod = String(cp && cp.cod ? cp.cod : '').trim();
              const nome = String(cp && cp.nome ? cp.nome : '').trim();
              return nome ? `${cod} - ${nome}` : cod;
            }).filter(Boolean)
          : [];
        cachedData.stats.pendingFranchiseDrives = pendingFromFvc;
        cachedData.stats.pendingFranchiseDriveCount = Number(fvcInfo.totalCpSemEmailCadastro) || pendingFromFvc.length;
      }

      if (cachedData.stats && !cachedData.stats.dispoRule) {
        const cachedDispo = Number(cachedData.stats.dispo) || 0;
        cachedData.stats.dispoRule = computeDisponibilizacaoRegra2025e2026_(normalizedFilters, cachedDispo);
        cachedData.stats.dispo = Number(cachedData.stats.dispoRule.total) || cachedDispo;
      }
      
      // OtimizaÃ§Ã£o de Performance: Se for carga inicial (light mode),
      // retorna apenas os primeiros 200 itens para reduzir payload.
      if (filterParams && filterParams.light === true && cachedData.list) {
        cachedData.list = cachedData.list.slice(0, 200);
        cachedData.isLimited = true;
      }
      
      return cachedData;
      }
    }
    
    // â•â•â• ROTA PRIMÃRIA: JSONs mensais gerados pelo Colab ETL â•â•â•
    Logger.log('[INFO] Carregando JSONs mensais (fonte primaria)...');
    const jsonResult = loadAllJsonMonthlyData_();
    let freshData;

    if (jsonResult.success && jsonResult.data.length > 0) {
      Logger.log('[OK] JSON carregado: ' + jsonResult.fileCount + ' arquivos, ' + jsonResult.data.length + ' registros.');
      freshData = buildResponseFromJson_(jsonResult.data, normalizedFilters);
    }

    // â•â•â• ROTA FALLBACK: Sheets (caso JSON falhe) â•â•â•
    if (!freshData || !freshData.success) {
      Logger.log('[WARN] JSON falhou, usando fallback Sheets.');
      freshData = internal_processSheetsData_(normalizedFilters);
      if (freshData.success) freshData.sourceMethod = 'SHEETS (Fallback)';
    }

    // Salva o snapshot no Drive para o prÃ³ximo usuÃ¡rio
    if (freshData && freshData.success && !hasActiveFilters) {
      saveDataToCacheFile_(freshData);
    }

    // Aplica o mesmo limite para o retorno imediato do processamento
    if (freshData && normalizedFilters && normalizedFilters.light === true && freshData.list) {
      freshData.list = freshData.list.slice(0, 200);
      freshData.isLimited = true;
    }

    return freshData || { error: 'Nenhuma fonte de dados disponÃ­vel.' };
  } catch(e) {
    return { error: "Erro no Motor JSON: " + e.toString() };
  }
}

/**
 * FunÃ§Ã£o Administrativa para forÃ§ar a atualizaÃ§Ã£o do Banco de Dados JSON
 */
/**
 * Admin Refresh Seguro
 * Alterado Patty - 10/05/2026
 * Motivo: impedir que o Apps Script reprocesse JSONs mensais pesados.
 * Agora o snapshot deve ser gerado pelo Python/Colab.
 */
// Alterado Patty - 10/05/2026
// Teste rápido: valida se getHomeSnapshotFast está funcional e mede tempo de resposta.
function admin_testHomeFast() {
  requireSherlockAdmin_();
  const ini = Date.now();
  const res = getHomeSnapshotFast();
  return {
    success: !(res && res.error),
    elapsedMs: Date.now() - ini,
    sourceMethod: res && res.sourceMethod ? res.sourceMethod : '',
    listLength: res && res.list ? res.list.length : 0,
    audit: res && res.stats ? res.stats.audit : null,
    cps: res && res.stats ? res.stats.cps : null,
    audit2025: res && res.stats ? res.stats.audit2025 : null,
    audit2026: res && res.stats ? res.stats.audit2026 : null,
    semFvcCount: res && res.stats ? res.stats.semFvcCount : null,
    error: res && res.error ? res.error : ''
  };
}

function admin_forceRefreshSnapshot() {
  requireSherlockAdmin_();
  const snapshot = getHomeSnapshotFast();
  if (!snapshot || snapshot.error || snapshot.success === false) {
    return {
      success: false,
      error: snapshot && snapshot.error
        ? snapshot.error
        : 'Snapshot não encontrado. Execute o Colab para gerar ' + CACHE_FILE_NAME + '.'
    };
  }
  return {
    success: true,
    message: 'Snapshot validado com sucesso. Nenhum reprocessamento pesado foi executado no Apps Script.',
    loadedAt: new Date().toISOString(),
    stats: snapshot.stats || {},
    sourceMethod: snapshot.sourceMethod || 'SNAPSHOT_FAST_ONLY'
  };
}

function admin_getTechPanelData() {
  requireSherlockAdmin_();
  const snapshotTest = admin_testHomeFast();
  const logs = getAdminLogs();
  return {
    success: true,
    loadedAt: new Date().toISOString(),
    snapshotTest: snapshotTest,
    logs: logs && logs.logs ? logs.logs.slice(0, 100) : [],
    logsError: logs && logs.error ? logs.error : '',
    sourceMethod: 'ADMIN_TECH_PANEL'
  };
}

/**
 * Home rápida — lê apenas o snapshot pronto.
 * Alterado Patty - 10/05/2026
 * Motivo: impedir que a página inicial reprocesse JSON/Sheets/CSV e demore minutos.
 */
function getHomeSnapshotFast() {
  try {
    const folder = DriveApp.getFolderById(SHARD_FOLDER_ID);
    const files = folder.searchFiles("title = '" + CACHE_FILE_NAME + "' and trashed = false");

    if (!files.hasNext()) {
      return {
        success: false,
        error: 'Snapshot não encontrado. Gere o arquivo ' + CACHE_FILE_NAME + ' pelo Colab.',
        sourceMethod: 'SNAPSHOT_FAST_ONLY'
      };
    }

    const file = files.next();
    const content = file.getBlob().getDataAsString('UTF-8');
    let data = JSON.parse(content);

    // Aplica camada oficial de negócio sem reprocessar JSON/Sheets.
    // 2025 preserva histórico validado. 2026 usa snapshot + paridade CI/CA.
    data = applySherlockOfficialMetrics_(data);

    const AUDIT_ALLOWED_EMAILS = [
      'patricia.wojciechowski@grupoboticario.com.br',
      'opatriciakelly@gmail.com'
    ];
    let activeUser = '';
    try { activeUser = Session.getActiveUser().getEmail().toLowerCase(); } catch(e) {}
    data.canAudit = AUDIT_ALLOWED_EMAILS.includes(activeUser);

    if (data.stats) {
      // Não recalcular FVC/carteira na home.
      // applySherlockOfficialMetrics_ já injeta fvcOutsideCurrentCycle leve.
    }

    // A home não precisa receber lista gigante
    if (Array.isArray(data.list)) {
      data.list = sanitizeAuditListEmailOnly_(data.list).slice(0, 300);
      data.isLimited = true;
    }

    data.success = true;
    data.sourceMethod = 'SNAPSHOT_FAST_ONLY';
    data.loadedAt = new Date().toISOString();
    return data;

  } catch(e) {
    return {
      success: false,
      error: 'Erro ao carregar snapshot rápido: ' + e.toString(),
      sourceMethod: 'SNAPSHOT_FAST_ONLY'
    };
  }
}

/**
 * Calcula CPs fora da carteira FVC sem buscar e-mails por CP.
 * Alterado Patty - 10/05/2026
 * Motivo: evitar chamada pesada na abertura.
 */
function buildFvcOutsideFastFromSnapshot_(stats) {
  try {
    const catalog = getCurrentFvcCycleCatalog_();
    if (!catalog || !catalog.success) {
      return {
        success: false, totalCpFvc: 0, totalCpForaFvc: 0, cpForaFvc: [],
        error: catalog && catalog.error ? catalog.error : 'Carteira FVC indisponível.'
      };
    }

    const cpUniverseCodes = Array.isArray(stats && stats.cpUniverseCodes) ? stats.cpUniverseCodes : [];
    const cpNames = (stats && stats.cpNames) ? stats.cpNames : {};
    const uniq = {};
    const outside = [];

    cpUniverseCodes.forEach(codRaw => {
      const cod = normalizeCpCode_(codRaw);
      if (cod) uniq[cod] = true;
    });

    Object.keys(uniq).forEach(cod => {
      if (catalog.byCode && catalog.byCode[cod]) return;
      outside.push({ cod: cod, nome: String(cpNames[cod] || '').trim() });
    });

    outside.sort((a, b) => String(a.cod).localeCompare(String(b.cod), 'pt-BR'));

    return {
      success: true,
      sourceUrl: catalog.sourceUrl,
      sheetName: catalog.sheetName,
      totalCpFvc: Number(catalog.totalCp) || 0,
      totalCpForaFvc: outside.length,
      cpForaFvc: outside.slice(0, 200),
      cpForaFvcTruncado: outside.length > 200,
      error: ''
    };
  } catch(e) {
    return {
      success: false, totalCpFvc: 0, totalCpForaFvc: 0, cpForaFvc: [],
      error: 'Erro no cálculo rápido de CPs fora FVC: ' + e.toString()
    };
  }
}

/**
 * DIAGNÃ“STICO COMPLETO â€” Execute no editor Apps Script.
 */
function admin_diagnosticFull() {
  const adminEmails = ['patricia.wojciechowski@grupoboticario.com.br', 'opatriciakelly@gmail.com'];
  const activeUser = Session.getActiveUser().getEmail().toLowerCase();
  if (!adminEmails.includes(activeUser)) return { error: 'Apenas admins.' };

  const result = { driveFiles: [], jsonMonthsFound: [], cache: {}, sampleRow2026: null };
  try {
    const folder = DriveApp.getFolderById(SHARD_FOLDER_ID);
    const files = folder.getFiles();
    const monthPattern = /^sherlock_logs_(\d{4}-\d{2})\.json$/i;
    while (files.hasNext()) {
      const f = files.next();
      const nm = f.getName();
      result.driveFiles.push(nm);
      const m = nm.match(monthPattern);
      if (m) result.jsonMonthsFound.push(m[1]);
    }
    result.jsonMonthsFound.sort();
    const cacheFiles = folder.searchFiles("title = '" + CACHE_FILE_NAME + "'");
    if (cacheFiles.hasNext()) {
      try {
        const cacheData = JSON.parse(cacheFiles.next().getBlob().getDataAsString());
        const trendKeys = Object.keys((cacheData.stats || {}).trend || {}).sort();
        result.cache = {
          exists: true, audit: (cacheData.stats || {}).audit,
          trendKeyCount: trendKeys.length,
          firstTrendKey: trendKeys[0] || 'N/A', lastTrendKey: trendKeys[trendKeys.length - 1] || 'N/A',
          trendKeys2026: trendKeys.filter(k => k.startsWith('2026'))
        };
      } catch(e) { result.cache = { exists: true, parseError: e.toString() }; }
    } else { result.cache = { exists: false }; }
  } catch(e) { result.fatalError = e.toString(); }
  Logger.log(JSON.stringify(result, null, 2));
  return result;
}

function saveDataToCacheFile_(data) {
  try {
    const folder = DriveApp.getFolderById(SHARD_FOLDER_ID);
    const files = folder.searchFiles("title = '" + CACHE_FILE_NAME + "'");
    
    if (files.hasNext()) {
      files.next().setContent(JSON.stringify(data));
    } else {
      folder.createFile(CACHE_FILE_NAME, JSON.stringify(data), MimeType.PLAIN_TEXT);
    }
  } catch(e) { console.error("Falha ao salvar cache:", e); }
}

function sumTrendRange_(trendMap, dateFrom, dateTo) {
  let total = 0;
  const start = dateFrom || '0000-01-01';
  const end = dateTo || '9999-12-31';
  const keys = Object.keys(trendMap || {});
  for (let i = 0; i < keys.length; i++) {
    const k = keys[i];
    if (k >= start && k <= end) {
      total += Number(trendMap[k]) || 0;
    }
  }
  return total;
}

// Alterado Patty - 10/05/2026
// Motivo: substituir getConsolidatedData({ light: true }) por getHomeSnapshotFast() — sem mineração pesada.
function getStrategicAccessView() {
  try {
    const snapshot = getHomeSnapshotFast();
    if (!snapshot || snapshot.success === false || snapshot.error) {
      return { success: false, error: snapshot && snapshot.error ? snapshot.error : 'Snapshot indisponível.' };
    }
    const stats = snapshot.stats || {};
    const trend = stats.trend || {};
    const access2025 = Number(stats.audit2025 || stats.viewsOfficial2025) || sumTrendRange_(trend, '2025-01-01', '2025-12-31');
    const access2026 = Number(stats.audit2026 || stats.viewsOfficial2026) || sumTrendRange_(trend, '2026-01-01', '2026-12-31');
    return {
      success: true,
      ranges: {
        y2025: access2025,
        y2026FirstTwoHalfMonths: access2026,
        y2026Accumulated: access2026
      },
      labels: {
        y2025: '2025 oficial - histórico validado',
        y2026FirstTwoHalfMonths: '2026 snapshot + paridade CI/CA',
        y2026Accumulated: '2026 acumulado oficial'
      },
      sourceMethod: 'SNAPSHOT_FAST_ONLY_OFFICIAL'
    };
  } catch (e) {
    return { success: false, error: 'Erro ao calcular visão estratégica via snapshot: ' + e.toString() };
  }
}

// Alterado Patty - 10/05/2026
// Motivo: substituir getConsolidatedData por getHomeSnapshotFast() — sem mineração pesada.
function getYearlyLineAccessSummary() {
  try {
    const snapshot = getHomeSnapshotFast();
    if (!snapshot || snapshot.success === false || snapshot.error) {
      return { success: false, error: snapshot && snapshot.error ? snapshot.error : 'Snapshot indisponível.' };
    }
    const stats = snapshot.stats || {};
    const trend = stats.trend || {};
    const access2025 = Number(stats.audit2025 || stats.viewsOfficial2025) || sumTrendRange_(trend, '2025-01-01', '2025-12-31');
    const access2026 = Number(stats.audit2026 || stats.viewsOfficial2026) || sumTrendRange_(trend, '2026-01-01', '2026-12-31');
    return {
      success: true,
      ranges: { y2025: access2025, y2026: access2026 },
      labels: { y2025: '2025 oficial - histórico validado', y2026: '2026 snapshot + paridade CI/CA' },
      sourceMethod: 'SNAPSHOT_FAST_ONLY_OFFICIAL'
    };
  } catch (e) {
    return { success: false, error: 'Erro ao calcular resumo anual via snapshot: ' + e.toString() };
  }
}

function normalizeActorEmail_(value) {
  const raw = String(value || '').toLowerCase().trim();
  if (!raw) return '';
  const match = raw.match(/[a-z0-9._%+\-]+@[a-z0-9.\-]+\.[a-z]{2,}/i);
  return match ? String(match[0]).toLowerCase().trim() : '';
}

// ═══════════════════════════════════════════════════════
// LEITURA ROBUSTA DO CAMPO DE E-MAIL/ATOR
// Alterado Patty - 10/05/2026
// Motivo: JSONs podem ter 'RESPONSÁVEL (ATOR)' ou variantes com encoding.
// Admin/dev continua oculto; e-mail de cliente continua visível mascarado.
// ═══════════════════════════════════════════════════════

function getRowValueByAliases_(row, aliases) {
  if (!row || typeof row !== 'object') return '';
  for (var i = 0; i < aliases.length; i++) {
    var key = aliases[i];
    if (row[key] !== undefined && row[key] !== null && String(row[key]).trim() !== '') {
      return row[key];
    }
  }
  return '';
}

function getAuditActorRawFromRow_(row) {
  return getRowValueByAliases_(row, [
    'RESPONS\u00c1VEL (ATOR)',
    'RESPONSAVEL (ATOR)',
    'RESPONS\u00c3VEL (ATOR)',
    'RESPONS\u00c1VEL',
    'RESPONSAVEL',
    'ATOR',
    'E-MAIL',
    'EMAIL',
    'EMAIL ATOR',
    'USU\u00c1RIO',
    'USUARIO',
    'USER',
    'ACTOR'
  ]);
}

function getQuantityFromAuditRow_(row) {
  var raw = getRowValueByAliases_(row, [
    'QUANTIDADE DE ACESSOS',
    'QTD ACESSOS',
    'QTD',
    'VIEWS',
    'ACESSOS'
  ]);
  var n = Number(raw);
  return Number.isFinite(n) && n > 0 ? n : 1;
}

function isSystemActorEmail_(email) {
  if (!email) return true;
  const local = String(email).split('@')[0] || '';
  if (/^(noreply|no-reply|bot|googleapis|admin|postmaster|mailer|dlp|drive|sistema)/i.test(local)) return true;
  return String(email).indexOf('gserviceaccount.com') > -1;
}

// Alterado Patty - 10/05/2026 — segunda ocorrência substituída
// Motivo: esta cópia ainda chamava loadAllJsonMonthlyData_() com leitura pesada.
function getFranchiseBigNumber2026() {
  try {
    const snapshot = getHomeSnapshotFast();
    if (!snapshot || snapshot.success === false || snapshot.error) {
      return { success: false, error: snapshot && snapshot.error ? snapshot.error : 'Snapshot indisponível.' };
    }
    const stats = snapshot.stats || {};
    const trend = stats.trend || {};
    const total2026 = Number(stats.audit2026 || stats.viewsOfficial2026) || sumTrendRange_(trend, '2026-01-01', '2026-12-31');
    return {
      success: true,
      year: 2026,
      baseLines: total2026,
      total: total2026,
      multiplier: 1,
      label: 'Acessos 2026 via snapshot + paridade CI/CA',
      sourceMethod: 'SNAPSHOT_FAST_ONLY_OFFICIAL'
    };
  } catch (e) {
    return { success: false, error: 'Erro ao calcular Big Number 2026 via snapshot: ' + e.toString() };
  }
}

// ====== INTEGRAÇÃO ORGANIZZECI (JSON/BQ) PARA BUSCA DE EMAILS ======
// FOLDER IDs: Pastas que contêm shards JSON e CSVs de auditoria
const SHARD_FOLDER_ID = '1tIqgxus9VmC3rEaCj9z7JB1rmsahfJ_a';      // Pasta Principal (2026+)
const SHARD_FOLDER_ID_2025 = '0AOVybAy3QnD3Uk9PVA';               // Pasta Histórica (2025)

/**
 * Lê um arquivo JSON mensal específico do Sherlock.
 * Alterado Patty - 10/05/2026
 * Motivo: permitir consulta histórica 2025/2026 sem carregar todos os meses na home.
 */
function readSherlockMonthlyJson_(period) {
  const p = String(period || '').trim();
  if (!/^\d{4}-\d{2}$/.test(p)) throw new Error('Período inválido. Use YYYY-MM, exemplo: 2025-01.');

  const fileName = 'sherlock_logs_' + p + '.json';
  const folderIds = [SHARD_FOLDER_ID, SHARD_FOLDER_ID_2025];

  for (let i = 0; i < folderIds.length; i++) {
    const folder = DriveApp.getFolderById(folderIds[i]);
    const files = folder.searchFiles("title = '" + fileName + "' and trashed = false");
    if (files.hasNext()) {
      const content = files.next().getBlob().getDataAsString('UTF-8');
      const json = JSON.parse(content);
      if (!Array.isArray(json)) throw new Error(fileName + ' não está no formato de lista JSON.');
      return json;
    }
  }
  return [];
}

// Alterado Patty - 10/05/2026
// Garante que nomes de CP sem prefixo "CP" recebam o prefixo na exibição.
function normalizeCpNameForDisplay_(value) {
  var n = String(value || '').trim().replace(/\s+/g, ' ');
  if (!n) return '';

  n = n.replace(/^CP\s+MS\s+PERFUMARIA$/i, 'CP M S PERFUMARIA');
  n = n.replace(/^CP\s+M\.?\s*S\.?\s+PERFUMARIA$/i, 'CP M S PERFUMARIA');

  var lower = n.toLowerCase();
  var isNoise =
    lower.indexOf('evento n\u00e3o registrado') > -1 ||
    lower.indexOf('evento nao registrado') > -1 ||
    lower.indexOf('sem log') > -1 ||
    lower.indexOf('unknown') > -1 ||
    lower.indexOf('desconhecido') > -1;

  if (!isNoise && !/^CP\b/i.test(n)) {
    n = 'CP ' + n;
  }

  return n;
}

/**
 * API paginada para o Audit Log Explorer.
 * Carrega somente o mês selecionado — sem reprocessar tudo.
 * Alterado Patty - 10/05/2026
 */
function api_getAuditExplorerPage(params) {
  try {
    const p = params || {};
    const period   = String(p.period   || '').trim();
    const page     = Math.max(1, Number(p.page     || 1));
    const pageSize = Math.min(Math.max(10, Number(p.pageSize || 50)), 100);
    const term     = String(p.term || '').toLowerCase().trim();
    const type     = String(p.type || '').trim();

    const AUDIT_ALLOWED = ['patricia.wojciechowski@grupoboticario.com.br', 'opatriciakelly@gmail.com'];
    let canAudit = false;
    try { canAudit = AUDIT_ALLOWED.some(ae => Session.getActiveUser().getEmail().toLowerCase() === ae); } catch(e) {}

    if (!period) return { success: false, error: 'Selecione um mês para consultar o histórico. Exemplo: 2025-01.' };

    const rows   = readSherlockMonthlyJson_(period);
    const parsed = [];

    for (let i = 0; i < rows.length; i++) {
      const row     = rows[i] || {};
      const tipoRaw = String(row['TIPO DO REPORT'] || 'Outro').trim();
      const tipo    = tipoRaw === 'Outro' ? 'Acesso ao Drive' : tipoRaw;
      const cod     = String(row['COD DO CP']  || 'Unknown').trim();
      const nomeRaw = String(row['NOME DO CP'] || '').trim();
      const nome    = normalizeCpNameForDisplay_(nomeRaw);
      const ator    = String(getAuditActorRawFromRow_(row) || '').trim();
      // Alterado Patty - 10/05/2026: usar SH_getAccessDateBR_ para data real do acesso
      const dataBREx = SH_getAccessDateBR_(row);
      if (!dataBREx) continue;
      const dateKey = SH_brDateToIso_(dataBREx);
      const fDate   = dataBREx;

      // Alterado Patty - 11/05/2026
      // Motivo: regra única de qualidade para o Audit Log Explorer.
      // Remove CP vazio, Unknown, Evento Nao Registrado/Perdido e datas futuras.
      const codNorm = normalizeCpCode_(cod);
      const maxAllowedDate = SH_getResumoMaxDate_();

      if (dateKey && maxAllowedDate && dateKey > maxAllowedDate) continue;
      if (!codNorm) continue;
      if (String(cod || '').trim().toLowerCase() === 'unknown') continue;
      if (SH_isResumoNoiseCp_(cod, nomeRaw) || SH_isResumoNoiseCp_(cod, nome)) continue;

      if (type && tipo !== type) continue;
      if (term && !matchesCpSearchByCodeFirst_(term, cod, nome, ator)) continue;

      parsed.push({ t: tipo, c: codNorm, n: nome, d: fDate, dateKey: dateKey,
                    e: canAudit ? auditEmailForUiOnly_(ator) : null,
                    i: String(row['ARQUIVO_LOG'] || ''), v: Number(row['QUANTIDADE DE ACESSOS'] || 1) });
    }

    parsed.sort((a, b) => {
      const bk = String(b.dateKey || ''), ak = String(a.dateKey || '');
      if (bk !== ak) return bk.localeCompare(ak);
      return String(a.c || '').localeCompare(String(b.c || ''));
    });

    const total = parsed.length;
    const start = (page - 1) * pageSize;
    const totalPages = Math.max(1, Math.ceil(total / pageSize));

    return {
      success: true, period: period, page: page, pageSize: pageSize,
      total: total, totalPages: totalPages,
      list: sanitizeAuditListEmailOnly_(parsed.slice(start, start + pageSize))
    };
  } catch(e) {
    return { success: false, error: 'Erro ao carregar Audit Log Explorer: ' + e.toString() };
  }
}

// NOTA: BigQuery nÃ£o Ã© necessÃ¡rio aqui. Usar apenas Drive para leitura de JSON shards
// Se precisar BigQuery no futuro, substituir por ID apropriado
const BQ_CI_SSID = '1cJjAOlnVn1sUxq36Bhvay_WCCPF4jKo7KNFtQBYXNNs'; // Reutiliza Spreadsheet de auditoria

function fetchDbEmails_Unified_(cod) {
  if (!cod || String(cod).trim().length < 3) return [];
  cod = String(cod).trim().toUpperCase();

  // 1. Tenta SHARDS (Arquivos JSON no Drive) - MUITO MAIS RÃPIDO
  try {
    const folder = DriveApp.getFolderById(SHARD_FOLDER_ID);
    const files = folder.searchFiles("title = 'cp_" + cod + ".json'");
    if (files.hasNext()) {
      const file = files.next();
      const data = JSON.parse(file.getBlob().getDataAsString());
      const mailList = (data.emails || data.Emails || []).filter(e => e && String(e).includes('@'));
      if (mailList.length > 0) {
        // Remove duplicatas mantendo ordem (sem Set, compatÃ­vel com GAS antigas)
        const unique = [];
        const seen = {};
        mailList.forEach(e => {
          const normalized = String(e).toLowerCase().trim();
          if (!seen[normalized]) {
            seen[normalized] = true;
            unique.push(normalized);
          }
        });
        return unique;
      }
    }
  } catch(e) { /* ignore and fallback */ }

  return [];
}

/**
 * Teste de acesso ao Drive - valida permissÃµes
 * Execute esta funÃ§Ã£o para autorizar o acesso ao Drive
 */
function testDriveAccess() {
  try {
    const folder = DriveApp.getFolderById('1tIqgxus9VmC3rEaCj9z7JB1rmsahfJ_a');
    const name = folder.getName();
    Logger.log('[OK] ACESSO AO DRIVE FUNCIONANDO!');
    Logger.log('Pasta: ' + name);
    return { success: true, folderName: name };
  } catch(e) {
    Logger.log('[ERRO] ' + e.toString());
    return { success: false, error: e.toString() };
  }
}

// â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•
// MOTOR JSON â€” LÃª sherlock_logs_YYYY-MM.json gerados pelo Colab
// â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•

// Alterado Patty - 10/05/2026 — segunda ocorrência substituída
// Motivo: remover leitura de Google Sheets e CSV — ler SOMENTE JSON mensal oficial.
function loadAllJsonMonthlyData_() {
  try {
    const folderIds = [SHARD_FOLDER_ID, SHARD_FOLDER_ID_2025];
    let allData = [];
    let fileCount = 0;

    const MONTHLY_PATTERN = /^sherlock_logs_\d{4}-\d{2}\.json$/i;

    folderIds.forEach(folderId => {
      try {
        const folder = DriveApp.getFolderById(folderId);
        const files = folder.getFiles();

        Logger.log('[PASTA] Escaneando SOMENTE JSON mensal: ' + folder.getName());

        while (files.hasNext()) {
          const file = files.next();
          const fileName = file.getName();

          if (!MONTHLY_PATTERN.test(fileName)) {
            Logger.log('[SKIP] ' + fileName + ' — não é JSON mensal oficial');
            continue;
          }

          try {
            const content = file.getBlob().getDataAsString('UTF-8');
            const json = JSON.parse(content);

            if (!Array.isArray(json)) {
              Logger.log('[SKIP] ' + fileName + ' — JSON não é array');
              continue;
            }

            allData = allData.concat(json);
            fileCount++;
            Logger.log('[JSON OK] ' + fileName + ' (' + json.length + ' linhas)');

          } catch (jsonErr) {
            Logger.log('[ERRO JSON] ' + fileName + ': ' + jsonErr);
          }
        }

      } catch (folderErr) {
        Logger.log('[ERRO PASTA] ' + folderId + ': ' + folderErr);
      }
    });

    Logger.log('[OK] JSONs mensais carregados: ' + fileCount + ' arquivos | ' + allData.length + ' registros');

    return {
      success: true,
      data: allData,
      fileCount: fileCount,
      source: 'JSON_MENSAL_ONLY'
    };

  } catch (e) {
    Logger.log('[ERRO] loadAllJsonMonthlyData_ falhou: ' + e);

    return {
      success: false,
      error: e.toString(),
      data: [],
      fileCount: 0,
      source: 'JSON_MENSAL_ONLY'
    };
  }
}

function buildResponseFromJson_(data, filterParams) {
  try {
    const AUDIT_ALLOWED = ['patricia.wojciechowski@grupoboticario.com.br', 'opatriciakelly@gmail.com'];
    const cpProductMap = getCpProductMap_(); // Mapeamento dinâmico de produtos
    let canAudit = false;
    try { canAudit = AUDIT_ALLOWED.some(ae => Session.getActiveUser().getEmail().toLowerCase() === ae); } catch(e) {}

    const cpAgg = {}, cpNames = {}, trend = {};
    const sugs = new Set();
    const auditSeen = new Set(); // Controle de Unicidade por E-mail/Dia (Checklist Item 2)
    let audit = 0, historyTotal = 0, resultData = [];
    const ADMIN_EMAILS_EXCLUIR = [
      'patricia.wojciechowski@grupoboticario.com.br',
      'opatriciakelly@gmail.com'
    ];
    const isAdminActor_ = (email) => {
      const actorEmail = String(email || '').toLowerCase().trim();
      if (!actorEmail) return false;
      if (ADMIN_EMAILS_EXCLUIR.includes(actorEmail)) return true;
      return /^patricia\.wojciechowsk?i@grupoboticario\.com\.br$/.test(actorEmail);
    };
    const limitList = 3000;

    for (let i = 0; i < data.length; i++) {
      const row = data[i];
      const tipoRaw = String(row['TIPO DO REPORT'] || 'Outro').trim();
      const tipo = tipoRaw === 'Outro' ? 'Acesso ao Drive' : tipoRaw;
      const cod  = String(row['COD DO CP'] || 'Unknown').trim();
      const nom  = String(row['NOME DO CP'] || '').trim();
      const ator = String(getAuditActorRawFromRow_(row) || '').toLowerCase().trim();
      const qtd  = parseFloat(row['QUANTIDADE DE ACESSOS']) || 1;
      const dCtx = getNativeDateContext(row['DATA'] || '');
      const dateKey = dCtx ? dCtx.k : '';
      const fDate   = dCtx ? dCtx.f : '';

      // Alterado Patty - 11/05/2026
      // Motivo: impedir que eventos perdidos, CP vazio/Unknown e datas futuras
      // entrem no Explorer, nos totais JSON e na lista fullList.
      const codNormSherlock = normalizeCpCode_(cod);
      const maxAllowedDateSherlock = SH_getResumoMaxDate_();

      if (dateKey && maxAllowedDateSherlock && dateKey > maxAllowedDateSherlock) continue;
      if (!codNormSherlock) continue;
      if (String(cod || '').trim().toLowerCase() === 'unknown') continue;
      if (SH_isResumoNoiseCp_(cod, nom)) continue;

      const isAdminActor = isAdminActor_(ator);

      // 1. DISPONIBILIZAÃ‡Ã•ES (Contagem bruta para fins de controle, excluindo admins)
      if (!isAdminActor) historyTotal += qtd;

      // 2. FILTRO DE EXCLUSÃƒO DE ADMINS (NÃ£o conta como view de franquia)
      if (isAdminActor) continue;

      // 3. FILTRO DE RUÃDO (DLP e AÃ§Ãµes de Sistema)
      const tipoLower = tipo.toLowerCase();
      if (tipoLower.includes('dlp') || ator.includes('dlp') || ator.includes('rule')) continue;

      // 4. APLICA FILTROS DE PESQUISA DO USUÃRIO (FRONTEND)
      if (filterParams) {
        if (filterParams.type && filterParams.type !== '' && tipo !== filterParams.type) continue;
        if (filterParams.term && filterParams.term !== '') {
          if (!matchesCpSearchByCodeFirst_(filterParams.term, cod, nom, ator)) continue;
        }
        if (dateKey && filterParams.dateFrom && dateKey < filterParams.dateFrom) continue;
        if (dateKey && filterParams.dateTo   && dateKey > filterParams.dateTo)   continue;
      }

      // 5. REGRA DE OURO: Multiplicador x2 aplicado ao E-MAIL ÚNICO POR CP POR DIA (Checklist Item 2)
      const aggKey = cod !== 'Unknown' && cod !== '' ? cod : (nom || 'Unknown');
      const uniqAuditKey = `${aggKey}|${ator}|${dateKey}`;
      
      // Alterado Patty - 10/05/2026
      // Motivo: auditValue declarado fora do if para evitar ReferenceError quando chave já existe.
      let auditValue = 0;

      if (!auditSeen.has(uniqAuditKey) && ator && ator !== 'unknown') {
        auditSeen.add(uniqAuditKey);

        // Multiplicador Dinâmico baseada em Produtos (Checklist Item 4)
        const normCod = normalizeCpCode_(cod);
        const prods = cpProductMap[normCod] || 'CI';
        const multiplier = (prods.includes('CI') && prods.includes('CA')) ? 2 : 1;
        auditValue = multiplier;

        audit += auditValue;

        cpAgg[aggKey] = (cpAgg[aggKey] || 0) + auditValue;

        if (cod !== 'Unknown') {
          cpNames[cod] = nom;
          sugs.add(cod);
        }

        if (dateKey) trend[dateKey] = (trend[dateKey] || 0) + auditValue;
      }

      if (resultData.length < limitList - 1 && auditValue > 0) {
        const regAuditado = canAudit ? auditEmailForUiOnly_(ator) : null;
        resultData.push({ t: tipo, c: cod, n: nom, d: fDate, e: regAuditado, i: String(row['ARQUIVO_LOG'] || ''), v: auditValue });
      }
    }

    const rankEntries = Object.entries(cpAgg)
      .filter(([_, v]) => v > 0)
      .map(([c, v]) => ({ cod: c, name: cpNames[c] || c, val: v }))
      .filter(e => !shouldHideFromRanking_(e.name) && !shouldHideFromRanking_(e.cod));

    const top10    = rankEntries.slice().sort((a,b) => b.val - a.val).slice(0, 10);
    const bottom10 = rankEntries.slice().sort((a,b) => a.val - b.val).slice(0, 10);

    const cpUniverseCodes = [];
    const cpNameByCode = {};
    const cpUnresolvedLabels = [];
    const aggCodes = Object.keys(cpAgg || {});
    for (let i = 0; i < aggCodes.length; i++) {
      const rawCode = aggCodes[i];
      const normCode = normalizeCpCode_(rawCode);
      const cpName = String(cpNames[rawCode] || rawCode || '').trim();
      if (!normCode) {
        cpUnresolvedLabels.push(cpName);
        continue;
      }
      cpUniverseCodes.push(normCode);
      if (!cpNameByCode[normCode]) cpNameByCode[normCode] = cpName;
    }
    const fvcOutsideCurrentCycle = buildFvcOutsideInsight_(cpUniverseCodes, cpNameByCode, cpUnresolvedLabels);
    const pendingFranchiseDrives = Array.isArray(fvcOutsideCurrentCycle.cpSemEmailCadastro)
      ? fvcOutsideCurrentCycle.cpSemEmailCadastro.map(cp => {
          const cod = String(cp && cp.cod ? cp.cod : '').trim();
          const nome = String(cp && cp.nome ? cp.nome : '').trim();
          return nome ? `${cod} - ${nome}` : cod;
        }).filter(Boolean)
      : [];
    const pendingFranchiseDriveCount = Number(fvcOutsideCurrentCycle.totalCpSemEmailCadastro) || pendingFranchiseDrives.length;
    const dispoRule = computeDisponibilizacaoRegra2025e2026_(filterParams, historyTotal);
    const fvcMetadata = getFvcMetadata_();
    const cpProductMapFinal = fvcMetadata.map;
    const semFvcCount = fvcMetadata.semFvcCount;

    const totalCps = Object.keys(cpAgg).length;
    // CHECKLIST 09/05: forçar ×2 no Big Number (E-mails Únicos × 2 = CI+CA consolidado)
    const auditTotal = auditSeen.size * 2;

    // BLOCO 3 FIX: audit2025/2026 com força ×2 por entrada única (igual ao path Sheets)
    let audit2025 = 0;
    let audit2026 = 0;
    auditSeen.forEach(key => {
      const parts = key.split('|');
      const dateKey = parts[2];
      if (dateKey && dateKey.startsWith('2025')) audit2025 += 2;
      else if (dateKey && dateKey.startsWith('2026')) audit2026 += 2;
    });

    // BLOCO 4 FIX: qualityTotal dinâmico — CPs com e-mail válido / total carteira FVC
    const fvcCatalog = getCurrentFvcCycleCatalog_();
    const fvcCatalogTotal = (fvcCatalog && fvcCatalog.success && fvcCatalog.totalCp > 0) ? fvcCatalog.totalCp : totalCps;
    const cpsComEmail = Math.max(0, totalCps - pendingFranchiseDriveCount);
    const qualityTotal = fvcCatalogTotal > 0 ? Math.min(100, (cpsComEmail / fvcCatalogTotal) * 100) : 0;

    return {
      success: true,
      canAudit: canAudit,
      // return {
      //  success: true,
      //  canAudit: canAudit,
      //  sourceMethod: 'JSON (Colab ETL)',
      stats: {
        audit: auditTotal,
        audit2025: audit2025,
        audit2026: audit2026,
        dispo: Number(dispoRule.total) || 0,
        dispoRule: dispoRule,
        cps: totalCps,
        semFvcCount: semFvcCount,
        qualityTotal: qualityTotal,
        top10: top10,
        bottom10: bottom10,
        cpUniverseCodes: cpUniverseCodes,
        fvcOutsideCurrentCycle: fvcOutsideCurrentCycle,
        pendingFranchiseDrives: pendingFranchiseDrives,
        pendingFranchiseDriveCount: pendingFranchiseDriveCount,
        types: [],
        trend: trend,
        historyTotal: audit,
        suggestions: Array.from(sugs)
          .map(function(s) { return normalizeCpCode_(s); })
          .filter(function(s) { return s && s !== 'Unknown'; })
          .sort(function(a, b) { return a.localeCompare(b, 'pt-BR', { numeric: true }); }),
        cpNames: cpNames,
        lastUpdate: new Date().toISOString()
      },
      // Alterado Patty - 10/05/2026: sort só nos 3.000 itens já processados (campo d = DD/MM/YYYY)
      list: resultData.slice().sort((a, b) => {
        const toIso = d => { const p = String(d||'').split('/'); return p.length===3 ? p[2]+p[1]+p[0] : String(d||''); };
        return toIso(b.d).localeCompare(toIso(a.d));
      })
    };
//   list: resultData
// };
// -- finalizado --
  } catch(e) {
    Logger.log('[ERRO] buildResponseFromJson_ falhou: ' + e);
    return { success: false, error: e.toString() };
  }
}

function normalizeEvidenceSearchText_(value) {
  return String(value || '')
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '')
    .replace(/[^a-z0-9\s]/g, ' ')
    .replace(/\s+/g, ' ')
    .trim();
}

function extractEvidenceSearchCode_(value) {
  const raw = String(value || '').trim();
  const match = raw.match(/\d{3,}/);
  return normalizeCpCode_(match ? match[0] : raw);
}

// Alterado Patty 10/05/2026: helpers para comprovante leve por período
function getEvidencePeriodsFromRange_(dateFrom, dateTo) {
  const from = String(dateFrom || '').trim();
  const to = String(dateTo || '').trim();

  if (!/^\d{4}-\d{2}-\d{2}$/.test(from)) {
    throw new Error('Informe a data inicial para buscar o comprovante de forma otimizada.');
  }

  const startParts = from.split('-');
  const endValue = /^\d{4}-\d{2}-\d{2}$/.test(to) ? to : from;
  const endParts = endValue.split('-');

  let y = Number(startParts[0]);
  let m = Number(startParts[1]);
  const endY = Number(endParts[0]);
  const endM = Number(endParts[1]);
  const periods = [];

  while (y < endY || (y === endY && m <= endM)) {
    periods.push(String(y) + '-' + String(m).padStart(2, '0'));
    m++;
    if (m > 12) { m = 1; y++; }
    if (periods.length > 24) throw new Error('Período muito amplo para comprovante. Selecione uma janela menor.');
  }

  return periods;
}

function loadJsonMonthlyDataByPeriods_(periods) {
  const folder2026 = DriveApp.getFolderById(SHARD_FOLDER_ID);
  const folder2025 = DriveApp.getFolderById('0AOVybAy3QnD3Uk9PVA');
  const rows = [];
  const wanted = Array.isArray(periods) ? periods : [];

  for (let p = 0; p < wanted.length; p++) {
    const period = String(wanted[p] || '').trim();
    if (!/^\d{4}-\d{2}$/.test(period)) continue;

    const fileName = 'sherlock_logs_' + period + '.json';
    const folder = period.startsWith('2025') ? folder2025 : folder2026;
    const files = folder.getFilesByName(fileName);
    if (!files.hasNext()) continue;

    try {
      const content = files.next().getBlob().getDataAsString('UTF-8');
      const parsed = JSON.parse(content);
      if (Array.isArray(parsed)) { for (let i = 0; i < parsed.length; i++) rows.push(parsed[i]); }
    } catch (e) {
      Logger.log('[WARN] Falha ao ler ' + fileName + ': ' + e);
    }
  }

  return { success: true, data: rows };
}

// ═══════════════════════════════════════════════════════
// SHERLOCK — LEITURA ROBUSTA DO ATOR / E-MAIL
// Alterado Patty - 10/05/2026
// Motivo: restaurar contrato antigo do front, que espera o campo "e"
// preenchido com o registro auditado/e-mail mascarado.
// Admin/dev continua oculto. Cliente/franquia continua visível mascarado.
// ═══════════════════════════════════════════════════════

const SHERLOCK_AUDIT_ALLOWED_EMAILS = [
  'patricia.wojciechowski@grupoboticario.com.br',
  'opatriciakelly@gmail.com',
  'opatricia.kelly@gmail.com'
];

function canCurrentUserAudit_() {
  try {
    var email = String(Session.getActiveUser().getEmail() || '')
      .toLowerCase()
      .trim();
    return SHERLOCK_AUDIT_ALLOWED_EMAILS.indexOf(email) >= 0;
  } catch (e) {
    return false;
  }
}

function SH_getFirstValueByAliases_(row, aliases) {
  if (!row || typeof row !== 'object') return '';
  for (var i = 0; i < aliases.length; i++) {
    var key = aliases[i];
    if (row[key] !== undefined && row[key] !== null && String(row[key]).trim() !== '') {
      return row[key];
    }
  }
  return '';
}

// Alterado Patty - 10/05/2026
// Leitura com NFD normalização das chaves — resolve qualquer variação de acentuação
// nos cabeçalhos dos JSONs sem precisar listar todos os encodings possíveis.
function SH_normKey_(value) {
  return String(value || '')
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '')
    .toUpperCase()
    .replace(/[^A-Z0-9]/g, '');
}

function SH_getValueByAliases_(row, aliases) {
  if (!row || typeof row !== 'object') return '';
  var keyMap = {};
  Object.keys(row).forEach(function(k) {
    keyMap[SH_normKey_(k)] = k;
  });
  for (var i = 0; i < aliases.length; i++) {
    var normalizedAlias = SH_normKey_(aliases[i]);
    var realKey = keyMap[normalizedAlias];
    if (
      realKey &&
      row[realKey] !== undefined &&
      row[realKey] !== null &&
      String(row[realKey]).trim() !== ''
    ) {
      return row[realKey];
    }
  }
  return '';
}

function SH_parseDate_(value) {
  if (!value) return null;
  if (value instanceof Date && !isNaN(value.getTime())) return value;
  var s = String(value).trim();
  if (!s) return null;
  // YYYY-MM-DD
  var m = s.match(/^(\d{4})-(\d{2})-(\d{2})/);
  if (m) return new Date(Number(m[1]), Number(m[2]) - 1, Number(m[3]));
  // DD/MM/YYYY ou DD-MM-YYYY
  m = s.match(/^(\d{2})[\/\-](\d{2})[\/\-](\d{4})/);
  if (m) return new Date(Number(m[3]), Number(m[2]) - 1, Number(m[1]));
  // Serial planilha
  if (/^\d+(\.\d+)?$/.test(s)) {
    var serial = Number(s);
    if (serial > 20000 && serial < 70000) {
      return new Date(Math.round((serial - 25569) * 86400 * 1000));
    }
  }
  return null;
}

function SH_formatDateBR_(date) {
  if (!date || !(date instanceof Date) || isNaN(date.getTime())) return '';
  try {
    return Utilities.formatDate(date, Session.getScriptTimeZone(), 'dd/MM/yyyy');
  } catch(e) {
    var d = date.getDate(), mo = date.getMonth()+1, yr = date.getFullYear();
    return (d<10?'0':'')+d+'/'+(mo<10?'0':'')+mo+'/'+yr;
  }
}

function SH_getAccessDateBR_(row) {
  var rawDate = SH_getValueByAliases_(row, [
    'DATA DO ACESSO',
    'DATA ACESSO',
    'DATA/HORA DO ACESSO',
    'DATA HORA DO ACESSO',
    'DATA EVENTO',
    'DT ACESSO',
    'DT HR ACESSO',
    'EVENT TIME',
    'TIMESTAMP',
    'DATA',
    'DATE'
  ]);
  var parsed = SH_parseDate_(rawDate);
  return SH_formatDateBR_(parsed);
}

function SH_brDateToIso_(brDate) {
  var p = String(brDate || '').split('/');
  if (p.length !== 3) return '';
  return p[2] + '-' + p[1] + '-' + p[0];
}

function SH_getActorEmailRaw_(row) {
  // Usa SH_getValueByAliases_ com NFD — resolve RESPONSÁVEL/RESPONSAVEL/RESPONSÃVEL automaticamente
  return SH_getValueByAliases_(row, [
    'RESPONSÁVEL (ATOR)',
    'RESPONSAVEL (ATOR)',
    'RESPONSÃVEL (ATOR)',
    'RESPONSÁVEL',
    'RESPONSAVEL',
    'ATOR',
    'ACTOR',
    'USER',
    'USUARIO',
    'USUÁRIO',
    'EMAIL',
    'E-MAIL',
    'EMAIL ATOR',
    'E-MAIL ATOR',
    'EMAIL_USUARIO',
    'EMAIL USUARIO',
    'EMAIL USUÁRIO',
    'registro_auditado',
    'REGISTRO AUDITADO',
    'email',
    'e'
  ]);
}

function SH_normalizeCpName_(name) {
  var n = String(name || '').trim().replace(/\s+N[\u00b0\u00ba]?\d+\s*$/i, '').trim();
  if (!n) return n;
  if (!/^CP\s+/i.test(n)) n = 'CP ' + n;
  return n.replace(/\s+/g, ' ').trim();
}

function SH_normalizeEmail_(value) {
  return String(value || '')
    .toLowerCase()
    .trim()
    .replace(/^mailto:/i, '')
    .replace(/[<>"]/g, '');
}

function SH_isAdminActorToHide_(email) {
  var e = SH_normalizeEmail_(email);
  return SHERLOCK_AUDIT_ALLOWED_EMAILS.indexOf(e) >= 0;
}

function SH_maskEmailForAudit_(email) {
  var e = SH_normalizeEmail_(email);
  if (!e || e.indexOf('@') < 0) return '';
  var parts = e.split('@');
  var user = parts[0];
  var domain = parts[1];
  if (!user || !domain) return '';
  var start = user.substring(0, Math.min(2, user.length));
  var end = user.length > 4 ? user.substring(user.length - 1) : '';
  return start + '***' + end + '@' + domain;
}

function getCpEvidenceData(cpInput, dateFrom, dateTo) {
  try {
    const raw = String(cpInput || '').trim();
    if (!raw) return { error: 'Informe o codigo ou nome do CP.' };

    const queryCode = extractEvidenceSearchCode_(raw);
    const queryText = normalizeEvidenceSearchText_(raw.replace(/^\s*\d{3,}\s*[-|:]?\s*/, ''));
    const queryWords = queryText ? queryText.split(/\s+/).filter(w => w.length > 1) : [];
    const from = String(dateFrom || '').trim();
    const to = String(dateTo || '').trim();

    // Alterado Patty 10/05/2026: lê somente os meses do período informado
    const periods = getEvidencePeriodsFromRange_(from, to);
    const jsonResult = loadJsonMonthlyDataByPeriods_(periods);
    if (!jsonResult || !jsonResult.success) {
      return { error: 'Falha ao ler JSONs mensais para emissão do comprovante.' };
    }
    // Sem leitura de produto/FVC — cada linha representa evidência real do log
    const cpProductMap = {};

    // Alterado Patty - 10/05/2026
    // Motivo: a permissão de auditoria deve incluir opatricia.kelly@gmail.com.
    // Antes, AUDIT_ALLOWED só tinha opatriciakelly@gmail.com (sem ponto), causando canAudit=false.
    const canAudit = canCurrentUserAudit_();

    const ADMIN_EMAILS_EXCLUIR = {
      'patricia.wojciechowski@grupoboticario.com.br': true,
      'opatriciakelly@gmail.com': true,
      'opatricia.kelly@gmail.com': true
    };
    const todayKey = toIsoDateLocal_(new Date());
    const list = [];
    const sourceRows = Array.isArray(jsonResult.data) ? jsonResult.data : [];
    const auditSeen = new Set(); // Controle de Unicidade por E-mail/Dia (Checklist Item 2)

    for (let i = 0; i < sourceRows.length; i++) {
      const row = sourceRows[i] || {};

      // Alterado Patty - 10/05/2026: usar SH_getAccessDateBR_ para pegar data real do acesso,
      // não a data da carga. Se não tiver data válida, ignorar a linha.
      const dataBR = SH_getAccessDateBR_(row);
      if (!dataBR) continue;
      const dateKey = SH_brDateToIso_(dataBR);
      if (dateKey && todayKey && dateKey > todayKey) continue;
      const maxAllowedDate = SH_getResumoMaxDate_();
      if (dateKey && maxAllowedDate && dateKey > maxAllowedDate) continue;
      if (dateKey && from && dateKey < from) continue;
      if (dateKey && to && dateKey > to) continue;

      const cod = String(SH_getValueByAliases_(row, ['COD DO CP','COD CP','COD','CODIGO','CÓDIGO']) || '').trim();
      const nomeRawEv = String(SH_getValueByAliases_(row, ['NOME DO CP','NOME CP','NOME','NAME']) || '').trim();
      const nome = SH_normalizeCpName_(nomeRawEv);
      if (!cod && !nome) continue;
      if (cod === 'Unknown' || SH_isResumoNoiseCp_(cod, nomeRawEv)) continue;

      const codNorm = normalizeCpCode_(cod);
      const nameNorm = normalizeEvidenceSearchText_(nome);
      const rowText = normalizeEvidenceSearchText_(cod + ' ' + nome);
      let matches = false;

      if (queryCode && codNorm && codNorm === queryCode) {
        matches = true;
      } else if (queryCode && codNorm && codNorm.indexOf(queryCode) > -1 && queryCode.length >= 4) {
        matches = true;
      } else if (queryWords.length > 0 && queryWords.every(w => rowText.indexOf(w) > -1 || nameNorm.indexOf(w) > -1)) {
        matches = true;
      }

      if (!matches) continue;

      // Alterado Patty - 10/05/2026: leitura robusta do ator; oculta apenas admin/dev
      const actorRaw = getAuditActorRawFromRow_(row);
      const actor = normalizeActorEmail_(actorRaw);
      if (ADMIN_EMAILS_EXCLUIR[actor]) continue;
      if (isDevAdminEmailToHideOnly_(actor)) continue;

      const tipoRaw = String(row['TIPO DO REPORT'] || row['TIPO'] || 'Outro').trim();
      const tipo = tipoRaw === 'Outro' ? 'Acesso ao Drive' : tipoRaw;

      // Regra de Unicidade e Multiplicador para o Comprovante (Checklist Item 3)
      const uniqKey = `${codNorm}|${actor}|${dateKey}`;
      let auditValue = 0;
      if (!auditSeen.has(uniqKey) && actor && actor !== 'unknown') {
        auditSeen.add(uniqKey);
        // Comprovante otimizado: cada linha é evidência real do log, sem FVC/produto
        auditValue = getQuantityFromAuditRow_(row);
      }

      const actorRawSH = SH_getActorEmailRaw_(row);
      const actorEmailSH = SH_normalizeEmail_(actorRawSH);

      let auditEmail = '';
      if (actorEmailSH && !SH_isAdminActorToHide_(actorEmailSH)) {
        auditEmail = SH_maskEmailForAudit_(actorEmailSH);
      } else if (actorEmailSH && SH_isAdminActorToHide_(actorEmailSH)) {
        auditEmail = '\u2014 sistema \u2014';
      }

      list.push({
        t: tipo,
        c: cod,
        n: nome,
        d: dataBR,
        e: canAudit ? auditEmail : '',
        i: String(row['ARQUIVO_LOG'] || row['ARQUIVO'] || ''),
        v: auditValue
      });
    }

    return {
      success: true,
      canAudit: canAudit,
      query: raw,
      normalizedCode: queryCode,
      list: list
    };
  } catch(e) {
    return { error: 'Erro ao localizar CP para comprovante: ' + e.toString() };
  }
}

function onOpen() {
  try {
    const ui = SpreadsheetApp.getUi();
    ui.createMenu('Sherlock Holmes')
      .addItem('ForÃ§ar AtualizaÃ§Ã£o do Banco (Snapshot)', 'admin_forceRefreshSnapshot')
      .addItem('Executar DiagnÃ³stico Completo', 'admin_diagnosticFull')
      .addToUi();
  } catch (e) {
    // getUi() fails when run from contexts without UI (like Web App deployment)
  }
}

function sherlockDiagError_(e) {
  return {
    message: e && e.message ? e.message : String(e),
    name: e && e.name ? e.name : 'Error',
    stack: e && e.stack ? String(e.stack).substring(0, 1200) : ''
  };
}

function sherlockDiagPing() {
  const started = Date.now();
  try {
    let user = 'indisponivel';
    try {
      user = Session.getActiveUser().getEmail() || 'sem-email';
    } catch (e) {
      user = 'erro-session: ' + e;
    }

    return {
      success: true,
      app: 'Sherlock Holmes',
      serverTime: new Date().toISOString(),
      elapsedMs: Date.now() - started,
      user: user,
      timezone: Session.getScriptTimeZone()
    };
  } catch (e) {
    return { success: false, elapsedMs: Date.now() - started, error: sherlockDiagError_(e) };
  }
}

function sherlockDiagDrive() {
  const started = Date.now();
  const result = {
    success: true,
    serverTime: new Date().toISOString(),
    elapsedMs: 0,
    tests: []
  };

  function addTest(name, ok, data) {
    result.tests.push({ name: name, ok: ok, data: data || null });
    if (!ok) result.success = false;
  }

  try {
    try {
      addTest('Session.getActiveUser', true, { email: Session.getActiveUser().getEmail() || 'sem-email' });
    } catch (e) {
      addTest('Session.getActiveUser', false, sherlockDiagError_(e));
    }

    try {
      const cache = CacheService.getScriptCache();
      cache.put('sherlock_diag_ping', String(Date.now()), 60);
      addTest('CacheService', true, { writable: true });
    } catch (e) {
      addTest('CacheService', false, sherlockDiagError_(e));
    }

    try {
      const folder = DriveApp.getFolderById(SHARD_FOLDER_ID);
      addTest('DriveApp.getFolderById', true, { folderName: folder.getName(), folderId: SHARD_FOLDER_ID });

      const cacheFiles = folder.searchFiles("title = '" + CACHE_FILE_NAME + "' and trashed = false");
      addTest('Cache JSON', true, { exists: cacheFiles.hasNext(), fileName: CACHE_FILE_NAME });

      const monthlyFiles = folder.searchFiles("title contains 'sherlock_logs_' and trashed = false");
      let count = 0;
      const sample = [];
      while (monthlyFiles.hasNext() && count < 8) {
        const f = monthlyFiles.next();
        sample.push({ name: f.getName(), size: f.getSize(), updated: f.getLastUpdated().toISOString() });
        count++;
      }
      addTest('Arquivos mensais JSON', true, { sampleCount: count, sample: sample });
    } catch (e) {
      addTest('DriveApp / JSON folder', false, sherlockDiagError_(e));
    }
  } catch (e) {
    result.success = false;
    result.fatal = sherlockDiagError_(e);
  }

  result.elapsedMs = Date.now() - started;
  return result;
}

function sherlockDiagConsolidatedSmoke() {
  const started = Date.now();
  try {
    const res = getConsolidatedData({}, false);
    const stats = res && res.stats ? res.stats : {};
    return {
      success: !(res && res.error),
      elapsedMs: Date.now() - started,
      error: res && res.error ? res.error : '',
      listLength: res && res.list ? res.list.length : 0,
      canAudit: !!(res && res.canAudit),
      stats: {
        audit: Number(stats.audit) || 0,
        cps: Number(stats.cps) || 0,
        lastUpdate: stats.lastUpdate || '',
        trendKeys: stats.trend ? Object.keys(stats.trend).length : 0
      }
    };
  } catch (e) {
    return {
      success: false,
      elapsedMs: Date.now() - started,
      error: sherlockDiagError_(e)
    };
  }
}

/**
 * LÃª todas as abas de uma planilha de resumo e converte para o formato JSON.
 */
function readYearlySummarySheet_(ssId) {
  try {
    const ss = SpreadsheetApp.openById(ssId);
    const sheets = ss.getSheets();
    let allRows = [];

    sheets.forEach(sh => {
      const lastRow = sh.getLastRow();
      if (lastRow < 2) return;
      
      const data = sh.getRange(1, 1, lastRow, 8).getValues();
      let startIdx = 1;
      for (let r=0; r<Math.min(10, data.length); r++) {
         const txt = data[r].join('|').toUpperCase();
         if (txt.includes('DATA') || txt.includes('COD')) { startIdx = r + 1; break; }
      }
      
      for (let i = startIdx; i < data.length; i++) {
        const row = data[i];
        if (!row[1] && !row[2]) continue;
        
        allRows.push({
          "TIPO DO REPORT": String(row[0] || 'Acesso ao Drive'),
          "COD DO CP": String(row[1] || 'Unknown'),
          "NOME DO CP": String(row[2] || ''),
          "DATA": row[3],
          "RESPONSÃVEL (ATOR)": String(row[4] || ''),
          "QUANTIDADE DE ACESSOS": row[7] || 1,
          "ARQUIVO_LOG": String(row[6] || '')
        });
      }
    });
    return allRows;
  } catch(e) {
    Logger.log('[ERRO readSheet] ' + e);
    return [];
  }
}

/**
 * LÃª arquivo CSV e converte para o formato Sherlock.
 */
function readYearlySummaryCsv_(file) {
  try {
    const csvContent = file.getBlob().getDataAsString('ISO-8859-1'); 
    let data = Utilities.parseCsv(csvContent, ';'); 
    
    if (data[0] && data[0].length < 3) {
      data = Utilities.parseCsv(csvContent, ',');
    }

    let start = 0;
    for(let r=0; r<Math.min(10, data.length); r++) {
      const txt = data[r].join('|').toUpperCase();
      if (txt.includes('DATA') || txt.includes('COD')) { start = r+1; break; }
    }

    let rows = [];
    for (let i = start; i < data.length; i++) {
      const r = data[i];
      if (!r[1] && !r[2]) continue;
      rows.push({
        "TIPO DO REPORT": String(r[0] || 'Acesso ao Drive'),
        "COD DO CP": String(r[1] || 'Unknown').trim(),
        "NOME DO CP": String(r[2] || '').trim(),
        "DATA": r[3],
        "RESPONSÃVEL (ATOR)": String(r[4] || '').trim(),
        "QUANTIDADE DE ACESSOS": parseFloat(r[7]) || 1,
        "ARQUIVO_LOG": String(r[6] || '')
      });
    }
    return rows;
  } catch(e) { 
    Logger.log('[ERRO readCsv] ' + e);
    return []; 
  }
}

/** 
 * Obtém o mapeamento de produtos por CP para determinar o multiplicador de views (x2 se CI+CA)
 */
// }

/** 
 * Obtém metadados da FVC (Mapa de Produtos e Contagem SEM FVC)
 */
function getFvcMetadata_() {
  const cache = CacheService.getScriptCache();
  const cached = cache.get("FVC_METADATA_CACHE");
  if (cached) return JSON.parse(cached);

  try {
    const ss = SpreadsheetApp.openById(PRODUCT_MAPPING_SS_ID);
    const sh = ss.getSheetByName('CPS 062026');
    if (!sh) throw new Error("Aba 'CPS 062026' não encontrada.");
    
    const data = sh.getDataRange().getValues();
    const map = {};
    let semFvcCount = 0;
    
    for (let i = 1; i < data.length; i++) {
      const cod = normalizeCpCode_(data[i][0]);
      const prod = String(data[i][2]).trim().toUpperCase();
      if (cod) map[cod] = prod;
      if (prod === 'SEM FVC' || prod.includes('SEM FVC')) semFvcCount++;
    }
    
    const result = { map: map, semFvcCount: semFvcCount };
    cache.put("FVC_METADATA_CACHE", JSON.stringify(result), 600); // 10 min — releitura frequente
    return result;
  } catch (e) {
    console.error("Erro ao carregar metadados FVC:", e);
    return { map: {}, semFvcCount: 0 };
  }
}

function getCpProductMap_() {
  return getFvcMetadata_().map;
}

/**
 * Limpa todos os caches do Sherlock para forcar atualizacao completa
 */
function clearAllSherlockCache() {
  const cache = CacheService.getScriptCache();
  cache.remove('FVC_METADATA_CACHE');
  cache.remove('CP_PRODUCT_MAP_CACHE');
  cache.remove('FVC_CATALOG_CACHE_KEY');
  cache.remove('FVC_MISSING_EMAIL_CACHE_KEY');
  return { success: true };
}

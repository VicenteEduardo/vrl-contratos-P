# MED — Memo de Engenharia e Desenvolvimento

## Projecto: linkedinsms.vercel.app (EDUALL Document Generator)

### Ficheiros
- `index.html` — aplicação single‑page (HTML + CSS + JS)
- `assinatura.png` — imagem da assinatura + carimbo da VRL (141 KB, embutida como base64 no JS)

---

## Resumo das Alterações

### 1. Termo de Demonstração (documento novo)
- Adicionado botão "Termo de Demonstração", card de funcionalidade, entrada na sidebar
- Criado modal `#termModal` com campos: Instituição, NIF, Sede, Representante, Cargo, Tel, Email, VRL Nome, VRL Cargo, Duração (15/30 dias), Dia, Mês, Ano, Funcionalidades
- Implementado `generateTerm()`, `renderTermPages(d)` — capa + 10 cláusulas + bloco de assinaturas
- Implementado `restoreTermModal(d)` para edição e reabertura
- Implementado `buildTermDOCX()` com estrutura completa de cláusulas e assinatura
- Adicionado listener de abertura do modal para restaurar último termo salvo do `localStorage('vrl_terms')`
- Selector de duração com cálculo automático da data de fim (`atualizarFimTermo()`)
- Actualizados `editContract()`, `getDefaultFilename()` e `doExportDOCX()` para suportar tipo `term`

### 2. Imagem da Assinatura
- Substituído texto‑based VRL signature (`${blank(200)}` etc.) por `IMG_ASSINATURA = 'assinatura.png'` em todos os tipos de documento
- Removido `crossorigin="anonymous"` de todas as tags `<img>` de assinatura (bloqueava a exibição)
- Removido texto "Director Técnico" abaixo da imagem
- Ajustado `.sign-wrap` `margin-top: 60px → 10px` e largura da imagem `200px → 180px`
- **Assinatura embutida como base64 data URL** (188 KB) — resolve problemas de CORS em file:// e Vercel, e preserva transparência PNG

### 3. Cores / Capas — Economia de Tinta
Todas as 5 capas (Cliente, Parceiro, Carta, Termo, Proposta Comercial, Proposta Pública) foram redesenhadas de fundo laranja gradiente para **fundo branco minimalista com acentos laranja**:
- Barra superior laranja sólida de 10px
- "VRL" em laranja (#E8631A) bold; "identidade" em cinza claro
- "Sistema EDUALL" em laranja, letter‑spacing 6px
- Tag do documento com borda laranja (1.5px) e texto laranja
- Título a preto (#1a1a1a)
- Linha de acento laranja (3px height)
- Dois círculos decorativos **apenas com borda** (sem preenchimento):
  - Grande: 500px, borda 2.5px, opacidade 35%
  - Pequeno: 200px, borda 1.5px, opacidade 25%
- Rodapé: texto cinza, separador `#eee`
- **Zero tinta de cor no fundo** — imprime só o que é essencial

### 4. Números de Telefone
- Actualizado de `+244 944 968 819` para `945 299 410 / 945 299 629` em 11 ocorrências (formulários, fallbacks, assinaturas CRM)

### 5. Campos Padrão
- Removido "Lumanisa Manuel" / "Director Geral" como valores padrão de todos os campos e fallbacks JS para todos os tipos de documento

### 6. Datas Automáticas
- Campos Dia, Mês, Ano preenchem automaticamente com a data de hoje no `DOMContentLoaded`

### 7. Exportação PDF
- Removido código de conversão manual de imagens para canvas (causava perda de qualidade e desaparecimento da assinatura)
- Adicionado `useCORS: true` ao `html2canvas` — recarrega imagens CDN com `crossOrigin: 'Anonymous'` (ImageKit devolve `Access-Control-Allow-Origin: *`)
- Assinatura é data URL base64 → não precisa de CORS

### 8. Exportação DOCX
- Removido termo da lista de skip (já não precisa)
- `buildTermDOCX()` totalmente funcional

### 9. Variáveis Globais
- `IMG_CAPA`, `IMG_INICIO`, `IMG_RODAPE`, `IMG_ASSINATURA` mudaram de `const` para `let`

---

## Arquitectura
- A aplicação é um único ficheiro `index.html`
- Todas as capas são geradas dinamicamente via JavaScript (inline styles)
- `coverPage(wrap, type)` — função central para capas de Cliente e Parceiro
- Cada documento tem a sua função de renderização: `renderClientPages`, `renderPartnerPages`, `renderTermPages`, `renderLetterPages`, `renderProposalPages`, `renderPublicPartnershipPages`
- PDF: html2canvas 1.4.1 + jsPDF 2.5.1
- DOCX: biblioteca `docx` (Packer, Document, Paragraph, TextRun, ImageRun)

---

## Comandos Úteis

```bash
# Verificar sintaxe JS
node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');const m=h.match(/<script>([\s\S]*?)<\/script>/);try{new Function(m[1]);console.log('OK')}catch(e){console.log(e.message)}"

# Verificar tamanho do ficheiro
ls -lh index.html

# Contar linhas
wc -l index.html
```

---

## TODO / Problemas Conhecidos
- [ ] Nenhum por agora

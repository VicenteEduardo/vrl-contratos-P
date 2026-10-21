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

### 10. Contrato de Cartões (RFID) — novo tipo de contrato
- Adicionado um 3.º tipo no modal de Contratos: **Implementação / Parceria / Cartões & RFID** (`btnTypeCards`, `#formCards`, `selectType('cards')`)
- Tabela de itens dinâmica (`#c_itemsBody`): linhas com **Tipo** (Cartão RFID / Eduall Smart / Personalizado), **Descrição**, **Quantidade**, **Preço unitário (Kz)** e **Subtotal**; total calculado automaticamente (`addCardItem`, `removeCardItem`, `recalcCardItems`, `readCardItems`, `loadCardItems`)
- Botões rápidos “+ Cartão RFID”, “+ Eduall Smart”, “+ Personalizado”
- **Modalidade** (`#c_modo`): *Com software EDUALL* ou **EDUALL Smart** (instituições que **não** usam o software EDUALL) — altera o objeto, a Cláusula 9 (licença vs. exclusão de licença) e a capa/subtítulo
- **Pagamento por modalidade** (`atualizarModoCartao`, `cardPaymentSummary`, `atualizarPagamentoCartao`):
  - **Cartões (com software)** → apenas **Pagamento Único**; o valor dos itens é o valor total
  - **Eduall Smart** → **Pagamento Único / Anual / Semestral / Mensal / A definir**; o valor dos itens é o **valor anual de referência**, dividido pela periodicidade (Mensal = ÷12, Semestral = ÷2)
  - **A definir** → gera apenas menção de que as condições ficam em aberto e que nada é exigível no período (sem valor de prestação)
  - Caixa “Resumo do pagamento” no modal mostra o valor anual e o valor por período
- Implementado `renderCardsPages(d)` (capa + 15 cláusulas + tabela de itens + bloco de assinaturas) e `cardsItemsTableHTML(d)`
- Implementado `buildCardsDOCX(d,h)` com a mesma estrutura para exportação DOCX
- `coverPage()` estendido para o tipo `cards`
- `getDefaultFilename()`, `historyTitle()`, `editContract()` e `restoreFromHistory()` atualizados para o tipo `cards` (com `restoreCardsModal(d)`)
- Adicionados botão no hero, card na grelha de documentos e entrada no painel “Documentos EDUALL”

### 11. Contrato de Parceria — dados do parceiro e remuneração configurável
- Adicionados ao formulário `#formPartner` os campos: **Nome do Parceiro**, **NIF (opcional)**, **Sede/Localidade (opcional)**, **Nome do Representante**, **Cargo do Representante**, **Duração da Parceria (meses)** e **Data da Assinatura** (dia/mês/ano)
- Adicionado selector **Tipo de Remuneração**: **Percentagem (%)** ou **Valor fixo (Kz)**, com campo de valor e periodicidade (`por cliente angariado`, `por mês`, `pagamento único`)
- `aplicarModeloParceiro()` aplica valores padrão por modelo (Reseller 15%, Referral 200.000 Kz, Expert 500.000 Kz) e `atualizarRemuneracaoParceiro()` alterna os campos e mostra um resumo em tempo real
- `renderPartnerPages()` passou a preencher nome, NIF, sede, representante e cargo no contrato, usando espaços em branco quando os campos estão vazios; a Cláusula 5 é gerada dinamicamente por `partnerClause5HTML(d)` com a remuneração escolhida; Cláusula 9 usa a duração indicada e o bloco de assinaturas mostra os dados do parceiro
- `buildPartnerDOCX()` e `partnerClause5DOCX(d,h)` replicam a mesma lógica no DOCX
- `restorePartnerModal(d)`, `editContract()`, `restoreFromHistory()`, `historyTitle()` e `getDefaultFilename()` actualizados para guardar/restaurar os novos campos e usar o nome do parceiro no título e no nome do ficheiro
- Campos Dia/Mês/Ano do parceiro (`f_pdia`, `f_pmes`, `f_pano`) incluídos no preenchimento automático com a data de hoje

### 12. Contrato de Parceria para pessoa singular
- Formulário ajustado para pessoas singulares: **Nome Completo do Parceiro**, **NIF (opcional)**, **Residência / Endereço (opcional)** e **Duração**; removidos os campos de representante e cargo
- Texto das partes passou a usar **“residente em”** em vez de **“sede em / representada por / na qualidade de”**
- Cláusula 2 identifica o parceiro pelo nome, NIF e residência, actuando em nome próprio
- Bloco de assinaturas mostra apenas nome e NIF do parceiro
- `buildPartnerDOCX()` actualizado com a mesma estrutura
- A página **“Programa de Parceiros — Modelos Disponíveis”** deixa de ser incluída quando o contrato é por **percentagem**, evitando contradizer a remuneração acordada; o Sumário é ajustado automaticamente (páginas e entrada removida)
- Removidas da Cláusula 5 as cláusulas automáticas de transição de estatuto (Referral→Reseller, Reseller→Expert) e a atribuição de território exclusivo com referência a “Anexo I”, por não fazerem parte das condições acordadas

### 13. Exportação PDF mais rápida
- Escala de renderização reduzida de `2` para `1,5–1,6` (~150 dpi), mantendo o texto nítido
- Imagens convertidas para **JPEG (qualidade 0,92)** em vez de PNG — PDF final bastante mais leve
- Cabeçalho e rodapé pré-carregados uma só vez antes de renderizar as páginas
- Progresso visível no botão: **“A gerar PDF… 2/6”** e **“A descarregar…”**
- Inserção no PDF com compressão `FAST`


---

## Arquitectura
- A aplicação é um único ficheiro `index.html`
- Todas as capas são geradas dinamicamente via JavaScript (inline styles)
- `coverPage(wrap, type)` — função central para capas de Cliente e Parceiro
- Cada documento tem a sua função de renderização: `renderClientPages`, `renderPartnerPages`, `renderCardsPages`, `renderTermPages`, `renderLetterPages`, `renderProposalPages`, `renderPublicPartnershipPages`
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

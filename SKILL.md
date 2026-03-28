---
name: brd-specialist
description: "Use this skill to generate Business Requirements Documents (BRDs) for financial systems projects. Triggers when user asks to create, write, or draft a BRD. Accepts three input types: existing documents (PDF/DOCX), meeting transcriptions, or structured questionnaire responses. Delivers output in both DOCX (BIP template) and Markdown (Confluence-ready). Automatically activates a Brazil Tax Reform module (LC 214/2025) when CBS, IBS, DeRE, or Split Payment context is detected. Do NOT use for evaluating/scoring existing BRDs or for non-financial system projects."
---

# BRD SPECIALIST - GERADOR DE BUSINESS REQUIREMENTS DOCUMENT

**Versão**: 1.0.0
**Domínio**: Sistemas financeiros (agnóstico de cliente e regulação)
**Módulo Condicional**: Reforma Tributária LC 214/2025 (ativado automaticamente quando detectado)

---

## QUANDO USAR ESTE SKILL

Use este skill quando:
- Usuário pedir para "gerar", "criar" ou "redigir" um BRD
- Input fornecido for: documento de requisitos (PDF/DOCX), transcrição de reunião, ou respostas a questionário
- Contexto for sistema financeiro (banking, insurance, investimentos, tesouraria, fiscal/contábil)
- Usuário mencionar "Business Requirements Document", "BRD", "documento de requisitos de negócio"

NÃO use este skill quando:
- Tarefa for avaliar/pontuar um BRD existente (este skill só gera)
- Contexto for desenvolvimento de produto não-financeiro
- Usuário pedir apenas um resumo ou análise de documento (use file-reading)
- Output desejado for FRD (Functional Requirements Document) ou SRS (System Requirements Specification)

---

## O QUE ESTE SKILL FAZ

Gera BRDs completos e estruturados para projetos de sistemas financeiros a partir de três tipos de input: documentos existentes (PDF/DOCX), transcrições de reuniões, ou respostas a um questionário guiado.

**Output entregue sempre em dois formatos:**
- `brd_[sistema]_v1.0.md` - Markdown estruturado (pronto para Confluence)
- `brd_[sistema]_v1.0.docx` - Word formatado no template BIP

**Ativa automaticamente o Módulo LC 214/2025** quando detecta contexto de Reforma Tributária (CBS, IBS, DeRE, Nota Fiscal, tributos indiretos).

---

## INSTRUÇÕES PRINCIPAIS

### PASSO 0: CARREGAR SKILLS DE SUPORTE (UMA VEZ, NO INÍCIO)

```bash
# Sempre carregar antes de começar - não reler durante o workflow
view /mnt/skills/public/docx/SKILL.md
```

Se o input contiver arquivo PDF:
```bash
view /mnt/skills/public/pdf-reading/SKILL.md
```

---

### PASSO 1: IDENTIFICAR MODO DE OPERAÇÃO

Detectar qual tipo de input foi fornecido:

| Modo | Trigger | Ação |
|------|---------|------|
| **DOCUMENTO** | Arquivo .pdf ou .docx anexado | Ir para Extração de Documento |
| **TRANSCRIÇÃO** | Texto bruto de reunião colado/anexado | Ir para Extração de Transcrição |
| **QUESTIONÁRIO** | Nenhum input estruturado / input vago | Ir para Modo Questionário |
| **MISTO** | Combinação de dois ou mais acima | Processar todos, depois consolidar |

**Se não houver input suficiente para gerar o BRD**, ativar Modo Questionário antes de gerar.

---

### PASSO 2A: EXTRAÇÃO - DOCUMENTO (PDF ou DOCX)

```bash
# Verificar arquivo em uploads
ls /mnt/user-data/uploads/

# Para PDF: extrair texto
pdftotext -layout /mnt/user-data/uploads/[arquivo].pdf /home/claude/input_text.txt
cat /home/claude/input_text.txt

# Para DOCX: extrair via pandoc
pandoc /mnt/user-data/uploads/[arquivo].docx -o /home/claude/input_text.md
cat /home/claude/input_text.md
```

Após extração, identificar e mapear para as seções do BRD:
- Qual é o sistema/produto sendo especificado?
- Quais são os requisitos funcionais explícitos?
- Existem regras de negócio documentadas?
- Há integrações mencionadas?
- Existem critérios de aceite ou métricas?

---

### PASSO 2B: EXTRAÇÃO - TRANSCRIÇÃO DE REUNIÃO

Processar o texto bruto da transcrição identificando:

1. **Participantes e papéis** (extrair para seção Stakeholders)
2. **Problemas/necessidades declarados** (extrair para Objetivo e Contexto)
3. **Funcionalidades discutidas** (extrair para Requisitos Funcionais)
4. **Restrições mencionadas** (extrair para Requisitos Não-Funcionais e RAID)
5. **Decisões tomadas** (extrair para Premissas ou Regras de Negócio)
6. **Pontos em aberto** (extrair para Issues no RAID)
7. **Datas e prazos citados** (extrair para Cronograma/RAID)

SEMPRE sinalizar no BRD gerado quais informações vieram da transcrição com nota:
`> Fonte: Transcrição de reunião - requer validação com stakeholders`

---

### PASSO 2C: MODO QUESTIONÁRIO

Quando não houver input suficiente, coletar informações via perguntas estruturadas.
Fazer as perguntas em blocos, não todas de uma vez.

**BLOCO 1 - Contexto (fazer primeiro):**
```
1. Qual é o nome do sistema ou módulo a ser especificado?
2. Em uma frase, qual é o problema de negócio que este sistema resolve?
3. Quem são os usuários finais (personas)?
4. Qual é a data prevista de entrega ou aprovação do BRD?
```

**BLOCO 2 - Escopo (após Bloco 1):**
```
5. Quais são as 3-5 funcionalidades principais?
6. O que está FORA do escopo desta fase?
7. Quais sistemas existentes serão impactados ou integrados?
8. Há regulação específica aplicável? (ex: Banco Central, CVM, Receita Federal)
```

**BLOCO 3 - Requisitos Detalhados (após Bloco 2):**
```
9. Existe alguma regra de negócio crítica que o sistema deve respeitar?
10. Quais são os critérios de aceite mínimos?
11. Há restrições de performance, disponibilidade ou segurança?
12. Quem aprova o BRD? (sponsor, gestor, área de negócio)
```

**SE contexto de Reforma Tributária for detectado, adicionar Bloco Fiscal.**
Ver seção MÓDULO LC 214/2025 abaixo.

---

### PASSO 3: DETECTAR MÓDULO LC 214/2025

Verificar se o conteúdo extraído contém algum destes indicadores:

**Indicadores primários (qualquer um ativa o módulo):**
- Menção a "CBS", "IBS", "Reforma Tributária", "LC 214", "Split Payment"
- Menção a "DeRE" (Declaração Eletrônica de Regime Específico)
- Menção a "Nota Fiscal eletrônica" em contexto de adequação regulatória
- Menção a tributos indiretos (PIS, COFINS, ISS, ICMS) em contexto de reforma

**Se ativado:** Incluir seção adicional "9. Impactos da Reforma Tributária (LC 214/2025)" no BRD.
**Se não ativado:** Omitir completamente. Não mencionar no BRD.

**ATENÇÃO CRÍTICA:** O termo correto é sempre **DeRE** (Declaração Eletrônica de Regime Específico).
Nunca usar "DERY" - este acrônimo está incorreto.

---

### PASSO 4: ESTRUTURAR O BRD

Montar o BRD seguindo esta estrutura padrão. Adaptar nível de detalhe ao input disponível.
Onde informação for insuficiente, preencher com `[A CONFIRMAR COM STAKEHOLDERS]`.

```
ESTRUTURA PADRÃO DO BRD:

1. DOCUMENTO DE CONTROLE
   1.1 Informações do Documento
   1.2 Histórico de Versões
   1.3 Lista de Aprovadores

2. OBJETIVO E CONTEXTO
   2.1 Objetivo do Projeto
   2.2 Contexto de Negócio
   2.3 Problema / Necessidade
   2.4 Benefícios Esperados

3. ESCOPO
   3.1 In-Scope (o que está incluído)
   3.2 Out-of-Scope (o que está excluído)
   3.3 Premissas

4. PARTES INTERESSADAS (STAKEHOLDERS)
   4.1 Patrocinador (Sponsor)
   4.2 Gestor do Projeto
   4.3 Áreas Envolvidas
   4.4 Usuários Finais

5. REQUISITOS FUNCIONAIS
   5.1 [Funcionalidade 1]
   5.2 [Funcionalidade 2]
   ... (quantas forem necessárias)

6. REQUISITOS NÃO-FUNCIONAIS
   6.1 Performance e Disponibilidade
   6.2 Segurança e Controle de Acesso
   6.3 Auditoria e Rastreabilidade
   6.4 Integração e Compatibilidade

7. REGRAS DE NEGÓCIO

8. INTEGRAÇÕES E DEPENDÊNCIAS
   8.1 Sistemas de Origem (Upstream)
   8.2 Sistemas de Destino (Downstream)
   8.3 APIs e Contratos de Dados

9. [CONDICIONAL] IMPACTOS DA REFORMA TRIBUTÁRIA (LC 214/2025)
   - Ver seção MÓDULO LC 214/2025 abaixo

10. MODELO DE DADOS / DICIONÁRIO DE DADOS
    (Incluir apenas se houver informação suficiente no input)

11. CRITÉRIOS DE ACEITE (DEFINITION OF DONE)

12. RAID
    12.1 Riscos
    12.2 Premissas
    12.3 Issues / Impedimentos
    12.4 Dependências
```

---

### PASSO 5: GERAR MARKDOWN

Criar o arquivo Markdown pronto para Confluence.

```bash
# Criar arquivo markdown em /home/claude
```

Formato Confluence-friendly:
- Usar `#` para H1, `##` para H2, `###` para H3
- Tabelas com `|col|col|` para seções de dados estruturados
- Blocos `> ⚠️` para alertas e pontos a confirmar
- Blocos `> 📌` para decisões tomadas em reunião
- IDs de requisito no formato `RF-001`, `RNF-001`, `RN-001`
- Critérios de aceite numerados `CA-001`, `CA-002`

Salvar em:
```
/home/claude/brd_[nome_sistema]_v1.0.md
```

---

### PASSO 6: GERAR DOCX (TEMPLATE BIP)

Instalar dependência se necessário:
```bash
npm list -g docx 2>/dev/null || npm install -g docx
```

Criar script Node.js com as especificações exatas do template BIP:

**Especificações do Template BIP:**
```javascript
// PÁGINA
// Formato: A4 (11906 x 16838 DXA)
// Margem esquerda/direita: 1701 DXA cada
// Content width: 8504 DXA (11906 - 1701 - 1701)
// Margem superior/inferior: 1134 DXA (~2cm)

// TIPOGRAFIA (em half-points para docx-js: pt * 2)
// H1: Work Sans Light, 64pt = size: 128, bold: false
// H2: Work Sans SemiBold, 28pt = size: 56, bold: true
// H3: Work Sans SemiBold, 20pt = size: 40, bold: true
// Body: Work Sans, 20pt = size: 40
// Cor de todo texto: #0C2340 (navy)
// Fallback de fonte se Work Sans não disponível: Arial

// CORES BIP
// Primárias: #DA291C (vermelho), #0C2340 (navy), #FFFFFF (branco)
// Header de tabela: #DADEE2 (cinza claro)

// TABELAS
// Bordas: BorderStyle.NONE (sem bordas visíveis)
// Largura total: 8494 DXA
// Header row fill: #DADEE2 com ShadingType.CLEAR
// Texto de header: Work Sans SemiBold, color #0C2340

// RODAPÉ
// Texto esquerdo: bip-group.com
// Centro: checkboxes de classificação (padrão: Internal)
// Direito: número de página (PageNumber.CURRENT)
```

Criar script completo em `/home/claude/generate_brd.js` e executar:
```bash
node /home/claude/generate_brd.js
```

Validar arquivo gerado:
```bash
python scripts/office/validate.py /home/claude/brd_[nome_sistema]_v1.0.docx 2>/dev/null || \
  ls -lh /home/claude/brd_[nome_sistema]_v1.0.docx
```

---

### PASSO 7: ENTREGAR OS DOIS ARQUIVOS

```bash
# Copiar ambos para outputs
cp /home/claude/brd_[nome_sistema]_v1.0.md /mnt/user-data/outputs/
cp /home/claude/brd_[nome_sistema]_v1.0.docx /mnt/user-data/outputs/
```

Chamar `present_files` com os dois arquivos:
```
present_files([
  "/mnt/user-data/outputs/brd_[nome_sistema]_v1.0.docx",
  "/mnt/user-data/outputs/brd_[nome_sistema]_v1.0.md"
])
```

Mensagem de entrega: informar brevemente quantas seções foram geradas, quais campos ficaram como `[A CONFIRMAR]`, e se o Módulo LC 214/2025 foi ativado.

---

### REGRAS OBRIGATÓRIAS

**SEMPRE:**
- [ ] Carregar docx SKILL.md no início (Passo 0)
- [ ] Usar IDs padronizados para requisitos: `RF-001`, `RNF-001`, `RN-001`, `CA-001`
- [ ] Marcar lacunas com `[A CONFIRMAR COM STAKEHOLDERS]` em vez de inventar
- [ ] Entregar DOIS arquivos: .md + .docx
- [ ] Usar `present_files` para entregar (não apenas imprimir o caminho)
- [ ] Trabalhar em `/home/claude` e copiar para `/mnt/user-data/outputs` no final
- [ ] Usar **DeRE** (não "DERY") quando tratar de Reforma Tributária

**NUNCA:**
- [ ] Inventar dados de negócio não presentes no input
- [ ] Incluir nomes de clientes reais no SKILL.md (agnóstico)
- [ ] Usar a seção LC 214/2025 se contexto fiscal não foi detectado
- [ ] Criar arquivos diretamente em `/mnt/user-data/outputs` (usar `/home/claude` primeiro)
- [ ] Criar o DOCX com python-docx (usar docx-js conforme docx SKILL.md)

---

## MÓDULO LC 214/2025 - REFORMA TRIBUTÁRIA

Ativar apenas quando contexto fiscal/tributário for detectado (ver Passo 3).

### Estrutura da Seção 9 (quando ativada):

```markdown
## 9. IMPACTOS DA REFORMA TRIBUTÁRIA (LC 214/2025)

### 9.1 Tributos Impactados
| Tributo | Status | Observação |
|---------|--------|-----------|
| CBS (Contribuição sobre Bens e Serviços) | [Impactado/Não impactado] | ... |
| IBS (Imposto sobre Bens e Serviços) | [Impactado/Não impactado] | ... |
| IS (Imposto Seletivo) | [Impactado/Não impactado] | ... |

### 9.2 DeRE (Declaração Eletrônica de Regime Específico)
[Descrever obrigações acessórias do sistema sob a DeRE, se aplicável]

### 9.3 Split Payment
[Descrever impacto do mecanismo de Split Payment nos fluxos financeiros do sistema]

### 9.4 Calendário de Conformidade
| Marco | Data Prevista | Sistemas Afetados | Status |
|-------|--------------|-------------------|--------|
| ... | ... | ... | [A CONFIRMAR] |

### 9.5 Mapeamento de Sistemas Afetados
[Listar quais funcionalidades/integrações serão impactadas e de que forma]

### 9.6 Requisitos Específicos LC 214/2025
[RF-LC-001] ...
[RF-LC-002] ...

### 9.7 Riscos Regulatórios
[Riscos de não-conformidade, penalidades, calendário restritivo]
```

### Perguntas Adicionais do Questionário (Bloco Fiscal):

```
13. Quais tributos indiretos incidem sobre as operações do sistema hoje?
14. O sistema processa Notas Fiscais ou documentos fiscais eletrônicos?
15. Há integração atual com SPED, PGDAS ou outros sistemas da Receita Federal?
16. Qual é o prazo regulatório de adequação à CBS/IBS para esta operação?
17. O sistema participará do mecanismo de Split Payment?
18. Haverá necessidade de DeRE para este regime tributário específico?
```

---

## EXEMPLOS

### Exemplo 1: Input via Documento PDF (Happy Path)

**Input do Usuário:**
> "Tenho um levantamento de requisitos em PDF. Gere o BRD formal a partir dele."
> [anexa: levantamento_requisitos_sistema_pagamentos.pdf]

**Ação do Claude:**
1. Carrega docx SKILL.md e pdf-reading SKILL.md
2. Extrai texto do PDF:
   ```bash
   pdftotext -layout /mnt/user-data/uploads/levantamento_requisitos_sistema_pagamentos.pdf \
     /home/claude/input_text.txt
   ```
3. Analisa texto: detecta sistema de pagamentos, não detecta contexto de Reforma Tributária
4. Estrutura BRD com seções 1-8, 10-12 (sem seção 9)
5. Gera `/home/claude/brd_sistema_pagamentos_v1.0.md`
6. Gera `/home/claude/brd_sistema_pagamentos_v1.0.docx` via docx-js com template BIP
7. Copia ambos para outputs, chama `present_files`
8. Mensagem: "BRD gerado com 12 seções. 4 campos marcados como [A CONFIRMAR]. Módulo LC 214/2025 não ativado."

---

### Exemplo 2: Input via Transcrição de Reunião com Contexto Fiscal

**Input do Usuário:**
> "Aqui está a transcrição da nossa reunião de kickoff sobre adequação à Reforma Tributária:
> [João]: Precisamos adaptar o módulo de NF-e para suportar CBS e IBS até Q2...
> [Maria]: Também vai precisar de integração com o novo layout de DeRE...
> [Carlos]: O Split Payment vai impactar o fluxo de caixa, temos que mapear isso..."

**Ação do Claude:**
1. Carrega docx SKILL.md
2. Processa transcrição: identifica participantes (João, Maria, Carlos), sistema (módulo NF-e), funcionalidades discutidas
3. Detecta indicadores fiscais: CBS, IBS, DeRE, Split Payment - **ATIVA MÓDULO LC 214/2025**
4. Gera BRD completo incluindo seção 9 (Reforma Tributária)
5. Marca como `[A CONFIRMAR]` os campos sem informação: critérios de aceite, datas específicas, stakeholders formais
6. Adiciona nota `> Fonte: Transcrição de reunião - requer validação com stakeholders` nas seções extraídas da transcrição
7. Entrega brd_modulo_nfe_v1.0.md + brd_modulo_nfe_v1.0.docx

---

### Exemplo 3: Modo Questionário (Sem Input Prévio)

**Input do Usuário:**
> "Preciso de um BRD para um novo sistema de gestão de fundos de investimento."

**Ação do Claude:**
1. Detecta: sem documento anexado, sem transcrição - ativar Modo Questionário
2. Apresenta Bloco 1 de perguntas (contexto)
3. Após respostas, apresenta Bloco 2 (escopo)
4. Detecta em "fundos de investimento" possível contexto regulatório - pergunta sobre CMN/CVM
5. Se contexto fiscal identificado, apresenta Bloco Fiscal
6. Após coletar todos os blocos, gera BRD sem perguntas adicionais
7. Entrega os dois arquivos

---

### Exemplo 4: Tratamento de Erro - Arquivo Não Extraível

**Problema:** PDF é um scan sem texto extraível.

**Detecção:**
```bash
pdftotext -f 1 -l 1 /mnt/user-data/uploads/arquivo.pdf - | wc -c
# Output: 0 ou menos de 50 caracteres = provavelmente scan
```

**Ação do Claude:**
1. Detecta PDF escaneado
2. Rasteriza as primeiras páginas para leitura visual:
   ```bash
   pdftoppm -jpeg -r 150 -f 1 -l 3 /mnt/user-data/uploads/arquivo.pdf /home/claude/page
   ls /home/claude/page-*.jpg
   ```
3. Usa visão para interpretar as páginas rasterizadas
4. Extrai requisitos visualmente e prossegue com geração normal
5. Adiciona nota no BRD: `> Fonte: Documento escaneado - qualidade de extração pode ser limitada`

---

## NOTAS IMPORTANTES

### Limitações Conhecidas

- Transcrições com múltiplos participantes e temas sobrepostos podem gerar mapeamento impreciso de responsabilidades. Sempre marcar com `[A CONFIRMAR]`.
- PDFs com formatação complexa (tabelas aninhadas, layouts multi-coluna) podem ter extração de texto degradada. Usar rasterização como fallback.
- O template BIP usa a fonte Work Sans. Se não estiver instalada no ambiente de destino (Word), o documento será renderizado com a fonte fallback (Arial). A estrutura permanece correta.
- BRDs gerados a partir de inputs incompletos terão muitos campos `[A CONFIRMAR]`. Isso é intencional e esperado - não preencher lacunas com suposições.

### Armadilhas Comuns

#### Módulo LC 214/2025 ativado incorretamente
- **Causa:** Menção genérica a "impostos" ou "tributação" sem relação com a Reforma
- **Solução:** Só ativar se houver ao menos um indicador primário (CBS, IBS, DeRE, Split Payment, LC 214)

#### DOCX gerado com fonte errada ou formatação quebrada
- **Causa:** docx-js não instalado, ou script com erro de sintaxe
- **Solução:**
  ```bash
  npm install -g docx  # instalar se ausente
  node /home/claude/generate_brd.js 2>&1  # ver erro completo
  ```
  Validar com `python scripts/office/validate.py` se disponível

#### Arquivo não aparece para o usuário
- **Causa:** Esqueceu de copiar para `/mnt/user-data/outputs` ou não chamou `present_files`
- **Solução:** Sempre executar os dois passos antes de encerrar

#### Requisitos sem ID padronizado
- **Causa:** Geração livre de texto sem aplicar convenção de IDs
- **Solução:** Sempre usar `RF-001`, `RNF-001`, `RN-001`, `CA-001` - nunca "Requisito 1" ou "REQ01"

#### DeRE grafado incorretamente como "DERY"
- **Causa:** Confusão com acrônimo em inglês
- **Solução:** Sempre **DeRE** (Declaração Eletrônica de Regime Específico). Sem exceção.

### Dependências

- `docx` via npm (`npm install -g docx`) - para geração do DOCX
- `poppler-utils` (pdftotext, pdftoppm) - para extração de PDF
- `pandoc` - para extração de DOCX existente

### Interações com Outros Skills

- Complementa `docx` skill: este skill usa docx skill para geração do Word final
- Complementa `pdf-reading` skill: usado quando input é PDF
- Complementa `pdf` skill: se usuário quiser o BRD também em PDF após geração
- Não conflita com nenhum skill existente (escopo exclusivo: geração de BRD)

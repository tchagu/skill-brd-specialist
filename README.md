# skill-brd-specialist

> 🇧🇷 [Português](#português) | 🇺🇸 [English](#english)

---

## Português

### O que é este skill?

Um skill para Claude AI que gera **Business Requirements Documents (BRDs)** completos para projetos de sistemas financeiros.

Aceita três tipos de input e entrega sempre dois formatos de output, sem precisar de configuração adicional.

### Inputs suportados

- Documento de requisitos existente (PDF ou DOCX)
- Transcrição de reunião (texto bruto)
- Questionário guiado (quando não há documento prévio)

### Outputs entregues

- `brd_[sistema]_v1.0.md` — Markdown estruturado, pronto para colar no Confluence
- `brd_[sistema]_v1.0.docx` — Word formatado no template BIP

### Módulo Reforma Tributária (LC 214/2025)

O skill detecta automaticamente contexto de Reforma Tributária (CBS, IBS, DeRE, Split Payment) e ativa uma seção adicional no BRD com mapeamento de impactos, requisitos específicos e calendário de conformidade. Se o contexto fiscal não for detectado, a seção é omitida.

### Como instalar

Copie a pasta `brd-specialist/` para o diretório de skills do seu Claude:

```
~/.claude/skills/brd-specialist/SKILL.md
```

### Como usar

Basta pedir ao Claude para gerar um BRD:

> "Gere um BRD para o sistema de pagamentos a partir deste documento."

> "Preciso de um BRD para adequação à Reforma Tributária. Tenho a transcrição da reunião de kickoff."

> "Crie um BRD para o módulo de gestão de fundos."

### Requisitos

- Claude Pro, Max, Team ou Enterprise com execução de código habilitada
- Dependências instaladas automaticamente pelo skill: `docx` (npm), `poppler-utils`, `pandoc`

### Autor

**Thiago Lopes** — Senior Project Manager e AI-Driven Delivery Lead com experiência em projetos de sistemas financeiros, seguros e tecnologia empresarial.

[LinkedIn](https://linkedin.com/in/tchagu/)

---

## English

### What this skill does

A Claude AI skill that generates complete **Business Requirements Documents (BRDs)** for financial systems projects.

Accepts three input types and always delivers two output formats, with no additional configuration required.

### Supported inputs

- Existing requirements document (PDF or DOCX)
- Meeting transcription (raw text)
- Guided questionnaire (when no prior document is available)

### Delivered outputs

- `brd_[system]_v1.0.md` — Structured Markdown, ready to paste into Confluence
- `brd_[system]_v1.0.docx` — Word document formatted with the BIP template

### Brazil Tax Reform Module (LC 214/2025)

The skill automatically detects Tax Reform context (CBS, IBS, DeRE, Split Payment) and activates an additional BRD section covering impact mapping, specific requirements, and compliance calendar. If no fiscal context is detected, the section is omitted entirely.

### Installation

Copy the `brd-specialist/` folder to your Claude skills directory:

```
~/.claude/skills/brd-specialist/SKILL.md
```

### Usage

Just ask Claude to generate a BRD:

> "Generate a BRD for the payments system based on this document."

> "I need a BRD for Tax Reform compliance. I have the kickoff meeting transcript."

> "Create a BRD for the fund management module."

### Requirements

- Claude Pro, Max, Team or Enterprise with code execution enabled
- Dependencies installed automatically by the skill: `docx` (npm), `poppler-utils`, `pandoc`

### Author

**Thiago Lopes** — Senior Project Manager and AI-Driven Delivery Lead with experience in financial services, insurance, and enterprise technology projects.

[LinkedIn](https://linkedin.com/in/tchagu/)

---

## License

MIT — free to use, adapt and share with attribution.

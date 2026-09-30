---
name: orcamento-horas-simples
description: >
  Gera a versão SIMPLES do DOCX "Detalhamento do Orçamento de Horas" no padrão Sankhya
  (mesmo layout DSTECH v.4 sobre o Modelo de Documento Padrão Sankhya 2026), com identificação, resumo,
  desenvolvimento e premissas — SEM fases de alinhamento, homologação e documentação,
  SEM resumo consolidado, SEM pontos a definir e SEM escopo negativo. Acionar para
  "documento simples de orçamento de horas", "orçamento de horas simples", "orçamento
  simples", "versão simples do orçamento". As horas vêm ditadas pelo usuário — esta skill
  não estima nada. Orçamento completo (com fases e consolidado) é a orcamento-horas-docx.
---

# Orçamento de Horas — versão simples

Formato reduzido para quando o usuário já sabe o que será feito e quanto custa cada item. O documento é só o espelho disso, no visual padrão.

**Total do orçamento = soma dos itens de desenvolvimento.** Não existe fase adicional.

## Ordem fixa das seções

1. Título **"Detalhamento do Orçamento de Horas"** (fixo) + subtítulo com o nome da demanda (filete verde) + tabela de identificação
2. `1. Resumo` — parágrafo + tabela Assunto/Descrição
3. `2. Escopo do desenvolvimento` (nova página) — itens agrupados em H3 por assunto, com subtotal por grupo, e **Total do orçamento**
4. `3. Premissas`

Nada além disso. Se o usuário pedir consolidado, pontos a definir, escopo negativo ou fases, o documento pedido é o completo — usar a skill `orcamento-horas-docx`.

## Entrada — o usuário dita, a skill não calcula

O usuário informa item e horas diretamente. Regras:

- **Nunca estimar horas.** Item sem hora informada → perguntar, não chutar. Não acionar a `sankhya-estimativa-planejador`.
- **Somar, não arredondar.** Subtotal de grupo = soma dos itens do grupo; total = soma dos subtotais. Divergência entre o total ditado pelo usuário e a soma dos itens → apontar e perguntar qual vale.
- **Horas cravadas, sem faixas.** Faixa informada (16–24h) → perguntar o valor.
- Um único grupo, ou nenhum: item sem classificação de assunto entra direto sob `2.1 Desenvolvimento`, sem H3.

## Regras de conteúdo

Valem as mesmas da skill completa:

- **Linguagem funcional**, público misto. Sem nome interno de tabela nem jargão ("TGFCAB", "de-para", "staging", "idempotência", "rollback").
- **Nunca citar banco de dados** — nada de "Oracle", "SQL Server", "dual-dialeto" ou variação.
- **Nunca expor a base de cálculo das horas** — complexidade, fator, multiplicador, produtividade e LOC são internos. Vale inclusive em `Premissas`.
- **Não incluir** confiança da estimativa, abordagem técnica nem stack de frontend.
- **Tabela de identificação:** `ID DSTech` (nunca "Chamado/OS"). `Consultor Funcional` = autor do escopo lido; sem autor identificável, **omitir a linha inteira**. `Orçamento Realizado por` é sempre `João Coluci`, obrigatório, salvo outro nome informado.
- Nomes de grupo idênticos entre `resumo.rotinas` e os H3 do `desenvolvimento`. Divergência quebra a localização e é o erro mais comum.
- Letra dos itens (`a)`, `b)`, ...) reinicia a cada grupo.

## Humanizer — obrigatório

**Todo texto redigido passa pela skill `humanizer` antes de virar JSON.** Campos que passam: `resumo.intro` · `resumo.rotinas[]` (2ª coluna) · `escopos[0].intro` · `escopos[0].desenvolvimento[].item` e `.subs[]` · `observacao.texto` · `premissas.itens[]`.

**Não passa:** rótulos da identificação, nomes de grupo e rotina, qualquer valor numérico, títulos de seção padronizados.

## JSON

Mesmo schema da skill completa (`~/.claude/skills/orcamento-horas-docx/references/schema-orcamento.md`), com estes campos **omitidos**:

- `escopos[0].tituloFases` e `escopos[0].fases`
- `escopos[0].subtotalDevLabel` e `escopos[0].subtotalDev` — no formato simples o subtotal sempre iguala o total; duas linhas com o mesmo número só confundem
- `escopos[0].naoEscopo`
- `consolidado`
- `pontosDefinir`
- `escopoNegativo`

O gerador já trata cada um como opcional — omitir basta, não existe flag. Modelo: `examples/orcamento-simples-exemplo.json`.

## Padrão visual

Idêntico ao da skill completa — mesmo gerador, mesmos assets DSTECH v.4 (capa, contracapa, logos, Work Sans embutida), mesma paleta. **Não sobrescrever cores nem fonte por documento.** Detalhes em `~/.claude/skills/orcamento-horas-docx/references/design-sankhya.md`.

## Fluxo

1. Coletar os itens e horas ditados pelo usuário; conferir a soma.
2. Redigir resumo, descrições e premissas.
3. Passar cada texto pelo **`humanizer`**.
4. Montar o JSON e salvar em local temporário.
5. Gerar, reusando o gerador da skill completa:

```bash
npm ls -g docx || npm install -g docx
export NODE_PATH="$(npm root -g)"
node "$HOME/.claude/skills/orcamento-horas-docx/scripts/gerar-orcamento-docx.js" \
  --content /caminho/dados.json \
  --output "/caminho/Orcamento Nome da Demanda.docx"
```

6. Validar (forçar UTF-8; o validador quebra com cp1252 no Windows):

```bash
export PYTHONUTF8=1
python "$HOME/.claude/skills/docx/scripts/office/validate.py" "/caminho/Orcamento.docx"
```

Esperado: `All validations PASSED!`.

## Cuidados

- DOCX **aberto no Word** → gravação falha com `EBUSY`; pedir para fechar e regerar.
- Não guardar dado real de cliente na skill; `examples/` é genérico.

## Recursos

`examples/orcamento-simples-exemplo.json` modelo genérico · gerador, schema, design e assets vêm da skill `orcamento-horas-docx` (não duplicar).

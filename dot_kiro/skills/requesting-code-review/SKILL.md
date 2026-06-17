---
name: requesting-code-review
description: >
  Use ao completar tasks, implementar features significativas, ou antes de merge
  para verificar se o trabalho atende requisitos e padrões de qualidade. Aciona
  automaticamente validação com critérios do Clean Code para Agentes de IA.
  Ative sempre que o usuário pedir review, code review, revisão de código,
  validação de implementação, ou quando terminar uma tarefa de implementação.
---

# Requesting Code Review

Realize revisão de código para capturar problemas antes que se propaguem. A revisão usa contexto preciso para avaliação — diff do git, requisitos, e os critérios do Clean Code para Agentes.

**Princípio central:** Revise cedo, revise frequentemente.

## Integração com Clean Code para Agentes

Toda revisão de código DEVE aplicar o checklist da skill `clean-code-agentes` como critério adicional de qualidade. Isso significa que além dos checks tradicionais (bugs, segurança, arquitetura), o reviewer valida:

1. **Funções e arquivos pequenos** — funções 4-20 linhas, arquivos < 500 linhas
2. **SRP** — cada módulo/classe faz UMA coisa
3. **Nomes grepáveis** — nomes únicos e específicos, sem genéricos
4. **Comentários de contexto** — PORQUÊ, não QUÊ; proveniência documentada
5. **Tipos explícitos** — sem `any`, `dynamic`, `object` genérico
6. **DRY** — sem duplicação que o agente possa esquecer de atualizar
7. **Testes** — existem e são executáveis
8. **Aninhamento** — máximo 2 níveis de indentação
9. **Erros com contexto** — mensagens incluem valor recebido e formato esperado

Referência completa: `.kiro/skills/clean-code-agentes/SKILL.md`

---

## Quando Solicitar Review

**Obrigatório:**
- Após completar cada task em desenvolvimento orientado a specs
- Após implementar feature significativa
- Antes de merge em branch principal

**Opcional mas valioso:**
- Quando travado (perspectiva fresca)
- Antes de refatoração (baseline check)
- Após corrigir bug complexo

## Como Realizar a Review

**1. Obter os SHAs do git:**
```bash
git log --oneline -5
# Identificar o commit base e o commit head
```

**2. Obter o diff para análise:**
```bash
git diff --stat BASE_SHA..HEAD_SHA
git diff BASE_SHA..HEAD_SHA
```

**3. Executar a revisão usando o template abaixo:**

Delegue a revisão a um subagent com o prompt do template em [code-reviewer.md](code-reviewer.md), ou execute inline se subagents não estiverem disponíveis.

**4. Agir sobre o feedback:**
- Corrija issues **Critical** imediatamente
- Corrija issues **Important** antes de prosseguir
- Anote issues **Minor** para depois
- Conteste se o reviewer estiver errado (com raciocínio técnico)

## Checklist de Review (Resumo)

Ao revisar, verifique TODOS estes aspectos:

### Alinhamento com Requisitos
- Implementação corresponde ao plano/requisitos?
- Desvios são melhorias justificadas ou problemas?
- Toda funcionalidade planejada está presente?

### Qualidade de Código (Clean Code para Agentes)
- Funções ≤ 20 linhas? Arquivos ≤ 500 linhas?
- Separação clara de responsabilidades (SRP)?
- Nomes grepáveis e distintivos?
- Comentários explicam o PORQUÊ de decisões não-óbvias?
- Tipos explícitos em assinaturas?
- Sem duplicação (DRY)?
- Early returns em vez de aninhamento profundo?
- Erros com contexto (valor recebido + formato esperado)?
- Tratamento adequado de erros e edge cases?

### Arquitetura
- Decisões de design sólidas?
- Performance e escalabilidade razoáveis?
- Preocupações de segurança endereçadas?
- Integra-se de forma limpa com código existente?
- Dependency Injection para dependências externas?

### Testes
- Testes verificam comportamento real, não mocks?
- Edge cases cobertos?
- Testes de integração onde relevante?
- Todos os testes passando?
- Testes executáveis sem setup manual?

### Prontidão para Produção
- Estratégia de migração se schema mudou?
- Backward compatibility considerada?
- Documentação completa?
- Sem bugs óbvios?

## Calibração

Categorize issues pela severidade real. Nem tudo é Critical.
Reconheça o que foi bem feito antes de listar problemas — elogio preciso ajuda o implementador a confiar no resto do feedback.

Se encontrar desvios significativos do plano, sinalize especificamente para que o implementador confirme se foi intencional.

## Formato de Output

```markdown
### Pontos Fortes
[O que está bem feito? Seja específico.]

### Issues

#### Critical (Deve Corrigir)
[Bugs, problemas de segurança, riscos de perda de dados, funcionalidade quebrada]

#### Important (Deveria Corrigir)
[Problemas de arquitetura, features faltando, tratamento de erros insuficiente, gaps de teste]

#### Minor (Bom Ter)
[Estilo de código, oportunidades de otimização, polish de documentação]

Para cada issue:
- Arquivo:linha referência
- O que está errado
- Por que importa
- Como corrigir (se não for óbvio)

### Clean Code para Agentes — Conformidade
[Avaliação rápida dos 9 critérios principais da skill clean-code-agentes]
- ✅ ou ❌ para cada critério com nota breve se necessário

### Recomendações
[Melhorias para qualidade, arquitetura ou processo]

### Veredito

**Pronto para merge?** [Sim | Não | Com correções]

**Justificativa:** [1-2 frases de assessment técnico]
```

## Integração com Workflows

**Desenvolvimento orientado a Specs (Kiro):**
- Review após CADA task completada
- Capturar issues antes que se acumulem
- Corrigir antes de prosseguir para próxima task

**Implementação Ad-Hoc:**
- Review antes de merge
- Review quando travado

## Red Flags

**Nunca:**
- Pule review porque "é simples"
- Ignore issues Critical
- Prossiga com issues Important não corrigidos
- Discuta com feedback técnico válido

**Se o reviewer estiver errado:**
- Conteste com raciocínio técnico
- Mostre código/testes que provam que funciona
- Peça esclarecimento

---

Veja template completo do reviewer em: [code-reviewer.md](code-reviewer.md)

---
name: clean-code-agentes
description: >
  Valida e aplica práticas de Clean Code otimizadas para agentes de IA, baseadas
  no artigo "Clean Code pra Agentes de IA" de Akita. Use esta skill sempre que
  estiver escrevendo, revisando, refatorando ou gerando código novo. Aplique
  automaticamente ao criar funções, classes, módulos ou arquivos. Também use
  quando o usuário mencionar clean code, qualidade de código, boas práticas,
  legibilidade para agentes, otimização de contexto, ou quando pedir revisão
  de código. Esta skill complementa as steering rules do projeto com foco
  específico em tornar o código mais eficiente para leitura e edição por LLMs.
---

# Clean Code para Agentes de IA

Skill baseada no artigo ["Clean Code pra Agentes de IA"](https://akitaonrails.com/2026/04/20/clean-code-para-agentes-de-ia) de Fabio Akita, que re-ranqueia os princípios do Clean Code de Uncle Bob para o contexto onde o leitor primário do código é um agente LLM.

## Por que isso importa

Agentes de IA têm restrições técnicas concretas que mudam o peso de cada prática:

- **Truncamento**: CLIs de agente leem ~2000 linhas por vez. Arquivo grande não entra inteiro na janela de contexto.
- **Atenção degrada com contexto**: quanto mais coisa na janela, pior a precisão de detalhe.
- **Grep é mais barato que read**: o agente prefere `rg "funcName"` a carregar arquivo inteiro. Nomes únicos tornam isso eficaz.
- **Tool calls custam tokens**: cada Read/Edit/Bash gasta tokens. Arquivo curto e output enxuto mantêm o agente produtivo.
- **Latência importa**: arquivo grande demora para processar e vira gargalo de sessão.

## Checklist de validação (em ordem de prioridade)

Ao escrever ou revisar código, valide cada item na ordem abaixo. Os primeiros são os mais críticos para produtividade do agente.

### 1. Funções pequenas e arquivos pequenos

- Funções: 4-20 linhas. Se passou de 20, divida.
- Arquivos: abaixo de 500 linhas, idealmente 200-300.
- Uma função pequena cabe numa única tool call sem truncamento.
- Um arquivo curto cabe numa única leitura com atenção cheia.

### 2. Single Responsibility Principle (SRP)

- Cada módulo/classe faz UMA coisa e tem UMA razão para mudar.
- Classe de 800 linhas que faz três coisas é pior que três classes de 250 linhas.
- O agente consegue isolar a unidade, entender sem carregar o resto, testar focado e editar sem efeito colateral.

### 3. Nomes significativos e únicos (grepáveis)

- Nomes devem ser específicos e retornar poucos resultados no grep.
- Evite nomes genéricos: `data`, `process`, `handler`, `Manager`, `Service`, `Helper`, `Utils`.
- Prefira nomes distintivos: `UserRegistrationValidator`, `InvoiceLineItemTotal`.
- **Regra prática**: se grep do nome retorna muita coisa irrelevante, o nome está ruim para o agente.

### 4. Comentários com contexto e proveniência

Esta é a inversão mais importante em relação ao Clean Code original.

- **Escreva o PORQUÊ, não o QUÊ**. O agente sabe o que `x++` faz. Ele não sabe por que você escolheu essa abordagem.
- Documente: qual bug motivou a lógica, qual constraint de negócio força a ordem, qual workaround existe por bug de lib upstream, qual issue/commit é referência.
- **Não apague comentários que o agente escreveu** durante refatoração. Eles carregam contexto que o agente vai querer ler na próxima interação.
- Docstrings em funções públicas: intenção + exemplo de uso.
- Referencie números de issue/commit quando uma linha existe por causa de bug específico.
- O único comentário ruim é o óbvio e redundante (`// incrementa i` acima de `i++`).

### 5. Tipos explícitos

- Sem `any`, sem `dynamic`, sem `object` genérico, sem funções sem tipo de retorno.
- Código tipado dá gabarito imediato ao agente: assinatura fala o que entra e o que sai.
- Código dinâmico sem anotação obriga o agente a inferir tipo por uso, custando raciocínio e errando frequentemente.

### 6. DRY rigoroso

- Duplicação é pior para agente que para humano: ao mudar algo replicado, o agente pode atualizar uma cópia e esquecer das outras.
- A janela de atenção do agente não tem "gravidade natural" para puxar "tem mais cópias disso".
- Fatore em função ou módulo reutilizável. Não é estética, é segurança de refactor automatizado.

### 7. Testes executáveis pelo agente

- Comando para rodar testes deve estar documentado (README, CLAUDE.md, Makefile, package.json).
- Output com formato previsível que o agente parseia.
- Sem dependência de setup manual, seed de banco, config fora do repo, credencial secreta.
- Cada função nova ganha teste. Bug fix ganha teste de regressão.
- Mock de I/O externo com classes fake nomeadas, não stubs inline.
- F.I.R.S.T: Fast, Independent, Repeatable, Self-Validating, Timely.
- TDD é obrigação técnica, não filosofia. Sem teste, o agente fica chutando.

### 8. Estrutura de diretório previsível

- Siga convenções do framework (Rails, Django, Next.js, Laravel, ASP.NET MVC).
- Paths previsíveis: se existe `controllers/users`, o agente antecipa `models/user`.
- Prefira módulos pequenos e focados a "god files".

### 9. Dependency Injection e testabilidade

- Injete dependências via construtor/parâmetro, não via global/import hardcoded.
- Envolva libs de terceiros em interface fina do projeto.
- DI permite substituir dependência real por fake no teste sem gambiarra.

### 10. Evitar aninhamento profundo

- Máximo 2 níveis de indentação.
- Use early returns, guard clauses, pattern matching.
- Cada nível extra de indentação custa atenção desproporcional ao agente.

### 11. Erros com contexto

- Mensagens de exceção devem incluir o valor recebido e o formato esperado.
- `raise ValueError("invalid input")` não ajuda. `raise ValueError(f"invalid input: received {repr(x)}, expected non-empty string of digits")` ajuda.
- O agente usa mensagem de exceção como sinal para debug. Mensagem vaga = round extra.

### 12. Formatação consistente

- Use o formatador padrão da linguagem e não discuta estilo além disso.
- O agente lida bem com qualquer estilo consistente.

### 13. Nunca comentário óbvio

- `// increment i by 1` acima de `i++` desperdiça tokens.
- Em 2008 era ruim porque poluía visual. Em 2026 é ruim porque custa dinheiro real em tokens.

## Práticas novas (não previstas no Clean Code original)

- **Meta-documentação para agentes**: CLAUDE.md, AGENTS.md, steering files. Curto, direto, imperativo, orientado a ações.
- **README com arquitetura alto nível**: diagrama simples em ASCII ou Mermaid encurta o caminho do agente.
- **Logging estruturado**: JSON com campos nomeados. O agente parseia JSON trivialmente. Printf solto em texto livre obriga parsing heurístico.
- **Comandos de observabilidade acessíveis**: `pnpm test`, `make lint`, `cargo check`. Quanto mais previsível, melhor.
- **Scripts de setup idempotentes**: o agente deve poder rodar `bin/setup` numa máquina limpa.
- **Defensive programming explícito**: circuit breaker, retry com backoff, timeout, graceful degradation. O agente não propõe sozinho — precisa de instrução.

## Como aplicar

Ao gerar ou revisar código:

1. Verifique se funções e arquivos estão dentro dos limites de tamanho
2. Confirme que cada módulo tem responsabilidade única
3. Valide que nomes são grepáveis e distintivos
4. Preserve comentários de proveniência e adicione quando houver decisão não-óbvia
5. Garanta tipos explícitos em assinaturas
6. Elimine duplicação real
7. Confirme que testes existem e são executáveis sem setup manual
8. Mantenha estrutura de diretório previsível
9. Use DI para dependências externas
10. Achate lógica aninhada com early returns
11. Inclua contexto em mensagens de erro
12. Rode formatador automático
13. Remova comentários óbvios, preserve comentários de contexto

---

*Fonte: [Clean Code pra Agentes de IA](https://akitaonrails.com/2026/04/20/clean-code-para-agentes-de-ia) — Fabio Akita, abril 2026. Conteúdo parafraseado e reorganizado para uso como skill de validação.*

# Codereview - Front Talk
## Apresentação de 30 minutos

---

# Sumário

**Introdução** (~2 min)
- O que vamos abordar?

**@Desenvolvedor** (~10 min)
- Problema
- Changelist
- PR Description
- Lidando com Comentários

**@Reviewer** (~15 min)
- Analisando Description
- Entendendo o problema
- Analisando solução
- Análise técnica
- Análise de padrões
- Comentando na PR
- Lidando com respostas e resolveds
- Request Change, Comment ou Approve

**Pós-créditos**
- IA na pipeline
- Hora de fazer cagada

---

# Introdução

## O que vamos abordar?

Code review é um processo onde **alguém que não é o autor** examina o código.

**Objetivo principal:** é garantir segurança, qualidade, evolução saudável da codebase e principalmente dos produtos.

Hoje vamos ver dois lados da mesma moeda:
1. **Como desenvolvedor** - Como criar PRs que facilitam o review
2. **Como reviewer** - Como revisar de forma eficiente e construtiva

> "Otimizamos para a velocidade que um **time** consegue entregar um produto junto, não para a velocidade que um **indivíduo** consegue escrever código."

> "Toda regra tem a sua exceção desde que seja bem defendida."

---

# @Desenvolvedor

## Problema

Uma boa changelist começa pelo entendimento do problema. O primeiro passo é entender claramente qual problema está sendo solucionado.

**Perguntas que te ajudam a entender:**
- Qual bug estou corrigindo?
- Qual funcionalidade estou implementando?
- Por que isso é necessário agora?
- Quem é afetado por esse problema?

Um bom entendimento vai gerar uma boa explicação que vai fazer com que um codereview seja executado com mais eficiência. Tanto por um humano mas principlamente para uma IA.

> Esse entendimento nós temos que transcrever a nível técnico e a nível de produto em uma PR description.

---

## Changelist (CL)

### O que é uma CL pequena?

Uma CL deve ser **uma mudança auto-contida** que:
- Aborda **uma única coisa** (uma parte de uma feature, não a feature inteira)
- Inclui os testes relacionados (quanto existir)
- Contém tudo que o reviewer precisa para entender a mudança
- Mantém o sistema funcionando após o merge

### Por que CLs pequenas?

| Benefício | Explicação |
|-----------|------------|
| **Review mais rápido** | É mais fácil achar 5 minutos várias vezes do que 30 minutos de uma vez |
| **Review mais completo** | Volumes grandes geram frustração e pontos importantes são perdidos |
| **Menos bugs** | Menos mudanças = mais fácil raciocinar sobre o impacto |
| **Menos trabalho perdido** | Se rejeitado, menos código para jogar fora ou corrigido|
| **Mais fácil de mergear** | Menos conflitos com outras mudanças |
| **Mais fácil fazer rollback** | Uma CL grande provavelmente toca arquivos que mudam frequentemente |

### Tamanho ideal

- **~400 linhas** = geralmente razoável
- **~acima de 800 linhas** = geralmente muito grande
- O número de **arquivos** também importa: 400 linhas em 8 arquivo pode ser ok, em 50 arquivos é demais

> **Reviewers podem rejeitar uma CL apenas por ser muito grande.**

### Estratégias para dividir CLs

1. **Stacking** - Empilhar várias CLs uma baseada na outra
2. **Por arquivos** - Agrupar por arquivos que precisam de revisores diferentes
3. **Horizontal** - Dividir por camadas (componente, ui, hooks e repository)
4. **Vertical** - Dividir por features independentes
5. **Refactoring separado** - Refatoração em CL separada de bug fix/feature

---

## PR Description

A PR description é um **registro público e permanente** da mudança. Ela deve comunicar:

1. **O QUE** está sendo alterado?
2. **POR QUE** essas alterações estão sendo feitas?
3. **COMO** validar que está funcionando?

### Título da PR

Seguimos o padrão: `[XXX-0000] - título do PR`

```
✅ BOM: "[DEV-1234] - Corrige validação de CPF no formulário de cadastro"
✅ BOM: "[HOTFIX] [FE-5678] - Corrige crash ao abrir modal de pagamento"
❌ RUIM: "Fix bug"
❌ RUIM: "Ajustes"
❌ RUIM: "WIP"
```

**Para hotfixes no app:** adicione a tag `[HOTFIX]` antes do código do ticket.

### Estrutura do corpo da PR

| Seção | O que incluir | Obrigatório? |
|-------|---------------|--------------|
| **🔖 Escopo** | Descrição detalhada do problema e da solução. Vincule o ticket. | ✅ Sim |
| **🚩 Known issues** | Limitações ou problemas conhecidos introduzidos. Se não houver, escreva "N/A". | ✅ Sim |
| **📁 Evidências** | Prints, GIFs ou vídeos demonstrando a mudança funcionando | ✅ Sim |
| **Roteiro de testes** | Passo a passo do caminho feliz + critérios de aceitação | ✅ Sim |
| **💥 Pontos de impacto** | Quais partes do sistema podem ter sido afetadas | ✅ Sim |

### 🔖 Escopo - Como escrever bem

O escopo deve responder:
- **Qual era o problema?** (contexto)
- **O que foi feito para resolver?** (solução)
- **Por que essa abordagem?** (justificativa)

```
❌ RUIM:
"Corrigido o bug do cadastro"

✅ BOM:
"O formulário de cadastro não validava CPF com pontuação (ex: 123.456.789-00),
causando erro ao submeter. 

Adicionada normalização do CPF antes da validação, removendo pontos e traços.
Essa abordagem foi escolhida para manter compatibilidade com CPFs copiados de
documentos formatados."
```

### 📁 Evidências - Sempre obrigatório

**Por que evidências são obrigatórias?**
- Provam que a mudança funciona como esperado
- Facilitam o review (reviewer não precisa rodar local para casos visuais)
- Servem como documentação futura
- Mostra que você teve cuidado e executou os testes mínimos

| Tipo de mudança | Evidência recomendada |
|-----------------|----------------------|
| UI/Visual | GIF ou vídeo mostrando antes/depois |
| Fluxo/Interação | Vídeo do caminho feliz |
| Bug fix | Print ou vídeo do comportamento corrigido |
| Refactoring | Print dos testes passando |

### Roteiro de testes - O caminho feliz

Descreva o passo a passo para o reviewer validar a mudança:

```markdown
## Roteiro de testes: Validação de CPF

Validar que CPFs com e sem formatação são aceitos no cadastro.

### Critérios de aceitação
- [ ] CPF sem pontuação (12345678900) é aceito
- [ ] CPF com pontuação (123.456.789-00) é aceito  
- [ ] CPF inválido mostra mensagem de erro
- [ ] Campo não quebra com caracteres especiais
```

### Revise a description antes de submeter

CLs mudam significativamente durante o review. Antes de fazer merge, revise se a description ainda reflete o que a CL faz.

> **Dica:** Uma boa description com evidências claras acelera drasticamente o tempo de review.

---

## Lidando com Comentários

### Não leve para o lado pessoal

O objetivo do review é manter a qualidade da codebase, não criticar você.

Quando um reviewer adiciona uma request change, pense nisso como uma tentativa de ajudar você, o codebase e a empresa, nunca como um ataque pessoal.

> **Nunca responda com raiva a comentários de code review.** É uma quebra séria de etiqueta profissional que ficará para sempre registrada.

### Corrija o código

Se um reviewer diz que não entendeu algo:

1. **Primeira opção:** Clarifique o código em si
3. **Segunda opção:** Explique na ferramenta de review (apenas se a opção anterior não fizer sentido)

> Se um reviewer não entendeu, futuros leitores provavelmente também não vão entender.

### Pense colaborativamente

```
❌ RUIM: "Não, eu não vou fazer isso."
❌ RUIM: Marcar como resolved, sem discutir.

✅ BOM: "Eu fui com X por causa de [prós/contras] com [tradeoffs].
Meu entendimento é que usar Y seria pior porque [razões].
Você está sugerindo que Y serve melhor os tradeoffs originais,
que devemos pesar os tradeoffs diferentemente, ou algo mais?
```

### Resolvendo conflitos

1. Tente chegar a um consenso com o reviewer
2. Se não conseguir, consulte os princípios do "Standard of Code Review"
3. Escale para o time lead ou manager se necessário

> **Não deixe uma CL parada porque autor e reviewer não conseguem concordar.**

---

# @Reviewer

---

## Analisando Description

### Primeiro passo: Visão geral da mudança

Antes de olhar o código, leia a description e entenda:
- Essa mudança faz sentido?
- Ela deveria estar acontecendo?

Se a mudança não deveria ter sido feita:
1. Responda imediatamente explicando o porquê
2. Sugira o que deveria ser feito em vez disso
3. Seja cortês

```
✅ "Parece que você colocou um bom trabalho nisso, obrigado! 
Porém, estamos na direção de remover o sistema FooWidget que você está 
modificando aqui, então não queremos fazer novas modificações nele agora. 
Que tal ao invés disso você refatorar nossa nova classe BarWidget?"
```

---

## Entendendo o problema

Antes de analisar a solução, você precisa entender:
- Qual problema está sendo resolvido?
- O problema é válido?
- A solução está no escopo correto?

**Se você recebe muitas CLs que representam mudanças que não deveriam ser feitas:**
- Repense o processo de desenvolvimento do time
- Melhor dizer "não" antes do trabalho ser feito

---

## Analisando solução

### Segundo passo: Examine as partes principais

1. Encontre o arquivo ou arquivos que são a parte "principal" da CL
2. Geralmente é o arquivo com o maior número de mudanças lógicas
3. Olhe essas partes principais primeiro
4. Isso dá contexto para as partes menores

**Se a CL é muito grande para identificar as partes principais:**
- Pergunte ao desenvolvedor o que olhar primeiro
- Ou peça para dividir a CL em CLs menores

### Se houver problemas de design na parte principal

Envie esses comentários **imediatamente**, mesmo que não tenha tempo de revisar o resto.

**Por quê?**
1. Desenvolvedores frequentemente já começam trabalho novo baseado na CL
2. Mudanças de design demoram mais que pequenas mudanças

---

## Análise técnica

### O que procurar em um code review

| Aspecto | Perguntas |
|---------|-----------|
| **Design** | O código está bem desenhado? É apropriado para o sistema? |
| **Funcionalidade** | O código faz o que o desenvolvedor pretendia? É bom para os usuários? |
| **Complexidade** | Poderia ser mais simples? Outro dev conseguiria entender facilmente? |
| **Testes** | Tem testes corretos e bem desenhados? |
| **Naming** | Os nomes são claros? |
| **Comentários** | Os comentários são claros e úteis? Explicam o "porquê"? |
| **Style** | Segue o style guide? |
| **Documentação** | A documentação relevante foi atualizada? |

> "Essa changelist deixou a codebase mais saudável?"

### Complexidade

"Muito complexo" significa:
- Não pode ser entendido rapidamente por quem lê
- Desenvolvedores vão introduzir bugs ao tentar modificar

**Over-engineering:** Desenvolvedores fizeram o código mais genérico do que precisa ser ou adicionaram funcionalidade que não é necessária agora.

> Resolva o problema que você **sabe** que precisa ser resolvido agora, não o problema que você **especula** que pode precisar resolver no futuro.

### Cada linha

No caso geral, olhe **cada linha** de código que você foi designado para revisar.

Se é muito difícil ler o código:
- Peça ao desenvolvedor para clarificar
- Se você não consegue entender, outros desenvolvedores provavelmente também não vão

---

## Análise de padrões

### Consistência

Se o código existente é inconsistente com o style guide:
- O style guide é autoridade absoluta
- A CL deve seguir o guide

Se o style guide faz recomendações (não exigências):
- Decida se o código novo deve ser consistente com as recomendações ou com o código ao redor
- Tendência: siga o style guide, a menos que a inconsistência local seja muito confusa

### Contexto

Olhe a CL no contexto amplo:
- A CL está melhorando a saúde do código do sistema?
- Ou está tornando o sistema mais complexo, menos testado?

> **Não aceite CLs que degradam a saúde do código do sistema.**

A maioria dos sistemas se torna complexa através de muitas pequenas mudanças que se acumulam.

---

## Comentando na PR

### Resumo

- Seja gentil
- Explique seu raciocínio
- Siga o Conventional Comments (vai deixar seu comentário bem mais claro)
- Balance entre dar direções explícitas e apenas apontar problemas
- Encoraje o desenvolvedor a simplificar o código ao invés de apenas explicar a complexidade para você

### Cortesia

Sempre faça comentários sobre o **código**, não sobre o **desenvolvedor**.

```
❌ RUIM: "Por que VOCÊ usou threads aqui quando obviamente não há benefício 
de concorrência?"

✅ BOM: "O modelo de concorrência aqui está adicionando complexidade ao sistema 
sem nenhum benefício de performance que eu consiga ver. Por não haver benefício 
de performance, seria melhor este código ser single-threaded."
```

### Explique o porquê

Ajude o desenvolvedor a entender **por que** você está fazendo o comentário.

### Rotule a severidade com Conventional Comments

Adotamos o padrão [Conventional Comments] para padronizar a comunicação nos reviews.
https://conventionalcomments.org/
https://github.com/px-center/px-docs/blob/main/docs/front-guild/decisions/023-tags-nos-comentarios-de-review.md

**Por que usar?**
- Deixa clara a intenção de cada comentário
- Reduz ambiguidade sobre prioridade e severidade
- Facilita triagem entre itens obrigatórios vs opcionais
- Promove ambiente de colaboração saudável

**Formato:** `<tag>: <comentário>`

#### Tags principais

| Tag | Uso | Bloqueante? |
|-----|-----|-------------|
| **praise:** | Destaca algo positivo. Seja genuíno, evite elogios artificiais. | Não |
| **nitpick:** | Pedidos triviais, baseados em preferência pessoal. | Não |
| **suggestion:** | Propõe melhorias específicas e explica por que são melhores. | Pode ser |
| **issue:** | Aponta um problema real ou potencial. Idealmente com sugestão de correção. | Sim |
| **todo:** | Mudanças pequenas, triviais, porém necessárias antes do merge. | Sim |
| **question:** | Dúvida ou possível preocupação, mas sem certeza de que há um problema. | Não |
| **thought:** | Ideias ou reflexões que surgiram durante a revisão. | Não |
| **chore:** | Tarefas simples necessárias para atender processos ou padrões internos. | Sim |
| **note:** | Apenas destaca algo que o autor deve notar. | Não |

#### Tags opcionais (mais expressivas)

| Tag | Uso |
|-----|-----|
| **typo:** | Correção de erros de digitação (variação de `todo`). |
| **polish:** | Similar a `suggestion`, mas focado em melhorar qualidade, mesmo sem erro. |
| **quibble:** | Semelhante a `nitpick`, mas sem a conotação negativa. |

#### Exemplos práticos

```
❌ RUIM: "Esse nome de variável está confuso"

✅ BOM: "suggestion: Renomear `x` para `userCount` deixaria o propósito 
mais claro para quem ler o código depois."

✅ BOM: "praise: Excelente uso de early return aqui, deixou o código 
muito mais legível!"

✅ BOM: "issue: Essa query pode causar N+1. Considere usar um join 
ou eager loading."

✅ BOM: "nitpick: Prefiro aspas simples, mas é preferência pessoal."

✅ BOM: "question: Esse timeout de 30s é suficiente para requests 
que envolvem upload de arquivo?"
```

Isso evita mal-entendidos onde o autor interpreta todos os comentários como obrigatórios.

### Dê guidance

**É responsabilidade do desenvolvedor corrigir a CL, não do reviewer.**

Balance entre:
- Apontar problemas e deixar o desenvolvedor decidir (ajuda a aprender)
- Dar instruções diretas quando for mais útil
- Nunca commitar em branch alheia sem conversar antes

> Lembre-se: pessoas aprendem com reforço do que estão fazendo bem, não só do que poderiam fazer melhor.

---

## Lidando com respostas e resolveds

### Quando o desenvolvedor discorda

1. Considere se eles estão corretos (eles estão mais próximos do código)
2. O argumento deles faz sentido do ponto de vista de saúde do código?
3. Se sim, deixe o assunto de lado
4. Se não, explique melhor por que você acredita que sua sugestão está correta

### "Vou limpar depois"

Desenvolvedores frequentemente dizem que vão limpar algo em uma CL futura.

**Realidade:** A experiência mostra que quanto mais tempo passa após a CL original, menos provável é que a limpeza aconteça.

> Geralmente é melhor insistir que o desenvolvedor limpe **agora**, antes do código estar "done".

### Reclamações sobre grau de rigidez

Se você mudou de reviews laxos para rigorosos:
- Alguns desenvolvedores vão reclamar alto
- Melhorar a **velocidade** dos reviews geralmente faz essas reclamações desaparecerem
- Pode levar meses, mas eventualmente os desenvolvedores veem o valor

---

## Request Change, Comment ou Approve

### O Standard de Code Review

> **Em geral, reviewers devem aprovar uma CL quando ela está em um estado onde definitivamente melhora a saúde geral do código do sistema, mesmo que a CL não seja perfeita.**

### Não existe código "perfeito"

Existe apenas código **melhor**.

- Não exija que o autor polir cada pequeno detalhe
- Balance a necessidade de fazer progresso com a importância das mudanças sugeridas
- Busque **melhoria contínua**, não perfeição

### LGTM with Comments

Dê LGTM mesmo com comentários não resolvidos quando:
- Você confia que o desenvolvedor vai endereçar os comentários apropriadamente
- Os comentários não **precisam** ser resolvidos pelo desenvolvedor
- As sugestões são menores (ordenar imports, typo, etc.)

**Especialmente útil** quando desenvolvedor e reviewer estão em fusos horários diferentes.

### Velocidade

**Um dia útil é o tempo máximo** para responder a um pedido de code review.

Se você está no meio de uma tarefa focada:
- Não se interrompa para fazer o review
- Espere um break point no seu trabalho

> A maioria das reclamações sobre o processo de code review são resolvidas tornando o processo **mais rápido**.

### Emergências

Em emergências (bug crítico em produção, problema de segurança, etc.):
- Velocidade e corretude importam mais que tudo
- Mas depois da emergência, revise a CL novamente com mais cuidado

**O que NÃO é emergência:**
- Querer lançar essa semana ao invés da próxima
- O desenvolvedor trabalhou muito tempo e quer fazer merge
- É sexta-feira à tarde e seria legal fazer merge antes do fim de semana
- Manager disse que precisa ser feito hoje (soft deadline)

---

# Pós-créditos

---

## IA na pipeline

A mesma clareza que ajuda reviewers humanos vai ajudar IAs a fazerem reviews melhores:

- **PRs bem descritas** = IA entende melhor o contexto
- **CLs pequenas e focadas** = IA consegue analisar com mais precisão

> Um bom entendimento do problema vai gerar uma boa explicação que vai fazer com que um codereview seja executado com mais eficiência. Tanto por um humano quanto mais para uma IA.

---

## Hora de fazer cagada

### Mentoria através do Code Review

Code review é uma oportunidade de **ensinar** algo novo sobre:
- Uma linguagem
- Um framework
- Princípios gerais de design de software

Se seu comentário é puramente educacional:
- Prefixe com "Nit:" ou indique que não é obrigatório resolver nessa CL
- Compartilhar conhecimento é parte de melhorar a saúde do código ao longo do tempo

### A melhoria é gradual

> "Melhorar a saúde do código tende a acontecer em pequenos passos."

Não espere que cada CL seja perfeita. Espere que cada CL deixe o código **um pouco melhor** do que estava antes.

---

# Resumo

## Para o Desenvolvedor

1. **Entenda o problema** claramente antes de codar
2. **Faça CLs pequenas** (< 400 linhas, uma coisa só)
3. **Escreva descriptions boas** (O QUÊ + POR QUÊ)
4. **Responda com educação** e colabore com o reviewer

## Para o Reviewer

1. **Leia a description** antes de olhar o código
2. **Entenda o problema** antes de analisar a solução
3. **Seja cortês** nos comentários (critique o código, não a pessoa)
4. **Rotule a severidade** (Nit, Optional, FYI)
5. **Responda rápido** (máximo 1 dia útil)
6. **Aprove quando melhorar** o código, não quando for perfeito

---

# Perguntas?

---

*Baseado no Google Engineering Practices Documentation*
*https://google.github.io/eng-practices/*

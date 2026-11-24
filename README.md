# Locadora de Equipamentos Industriais

Trabalho Prático em Dupla — Programação Orientada a Objetos
Professor: Prof. Mestre Marcos Roberto de Moraes

## Identificação da dupla

| ALUNO                  | 
|------------------------|
| Matheus K. de Carvalho |
| Murilo                 |

**Tema escolhido:** 10 — Locadora de Equipamentos Industriais
**Entidade principal:** `contratoLocacao` (ordenação natural por prazo final de devolução)
**Interface de negócio:** `seguravel` (método `calcularApoliceSeguro()`)
**Exceção checked customizada:** `itemIndisponivelException`

## Funcionalidades

- Abertura de locação (botão **Abrir Locação**)
- Devolução com vistoria técnica (botão **Devolver / Vistoria**)
- Prorrogação do prazo de devolução (botão **Prorrogar**)
- Ordenação dos contratos por prazo final de devolução (botão **Ordenar**)
- Listagem de contratos ativos com o valor das apólices de seguro (botão **Listar Ativos**)
- Saída com confirmação, tanto pelo botão **Sair** quanto pelo fechamento da janela (`WindowListener`)

Devolver e Prorrogar operam sobre o contrato cujo número é digitado no campo **Nº Contrato (devolver/prorrogar)**; o campo **Nº Contrato (próxima locação)** apenas exibe o número que será atribuído à próxima locação e não é editável.

Todos os dados ficam em memória (`List<equipamento>` e `List<contratoLocacao>`), sem banco de dados e sem arquivos.

## Arquitetura das classes

```
src/
├── model/
│   ├── equipamento.java               # entidade do catálogo (código, descrição, diária, disponibilidade)
│   └── contratoLocacao.java           # entidade principal; implements Comparable<contratoLocacao>, seguravel
├── interfaces/
│   └── seguravel.java                 # contrato com método abstrato, default e static
├── excecoes/
│   └── itemIndisponivelException.java # checked exception (extends Exception)
├── visao/
│   └── telaLocadora.java              # JFrame com Swing
└── principal/
    └── main.java                      # main(): cria telaLocadora na Event Dispatch Thread e chama setVisible(true)
```

**Pacote `model`**
- `equipamento`: campos privados, getters/setters, construtor e `toString()`.
- `contratoLocacao`: campos privados, getters, `setAtivo()`, construtor, `toString()`, `equals()` e `hashCode()` (ambos baseados em `numeroContrato`).
- `contratoLocacao.compareTo()` ordena pelo `prazoFinalDevolucao`, permitindo `Collections.sort(contratos)`.
- `contratoLocacao` oferece `getDiasContratados()` e `calcularValorTotal()`; a devolução libera o equipamento novamente.

**Pacote `interfaces`**
- `seguravel` declara `calcularApoliceSeguro()` e os métodos `default descricaoCobertura()` e `static aplicarFranquia()`.
- `contratoLocacao` implementa `calcularApoliceSeguro()` calculando 8% do valor total do contrato como prêmio de seguro, e herda `descricaoCobertura()` sem sobrescrevê-lo.

**Pacote `excecoes`**
- `itemIndisponivelException` é lançada quando o equipamento não existe ou já está locado, e é capturada na tela, que mostra o alerta com `JOptionPane`.
- A tela também captura `NumberFormatException` na conversão dos campos numéricos, `DateTimeParseException` para datas inválidas e `IllegalArgumentException` para a validação dos campos Cliente e Cód. Equipamento vazios e para as regras de negócio (datas incoerentes, contrato não encontrado).

**Pacote `visao`**
- `telaLocadora` estende `JFrame`, usa `JTextField`, `JButton`, `JTextArea` (relatório dentro de `JScrollPane`) e `JOptionPane` para diálogos de erro, validação e confirmação.
- O formulário tem seis campos (`GridLayout` 6x2): os cinco da especificação inicial mais **Nº Contrato (devolver/prorrogar)**. Formulário e botões ficam juntos no topo e o relatório ocupa o centro da janela.
- As datas são lidas com `DateTimeFormatter` em modo estrito (`ResolverStyle.STRICT`), de forma que datas inexistentes, como `31/02/2026`, geram `DateTimeParseException`. Na abertura da locação, as datas são validadas antes da consulta ao catálogo.
- Eventos de clique tratados via `ActionListener` na forma clássica (classe anônima); fechamento da janela via `WindowListener` (com `WindowAdapter`).

## Como executar

Requisito: JDK 21 ou superior.

```bash
# na raiz do repositório
mkdir -p out
javac -encoding UTF-8 -d out $(find src -name "*.java")
java -cp out principal.main
```

No Windows (PowerShell):

```powershell
mkdir out
javac -encoding UTF-8 -d out (Get-ChildItem -Recurse src -Filter *.java).FullName
java -cp out principal.main
```

Também é possível abrir a pasta `src` no IntelliJ IDEA e executar a classe `main`.

### Roteiro rápido de teste

1. Preencha **Cliente** = `Construtora Alfa`, **Cód. Equipamento** = `EQ01` e clique em **Abrir Locação**.
2. Repita com `EQ01` (e qualquer cliente) para ver o alerta de `itemIndisponivelException`.
3. Preencha **Cliente**, use `EQ02` (disponível) e digite uma data em formato errado, como `2026-11-19`, para ver o erro de `DateTimeParseException`.
4. Digite o número do contrato em **Nº Contrato (devolver/prorrogar)**, informe em **Prazo Final Devolução** uma data posterior ao prazo atual e clique em **Prorrogar**.
5. Clique em **Listar Ativos** e **Ordenar** para ver as apólices e a ordenação.
6. Digite o número do contrato em **Nº Contrato (devolver/prorrogar)** e clique em **Devolver / Vistoria** para encerrar o contrato e liberar o equipamento.

## Histórico de prompts de IA utilizados

A regra do trabalho exige transparência. Cada entrada abaixo registra o que foi solicitado à inteligência artificial, a resposta obtida e os ajustes realizados pela dupla durante o desenvolvimento do projeto.

Os prompts estão organizados em sequência, acompanhando as etapas de desenvolvimento do sistema e demonstrando como a IA foi utilizada como ferramenta de apoio.

### Prompt 1 — Geração da estrutura inicial do projeto

- **Ferramenta:** Claude
- **O que foi pedido:** O enunciado completo do trabalho (tema 10, Locadora de Equipamentos Industriais), solicitando a construção do sistema em Java com Swing, Collections, exceção checked, `Comparable` e interface polimórfica.
- **Resposta obtida:** Projeto dividido em pacotes `model`, `interfaces`, `excecoes`, `visao` e `principal`, com todas as classes descritas.
- **Refinamentos feitos pela dupla:** Revisamos as regras de negócio e definimos que o seguro corresponderia a 8% do valor total do contrato, ajustando a proposta inicial da IA para atender à regra estabelecida pela dupla.

### Prompt 2 — Construção da tela Swing

- **Ferramenta:** Claude
- **O que foi pedido:** "Preciso montar uma tela Swing em Java para uma locadora de equipamentos. A tela tem cinco campos de texto no topo (número do contrato, cliente, código do equipamento, data de retirada e data de devolução), seis botões no meio e um `JTextArea` grande embaixo para o relatório. Qual `LayoutManager` combina melhor com essa divisão e como faço o `JTextArea` rolar com `JScrollPane`?"
- **Resposta obtida:** A IA sugeriu `BorderLayout` como layout principal, com `GridLayout` no painel de formulário e `FlowLayout` no painel de botões, além de mostrar o `JScrollPane` envolvendo o `JTextArea` e o uso de `BorderFactory.createTitledBorder()` para separar visualmente as áreas.
- **Refinamentos feitos pela dupla:** Aplicamos a sugestão, mas ajustamos o `GridLayout` para 5 linhas e 2 colunas (rótulo + campo), porque a sugestão inicial deixava os campos espremidos. A dupla também definiu quais campos o formulário teria, o que a IA não tinha como saber. *Atualização posterior:* na versão final, o formulário passou a ter seis campos (`GridLayout` 6x2), com o acréscimo de **Nº Contrato (devolver/prorrogar)**, necessário para que Devolver e Prorrogar saibam qual contrato operar; o campo **Nº Contrato (próxima locação)** continua apenas exibindo o próximo número.

### Prompt 3 — Registro dos eventos de clique

- **Ferramenta:** Claude
- **O que foi pedido:** "Como registro `ActionListener` em vários botões diferentes de um `JFrame`, usando a forma clássica com classe anônima, sem lambda? Preciso que cada botão chame um método diferente da própria tela."
- **Resposta obtida:** A IA mostrou o padrão `btn.addActionListener(new ActionListener() { public void actionPerformed(ActionEvent e) { metodo(); } });` para cada botão, e explicou que a classe `telaLocadora` também pode implementar `ActionListener` diretamente.
- **Refinamentos feitos pela dupla:** Optamos por manter a forma clássica com classe anônima (e não lambda) para ficar alinhada ao que foi ensinado em POO_07. A dupla também decidiu qual método cada botão chamaria, o que a IA não tinha como saber.

### Prompt 4 — Fechamento da janela com confirmação

- **Ferramenta:** Claude
- **O que foi pedido:** "Quero que, ao clicar no X da janela, o sistema peça confirmação antes de fechar. Como uso `WindowListener` para isso, já que `setDefaultCloseOperation` fecha direto?"
- **Resposta obtida:** A IA mostrou o uso de `WindowAdapter` sobrescrevendo `windowClosing`, com `JOptionPane.showConfirmDialog()` e `System.exit(0)` apenas se o usuário confirmar.
- **Refinamentos feitos pela dupla:** Implementamos com `WindowAdapter` (classe adaptadora) em vez de implementar todos os sete métodos de `WindowListener`, e trocamos `EXIT_ON_CLOSE` por `DO_NOTHING_ON_CLOSE` na configuração do frame.

### Prompt 5 — Mensagens de erro na tela

- **Ferramenta:** Claude
- **O que foi pedido:** "Como exibo mensagens de erro e de validação para o usuário dentro de uma tela Swing, sem deixar o erro aparecer no console?"
- **Resposta obtida:** A IA mostrou `JOptionPane.showMessageDialog()` com os tipos `ERROR_MESSAGE`, `WARNING_MESSAGE` e `showConfirmDialog()` para confirmação.
- **Refinamentos feitos pela dupla:** Padronizamos o uso — `ERROR_MESSAGE` para falhas de conversão e exceções, `WARNING_MESSAGE` para validações de regra de negócio, e `showConfirmDialog()` apenas no botão Sair.

### Prompt 6 — Implementação de `Comparable`

- **Ferramenta:** Claude
- **O que foi pedido:** "Minha entidade `contratoLocacao` tem os campos `prazoFinalDevolucao` (LocalDate) e `numeroContrato` (int). Preciso implementar `Comparable<contratoLocacao>` para ordenar pelo prazo final de devolução. Como fica o `compareTo()` e como o `Collections.sort()` usa esse método?"
- **Resposta obtida:** A IA mostrou o `compareTo()` delegando para `this.prazoFinalDevolucao.compareTo(outro.prazoFinalDevolucao)` e explicou que `Collections.sort()` chama esse método internamente.
- **Refinamentos feitos pela dupla:** Aceitamos a implementação. Testamos com três contratos de datas diferentes para confirmar a ordem crescente e verificamos que a lista realmente era reordenada após a chamada.

### Prompt 7 — `equals()` e `hashCode()`

- **Ferramenta:** Claude
- **O que foi pedido:** "Se eu sobrescrever só o `equals()` na minha entidade e deixar o `hashCode()` padrão, o que acontece de errado ao usar a lista?"
- **Resposta obtida:** A IA explicou o contrato entre `equals()` e `hashCode()` e mostrou como gerar os dois métodos usando o `numeroContrato` como base.
- **Refinamentos feitos pela dupla:** Implementamos os dois métodos manualmente em `contratoLocacao`, usando `Integer.hashCode(numeroContrato)`, em vez de copiar uma versão gerada pela IDE, para garantir que entendíamos o que estava sendo comparado.

### Prompt 8 — Interface com métodos `default` e `static`

- **Ferramenta:** Claude
- **O que foi pedido:** "Preciso criar uma interface `seguravel` com um método abstrato `calcularApoliceSeguro()`, mas quero também um método que devolva a descrição padrão da cobertura sem obrigar toda classe a reescrever. Qual é a diferença entre `default` e `static` dentro de uma interface e quando usar cada um?"
- **Resposta obtida:** A IA explicou que `default` é herdado pelas implementações e pode ser sobrescrito, enquanto `static` pertence à interface e é chamado por ela mesma.
- **Refinamentos feitos pela dupla:** Usamos `default descricaoCobertura()` para o texto da cobertura e declaramos `static aplicarFranquia()` como cálculo auxiliar da interface (ainda não utilizado pela tela). A dupla optou por não sobrescrever o `default` em `contratoLocacao`, justamente para demonstrar o comportamento herdado.

### Prompt 9 — Checked Exception customizada

- **Ferramenta:** Claude
- **O que foi pedido:** "Quero criar uma exceção `itemIndisponivelException` que obrigue quem chama o método a tratar o erro. Ela deve herdar de `Exception` ou de `RuntimeException`? Qual é a diferença prática no `try/catch`?"
- **Resposta obtida:** A IA explicou que herdar de `Exception` gera uma checked exception, exigindo `try/catch` ou `throws`, e que `RuntimeException` é unchecked.
- **Refinamentos feitos pela dupla:** Optamos por herdar de `Exception`, como o enunciado exige. A mensagem da exceção identifica o equipamento: pelo código, quando ele não existe no catálogo, e pela descrição, quando está indisponível, para o usuário saber exatamente o que deu errado.

### Prompt 10 — Conversão de datas e números

- **Ferramenta:** Claude
- **O que foi pedido:** "O usuário digita valores em `JTextField` e preciso converter para `LocalDate` e para `int`. Quais exceções eu preciso capturar se ele digitar errado, e como tratar sem deixar o erro aparecer no console?"
- **Resposta obtida:** A IA mostrou `DateTimeFormatter.ofPattern("dd/MM/yyyy")` com `LocalDate.parse()` e `Integer.parseInt()`, indicando capturar `DateTimeParseException` e `NumberFormatException`, ambos com `JOptionPane.showMessageDialog()`.
- **Refinamentos feitos pela dupla:** Criamos um `DateTimeFormatter` reaproveitado em todos os métodos e adicionamos a validação dos campos Cliente e Cód. Equipamento vazios antes da conversão, para dar uma mensagem mais clara ao usuário. Datas e número de contrato vazios são tratados pelas exceções de conversão (`DateTimeParseException` e `NumberFormatException`).

### Prompt 11 — Revisão geral do código

- **Ferramenta:** Claude
- **O que foi pedido:** "Segue meu `contratoLocacao`. Ele implementa `Comparable` e `seguravel`, tem `equals()` e `hashCode()`. Pode apontar o que está fora das boas práticas de POO ou o que eu deixei de tratar?"
- **Resposta obtida:** A IA apontou que o cálculo do valor total poderia ser um método em vez de ficar dentro do `toString()`, e sugeriu encapsular a lógica de dias contratados em um método próprio.
- **Refinamentos feitos pela dupla:** Aceitamos a sugestão e criamos `getDiasContratados()` e `calcularValorTotal()` como métodos separados. O `toString()` passou a apenas formatar a saída, usando esses métodos.

### Prompt 12 — Organização deste README

- **Ferramenta:** Claude
- **O que foi pedido:** "Preciso escrever o README do meu trabalho de POO. Ele precisa conter: identificação da dupla, tema escolhido, explicação da arquitetura das classes, instruções de execução e histórico de prompts de IA. Como organizo essas seções de forma clara e na ordem que faz sentido para quem vai ler?"
- **Resposta obtida:** A IA sugeriu a ordem: identificação → tema → funcionalidades → arquitetura → como executar → histórico de prompts. Explicou que a arquitetura deve vir antes das instruções de execução porque o leitor precisa entender o que está sendo executado antes de rodar.
- **Refinamentos feitos pela dupla:** Adotamos a ordem sugerida. A dupla acrescentou a seção "Roteiro rápido de teste", que a IA não havia sugerido, e o diagrama de pastas em bloco de código.

### Prompt 13 — Resumo das respostas de IA para o README

- **Ferramenta:** Claude
- **O que foi pedido:** "Tenho várias perguntas que fiz à IA durante o desenvolvimento, com as respostas e os ajustes que fizemos. Preciso transformar isso em uma seção de README que explique o que pedimos, o que a IA respondeu e como refinamos, sem ficar longo demais. Como estruturo cada item?"
- **Resposta obtida:** A IA sugeriu um formato de três blocos por prompt: **O que foi pedido**, **Resposta obtida** e **Refinamentos feitos pela dupla**. Recomendou que o bloco de refinamento fosse o mais importante, porque mostra o que a dupla fez além do que a IA entregou.
- **Refinamentos feitos pela dupla:** Aplicamos o formato de três blocos em todos os prompts e acrescentamos o campo **Ferramenta**, para deixar registrado qual IA foi usada em cada caso.

### Prompt 14 — Significado de `private final`

- **Ferramenta:** Claude
- **O que foi pedido:** "O IntelliJ está marcando vários campos da minha classe com aviso amarelo sugerindo `private final`. O que significa o `final` em um atributo? Quando devo usar e quando não devo?"
- **Resposta obtida:** A IA explicou que `final` impede que o valor seja alterado depois da atribuição, e que `private final` significa um campo privado que só recebe valor uma única vez, normalmente no construtor. Mostrou também que o compilador recusa qualquer tentativa de reatribuição e que o IntelliJ sugere `final` justamente nos campos que não possuem setter.
- **Refinamentos feitos pela dupla:** Analisamos campo por campo do `contratoLocacao` e decidimos aplicar `final` apenas em `numeroContrato`, `nomeCliente`, `equipamento`, `dataRetirada` e `prazoFinalDevolucao`, mantendo `ativo` sem `final` porque ele muda na devolução com `setAtivo()`. A dupla também decidiu não adicionar setters para os campos marcados como `final`, para preservar a imutabilidade.

### Prompt 15 — Interpretação de avisos do IntelliJ

- **Ferramenta:** Claude
- **O que foi pedido:** "O IntelliJ mostra 5 erros vermelhos e 24 avisos amarelos no meu projeto. Quais eu preciso corrigir para o programa rodar e quais posso ignorar?"
- **Resposta obtida:** A IA explicou a diferença entre erro (vermelho) e aviso (amarelo): apenas os vermelhos impedem a compilação. Listou os avisos mais comuns no contexto do projeto — `Field can be final`, `Field name hides class name`, `Unused import` e `Method can be static` — e indicou que todos são sugestões de estilo.
- **Refinamentos feitos pela dupla:** Corrigimos os 5 erros vermelhos, que vinham de divergência entre o nome do arquivo e o nome da classe. Dos amarelos, resolvemos apenas os `Field can be final`; os demais, incluindo o `Field name hides class name` do campo `equipamento` em `contratoLocacao`, foram mantidos, porque não afetam a execução nem a legibilidade do código.

### Prompt 16 — Revisão do código e conferência do README

- **Ferramenta:** Claude
- **O que foi pedido:** Revisão do código das classes do projeto no IntelliJ e, em seguida, a conferência do README contra o código para apontar divergências.
- **Resposta obtida:** A IA apontou bugs no código (campo de número do contrato não editável impedindo Devolver e Prorrogar, layout do relatório, validação de datas) e oito divergências entre README e código: `equipamento` sem `implements seguravel`, `equipamento` sem `equals()`/`hashCode()`, roteiro de teste inválido, aviso do IntelliJ ainda presente, `aplicarFranquia()` sem uso, validação de campos vazios só em Cliente e `JOptionPane` sem diálogo de entrada. Em seguida entregou as classes `telaLocadora`, `contratoLocacao`, `equipamento` e `main` corrigidas: campo separado e editável para devolver/prorrogar, layout do relatório, datas em modo estrito, valores monetários e datas formatados no relatório, prorrogação mantendo a posição do contrato na lista e criação da janela na Event Dispatch Thread.

## Considerações finais

A IA foi usada como ferramenta de apoio em três frentes: aceleração da tela Swing (conforme a dica do enunciado), esclarecimento de dúvidas conceituais e apoio na redação deste README. Nenhuma classe foi entregue pronta pela ferramenta: cada sugestão foi testada, ajustada e, em vários casos, reescrita pela dupla para atender às regras específicas do enunciado.

O código final foi compilado e executado por ambos os integrantes, que validaram manualmente cada operação da tela antes da entrega.

## Referências dos materiais da disciplina

- **POO_00** — Ambiente: JDK 21, VS Code, IntelliJ IDEA.
- **POO_02** — Encapsulamento, getters/setters, construtores.
- **POO_04** — `List<T>`, associações, `package` e `import`.
- **POO_05** — Construtores, sobrecarga, `this`.
- **POO_07** — Swing: `JFrame`, `JTextField`, `JButton`, `JTextArea`, `JScrollPane`, `ActionListener`, `WindowListener`.
- **POO_08** — Exceções: checked vs unchecked, `try/catch`, exceção customizada.
- **POO_09** — `equals()`/`hashCode()`, `ArrayList`, Generics, `Comparable`, `Collections.sort()`.
- **POO_10** — Interfaces: `implements`, métodos `default` e `static`.
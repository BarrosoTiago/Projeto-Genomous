**Documento Lógico do Projeto: Genomous**  
*Este documento é um guia para desenvolvedores entender o funcionamento técnico e lógico do projeto.*

*Última Atualização: 01 de Setembro de 2025*

#### **I. A Visão Geral: O Painel de Controle Reativo e Assíncrono**

O projeto Genomous evoluiu de um sistema puramente reativo para um sistema que também gerencia **processos assíncronos**, ou seja, ações que não acontecem instantaneamente. A filosofia central permanece a mesma: a interface é um reflexo direto do estado do jogo. No entanto, o "estado" agora inclui não apenas dados persistentes (como `dinheiro` e `fósseis`), mas também **estados temporários** que descrevem processos em andamento (como `escavacaoEmAndamento`).

#### **II. A Estrutura e o Layout (O Chassi \- HTML & CSS)**

A arquitetura visual do projeto permanece sólida, com sua hierarquia de zonas (`header`, `container-jogo`, `footer`) servindo como uma base robusta. As recentes adições no CSS aprimoraram a imersão visual, mas a lógica estrutural fundamental não foi alterada.

A principal evolução na camada visual foi a introdução de **UI (Interface do Usuário) condicional e dinâmica**, que responde a mudanças de estado de maneiras mais complexas:

1. **Visibilidade Condicional (`{#if}`):** A interface agora decide qual botão mostrar (`Pesquisar` vs. `Incubar`) com base no estado de um item de dados (`fossil.progressoGenoma`). Isso torna a interface mais inteligente, apresentando ao jogador apenas as ações relevantes para o contexto atual.  
2. **Estado de Elementos (`disabled`):** O botão "Escavar" agora tem sua interatividade diretamente vinculada a uma variável de estado booleana (`escavacaoEmAndamento`). Isso demonstra um princípio chave de design de interface: fornecer feedback claro e prevenir ações inválidas do usuário.  
3. **Texto Dinâmico (`{#if}` em botões):** O texto do botão "Escavar" também muda para "Escavando...", provendo feedback textual imediato sobre o processo que foi iniciado.

#### **III. O Cérebro e o Sistema Nervoso (A Lógica Reativa \- JavaScript)**

A camada lógica tornou-se mais sofisticada. Além de reagir a mudanças de estado, ela agora também gerencia o **tempo** como um recurso e um mecanismo de jogo.

1. **As Variáveis de Estado (A Fonte da Verdade):** A lista de variáveis de estado foi expandida.  
   * **Estados Persistentes:** `dinheiro`, `fosseisInventario` e `dinosNoParque` continuam a ser o coração do progresso do jogador. A variável `rendaPorSegundo` agora é exibida na interface, dando ao jogador uma métrica clara de seu poder econômico.  
   * **Estado Temporário:** A nova variável `let escavacaoEmAndamento` representa um estado transitório. Sua função é gerenciar a "trava" de um processo, garantindo que a lógica e a interface se comportem de maneira diferente enquanto uma ação demorada está ocorrendo.  
2. **O Pulso do Jogo (O Motor de Renda):** A lógica do `onMount` com `setInterval` permanece inalterada, funcionando como o batimento cardíaco econômico do jogo, modificando continuamente o estado `dinheiro`.  
3. **Ações do Jogador (As Funções Transacionais):** As funções que o jogador pode invocar evoluíram significativamente.  
   * **`escavar()` (Uma Ação Assíncrona):** Esta função foi a que mais mudou. Ela não é mais uma transação instantânea. Sua nova lógica de fluxo é: a. **Início Imediato:** Valida as condições (dinheiro, se já está escavando), debita o custo e, crucialmente, **muda o estado temporário** (`escavacaoEmAndamento = true`). Esta mudança de estado é o que aciona a desativação e a mudança de texto do botão na interface. b. **Período de Espera (`setTimeout`):** A função `setTimeout` introduz o tempo como um elemento de jogabilidade. Ela agenda a execução da lógica de "resultado" para o futuro, criando um período de espera. c. **Conclusão Futura:** Após o tempo definido, o código dentro do `setTimeout` é executado, modificando o estado persistente (`fosseisInventario`) e, finalmente, **revertendo o estado temporário** (`escavacaoEmAndamento = false`) para destravar a interface e permitir que o jogador inicie uma nova ação.  
   * **`pesquisar()` e `incubar()` (Ações Instantâneas):** Estas funções continuam a operar como transações instantâneas. Elas seguem o padrão de validar, modificar o estado (`dinheiro`, `fosseisInventario`, `dinosNoParque`, `rendaPorSegundo`) e deixar que a reatividade do Svelte atualize a interface. A função `incubar`, em particular, demonstra um ciclo completo de transformação de dados: um item é removido de uma lista (`fosseisInventario.filter`) e um novo item correspondente é adicionado a outra (`...dinosNoParque`), com um impacto direto na economia do jogo (`rendaPorSegundo`).  
4. **O Reflexo Visual (Os Blocos `#each` e `{#if}`):** A interface continua a ser um espelho fiel do estado. O bloco `#each` desenha o inventário, enquanto o bloco `{#if}` aninhado dentro dele funciona como um "filtro" inteligente, decidindo qual ação (`Pesquisar` ou `Incubar`) é apropriada para cada fóssil individualmente, com base em seu `progressoGenoma`.


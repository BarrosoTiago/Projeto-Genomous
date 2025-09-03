<script>
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';

  let vistaAtual = 'pesquisa';

  let fosseisInventario = [];
  let dinosNoParque = [];
  let escavacaoEmAndamento = false;
  let alertaFossil = '';
  let proximoIdInstanciaDino = 1;
  let dinoSelecionado = null;
  let dinheiro = 5000; // conforme RF-MVP-01
  let rendaPorSegundo = 10000;
  const custoEscavacao = 250;
  const custoPesquisa = 100;
  const custoIncubacao = 1000;
  const custoAlimentacao = 50;
  const pontosAlimentacao = 1; // ganho ao alimentar
  const custoEstudo = 75;
  const pontosEstudo = 1; // ganho ao estudar

  onMount(() => {
    carregarJogo();

    const intervalo = setInterval(() => {
      dinheiro = dinheiro + rendaPorSegundo;
    }, 1000);

    return () => {
      clearInterval(intervalo)
    };
  });

  const tiposDeFosseis = [
    { nome: 'Âmbar com Inseto', cargaGenoma: 35, raridade: 'Lendário' },
    { nome: 'Crânio', cargaGenoma: 25, raridade: 'Muito Raro' },
    { nome: 'Pelve', cargaGenoma: 18, raridade: 'Incomum' },
    { nome: 'Fêmur', cargaGenoma: 15, raridade: 'Incomum' },
    { nome: 'Garra', cargaGenoma: 12, raridade: 'Comum' },
    { nome: 'Úmero', cargaGenoma: 9, raridade: 'Comum' },
    { nome: 'Vértebra', cargaGenoma: 8, raridade: 'Comum' },
    { nome: 'Costela', cargaGenoma: 6, raridade: 'Muito Comum' },
    { nome: 'Impressão de Pele', cargaGenoma: 5, raridade: 'Muito Comum' },
    { nome: 'Dente', cargaGenoma: 4, raridade: 'Muito Comum' },
  ];

  const dinosDoBrasil = [
    { 
      id: 1, 
      nome: 'Staurikosaurus pricei',
      rendaGerada: 80,
      custoManutencao: 30,
      periodo: 'Triássico Superior',
      alimentacao: 'Carnívoro',
      caracteristicas: 'Bípede, ágil, um dos primeiros dinossauros conhecidos.',
      tamanho: '2m de comprimento, 0.8m de altura',
      peso: '30 kg',
      tamanhoVsHumano: 'Cerca de metade da altura de um humano (0.47x)'
    },
    { 
      id: 2, 
      nome: 'Irritator challengeri',
      rendaGerada: 150,
      custoManutencao: 50,
      periodo: 'Cretáceo Inferior',
      alimentacao: 'Piscívoro/Carnívoro',
      caracteristicas: 'Crânio longo e estreito, similar ao de um crocodilo, com dentes cônicos.',
      tamanho: '8m de comprimento',
      peso: '1 tonelada',
      tamanhoVsHumano: 'Comprimento de quase 5 humanos enfileirados (4.7x)'
    },
    { 
      id: 3, 
      nome: 'Oxalaia quilombensis',
      rendaGerada: 280,
      custoManutencao: 80,
      periodo: 'Cretáceo Superior',
      alimentacao: 'Piscívoro/Carnívoro',
      caracteristicas: 'O maior espinossaurídeo do Brasil, provavelmente com uma vela nas costas.',
      tamanho: '14m de comprimento',
      peso: '5 a 7 toneladas',
      tamanhoVsHumano: 'Comprimento de mais de 8 humanos enfileirados (8.2x)'
    },
    { 
      id: 4, 
      nome: 'Santanaraptor placidus',
      rendaGerada: 70,
      custoManutencao: 25,
      periodo: 'Cretáceo Inferior',
      alimentacao: 'Carnívoro',
      caracteristicas: 'Predador pequeno e veloz, parente primitivo do T-Rex.',
      tamanho: '1.5m de comprimento',
      peso: '20 kg',
      tamanhoVsHumano: 'Ligeiramente menor em comprimento que um humano (0.88x)'
    },
    { 
      id: 5, 
      nome: 'Uberabatitan riberoi',
      rendaGerada: 300,
      custoManutencao: 120,
      periodo: 'Cretáceo Superior',
      alimentacao: 'Herbívoro',
      caracteristicas: 'Titanossauro de pescoço longo com corpo robusto e pernas como colunas.',
      tamanho: '15m de comprimento, 4m de altura',
      peso: '16 toneladas',
      tamanhoVsHumano: 'Altura de mais de 2 vezes um humano (2.35x)'
    },
    { 
      id: 6, 
      nome: 'Pycnonemosaurus nevesi',
      rendaGerada: 350,
      custoManutencao: 100,
      periodo: 'Cretáceo Superior',
      alimentacao: 'Carnívoro',
      caracteristicas: 'Grande predador com braços muito curtos e cabeça robusta.',
      tamanho: '9m de comprimento, altura de 3m',
      peso: '2 a 3 toneladas',
      tamanhoVsHumano: 'Quase 2 vezes a altura de um humano (1.76x)'
    },
  ];

  function escavar() {
    if (escavacaoEmAndamento) return; 
    if (dinheiro < custoEscavacao) {
      alert('Dinheiro insuficiente para escavar.');
      return; 
    }

    dinheiro -= custoEscavacao;
    escavacaoEmAndamento = true;

    console.log('Escavação iniciada...');

    setTimeout(() => {
      let mensagemAlerta = ''; 
      const dinoEncontrado = dinosDoBrasil[Math.floor(Math.random() * dinosDoBrasil.length)];
      const parteEncontrada = tiposDeFosseis[Math.floor(Math.random() * tiposDeFosseis.length)];
      const fossilExistente = fosseisInventario.find(f => f.dinoId === dinoEncontrado.id);

      if (fossilExistente) {
        fossilExistente.progressoGenoma += parteEncontrada.cargaGenoma;
        mensagemAlerta = `Um(a) ${parteEncontrada.nome} de ${fossilExistente.nome}! (+${parteEncontrada.cargaGenoma}%)`;

        if (fossilExistente.progressoGenoma > 100) {
          fossilExistente.progressoGenoma = 100;
        }
        
        console.log(`Encontrou um(a) ${parteEncontrada.nome} de ${fossilExistente.nome}! Progresso +${parteEncontrada.cargaGenoma}%.`);

      } else {
        const novoFossil = {
          dinoId: dinoEncontrado.id,
          nome: dinoEncontrado.nome,
          progressoGenoma: parteEncontrada.cargaGenoma,
          estado: 'coletando', 
        };
        fosseisInventario = [...fosseisInventario, novoFossil];
        mensagemAlerta = `NOVA ESPÉCIE! Um(a) ${parteEncontrada.nome} de ${novoFossil.nome} (+${parteEncontrada.cargaGenoma}%)`;
        console.log(`Nova espécie descoberta: ${dinoEncontrado.nome}! O primeiro achado foi um(a) ${parteEncontrada.nome} (+${parteEncontrada.cargaGenoma}%).`);
      }
      
      alertaFossil = mensagemAlerta;
      
      setTimeout(() => {
        alertaFossil = '';
      }, 5000); 

      fosseisInventario = [...fosseisInventario];
        
      escavacaoEmAndamento = false;
      console.log('Escavação concluída.');

    }, 100); 
  }


  function pesquisar(fossilAlvo) {
    
    if (fossilAlvo.estado === 'pesquisando') return; 
    if (dinheiro < custoPesquisa) {
      alert('Dinheiro insuficiente para iniciar a pesquisa.');
      return;
    }

    dinheiro -= custoPesquisa;
    fossilAlvo.estado = 'pesquisando';
    fosseisInventario = [...fosseisInventario]; 

    const tempoDePesquisa = 60000; 
    console.log(`Iniciando pesquisa de viabilidade para ${fossilAlvo.nome}. Duração: ${tempoDePesquisa / 1000}s.`);

    setTimeout(() => {
      const chanceDeSucesso = Math.random(); 

      if (chanceDeSucesso < 0.75) {
        // 75% de chance de sucesso
        fossilAlvo.estado = 'pronto_para_incubar';
        console.log(`Pesquisa de ${fossilAlvo.nome} concluída com sucesso!`);
        alert(`Pesquisa de ${fossilAlvo.nome} concluída com sucesso! A incubação agora está disponível.`);
      } else {
        // 25% de chance de falha
        fossilAlvo.estado = 'pesquisa_falhou';
        console.log(`A pesquisa de ${fossilAlvo.nome} falhou! Contaminação na amostra.`);
        alert(`A pesquisa de ${fossilAlvo.nome} falhou! Será necessário tentar novamente.`);
      }

      fosseisInventario = [...fosseisInventario];

    }, tempoDePesquisa);
  }

  function incubar(fossilPronto) {
    if (dinheiro < custoIncubacao) {
      alert('Dinheiro insuficiente para incubar!');
      return;
    }
    
    // 1. Encontra os dados completos da espécie no nosso catálogo
    const especieInfo = dinosDoBrasil.find(d => d.id === fossilPronto.dinoId);
    if (!especieInfo) return; // Segurança: Se não encontrar a espécie, cancela.

    dinheiro -= custoIncubacao;

    const novoDinossauro = {
      instanceId: proximoIdInstanciaDino++,
      id: fossilPronto.dinoId,
      nome: fossilPronto.nome,
      renda: especieInfo.rendaGerada,      
      custo: especieInfo.custoManutencao,  
      estagio: 'Filhote', 
      pontosEvolucao: 0,
      pontosParaProximoEstagio: 100 
    };
    
    dinosNoParque = [...dinosNoParque, novoDinossauro];
    fosseisInventario = fosseisInventario.filter(f => f.dinoId !== fossilPronto.dinoId);

    recalcularRendaTotal();

    alert(`Um ${novoDinossauro.nome} nasceu! A economia do seu parque foi atualizada.`);
  }

  function selecionarDino(dino) {
    dinoSelecionado = dino;
  }

  const evolucaoConfig = {
    'Filhote': { proximoEstagio: 'Jovem',  metaPontos: 100,  multiplicadorRenda: 1.5, multiplicadorCusto: 1.2 },
    'Jovem':   { proximoEstagio: 'Adulto', metaPontos: 250,  multiplicadorRenda: 2.0, multiplicadorCusto: 1.5 },
    'Adulto':  { proximoEstagio: 'Alpha',  metaPontos: 500,  multiplicadorRenda: 2.5, multiplicadorCusto: 2.0 },
    'Alpha':   null 
  };

  function alimentarDino(dinoAlvo) {
    if (dinheiro < custoAlimentacao) {
      alert('Dinheiro insuficiente para alimentar.');
      return;
    }
    
    dinheiro -= custoAlimentacao;
    dinoAlvo.pontosEvolucao += pontosAlimentacao;

    dinoSelecionado = dinoSelecionado; 
    dinosNoParque = [...dinosNoParque];

    verificarEvolucao(dinoAlvo);

  }

  function estudarDino(dinoAlvo) {
    if (dinheiro < custoEstudo) {
      alert('Dinheiro insuficiente para estudar o espécime.');
      return;
    }
    
    dinheiro -= custoEstudo;
    dinoAlvo.pontosEvolucao += pontosEstudo;

    dinoSelecionado = dinoSelecionado; 
    dinosNoParque = [...dinosNoParque];

    verificarEvolucao(dinoAlvo);
  }

  function verificarEvolucao(dinoAlvo) {
    if (dinoAlvo.pontosEvolucao >= dinoAlvo.pontosParaProximoEstagio) {
      // EVOLUIU!
      const configEstagioAtual = evolucaoConfig[dinoAlvo.estagio];
      
      if (configEstagioAtual && configEstagioAtual.proximoEstagio) {
        // Guarda o "excesso" de pontos
        const pontosExcedentes = dinoAlvo.pontosEvolucao - dinoAlvo.pontosParaProximoEstagio;

        // Atualiza o estágio
        dinoAlvo.estagio = configEstagioAtual.proximoEstagio;

        // Atualiza os atributos
        dinoAlvo.renda = Math.round(dinoAlvo.renda * configEstagioAtual.multiplicadorRenda);
        dinoAlvo.custo = Math.round(dinoAlvo.custo * configEstagioAtual.multiplicadorCusto);

        // Define a nova meta de pontos
        const configProximoEstagio = evolucaoConfig[dinoAlvo.estagio];
        if (configProximoEstagio) {
          dinoAlvo.pontosParaProximoEstagio = configProximoEstagio.metaPontos;
          dinoAlvo.pontosEvolucao = pontosExcedentes; // Começa a nova barra com os pontos excedentes
        } else {
          // Atingiu o estágio final (Alpha)
          dinoAlvo.pontosEvolucao = dinoAlvo.pontosParaProximoEstagio; // Preenche a barra
        }

        // Recalcula a renda total do parque com os novos valores
        recalcularRendaTotal();
        
        alert(`${dinoAlvo.nome} #${dinoAlvo.instanceId} evoluiu para o estágio ${dinoAlvo.estagio}!`);
      }
    }
  }

    $: infoCientificaDino = dinoSelecionado // <- reage a seleção do dino puxando as informações dele
    ? dinosDoBrasil.find(d => d.id === dinoSelecionado.id) 
    : null;

  function recalcularRendaTotal() {
    let novaRendaTotal = 0;
    
    for (const dino of dinosNoParque) {
      novaRendaTotal += dino.renda;
      novaRendaTotal -= dino.custo;
    }

    const rendaBaseDoParque = 30;
    rendaPorSegundo = rendaBaseDoParque + novaRendaTotal;
  }


  function salvarJogo() {
    const estadoDoJogo = {
      dinheiro: dinheiro,
      fosseisInventario: fosseisInventario,
      dinosNoParque: dinosNoParque,
      proximoIdInstanciaDino: proximoIdInstanciaDino,
      // Nota: não precisamos salvar rendaPorSegundo, pois ela é recalculada ao carregar.
    };

    try {
      const estadoEmString = JSON.stringify(estadoDoJogo);
      localStorage.setItem('genomousSave', estadoEmString);
      alert('Jogo Salvo com Sucesso!'); 
    } catch (error) {
      console.error('Erro ao salvar o jogo:', error);
      alert('Ocorreu um erro ao salvar o jogo.');
    }
  }

  function carregarJogo() {
    try {
      const estadoSalvoEmString = localStorage.getItem('genomousSave');

      if (estadoSalvoEmString) {
        const estadoSalvo = JSON.parse(estadoSalvoEmString);

        // É uma boa prática verificar se as chaves existem para evitar erros
        // caso o formato do save mude no futuro.
        dinheiro = estadoSalvo.dinheiro || 5000;
        fosseisInventario = estadoSalvo.fosseisInventario || [];
        dinosNoParque = estadoSalvo.dinosNoParque || [];
        proximoIdInstanciaDino = estadoSalvo.proximoIdInstanciaDino || 1;

        // Após carregar os dinos, é crucial recalcular a renda total
        recalcularRendaTotal();
        
        console.log('Jogo carregado com sucesso!');
      }
    } catch (error) {
      console.error('Erro ao carregar o jogo salvo:', error);
    }
  }
</script>

<main class="tela-principal">
  
  {#if alertaFossil}
    <div class="alerta-visual" transition:fade>
      <p><strong>Fóssil Encontrado!</strong></p>
      <p>{alertaFossil}</p>
    </div>
  {/if}

  <header class="cabecalho">
    <h1>GENOMOUS - Área de Pesquisa</h1>
    <button on:click={salvarJogo}>Salvar Jogo</button>
  </header>

<div class="container-jogo">

    {#if vistaAtual === 'pesquisa'}
      <div class="cabecalho-container">
        <h2>Sítios de Escavação:</h2>
        <button on:click={() => vistaAtual = 'especimes'} class="botao-navegacao" title="Ir para a Área dos Dinossauros"></button>
      </div>

      <section class="secao-escavacao">
        <nav class="abas-sitios">
          <button class="aba-sitio ativo">Brasil</button>
          <button class="aba-sitio">Argentina</button>
          <button class="aba-sitio">China</button>
          <button class="aba-sitio">EUA</button>
        </nav>

        <div class="info-sitio">
          <div class="imagem-sitio">
            <p>Brasil</p> 
            <button class="botao-escavar" on:click={escavar} disabled={escavacaoEmAndamento}>
              {#if escavacaoEmAndamento}
                Escavando...
              {:else}
                Escavar
              {/if}
            </button>
          </div>
          <div class="lista-dinossauros">
            <h3>Espécimes Disponíveis:</h3>
            <ul>
              {#each dinosDoBrasil as dino}
                <li>{dino.nome}</li>
              {/each}
            </ul>
          </div>
        </div>
      </section>

      <section class="secao-fosseis">
        {#each fosseisInventario as fossil}
          <div class="fossil">
            <span>Genoma de {fossil.nome} ({fossil.progressoGenoma}/100%)</span>
            <div class="progress-bar-track">
              <div class="progress-bar-fill" style="width: {fossil.progressoGenoma}%;"></div>
            </div>
            {#if fossil.progressoGenoma < 100}
              <p class="info-estado">Colete mais fósseis para completar o genoma.</p>
            {:else if fossil.estado === 'coletando' || fossil.estado === 'pesquisa_falhou'}
              {#if fossil.estado === 'pesquisa_falhou'}
                <p class="info-estado erro">Pesquisa anterior falhou!</p>
              {/if}
              <button on:click={() => pesquisar(fossil)}>
                Pesquisar (${custoPesquisa})
              </button>
            {:else if fossil.estado === 'pesquisando'}
              <p class="info-estado">Pesquisando viabilidade...</p>
            {:else if fossil.estado === 'pronto_para_incubar'}
              <button class="botao-incubar" on:click={() => incubar(fossil)}>
                Incubar (${custoIncubacao})
              </button>
            {/if}
          </div>
        {/each}
      </section>

    {:else if vistaAtual === 'especimes'}
      <div class="cabecalho-container">
        <h2>Área dos Dinossauros</h2>
        <button on:click={() => vistaAtual = 'pesquisa'} class="botao-navegacao botao-voltar" title="Voltar para a Área de Pesquisa"></button>
      </div>

      <div class="conteudo-especimes">
        <section class="area-detalhes">
          {#if dinoSelecionado && infoCientificaDino}
            <h3>#{dinoSelecionado.instanceId}: {dinoSelecionado.nome}</h3>
            
            <div class="info-dino"><strong>Período:</strong> <span>{infoCientificaDino.periodo}</span></div>
            <div class="info-dino"><strong>Alimentação:</strong> <span>{infoCientificaDino.alimentacao}</span></div>
            <div class="info-dino"><strong>Tamanho:</strong> <span>{infoCientificaDino.tamanho}</span></div>
            <div class="info-dino"><strong>Peso:</strong> <span>{infoCientificaDino.peso}</span></div>
            <div class="info-dino"><strong>Comparativo:</strong> <span>{infoCientificaDino.tamanhoVsHumano}</span></div>
            <div class="info-dino descricao"><strong>Características:</strong> <p>{infoCientificaDino.caracteristicas}</p></div>
            <div class="info-dino renda"><strong>Renda Gerada:</strong> <span>${dinoSelecionado.renda}/s</span></div>

            <div class="gerenciamento-instancia">
              <h4>Gerenciamento da Instância</h4>
              
              <div class="info-dino">
                <strong>Estágio:</strong>
                <span>{dinoSelecionado.estagio}</span>
              </div>

              <div class="info-dino">
                <strong>Evolução:</strong>
                <div class="progress-bar-track">
                  <div 
                    class="progress-bar-fill evolucao" 
                    style="width: { (dinoSelecionado.pontosEvolucao / dinoSelecionado.pontosParaProximoEstagio) * 100 }%;"
                  >
                  </div>
                </div>
              </div>
              <span>{dinoSelecionado.pontosEvolucao} / {dinoSelecionado.pontosParaProximoEstagio} P.E.</span>

                  {#if dinoSelecionado.estagio !== 'Alpha'}
                    <div class="acoes-dino">
                      <button on:click={() => alimentarDino(dinoSelecionado)}>
                        Alimentar (${custoAlimentacao})
                      </button>
                      <button on:click={() => estudarDino(dinoSelecionado)}>
                        Estudar (${custoEstudo})
                      </button>
                    </div>
                  {:else}
                    <p class="info-estado">Estágio Máximo Atingido!</p>
                  {/if}
            </div>
          {:else}
            <div class="detalhes-placeholder">
              <p>Selecione um dinossauro na lista para ver os detalhes.</p>
            </div>
          {/if}
        </section>
        <section class="lista-mestre">
          <h3>Dinossauros no Parque ({dinosNoParque.length})</h3>
          <ul>
            {#each dinosNoParque as dino (dino.instanceId)}
              <li on:click={() => selecionarDino(dino)} class:ativo={dino === dinoSelecionado}>
                <span>#{dino.instanceId} - {dino.nome}</span>
              </li>
            {:else}
              <li>Nenhum dinossauro no parque.</li>
            {/each}
          </ul>
        </section>
      </div>
    {/if}
    </div>

  <footer class="rodape">
    <span class="info-dinheiro">{dinheiro} 💰 (+{rendaPorSegundo}/s)</span>
    <span class="info-data">22/08/2025</span>
  </footer>

</main>

<style>
  /* ESTILOS GLOBAIS E ATMOSFERA */
  :global(body) {
    background-image: url(../lib/assets/images/body-background-image.png);
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
    margin: 0;
    background-color: #1a1a1a; 
  }

  .tela-principal {
    color: #f0f0f0;
    font-family: 'Courier New', Courier, monospace;
    min-height: 100vh;
    padding: 1rem;
    box-sizing: border-box; 
    display: flex;
    flex-direction: column;
    gap: 1rem; 
  }

  /* LAYOUT DAS ZONAS PRINCIPAIS (HEADER, FOOTER) */
  .cabecalho, .rodape {
    display: flex;
    justify-content: space-between; 
    align-items: center;
    padding: 0.5rem 1rem;
    background-color: rgba(0, 0, 0, 0.429); 
    border: 1px solid #8b9a3b;
  }

  h1 {
    font-size: 1.2rem;
    color: #c7d187;
    text-transform: uppercase;
  }

  /* ARENA DE JOGO */
  .container-jogo {
    background-image: url(../lib/assets/images/textura-a-terra-seca-e-rachada-freepik.png);
    flex-grow: 1;
    margin: 2rem 10rem; 
    border: 2px solid #8b9a3b;
    box-shadow: 0 0 0 30px #000000; 
    padding: 1rem;
    display: flex;
    flex-direction: column;
    gap: 1rem; 
  }

  .cabecalho-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .botao-navegacao {
    background-image: url(../lib/assets/images/botao-especimes.png);
    background-size: cover;
    background-position: center;
    border: none;
    background-color: transparent;
    width: 5px;
    height: 50px;
  }

  /* SEÇÃO DE ESCAVAÇÃO E SUAS SUBDIVISÕES */
  .secao-escavacao {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .abas-sitios {
    display: flex;
    gap: 0.5rem;
  }

  .info-sitio {
    display: flex;
    gap: 1rem;
    background-color: rgba(0, 0, 0, 0.2);
    padding: 1rem;
  }

  .imagem-sitio {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    background-image: url(../lib/assets/images/brasil-pexels@florencia-potter.png);
    background-size: cover;
    background-position: center;
    width: 20%;
    color: #2e4b26;
  }


  .lista-dinossauros ul {
    list-style-type: none;
    padding: 0;
    margin: 0.5rem 0 0 0;
    display: grid;
    grid-template-columns: repeat(2, 1fr);  /* Define duas colunas de largura igual */
    gap: 0.5rem;
  }

  .lista-dinossauros li {
    background-color: rgba(0, 0, 0, 0.2);
    padding: 0.5rem;
    border: 1px solid #8b9a3b;
  }


  /* SEÇÃO DE FÓSSEIS (LAYOUT HORIZONTAL) */
  .secao-fosseis {
    display: flex;
    gap: 1rem;
    padding: 1rem;
    background-color: rgba(0, 0, 0, 0.2);
    border-top: 1px solid #8b9a3b;
    overflow-x: auto; 
  }

  .fossil {
    border: 1px solid #8b9a3b;
    padding: 0.5rem;
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    min-width: 150px;
  }

  .progress-bar-track {
    width: 100%;
    height: 10px;
    background-color: rgba(0, 0, 0, 0.5);
    border: 1px solid #8b9a3b;
    border-radius: 5px;
    overflow: hidden; 
  }

  .progress-bar-fill {
    height: 100%;
    background-color: #8b9a3b; 
    transition: width 0.8s ease-in-out; 
}

  /* ESTILO DOS ELEMENTOS INTERATIVOS */
  button {
    background-color: #3c5d33;
    color: #f0f0f0;
    border: 1px solid #8b9a3b;
    padding: 0.5rem 1rem;
    font-family: inherit; 
    cursor: pointer;
  }

  button:hover {
    background-color: #4a7340;
  }

  .aba-sitio.ativo {
    background-color: #8b9a3b;
    color: #1a1a1a;
    font-weight: bold;
  }

  .alerta-visual {
    position: fixed; 
    top: 20px;
    right: 20px;
    background-color: #c7d187; 
    color: #1a1a1a;
    padding: 1rem;
    border: 2px solid #2e4b26;
    border-radius: 5px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);
    z-index: 1000; 
    max-width: 300px;
  }
  .alerta-visual p {
    margin: 0;
  }

    /* ESTILOS PARA A ÁREA DOS ESPÉCIMES */
  .conteudo-especimes {
    display: flex;
    gap: 1rem;
    flex-grow: 1; 
    min-height: 0;
  }

  .area-padoques, .catalogo-especimes {
    background-color: rgba(0, 0, 0, 0.2);
    padding: 1rem;
    border: 1px solid #8b9a3b;
  }

  .area-padoques {
    flex-basis: 60%; 
  }

  .catalogo-especimes {
    flex-basis: 40%; 
  }

  /* ESTILOS PARA O LAYOUT MASTER-DETAIL */
  .area-detalhes {
    flex-basis: 60%;
    background-color: rgba(0, 0, 0, 0.2);
    padding: 1rem;
    border: 1px solid #8b9a3b;
  }

  .lista-mestre {
    display: flex;
    flex-direction: column;
    flex-basis: 40%;
    background-color: rgba(0, 0, 0, 0.2);
    padding: 1rem;
    border: 1px solid #8b9a3b;
    overflow-y: auto; 
    
  }

  .lista-mestre ul {
    list-style-type: none;
    padding: 0;
    margin: 0.5rem 0 0 0;
  }

  .lista-mestre li {
    padding: 0.8rem 0.5rem;
    border-bottom: 1px solid #3c5d33;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  .lista-mestre li:hover {
    background-color: rgba(139, 154, 59, 0.2);
  }

  .lista-mestre li.ativo {
    background-color: #8b9a3b;
    color: #1a1a1a;
    font-weight: bold;
  }

  .detalhes-placeholder {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    color: rgba(255, 255, 255, 0.5);
  }

  .info-dino {
    margin-top: 1rem;
    display: flex;
    justify-content: space-between;
    border-bottom: 1px dashed #8b9a3b;
    padding-bottom: 0.5rem;
  }

  .catalogo-especimes ul {
    list-style-type: none;
    padding: 0;
    margin: 0.5rem 0 0 0;
  }

  .catalogo-especimes li {
    padding: 0.5rem;
    border-bottom: 1px solid #3c5d33;
  }

  .botao-voltar {
    width: 50px;
    height: 50px;
    background-image: url(../lib/assets/images/frasco.png);
  }

  /* ESTILOS PARA A SEÇÃO DE GERENCIAMENTO DE INSTÂNCIA */
  .gerenciamento-instancia {
    margin-top: 1.5rem;
    padding-top: 1rem;
    border-top: 2px solid #3c5d33;
  }

  .gerenciamento-instancia h4 {
    margin: 0 0 1rem 0;
    text-transform: uppercase;
    color: #c7d187;
  }

  .progress-bar-fill.evolucao {
    background-color: #5a9bd6; 
  }

  .acoes-dino {
    margin-top: 1rem;
    display: flex;
    gap: 1rem;
  }

  .info-dino.descricao {
    border-bottom: none;
  }
</style>
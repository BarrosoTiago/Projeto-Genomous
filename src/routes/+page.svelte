<script>
  import { onMount } from 'svelte';
  import { fade } from 'svelte/transition';

  let fosseisInventario = [];
  let dinosNoParque = [];
  let escavacaoEmAndamento = false;
  let alertaFossil = '';
  let dinheiro = 5000; // conforme RF-MVP-01
  let rendaPorSegundo = 30;
  const custoEscavacao = 250;
  const custoPesquisa = 100;
  const custoIncubacao = 1000;

  onMount(() => {
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
    { id: 1, nome: 'Staurikosaurus pricei' },
    { id: 2, nome: 'Irritator challengeri'},
    { id: 3, nome: 'Oxalaia quilombensis'},
    { id: 4, nome: 'Santanaraptor placidus'},
    { id: 5, nome: 'Uberabatitan Riberoi'},
    { id: 6, nome: 'Pycnonemosaurus nevesi'},
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

    }, 10000); 
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
    if (dinheiro >= custoIncubacao) {

      dinheiro -= custoIncubacao;

      const novoDinossauro = {
        id: fossilPronto.dinoId,
        nome: fossilPronto.nome,
        renda: 250, 
      };
      
      dinosNoParque = [...dinosNoParque, novoDinossauro];

      fosseisInventario = fosseisInventario.filter(f => f.dinoId !== fossilPronto.dinoId);

      rendaPorSegundo += novoDinossauro.renda;

      alert(`Um ${novoDinossauro.nome} nasceu! Sua renda por segundo aumentou!`);
    } else {
      alert('Dinheiro insuficiente para incubar!');
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
    <button>Salvar Jogo</button>
  </header>

  <div class="container-jogo">
    <div class="cabecalho-container">
      <h2>Sítios de Escavação:</h2>
      <button class="botao-navegacao" title="Ir para a Área dos Dinossauros"></button>
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
</style>
<script>
  import { onMount } from 'svelte';

  let fosseisInventario = [];
  let dinheiro = 5000; // conforme RF-MVP-01
  let rendaPorSegundo = 50;
  const custoEscavacao = 250;

  onMount(() => {
    const intervalo = setInterval(() => {
      dinheiro = dinheiro + rendaPorSegundo;
    }, 1000);

    return () => {
      clearInterval(intervalo)
    };
  });

  const dinosDoBrasil = [
    { id: 1, nome: 'Staurikosaurus pricei' },
    { id: 2, nome: 'Irritator challengeri'},
  ];


  function escavar() {
    if (dinheiro >= custoEscavacao) {
      dinheiro -= custoEscavacao;
      const dinoEncontrado = dinosDoBrasil[Math.floor(Math.random() * dinosDoBrasil.length)];
      const fossilExistente = fosseisInventario.find(f => f.dinoId === dinoEncontrado.id);


      if (fossilExistente) {
        console.log(`Encontrou outro fóssil de ${dinoEncontrado}`)

      } else {
        const novoFossil = {
          dinoId: dinoEncontrado.id,
          nome: dinoEncontrado.nome,
          progressoGenoma: 1,
        };

        fosseisInventario = [...fosseisInventario, novoFossil];
        console.log(`Novo fóssil encontrado: ${novoFossil.nome}!`);
      }

      console.log('Escavação bem-sucedida! Novo saldo:', dinheiro);

    } else {
      alert('Dinheiro insuficiente para escavar.');
    }

  }
</script>

<main class="tela-principal">
  
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
          <button class="botao-escavar" on:click={escavar}>Escavar</button>
        </div>
        <div class="lista-dinossauros">
          <h3>Espécimes Disponíveis:</h3>
          <ul>
            <li>Staurikosaurus pricei</li>
            <li>Irritator challengeri</li>
            </ul>
        </div>
      </div>
    </section>

    <section class="secao-fosseis">
      {#each fosseisInventario as fossil}
        <div class="fossil">
          <span>{fossil.nome} ({fossil.progressoGenoma}/100%)</span>
          </div>
      {/each}
    </section>
  </div>

  <footer class="rodape">
    <span class="info-dinheiro">{dinheiro} 💰</span>
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
    background-color: rgba(0, 0, 0, 0.2); 
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
    margin: 0;
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
</style>
<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';

  let data = [];
  let isLoading = true;

  // PRNG determinista (mulberry32) para que el barajado sea reproducible a partir de una semilla.
  function mulberry32(seed) {
    return function () {
      seed |= 0;
      seed = (seed + 0x6d2b79f5) | 0;
      let t = Math.imul(seed ^ (seed >>> 15), 1 | seed);
      t = (t + Math.imul(t ^ (t >>> 7), 61 | t)) ^ t;
      return ((t ^ (t >>> 14)) >>> 0) / 4294967296;
    };
  }

  // La semilla vive en sessionStorage: misma pestaña y recargas (F5) reutilizan el mismo
  // orden; una pestaña/ventana nueva obtiene sessionStorage vacío y puede sortear otra semilla.
  function getSessionSeed() {
    const key = 'gridShuffleSeed';
    let seed = sessionStorage.getItem(key);
    if (seed === null) {
      seed = String(Math.floor(Math.random() * 2 ** 31));
      sessionStorage.setItem(key, seed);
    }
    return Number(seed);
  }

  function shuffleArray(array, random) {
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
  }

  async function loadAndShuffleData() {
    const url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vQZsnsur1Nec98cL4ujG9kBFmXU0TyvOtkd7TR61LsqhPuu4r7CUU-lZ9BF__cmYNHNA8rCcSsafvgq/pub?output=csv";
    try {
      const fetchedData = await csv(url);
      shuffleArray(fetchedData, mulberry32(getSessionSeed()));
      data = fetchedData;
    } catch (error) {
      console.error("Error fetching data:", error);
    } finally {
      isLoading = false;
    }
  }

  onMount(() => {
    loadAndShuffleData();
  });
</script>

<style>
  /* Contenedor propio de la portada (no usa la clase compartida `.container`: sus reglas
     son deliberadamente distintas — sin padding ni max-width — y no debe heredar las de
     las páginas interiores). */
  .home-grid {
    font-family: "Montserrat", sans-serif;
    background-color: #e6ebf1;
    text-align: justify;
    max-width: auto;
  }

  .grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    grid-auto-rows: 250px;
    gap: 20px;
  }

  .card {
    overflow: hidden;
    display: flex;
    justify-content: center;
    align-items: center;
  }

  .card-image img {
    max-width: 100%;
    height: auto;
    transition: all 0.5s ease;
    border: none;
    filter: grayscale(100%) brightness(80%) contrast(120%);
  }

  .card-image img:hover {
    box-shadow: 0 0 32px #333;
    transform: scale(1.05);
    filter: none;
  }

  @media screen and (max-width: 768px) {
    .grid {
      grid-template-columns: 1fr; /* Una sola columna para pantallas pequeñas */
      grid-auto-rows: auto;
    }
  }
</style>

<div class="home-grid">
  {#if isLoading}
    <p></p>
  {:else}
    <div class="grid">
      {#each data as item (item.indice)}
        <div class="card">
          {#if item.link}
            <a href={item.link} target="_blank" rel="noopener noreferrer">
              <div class="card-image">
                <img src={item.imagen} alt={`Publicación ${item.indice}`} loading="lazy" />
              </div>
            </a>
          {:else}
            <div class="card-image">
              <img src={item.imagen} alt={`Publicación ${item.indice}`} loading="lazy" />
            </div>
          {/if}
        </div>
      {/each}
    </div>
  {/if}
</div>

<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import '$lib/styles/structural.css';

  let data = [];
  let isLoading = true;

  // Copia local primero (si existe) para que la página se vea de inmediato al
  // navegar, y red siempre en paralelo para refrescarla — nunca al revés, y
  // nunca vaciando `data` antes de tener algo nuevo que mostrar.
  const CACHE_KEY = 'experienceData';

  async function consultarAPI() {
    const cached = localStorage.getItem(CACHE_KEY);
    if (cached) {
      data = JSON.parse(cached);
      isLoading = false;
    }

    const url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vQlN-YyegviJd-QDECGZZ3YJqw4edEV11bZYppkOuLxd63z7uca-VxI5psQQG_4TZdUtGSJx_JXy11G/pub?output=csv";
    try {
      const fetchedData = await csv(url);
      data = fetchedData;
      localStorage.setItem(CACHE_KEY, JSON.stringify(data));
    } catch (error) {
      console.error("Error fetching data:", error);
    } finally {
      isLoading = false;
    }
  }

  onMount(() => {
    consultarAPI();
  });
</script>

<div class="container">
  <h1>Experiencia Profesional</h1>

  {#if isLoading}
    <p></p>
  {:else}
    {#each data as item}
      <div class="card">
        <div class="card-content">
          <h2>{item.Cargo}</h2>
          <p class="meta">{item.Institución}</p>
          <p class="meta">{item.Periodo}</p>
          <p class="description">{item.Descripción}</p>
        </div>
      </div>
    {/each}
  {/if}
</div>

<style>
  .card {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    padding-block: var(--space-6);
  }

  .card + .card {
    border-top: var(--line-thin) solid var(--color-line);
  }

  .card-content {
    display: flex;
    flex-direction: column;
    gap: var(--space-2);
    max-width: var(--measure);
  }

  h1 {
    display: inline-block;
    margin: 0 0 var(--space-6);
    padding-bottom: var(--space-2);
    font-size: var(--fs-h1);
    font-weight: 600;
    line-height: var(--lh-tight);
    color: var(--color-ink);
    font-family: var(--font-sans);
    border-bottom: var(--line-thick) solid var(--color-accent);
  }

  h2 {
    margin: 0;
    font-family: var(--font-sans);
    font-weight: 600;
    font-size: var(--fs-h2);
    color: var(--color-ink);
  }

  .meta {
    margin: 0;
    font-family: var(--font-mono);
    font-size: var(--fs-meta);
    letter-spacing: 0.02em;
    color: var(--color-ink);
  }

  .description {
    margin: var(--space-2) 0 0;
    font-family: var(--font-sans);
    font-weight: 400;
    font-size: var(--fs-body);
    line-height: var(--lh-body);
  }

  @media (max-width: 768px) {
    .card {
      padding-block: var(--space-5);
    }
  }
</style>

<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import Icon from '@iconify/svelte';
  import '$lib/styles/structural.css';

  let data = [];
  let isLoading = true;

  // Copia local primero (si existe) para que la página se vea de inmediato al
  // navegar, y red siempre en paralelo para refrescarla — nunca al revés, y
  // nunca vaciando `data` antes de tener algo nuevo que mostrar.
  const CACHE_KEY = 'chaptersData';

  async function consultarAPI() {
    const cached = localStorage.getItem(CACHE_KEY);
    if (cached) {
      data = JSON.parse(cached);
      isLoading = false;
    }

    let url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vRgQUPIxtoE4v98W3bfYimpMXZwVSWPAomSC04M1HMK3fpU_LsHesfRIynehNKOc4sNlTuzRGUX0-6o/pub?output=csv";
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
  <a href="/publicaciones" class="back-link" aria-label="Volver a Publicaciones"><Icon icon="material-symbols:arrow-back" /></a>
  <h1>Capítulos de Libros</h1>

  {#each data as item}
    <div class="list-card">
      <div class="list-card-media">
        <a href={item.DOI} target='_blank' rel="noopener noreferrer"><img src={item.Imagen} alt={item.Título} loading="lazy" /></a>
      </div>
      <div class="list-card-body">
        <h2>{item.Título}</h2>
        <h3>{item.Título_libro}</h3>
        <p class="meta">Editor: {item.Editores}</p>
        <p class="meta">{item.Ciudad}</p>
        <p class="meta">{item.Editorial}</p>
        <p class="meta">{item.Año}</p>
        <p class="meta">pp: {item.Páginas}</p>
        <p class="meta">ISBN: {item.ISBN}</p>
        <p class='autor'>{item.Autor}</p>
        <!-- <p class="description">{item.Descripcion}</p> -->
      </div>
    </div>
  {/each}
</div>

<style>
  .back-link {
    display: inline-flex;
    color: var(--color-ink);
    font-size: 1.75rem;
    margin-bottom: var(--space-5);
    padding: var(--space-2);
    margin-left: calc(var(--space-2) * -1);
  }

  .back-link:hover,
  .back-link:focus-visible {
    color: var(--color-accent);
  }

  .list-card-media img {
    width: 100%;
    height: auto;
  }

  @media (hover: hover) and (pointer: fine) {
    .list-card-media img {
      transition: filter 0.4s ease;
      filter: grayscale(100%) brightness(90%);
    }

    .list-card-media a:hover img,
    .list-card-media a:focus-visible img {
      filter: none;
    }
  }

  h1 {
    display: inline-block;
    margin: var(--space-4) 0 var(--space-6);
    padding-bottom: var(--space-2);
    font-size: var(--fs-h1);
    font-weight: 600;
    line-height: var(--lh-tight);
    color: var(--color-ink);
    font-family: var(--font-sans);
    border-bottom: var(--line-thick) solid var(--color-accent);
  }

  h2 {
    margin: 0 0 var(--space-1);
    font-family: var(--font-sans);
    font-weight: 600;
    font-size: var(--fs-h2);
    color: var(--color-ink);
  }

  /* Título del libro contenedor: mismo peso de identificación que el resto
     (h2 = obra catalogada), pero un escalón más ligero/pequeño y con
     tracking, para leerse como referencia subordinada sin recurrir a
     cursiva/serifa (que ya no forman parte del sistema). */
  h3 {
    margin: 0 0 var(--space-2);
    font-family: var(--font-sans);
    font-style: normal;
    font-weight: 400;
    font-size: var(--fs-h3);
    letter-spacing: 0.02em;
    color: var(--color-ink);
  }

  .meta {
    margin: 0;
    font-family: var(--font-mono);
    font-size: var(--fs-meta);
    letter-spacing: 0.02em;
  }

  .autor {
    margin-top: var(--space-2);
    padding-top: var(--space-2);
    border-top: var(--line-thin) solid var(--color-line);
    font-family: var(--font-mono);
    font-weight: 500;
    font-size: var(--fs-meta);
    letter-spacing: 0.02em;
    color: var(--color-red-ink);
  }

  a {
    color: var(--color-orange-ink);
    text-decoration: none;
  }

  a:hover,
  a:focus-visible {
    text-decoration: underline;
    text-underline-offset: 3px;
  }
</style>

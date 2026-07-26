<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import '$lib/styles/structural.css';

  let isLoading = true;
  let raw = { articulos: [], libros: [], capitulos: [] };

  // Mismas hojas, URLs y claves de caché que ya usan /publicaciones/articulos,
  // /libros y /capitulos — no se crea una fuente de datos nueva, solo se
  // reutilizan las tres existentes para construir un índice combinado. El
  // campo "contenedor" es la publicación que aloja cada registro (revista,
  // editorial o el libro del que forma parte un capítulo).
  const SOURCES = [
    {
      key: 'articulos',
      cacheKey: 'articlesData',
      url: 'https://docs.google.com/spreadsheets/d/e/2PACX-1vThq4fhXW3GP7MsfMBrvjUgGN0SV-yQztsRwtWq5kCDpGVJdRAskzi0Mq2GYJuFsqIV4eHp9bfNoqVR/pub?gid=2145965265&single=true&output=csv',
      type: 'Artículo',
      titleField: 'Título',
      yearField: 'Año',
      linkField: 'DOI',
      containerField: 'Revista',
    },
    {
      key: 'libros',
      cacheKey: 'booksData',
      url: 'https://docs.google.com/spreadsheets/d/e/2PACX-1vSkGYClXLB0CD0KA30_fGfmmfVqnClIEnvsPh6AD8k_QPdkdlGr3bl3fikK3inXli40qRZng4qqp12n/pub?output=csv',
      type: 'Libro',
      titleField: 'Título',
      yearField: 'Año',
      linkField: 'DOI',
      containerField: 'Editorial',
    },
    {
      key: 'capitulos',
      cacheKey: 'chaptersData',
      url: 'https://docs.google.com/spreadsheets/d/e/2PACX-1vRgQUPIxtoE4v98W3bfYimpMXZwVSWPAomSC04M1HMK3fpU_LsHesfRIynehNKOc4sNlTuzRGUX0-6o/pub?output=csv',
      type: 'Capítulo',
      titleField: 'Título',
      yearField: 'Año',
      linkField: 'DOI',
      containerField: 'Título_libro',
    },
  ];

  // Copia local primero (si existe) para que la lista se vea de inmediato al
  // navegar, y red siempre en paralelo para refrescarla — mismo patrón de
  // caché que el resto del sitio, sin cambiarlo.
  async function loadSource(source) {
    const cached = localStorage.getItem(source.cacheKey);
    if (cached) {
      try {
        raw = { ...raw, [source.key]: JSON.parse(cached) };
      } catch (error) {
        console.error('Error leyendo caché:', source.key, error);
      }
    }
    try {
      const records = await csv(source.url);
      raw = { ...raw, [source.key]: records };
      localStorage.setItem(source.cacheKey, JSON.stringify(records));
    } catch (error) {
      console.error('Error fetching:', source.url, error);
    }
  }

  // Representación común a las tres fuentes. Sin título no hay fila útil; el
  // resto de campos se muestra solo cuando el dato existe de verdad — nunca
  // se homogeneiza inventando un valor.
  function toEntries(records, source) {
    return records
      .map((r) => ({
        title: r[source.titleField],
        year: r[source.yearField] || '',
        type: source.type,
        link: r[source.linkField] || '',
        container: source.containerField ? r[source.containerField] : '',
      }))
      .filter((e) => e.title);
  }

  // Orden cronológico descendente. Entre registros del mismo año, el orden es
  // el de concatenar Artículos, Libros y Capítulos (cada uno en el orden de
  // su propia hoja) antes de ordenar — Array.prototype.sort es estable, así
  // que ese orden de origen se conserva como criterio de desempate,
  // predecible y siempre igual para los mismos datos.
  $: entries = [
    ...toEntries(raw.articulos, SOURCES[0]),
    ...toEntries(raw.libros, SOURCES[1]),
    ...toEntries(raw.capitulos, SOURCES[2]),
  ].sort((a, b) => {
    const yearOf = (e) => (e.year ? Number(e.year) : -Infinity);
    return yearOf(b) - yearOf(a);
  });

  $: counts = {
    articulos: raw.articulos.length,
    libros: raw.libros.length,
    capitulos: raw.capitulos.length,
  };
  $: total = counts.articulos + counts.libros + counts.capitulos;

  onMount(() => {
    Promise.all(SOURCES.map(loadSource)).finally(() => {
      isLoading = false;
    });
  });
</script>

<style>
  .intro {
    max-width: var(--measure);
    margin: 0 0 var(--space-6);
    font-family: var(--font-sans);
    font-size: var(--fs-body);
    line-height: var(--lh-body);
    color: var(--color-ink);
  }

  h1 {
    display: inline-block;
    margin: var(--space-4) 0 var(--space-3);
    padding-bottom: var(--space-2);
    font-size: var(--fs-h1);
    font-weight: 600;
    line-height: var(--lh-tight);
    color: var(--color-ink);
    font-family: var(--font-sans);
    border-bottom: var(--line-thick) solid var(--color-accent);
  }

  .index-nav ul {
    display: flex;
    gap: var(--space-2) var(--space-5);
    list-style: none;
    margin: 0 0 var(--space-6);
    padding: 0 0 var(--space-4);
    border-bottom: var(--line-thin) solid var(--color-line);
    flex-wrap: wrap;
  }

  /* El color de .link (reposo/hover/focus-visible/aria-current) se define una
     sola vez en src/app.css ("Estados de navegación") — aquí solo el layout.
     Se reutiliza la misma clase que ya usaban estas pestañas, con un cuarto
     ítem ("Todo") y el conteo real de cada categoría. */
  .link {
    display: inline-flex;
    align-items: center;
    min-height: 44px;
    font-family: var(--font-sans);
    font-size: var(--fs-ui);
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 0.06em;
    padding: var(--space-2) var(--space-1);
  }

  .count {
    font-family: var(--font-mono);
    font-size: 0.75em;
    letter-spacing: 0;
    text-transform: none;
    margin-left: 0.35em;
  }

  /* Índice bibliográfico: filas separadas por línea fina, sin imágenes, sin
     relleno de color — año en una columna fija (mono), título como elemento
     principal, tipo/publicación contenedora como metadato secundario. */
  .index {
    list-style: none;
    margin: 0;
    padding: 0;
  }

  .entry {
    display: flex;
    align-items: flex-start;
    gap: var(--space-4);
    padding-block: var(--space-5);
  }

  .entry + .entry {
    border-top: var(--line-thin) solid var(--color-line);
  }

  .entry-year {
    flex: 0 0 auto;
    min-width: 3.5em;
    font-family: var(--font-mono);
    font-size: var(--fs-meta);
    color: var(--color-ink);
  }

  .entry-body {
    flex: 1;
    min-width: 0;
  }

  .entry-title {
    display: inline-block;
    font-family: var(--font-sans);
    font-weight: 600;
    font-size: var(--fs-body);
    line-height: var(--lh-tight);
    color: var(--color-ink);
    text-decoration: none;
  }

  a.entry-title:hover,
  a.entry-title:focus-visible {
    color: var(--color-accent);
  }

  .entry-meta {
    margin: var(--space-1) 0 0;
    font-family: var(--font-mono);
    font-size: var(--fs-meta);
    letter-spacing: 0.02em;
    color: var(--color-ink);
  }

  @media (max-width: 768px) {
    .index-nav ul {
      gap: var(--space-2) var(--space-4);
    }

    .entry {
      flex-direction: column;
      gap: var(--space-1);
      padding-block: var(--space-4);
    }

    .entry-year {
      min-width: 0;
    }
  }
</style>

<div class="container">
  <h1>Publicaciones</h1>
  <p class="intro">Artículos, libros y capítulos de libro, reunidos aquí en un solo índice ordenado por año.</p>

  <nav class="index-nav" aria-label="Categorías de publicaciones">
    <ul>
      <li><a href="/publicaciones" class="link" aria-current="page">Todo<span class="count">({total})</span></a></li>
      <li><a href="/publicaciones/articulos" class="link">Artículos<span class="count">({counts.articulos})</span></a></li>
      <li><a href="/publicaciones/libros" class="link">Libros<span class="count">({counts.libros})</span></a></li>
      <li><a href="/publicaciones/capitulos" class="link">Capítulos<span class="count">({counts.capitulos})</span></a></li>
    </ul>
  </nav>

  {#if isLoading && entries.length === 0}
    <p></p>
  {:else}
    <ul class="index">
      {#each entries as entry, i (entry.link || `${entry.title}-${i}`)}
        <li class="entry">
          <span class="entry-year">{entry.year || '—'}</span>
          <div class="entry-body">
            {#if entry.link}
              <a href={entry.link} target="_blank" rel="noopener noreferrer" class="entry-title">{entry.title}</a>
            {:else}
              <span class="entry-title">{entry.title}</span>
            {/if}
            <p class="entry-meta">{entry.type}{entry.container ? ` · ${entry.container}` : ''}</p>
          </div>
        </li>
      {/each}
    </ul>
  {/if}
</div>

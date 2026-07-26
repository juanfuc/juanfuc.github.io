<script>
  import { onMount, tick } from 'svelte';
  import { csv } from 'd3-fetch';
  import '$lib/styles/structural.css';

  let data = [];
  let isLoading = true;

  // Copia local primero (si existe) para que la página se vea de inmediato al
  // navegar, y red siempre en paralelo para refrescarla — nunca al revés, y
  // nunca vaciando `data` antes de tener algo nuevo que mostrar.
  const CACHE_KEY = 'projectsData';

  async function consultarAPI() {
    const cached = localStorage.getItem(CACHE_KEY);
    if (cached) {
      data = JSON.parse(cached);
      isLoading = false;
    }

    const url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vShkGiT3_6ivNBE2n_fkBCFoIbf2wJ50G0K3giN66uRJxnKvGrrYvlRXOBGHbnaLaqiHU_O3NzzrNRb/pub?output=csv";
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

  // La hoja ya usa los valores normalizados ("Programación creativa",
  // "Proyecto de Humanidades Digitales", "Videojuego", etc.), pero registros
  // antiguos — o una copia ya guardada en localStorage antes de normalizar la
  // hoja — pueden traer variantes históricas. Se corrigen aquí, en un mapa
  // pequeño y explícito: no es una taxonomía general, solo estas equivalencias
  // conocidas. Cualquier otro valor pasa tal cual (no se oculta ni se inventa).
  const TYPE_ALIASES = {
    'proyecto personal de humanidades digitales': 'Proyecto de Humanidades Digitales',
  };
  const SUBTYPE_ALIASES = {
    'video juego': 'Videojuego',
    'visualización/mapa': 'Visualización',
  };

  function normalizeField(value, aliases) {
    if (!value) return '';
    const trimmed = String(value).trim();
    return aliases[trimmed.toLowerCase()] || trimmed;
  }

  // Filtrado exclusivamente por tipo/subtipo, en dos niveles. El valor real
  // de Humanidades Digitales sigue siendo "Proyecto de Humanidades
  // Digitales"; "Humanidades digitales" es solo la etiqueta de interfaz.
  const TYPE_OPTIONS = [
    { value: 'Todos', label: 'Todos' },
    { value: 'Programación creativa', label: 'Programación creativa' },
    { value: 'Proyecto de Humanidades Digitales', label: 'Humanidades digitales' },
  ];

  const SUBTYPE_OPTIONS_BY_TYPE = {
    'Programación creativa': ['Videojuego', 'Sonido generativo', 'Visualización'],
    'Proyecto de Humanidades Digitales': ['Plataforma de investigación', 'Edición digital'],
  };

  let typeFilter = 'Todos';
  let subtypeFilter = 'Todos';

  function selectType(value) {
    typeFilter = value;
    subtypeFilter = 'Todos'; // cambiar de tipo siempre reinicia el subtipo
  }

  function selectSubtype(value) {
    subtypeFilter = value;
  }

  // Todo lo derivado sale de `data` por expresiones reactivas: una sola
  // fuente de verdad, sin copias independientes. Nada de esto vuelve a tocar
  // Google Sheets ni localStorage — es filtrado en memoria.
  $: normalized = data.map((item) => ({
    ...item,
    tipoNorm: normalizeField(item.tipo, TYPE_ALIASES),
    subtipoNorm: normalizeField(item.subtipo, SUBTYPE_ALIASES),
  }));

  $: typeCounts = {
    Todos: normalized.length,
    'Programación creativa': normalized.filter((it) => it.tipoNorm === 'Programación creativa').length,
    'Proyecto de Humanidades Digitales': normalized.filter(
      (it) => it.tipoNorm === 'Proyecto de Humanidades Digitales'
    ).length,
  };

  $: subtypeOptions = SUBTYPE_OPTIONS_BY_TYPE[typeFilter] || [];

  $: subtypeCounts = Object.fromEntries(
    subtypeOptions.map((s) => [
      s,
      normalized.filter((it) => it.tipoNorm === typeFilter && it.subtipoNorm === s).length,
    ])
  );

  $: filtered = normalized.filter((item) => {
    if (typeFilter !== 'Todos' && item.tipoNorm !== typeFilter) return false;
    if (subtypeFilter !== 'Todos' && item.subtipoNorm !== subtypeFilter) return false;
    return true;
  });

  // Identificador estable para iframe activo / imágenes fallidas / foco: el
  // enlace del proyecto, no la posición en la lista (que cambia con cada
  // filtro). Ningún iframe existe en el DOM hasta pulsar "Cargar vista
  // interactiva"; como máximo uno activo a la vez.
  function itemId(item) {
    return item.link || item.titulo;
  }

  let activeId = null;
  let iframeLoading = false;
  let failedImages = new Set();

  function hasPreview(item) {
    return Boolean(item.imagen) && !failedImages.has(itemId(item));
  }

  function markImageFailed(item) {
    failedImages.add(itemId(item));
    failedImages = failedImages;
  }

  let loadButtons = {};
  let closeButtons = {};

  async function openEmbed(item) {
    activeId = itemId(item);
    iframeLoading = true;
    await tick();
    closeButtons[activeId]?.focus();
  }

  async function closeEmbed(item) {
    const id = itemId(item);
    activeId = null;
    iframeLoading = false;
    await tick();
    loadButtons[id]?.focus();
  }

  // Si un cambio de filtro oculta la ficha cuyo iframe está abierto, se
  // desmonta solo (sin mover el foco: ya está en el botón de filtro que se
  // acaba de pulsar).
  $: if (activeId !== null && !filtered.some((item) => itemId(item) === activeId)) {
    activeId = null;
    iframeLoading = false;
  }

  onMount(() => {
    consultarAPI();
  });
</script>

<div class="container">
  <h1>Programación creativa y Proyectos HD</h1>

  <div class="filters">
    <fieldset class="filter-group filter-group-type">
      <legend class="filter-legend">Tipo</legend>
      {#each TYPE_OPTIONS as opt (opt.value)}
        <button
          type="button"
          class="filter-btn filter-btn-type"
          aria-pressed={typeFilter === opt.value}
          on:click={() => selectType(opt.value)}
        >
          {opt.label}<span class="filter-count">{typeCounts[opt.value]}</span>
        </button>
      {/each}
    </fieldset>

    {#if subtypeOptions.length > 0}
      <fieldset class="filter-group filter-group-subtype">
        <legend class="filter-legend">Subtipo</legend>
        <button
          type="button"
          class="filter-btn filter-btn-subtype"
          aria-pressed={subtypeFilter === 'Todos'}
          on:click={() => selectSubtype('Todos')}
        >
          Todos<span class="filter-count">{typeCounts[typeFilter]}</span>
        </button>
        {#each subtypeOptions as s (s)}
          <button
            type="button"
            class="filter-btn filter-btn-subtype"
            aria-pressed={subtypeFilter === s}
            on:click={() => selectSubtype(s)}
          >
            {s}<span class="filter-count">{subtypeCounts[s]}</span>
          </button>
        {/each}
      </fieldset>
    {/if}
  </div>

  <p class="results-count" aria-live="polite">
    {filtered.length} {filtered.length === 1 ? 'proyecto' : 'proyectos'}
  </p>

  {#if filtered.length === 0}
    <p class="empty-state">No hay proyectos en esta categoría.</p>
  {:else}
    {#each filtered as item (itemId(item))}
      <article class="list-card">
        {#if activeId === itemId(item)}
          <div class="list-card-media">
            <div class="embed">
              {#if iframeLoading}
                <p class="embed-status">Cargando…</p>
              {/if}
              <iframe
                src={item.link}
                title={`Vista interactiva: ${item.titulo}`}
                loading="lazy"
                allowfullscreen
                on:load={() => (iframeLoading = false)}
              ></iframe>
            </div>
            <button
              type="button"
              class="action action-close"
              bind:this={closeButtons[itemId(item)]}
              on:click={() => closeEmbed(item)}
            >
              Volver a la ficha
            </button>
          </div>
        {:else if hasPreview(item)}
          <div class="list-card-media">
            <div class="preview">
              <img src={item.imagen} alt="" loading="lazy" on:error={() => markImageFailed(item)} />
            </div>
          </div>
        {/if}
        <div class="list-card-body">
          <h2>{item.titulo}</h2>
          <p class="meta">{item.tipoNorm}{item.fecha ? ` · ${item.fecha}` : ''}</p>
          {#if item.descripcion}
            <p class="description">{item.descripcion}</p>
          {/if}
          <div class="actions">
            {#if activeId !== itemId(item)}
              <button
                type="button"
                class="action"
                bind:this={loadButtons[itemId(item)]}
                aria-label={`Cargar vista interactiva de ${item.titulo}`}
                on:click={() => openEmbed(item)}
              >
                Cargar vista interactiva
              </button>
            {/if}
            <a href={item.link} target="_blank" rel="noopener noreferrer" class="action">Abrir proyecto ↗</a>
          </div>
        </div>
      </article>
    {/each}
  {/if}
</div>

<style>
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

  /* Filtros como navegación editorial: texto con línea fina, nunca cápsulas
     ni botones de aplicación comercial. El primer nivel (tipo) tiene mayor
     jerarquía tipográfica que el segundo (subtipo). */
  .filters {
    margin-bottom: var(--space-5);
    padding-bottom: var(--space-4);
    border-bottom: var(--line-thin) solid var(--color-line);
  }

  .filter-group {
    display: flex;
    flex-wrap: wrap;
    align-items: baseline;
    gap: var(--space-1) var(--space-4);
    border: none;
    margin: 0;
    padding: 0;
  }

  .filter-group-subtype {
    margin-top: var(--space-3);
    padding-top: var(--space-3);
    border-top: var(--line-thin) solid var(--color-line);
  }

  .filter-legend {
    width: 100%;
    margin: 0 0 var(--space-1);
    padding: 0;
    font-family: var(--font-mono);
    font-size: var(--fs-meta);
    letter-spacing: 0.04em;
    text-transform: uppercase;
    color: var(--color-ink);
  }

  .filter-btn {
    display: inline-flex;
    align-items: center;
    gap: 0.4em;
    min-height: 44px;
    padding: var(--space-1) 0;
    border: none;
    border-bottom: var(--line-thin) solid transparent;
    background: none;
    font-family: var(--font-sans);
    text-transform: uppercase;
    letter-spacing: 0.04em;
    color: var(--color-ink);
    cursor: pointer;
  }

  .filter-btn-type {
    font-weight: 600;
    font-size: var(--fs-body);
  }

  .filter-btn-subtype {
    font-weight: 500;
    font-size: var(--fs-ui);
  }

  .filter-btn:hover,
  .filter-btn:focus-visible {
    color: var(--color-accent);
  }

  .filter-btn[aria-pressed='true'] {
    color: var(--color-accent);
    border-bottom-color: var(--color-accent);
  }

  .filter-count {
    font-family: var(--font-mono);
    font-size: 0.75em;
    letter-spacing: 0;
  }

  .results-count {
    margin: 0 0 var(--space-5);
    font-family: var(--font-mono);
    font-size: var(--fs-meta);
    letter-spacing: 0.04em;
    text-transform: uppercase;
    color: var(--color-ink);
  }

  .empty-state {
    margin: 0;
    padding-block: var(--space-5);
    font-family: var(--font-sans);
    font-size: var(--fs-body);
    color: var(--color-ink);
  }

  h2 {
    margin: 0 0 var(--space-1);
    font-family: var(--font-sans);
    font-weight: 600;
    font-size: var(--fs-h2);
  }

  .meta {
    margin: 0;
    font-family: var(--font-mono);
    font-size: var(--fs-meta);
    letter-spacing: 0.02em;
  }

  .description {
    margin: var(--space-2) 0 0;
    font-family: var(--font-sans);
    font-weight: 400;
    font-size: var(--fs-body);
    line-height: var(--lh-body);
  }

  /* Previsualización e incrustación comparten la misma caja (mismas
     dimensiones, mismo aspect-ratio): pasar de una a otra no produce salto
     de layout. Sin imagen, no se renderiza .list-card-media en absoluto —
     nada de espacio vacío ni placeholder ilustrado. */
  .preview,
  .embed {
    position: relative;
    width: 100%;
    aspect-ratio: 4 / 3;
    overflow: hidden;
  }

  .preview img {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .embed iframe {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    border: none;
  }

  .embed-status {
    position: absolute;
    inset: 0;
    z-index: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    margin: 0;
    background: var(--color-bg);
    font-family: var(--font-mono);
    font-size: var(--fs-meta);
    color: var(--color-ink);
  }

  @media (prefers-reduced-motion: no-preference) {
    .embed {
      animation: fade-in 0.2s ease;
    }
  }

  @keyframes fade-in {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  /* Acciones como texto, no como botones comerciales: sin relleno, sin
     borde redondeado, sin sombra — una línea fina que se vuelve ámbar en
     hover/foco, igual que el resto de los estados interactivos del sitio. */
  .actions {
    display: flex;
    flex-wrap: wrap;
    gap: var(--space-5);
    margin-top: var(--space-3);
  }

  .action {
    display: inline-flex;
    align-items: center;
    min-height: 44px;
    padding: var(--space-2) 0;
    border: none;
    border-bottom: var(--line-thin) solid var(--color-line);
    background: none;
    font-family: var(--font-sans);
    font-weight: 600;
    font-size: var(--fs-meta);
    text-transform: uppercase;
    letter-spacing: 0.06em;
    color: var(--color-ink);
    text-decoration: none;
    cursor: pointer;
  }

  .action:hover,
  .action:focus-visible {
    color: var(--color-accent);
    border-bottom-color: var(--color-accent);
  }

  .action-close {
    margin-top: var(--space-3);
  }

  @media (max-width: 768px) {
    .actions {
      flex-direction: column-reverse;
      align-items: flex-start;
      gap: var(--space-2);
    }
  }
</style>

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

  // Un registro con comillas sin escapar en Sheets (p. ej. una etiqueta <img>
  // pegada por error en la celda de imagen) rompe el CSV para esa fila: la URL
  // de imagen queda con comillas literales y no carga. Se descarta esa fila en
  // vez de mostrar una imagen rota; no se toca la hoja.
  function isUsableImage(url) {
    return typeof url === 'string' && url.trim() !== '' && !url.includes('"');
  }

  // El error real que motivó esto: un `indice` repetido en la hoja. Se avisa
  // solo en desarrollo (Vite elimina esta rama en el build de producción) y
  // nunca se bloquea ni se recorta la grilla por esto — es un aviso, no una
  // validación que impida renderizar.
  function warnDuplicateIndices(items) {
    if (!import.meta.env.DEV) return;
    const seen = new Set();
    const duplicated = new Set();
    for (const item of items) {
      if (seen.has(item.indice)) duplicated.add(item.indice);
      seen.add(item.indice);
    }
    if (duplicated.size > 0) {
      console.warn(
        '[portada] "indice" duplicado en la hoja de imágenes:',
        [...duplicated].join(', '),
        '— la grilla se muestra igual; corrige la hoja cuando sea posible.'
      );
    }
  }

  // Las imágenes de portada son puertas de entrada a contenidos que ya están
  // catalogados en otras hojas del sitio (Artículos, Libros, Capítulos,
  // Programación). En vez de pedir autoría/crédito en la hoja de portada, se
  // cruza su `link` contra el enlace real de cada conjunto para mostrar el
  // título y el tipo de contenido al que lleva. Sin coincidencia, no hay
  // ficha — nunca se inventa texto ni se deriva del dominio.
  //
  // Normaliza un enlace para comparar equivalencia entre hojas sin exigir
  // cadenas idénticas: recorta espacios, ignora mayúsculas, barra final,
  // fragmento #, y parámetros de idioma/tracking habituales que no cambian
  // el contenido identificado. Los DOI se comparan por su identificador
  // (10.xxxx/...), no por la URL completa del resolutor, para que distintos
  // prefijos (doi.org, con o sin https, con espacios o puntuación pegada
  // por error de captura) sigan siendo el mismo DOI.
  const IGNORED_PARAMS = ['atr', 'locale', 'utm_source', 'utm_medium', 'utm_campaign', 'single', 'output'];

  function normalizeLink(url) {
    if (!url) return '';
    const trimmed = String(url).trim();
    const doi = trimmed.match(/10\.\d{4,9}\/\S+/);
    if (doi) {
      return 'doi:' + doi[0].toLowerCase().replace(/[.,;]+$/, '');
    }
    try {
      const u = new URL(trimmed);
      const params = new URLSearchParams(u.search);
      IGNORED_PARAMS.forEach((p) => params.delete(p));
      const query = params.toString();
      const path = u.pathname.replace(/\/+$/, '');
      return `${u.hostname.toLowerCase()}${path}${query ? '?' + query : ''}`;
    } catch {
      return trimmed.toLowerCase().replace(/\/+$/, '');
    }
  }

  // Un conjunto por cada tabla local que puede ser destino de un enlace de
  // portada. `typeLabel` fija el tipo para las tres tablas bibliográficas;
  // Programación ya trae su propio campo `tipo`, más preciso que una
  // etiqueta genérica de "proyecto".
  const METADATA_SOURCES = [
    {
      cacheKey: 'articlesData',
      url: "https://docs.google.com/spreadsheets/d/e/2PACX-1vThq4fhXW3GP7MsfMBrvjUgGN0SV-yQztsRwtWq5kCDpGVJdRAskzi0Mq2GYJuFsqIV4eHp9bfNoqVR/pub?gid=2145965265&single=true&output=csv",
      linkField: 'DOI',
      titleField: 'Título',
      yearField: 'Año',
      typeLabel: 'Artículo',
    },
    {
      cacheKey: 'booksData',
      url: "https://docs.google.com/spreadsheets/d/e/2PACX-1vSkGYClXLB0CD0KA30_fGfmmfVqnClIEnvsPh6AD8k_QPdkdlGr3bl3fikK3inXli40qRZng4qqp12n/pub?output=csv",
      linkField: 'DOI',
      titleField: 'Título',
      yearField: 'Año',
      typeLabel: 'Libro',
    },
    {
      cacheKey: 'chaptersData',
      url: "https://docs.google.com/spreadsheets/d/e/2PACX-1vRgQUPIxtoE4v98W3bfYimpMXZwVSWPAomSC04M1HMK3fpU_LsHesfRIynehNKOc4sNlTuzRGUX0-6o/pub?output=csv",
      linkField: 'DOI',
      titleField: 'Título',
      yearField: 'Año',
      typeLabel: 'Capítulo',
    },
    {
      cacheKey: 'projectsData',
      url: "https://docs.google.com/spreadsheets/d/e/2PACX-1vShkGiT3_6ivNBE2n_fkBCFoIbf2wJ50G0K3giN66uRJxnKvGrrYvlRXOBGHbnaLaqiHU_O3NzzrNRb/pub?output=csv",
      linkField: 'link',
      titleField: 'titulo',
      yearField: 'fecha',
      typeField: 'tipo',
    },
  ];

  let metadataIndex = new Map();

  function indexRecords(records, source) {
    for (const record of records) {
      const norm = normalizeLink(record[source.linkField]);
      const title = record[source.titleField];
      if (!norm || !title) continue;
      if (metadataIndex.has(norm)) continue;
      metadataIndex.set(norm, {
        displayTitle: title,
        displayType: source.typeField ? record[source.typeField] : source.typeLabel,
        displayYear: source.yearField ? record[source.yearField] : '',
      });
    }
    metadataIndex = new Map(metadataIndex); // reasignar: dispara la reactividad de Svelte
  }

  // Cada tabla se resuelve de forma independiente y en paralelo: copia local
  // primero si existe (para no esperar red), red siempre después para
  // refrescarla. La grilla nunca espera a que esto termine — cuando cada
  // fuente resuelve, `metadataIndex` se reasigna y las fichas aparecen sobre
  // la grilla ya visible, sin desmontarla ni reconstruirla.
  async function loadMetadataSource(source) {
    const cached = localStorage.getItem(source.cacheKey);
    if (cached) {
      try {
        indexRecords(JSON.parse(cached), source);
      } catch (error) {
        console.error('Error leyendo caché de metadatos:', error);
      }
    }
    try {
      const records = await csv(source.url);
      indexRecords(records, source);
      localStorage.setItem(source.cacheKey, JSON.stringify(records));
    } catch (error) {
      console.error('Error fetching metadata source:', source.url, error);
    }
  }

  // Copia local primero (si existe) para que la retícula se vea de inmediato al
  // navegar, y red siempre en paralelo para refrescarla — nunca al revés, y
  // nunca vaciando `data` antes de tener algo nuevo que mostrar.
  const CACHE_KEY = 'homeGridData';
  let rawData = [];

  // `data` combina cada imagen con la metadata ya resuelta (si la hay). Al
  // ser una expresión reactiva, se recalcula sola cuando cambia `rawData` o
  // `metadataIndex`, sin volver a barajar ni a pedir nada de nuevo.
  $: data = rawData.map((item) => {
    const match = metadataIndex.get(normalizeLink(item.link));
    return match ? { ...item, ...match } : item;
  });

  // Con coincidencia, el nombre accesible del enlace es el título real (más
  // tipo/año si existen); sin coincidencia, un neutral "Abrir elemento N" —
  // nunca el dominio ni un texto inventado.
  function accessibleLabel(item) {
    if (!item.displayTitle) return `Abrir elemento ${item.indice}`;
    const meta = [item.displayType, item.displayYear].filter(Boolean).join(', ');
    return meta ? `${item.displayTitle}. ${meta}` : item.displayTitle;
  }

  // Clave del {#each}: no solo `indice` (puede repetirse en la hoja, como ya
  // ocurrió) ni solo la posición del array. Combina indice + enlace
  // normalizado — estable frente al recálculo reactivo de `data` cuando
  // llega la metadata — y añade la posición únicamente como último
  // desempate, para que dos filas realmente idénticas no colisionen.
  function entryKey(item, i) {
    return `${item.indice}|${normalizeLink(item.link)}|${i}`;
  }

  // Registro de imágenes que fallaron al cargar (por indice+enlace, no por
  // posición: sigue siendo la misma imagen aunque `data` se recalcule). Una
  // vez marcada, el <img> deja de renderizarse — sin reintentos.
  let failedImages = new Set();
  function contentKey(item) {
    return `${item.indice}|${normalizeLink(item.link)}`;
  }
  function markImageFailed(item) {
    failedImages.add(contentKey(item));
    failedImages = failedImages;
  }

  // Solo las primeras imágenes de la primera fila de escritorio (4 columnas)
  // compiten por ancho de banda con prioridad alta; el resto se difiere.
  const PRIORITY_COUNT = 4;

  async function loadAndShuffleData() {
    const seed = mulberry32(getSessionSeed());
    const cached = localStorage.getItem(CACHE_KEY);
    if (cached) {
      rawData = JSON.parse(cached);
      isLoading = false;
    }

    const url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vQZsnsur1Nec98cL4ujG9kBFmXU0TyvOtkd7TR61LsqhPuu4r7CUU-lZ9BF__cmYNHNA8rCcSsafvgq/pub?output=csv";
    try {
      const fetchedData = (await csv(url)).filter((item) => isUsableImage(item.imagen));
      warnDuplicateIndices(fetchedData);
      shuffleArray(fetchedData, seed);
      rawData = fetchedData;
      localStorage.setItem(CACHE_KEY, JSON.stringify(rawData));
    } catch (error) {
      console.error("Error fetching data:", error);
    } finally {
      isLoading = false;
    }
  }

  onMount(() => {
    loadAndShuffleData();
    METADATA_SOURCES.forEach(loadMetadataSource);
  });
</script>

<style>
  /* Contenedor propio de la portada (no usa la clase compartida `.container`: sus reglas
     son deliberadamente distintas — sin padding ni max-width — y no debe heredar las de
     las páginas interiores). Sin fondo propio: se apoya en el --color-bg de <body>, el
     mismo que el resto del sitio. El espacio antes de la retícula es el
     margin-bottom del propio <nav> (--space-header-gap, src/app.css) — no un
     padding local, para que el ritmo sea el mismo en toda página. */

  /* Retícula de celdas regulares (mismo criterio de simetría que el checkpoint anterior a
     R02: columnas iguales, altura fija por proporción — antes 250px fijos, ahora
     aspect-ratio para que la regularidad se mantenga en cualquier ancho de viewport).
     Las imágenes no conservan aquí su proporción individual: se recortan con object-fit
     para que todas las celdas midan lo mismo (decisión explícita de esta corrección,
     ver docs/renovacion-perfil/rondas/R02-sistema-visual.md). */
  .grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: var(--space-5);
    padding: 0 var(--space-5) var(--space-5);
  }

  /* Sin display:flex: el único hijo (el enlace, o .card-image cuando no hay
     enlace) debe ocupar el ancho completo de la celda con el comportamiento
     normal de bloque. Con display:flex ese hijo pasaba a ser un flex item
     con ancho "shrink-to-fit" en vez de "llenar el contenedor" — y como el
     <img> es position:absolute (no aporta tamaño intrínseco al cálculo de
     shrink-to-fit), la celda colapsaba a ancho/alto cero y la imagen
     desaparecía por completo, aunque los datos y el marcado eran correctos. */
  .card {
    width: 100%;
  }

  .card > a {
    display: block;
  }

  /* La celda reserva su espacio (aspect-ratio) y muestra un tono neutro
     derivado de la tinta de identidad antes de que la imagen termine de
     pintar — nunca blanco puro ni un hueco vacío, y sin cambiar de tamaño
     cuando la imagen llega. */
  .card-image {
    position: relative;
    width: 100%;
    aspect-ratio: 4 / 3;
    overflow: hidden;
    background-color: rgba(5, 79, 109, 0.06);
  }

  .card-image img {
    position: absolute;
    inset: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  /* Aparición breve al insertarse en el DOM (no al terminar de descargar: no
     depende de eventos de red ni de JS adicional, así que nunca deja una
     imagen invisible). Se omite por completo con prefers-reduced-motion. */
  @media (prefers-reduced-motion: no-preference) {
    .card-image img {
      animation: card-image-in 0.3s ease-out;
    }
  }

  @keyframes card-image-in {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  /* Variante tipográfica cuando la imagen falla: mismo tamaño de celda, sin
     icono de imagen rota, coherente con el resto de fichas de archivo del
     sitio (borde fino, fondo neutro, título + tipo/año cuando existen). */
  .image-fallback {
    position: absolute;
    inset: 0;
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 0.35em;
    padding: var(--space-3);
    border: var(--line-thin) solid var(--color-line);
  }

  .fallback-title {
    font-family: var(--font-sans);
    font-weight: 600;
    font-size: 0.9rem;
    line-height: 1.3;
    color: var(--color-ink);
  }

  .fallback-meta {
    font-family: var(--font-mono);
    font-size: 0.72rem;
    letter-spacing: 0.02em;
    color: var(--color-ink);
  }

  /* Ficha del contenido enlazado (título + tipo/año, cuando hay coincidencia
     en Artículos/Libros/Capítulos/Programación): superpuesta dentro de la
     misma celda, nunca cambia su tamaño. Oculta a lectores de pantalla
     (aria-hidden en el marcado): el nombre accesible del enlace ya lleva el
     mismo texto por su cuenta, así que exponer también la ficha duplicaría
     el anuncio. La veladura usa la propia tinta de identidad (--color-ink)
     en vez de un gris/negro genérico, lo bastante opaca para leer el texto
     en blanco sobre cualquier imagen. */
  .caption {
    position: absolute;
    inset: auto 0 0 0;
    display: flex;
    flex-direction: column;
    gap: 0.15em;
    padding: var(--space-2) var(--space-3);
    background: linear-gradient(to top, rgba(20, 20, 20, 0.92) 0%, rgba(15, 15, 15, 0.55) 60%, transparent 100%);
    color: #fff;
  }

  .caption-title {
    font-family: var(--font-sans);
    font-weight: 600;
    font-size: 0.85rem;
    line-height: 1.25;
  }

  .caption-meta {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    letter-spacing: 0.02em;
    color: rgba(255, 255, 255, 0.85);
  }

  /* Gris→color solo en dispositivos con puntero preciso y hover real; en
     táctil/puntero grueso la imagen se muestra en color y la ficha ya es
     visible desde el inicio (sin JS, sin doble toque). */
  @media (hover: hover) and (pointer: fine) {
    .card-image img {
      transition: filter 0.4s ease;
      filter: grayscale(100%) brightness(85%) contrast(110%);
    }

    .caption {
      opacity: 0;
      transition: opacity 0.4s ease;
    }

    a:hover .card-image img,
    a:focus-visible .card-image img,
    .card-image:hover img {
      filter: none;
    }

    a:hover .caption,
    a:focus-visible .caption,
    .card-image:hover .caption {
      opacity: 1;
    }
  }

  @media screen and (max-width: 1024px) {
    .grid {
      grid-template-columns: repeat(3, 1fr);
    }
  }

  @media screen and (max-width: 768px) {
    .grid {
      grid-template-columns: repeat(2, 1fr);
      gap: var(--space-4);
      padding: 0 var(--space-4) var(--space-4);
    }
  }
</style>

<div class="home-grid">
  {#if isLoading}
    <p></p>
  {:else}
    <div class="grid">
      {#each data as item, i (entryKey(item, i))}
        <div class="card">
          {#if item.link}
            <a href={item.link} target="_blank" rel="noopener noreferrer">
              <div class="card-image">
                {#if failedImages.has(contentKey(item))}
                  <div class="image-fallback">
                    <span class="fallback-title">{item.displayTitle || `Elemento ${item.indice}`}</span>
                    {#if item.displayType}
                      <span class="fallback-meta">{item.displayType}{item.displayYear ? ` · ${item.displayYear}` : ''}</span>
                    {/if}
                  </div>
                {:else}
                  <img
                    src={item.imagen}
                    alt={accessibleLabel(item)}
                    loading={i < PRIORITY_COUNT ? 'eager' : 'lazy'}
                    fetchpriority={i < PRIORITY_COUNT ? 'high' : 'low'}
                    decoding="async"
                    on:error={() => markImageFailed(item)}
                  />
                  {#if item.displayTitle}
                    <div class="caption" aria-hidden="true">
                      <span class="caption-title">{item.displayTitle}</span>
                      {#if item.displayType}
                        <span class="caption-meta">{item.displayType}{item.displayYear ? ` · ${item.displayYear}` : ''}</span>
                      {/if}
                    </div>
                  {/if}
                {/if}
              </div>
            </a>
          {:else}
            <div class="card-image">
              {#if failedImages.has(contentKey(item))}
                <div class="image-fallback">
                  <span class="fallback-title">{item.displayTitle || `Elemento ${item.indice}`}</span>
                  {#if item.displayType}
                    <span class="fallback-meta">{item.displayType}{item.displayYear ? ` · ${item.displayYear}` : ''}</span>
                  {/if}
                </div>
              {:else}
                <img
                  src={item.imagen}
                  alt={accessibleLabel(item)}
                  loading={i < PRIORITY_COUNT ? 'eager' : 'lazy'}
                  fetchpriority={i < PRIORITY_COUNT ? 'high' : 'low'}
                  decoding="async"
                  on:error={() => markImageFailed(item)}
                />
                {#if item.displayTitle}
                  <div class="caption" aria-hidden="true">
                    <span class="caption-title">{item.displayTitle}</span>
                    {#if item.displayType}
                      <span class="caption-meta">{item.displayType}{item.displayYear ? ` · ${item.displayYear}` : ''}</span>
                    {/if}
                  </div>
                {/if}
              {/if}
            </div>
          {/if}
        </div>
      {/each}
    </div>
  {/if}
</div>

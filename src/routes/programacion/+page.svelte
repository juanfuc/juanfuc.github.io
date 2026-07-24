<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import '$lib/styles/structural.css';

  let data = [];
  let isLoading = true;

  async function consultarAPI() {
    const CACHE_DURATION = 3600 * 1000; // 1 hora en milisegundos
    const now = Date.now();
    const cachedData = localStorage.getItem('projectsData');
    const cacheTimestamp = localStorage.getItem('projectsDataTimestamp');

    console.log('Cache Timestamp:', cacheTimestamp);
    console.log('Current Time:', now);
    console.log('Cache Duration:', CACHE_DURATION);

    if (cachedData && cacheTimestamp && (now - cacheTimestamp < CACHE_DURATION)) {
      console.log('Using cached data');
      data = JSON.parse(cachedData);
      isLoading = false;
    } else {
      console.log('Fetching new data from Google Sheets');
      const url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vShkGiT3_6ivNBE2n_fkBCFoIbf2wJ50G0K3giN66uRJxnKvGrrYvlRXOBGHbnaLaqiHU_O3NzzrNRb/pub?output=csv";
      try {
        const fetchedData = await csv(url);
        console.log('Fetched Data:', fetchedData);
        data = fetchedData;
        localStorage.setItem('projectsData', JSON.stringify(data));
        localStorage.setItem('projectsDataTimestamp', now);
      } catch (error) {
        console.error("Error fetching data:", error);
      } finally {
        isLoading = false;
      }
    }
  }

  onMount(() => {
    consultarAPI();
  });
</script>

<div class="container">
  <h1>Programación creativa y Proyectos HD</h1>
  <br><br>

  {#each data as item}
    <div class="list-card">
      <div class="list-card-media">
        <iframe src={item.link} title={`Proyecto interactivo: ${item.titulo}`} allowfullscreen style="border:none; overflow:hidden" loading="lazy"></iframe>
      </div>
      <div class="list-card-body">
       <a href={item.link} target="_blank" rel="noopener noreferrer" style="text-decoration: none; color: #054f6d;"> <h2>{item.titulo}</h2></a>
        <p>{item.tipo}</p>
        <p>{item.fecha}</p>
        <p class='autor'>{item.autor}</p>
        <p class="description">{item.descripcion}</p>
      </div>
    </div>
  {/each}
</div>

<style>
  .list-card-media iframe {
    width: 100%; /* Ajusta el iframe para que ocupe todo el ancho disponible */
    height: 350px; /* Puedes ajustar este valor para hacer el recuadro más grande o pequeño */
  }

  .list-card-body a h2:hover {
    text-decoration: underline;
    color: #054f6d;
  }

  h1 {
    margin-bottom: 1px;
    font-size: 2vw;
    font-weight: 550;
    color: #f4ba00;
    font-family: "Poppins", sans-serif;
  }

  h2, p {
    margin: 5px 0;
  }

  .autor {
    font-size: larger;
    color: #EE4E4E;
  }

  @media (max-width: 768px) {

    .container h1 {
      font-size: 1.5em;
    }

    .list-card-media iframe {
      width: 100%; /* Asegura que el iframe ocupe todo el ancho disponible */
      height: auto; /* Ajusta la altura automáticamente */
      min-height: 200px; /* Establece una altura mínima para pantallas pequeñas */
    }
  }
</style>

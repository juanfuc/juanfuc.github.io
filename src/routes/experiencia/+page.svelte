<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import '$lib/styles/structural.css';

  let data = [];
  let isLoading = true;

  async function consultarAPI() {
    const CACHE_DURATION = 3600 * 1000; // 1 hora en milisegundos
    const now = Date.now();
    const cachedData = localStorage.getItem('experienceData');
    const cacheTimestamp = localStorage.getItem('experienceDataTimestamp');

    console.log('Cache Timestamp:', cacheTimestamp);
    console.log('Current Time:', now);
    console.log('Cache Duration:', CACHE_DURATION);

    if (cachedData && cacheTimestamp && (now - cacheTimestamp < CACHE_DURATION)) {
      console.log('Using cached data');
      data = JSON.parse(cachedData);
      isLoading = false;
    } else {
      console.log('Fetching new data from Google Sheets');
      const url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vQlN-YyegviJd-QDECGZZ3YJqw4edEV11bZYppkOuLxd63z7uca-VxI5psQQG_4TZdUtGSJx_JXy11G/pub?output=csv";
      try {
        const fetchedData = await csv(url);
        console.log('Fetched Data:', fetchedData);
        data = fetchedData;
        localStorage.setItem('experienceData', JSON.stringify(data));
        localStorage.setItem('experienceDataTimestamp', now);
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
  <h1>Experiencia Profesional</h1>
  <br><br>

  {#if isLoading}
    <p></p>
  {:else}
    {#each data as item}
      <div class="card">
        <div class="card-content">
          <h2>{item.Cargo}</h2>
          <p>{item.Institución}</p>
          <p>{item.Periodo}</p>
          <p class="description">{item.Descripción}</p>
        </div>
      </div>
    {/each}
  {/if}
</div>

<style>
  .card {
    display: flex;
    overflow: hidden;
    justify-content: space-between;
    flex-direction: column;
    align-items: flex-start;
    margin-bottom: 20px;
    padding: 10px;
    border-radius: 8px;
    background-color: #e6ebf1;
    color: #054f6d;
  }

  .card-content {
    padding: 10px;
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

  .description {
    font-size: 14px;
    color: #333;
  }

  @media (max-width: 768px) {
    .container h1 {
      font-size: 1.5em;
    }
    .container {
      padding: 10px;
    }

    .card {
      flex-direction: column;
      height: auto;
      padding: 3vw;
    }
  }
</style>

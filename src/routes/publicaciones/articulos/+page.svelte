<script>
  import { onMount } from 'svelte';
  import { csv } from 'd3-fetch';
  import Icon from '@iconify/svelte';
  import '$lib/styles/structural.css';

  let data = [];
  let isLoading = true;

  async function consultarAPI() {
    const cachedData = localStorage.getItem('articlesData');
    if (cachedData) {
      data = JSON.parse(cachedData);
      isLoading = false;
    } else {
      let url = "https://docs.google.com/spreadsheets/d/e/2PACX-1vThq4fhXW3GP7MsfMBrvjUgGN0SV-yQztsRwtWq5kCDpGVJdRAskzi0Mq2GYJuFsqIV4eHp9bfNoqVR/pub?gid=2145965265&single=true&output=csv";

      try {
        const fetchedData = await csv(url);
        data = fetchedData;
        localStorage.setItem('articlesData', JSON.stringify(data));
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
  <a href="/publicaciones" class="back-link"> <Icon icon="material-symbols:arrow-back" /></a>
  <h1>Artículos académicos</h1>
  <br><br>

  {#each data as item}
    <div class="list-card">
      <div class="list-card-media">
        <a href={item.DOI} target='_blank' rel="noopener noreferrer"><img src={item.Imagen} alt={item.Título} loading="lazy" /></a>
      </div>
      <div class="list-card-body">
        <h2>{item.Título}</h2>
        <p>{item.Revista}</p>
        <p>{item.Volumen}</p>
        <p>{item.Número}</p>
        <p>{item.Año}</p>
        <p>{item.Páginas}</p>
        <p class='autor'>{item.Autor}</p>
        <!-- <p class="description">{item.Descripcion}</p> -->
      </div>
    </div>
  {/each}
</div>

<style>
  .back-link {
    color: #f4ba00;
    font-size: 3.5em;
  }

  .back-link:hover {
    font-size: 3.8em;
    box-shadow: 0 0 6px #333;
    transform: scale(1);
    transition: all 1.5s ease;
  }

  .list-card-media img {
    max-width: 100%;
    height: auto;
    transition: all 1.5s ease;
  }

  .list-card-media img:hover {
    box-shadow: 0 0 32px #333;
    transform: scale(1);
  }

  h1 {
    margin-bottom: 1px;
    font-size: 2vw;
    font-weight: 500;
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

  a {
    color: #E16526;
    text-decoration: none;
  }

  a:hover {
    text-decoration: underline;
  }

  @media (max-width: 768px) {


    .container h1 {
      font-size: 1.5em;
    }

    .list-card-media img {
      width: 100%;
    }
  }
</style>

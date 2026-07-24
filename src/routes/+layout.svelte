<script>
  import Icon from '@iconify/svelte';
  import { page } from '$app/stores';

  $: pathname = $page.url.pathname;
  $: isActive = (href) => (href === '/' ? pathname === '/' : pathname.startsWith(href));
</script>

<style>
/* Estilos para pantallas grandes */
nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 1em;
  color: #054f6d;
  margin-bottom: 20px;
  font-family: "Poppins", sans-serif;
  font-weight: 450;
  padding: 1%;
}

nav .left {
  flex: 1;
  display: flex;
  justify-content: flex-start;
  font-size: 2.5em; /* Tamaño de fuente mayor para el nombre */
}

nav .icons {
  display: flex;
  flex-direction: row;
  align-items: center;
  font-size: 20px !important;
  color: #f4ba00 !important;
  list-style: none;
  margin: 0 0 0 10px;
  padding: 0;
}

nav .right {
  flex: 1;
  display: flex;
  justify-content: space-evenly;
  list-style: none;
  margin: 0;
  padding: 0;
}

nav a {
  color: #054f6d;
  text-decoration: none;
  padding: 10px;
}

nav a:focus {
  color: #f4ba00;
}

nav a:hover {
  color: #f4ba00;
}

nav a[aria-current="page"] {
  color: #f4ba00;
}

/* Estilos para pantallas pequeñas (móviles) */
@media screen and (max-width: 768px) {
  nav {
    flex-direction: column;
    align-items: center;
    font-size: 0.8em; /* Ajusta el tamaño del texto del menú para móviles */
  }

  nav .left {
    align-items: center;
    margin-bottom: 10px;
  }

  nav .icons {
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
    margin-bottom: 10px;
  }

  nav .right {
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: center;
  }
}

</style>

<nav aria-label="Principal">
  <div class="left">
    <a href="/" aria-current={isActive('/') ? 'page' : undefined}>Juan Felipe Urueña Calderón</a>
    <ul class="icons">
      <li><a href="https://github.com/juanfuc" target="_blank" rel="noopener noreferrer" aria-label="GitHub"><Icon icon="mdi:github" /></a></li>
      <li><a href="https://x.com/@juuuuffff" target="_blank" rel="noopener noreferrer" aria-label="X (Twitter)"><Icon icon="pajamas:twitter" /></a></li>
      <li><a href="https://unal.academia.edu/JuanFelipeUrue%C3%B1aCalder%C3%B3n" target="_blank" rel="noopener noreferrer" aria-label="Academia.edu"><Icon icon="simple-icons:academia" /></a></li>
      <li><a href="https://www.researchgate.net/profile/Juan-Uruena" target="_blank" rel="noopener noreferrer" aria-label="ResearchGate"><Icon icon="simple-icons:researchgate" /></a></li>
      <li><a href="https://orcid.org/0000-0003-0576-159X" target="_blank" rel="noopener noreferrer" aria-label="ORCID"><Icon icon="simple-icons:orcid" /></a></li>
      <li><a href="https://observablehq.com/@juanfuc" target="_blank" rel="noopener noreferrer" aria-label="Observable"><Icon icon="simple-icons:observable" /></a></li>
    </ul>
  </div>
  <ul class="right">
    <li><a href="/perfil" aria-current={isActive('/perfil') ? 'page' : undefined}>Perfil</a></li>
    <li><a href="/experiencia" aria-current={isActive('/experiencia') ? 'page' : undefined}>Experiencia</a></li>
    <li><a href="/publicaciones" aria-current={isActive('/publicaciones') ? 'page' : undefined}>Publicaciones</a></li>
    <li><a href="/programacion" aria-current={isActive('/programacion') ? 'page' : undefined}>Programación/HD</a></li>
  </ul>
</nav>

<slot />

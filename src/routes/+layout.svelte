<script>
  import '../app.css';
  import Icon from '@iconify/svelte';
  import { page } from '$app/stores';

  $: pathname = $page.url.pathname;
  $: isActive = (href) => (href === '/' ? pathname === '/' : pathname.startsWith(href));
</script>

<style>
/* Encabezado en una franja: nombre a la izquierda, menú a la derecha, misma
   fila mientras haya espacio (se adapta con flex-wrap, no se apila por una
   regla fija de ancho). Sin línea ni caja bajo el nav: la separación con el
   contenido es aire, no un separador — un único `margin-bottom` (token
   --space-header-gap, definido en src/app.css) para que portada y todas las
   páginas interiores compartan el mismo ritmo tras el encabezado, en vez de
   que cada página fije su propio padding-top local. */
nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: var(--space-4) var(--space-6);
  font-family: var(--font-sans);
  padding: var(--space-6) var(--space-5);
  margin-bottom: var(--space-header-gap);
}

.identity {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: var(--space-3);
  min-width: 0;
}

/* El nombre es el elemento de mayor presencia del encabezado — notablemente
   mayor que el menú — pero sigue siendo texto plano: sin subrayado, caja ni
   fondo en ningún estado (color de estado en src/app.css). Tamaño moderado
   a propósito: debe convivir con el menú en la misma franja en escritorio,
   no solo verse grande en aislamiento. */
.site-name {
  font-size: clamp(1.75rem, 1.2rem + 1.6vw, 2.5rem);
  font-weight: 700;
  line-height: 1.1;
  padding: var(--space-1) 0;
}

.icons {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: var(--space-2);
  list-style: none;
  margin: 0;
  padding: 0;
  font-size: 1.15rem;
}

.menu {
  display: flex;
  gap: var(--space-2) var(--space-5);
  list-style: none;
  margin: 0;
  padding: 0;
  flex-wrap: wrap;
}

/* El color de nav a (reposo/hover/focus-visible/aria-current) se define una
   sola vez en src/app.css ("Estados de navegación") — aquí solo el layout. */
nav a {
  padding: var(--space-2) var(--space-2);
  display: inline-flex;
  align-items: center;
  min-height: 44px;
}

.menu a {
  font-size: var(--fs-ui);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

@media screen and (max-width: 768px) {
  nav {
    padding: var(--space-5) var(--space-4) var(--space-5);
    gap: var(--space-3);
  }

  .menu {
    gap: var(--space-2) var(--space-4);
  }
}
</style>

<nav aria-label="Principal">
  <div class="identity">
    <a class="site-name" href="/" aria-current={isActive('/') ? 'page' : undefined}>Juan Felipe Urueña Calderón</a>
    <ul class="icons">
      <li><a href="https://github.com/juanfuc" target="_blank" rel="noopener noreferrer" aria-label="GitHub"><Icon icon="mdi:github" /></a></li>
      <li><a href="https://x.com/@juuuuffff" target="_blank" rel="noopener noreferrer" aria-label="X (Twitter)"><Icon icon="pajamas:twitter" /></a></li>
      <li><a href="https://unal.academia.edu/JuanFelipeUrue%C3%B1aCalder%C3%B3n" target="_blank" rel="noopener noreferrer" aria-label="Academia.edu"><Icon icon="simple-icons:academia" /></a></li>
      <li><a href="https://www.researchgate.net/profile/Juan-Uruena" target="_blank" rel="noopener noreferrer" aria-label="ResearchGate"><Icon icon="simple-icons:researchgate" /></a></li>
      <li><a href="https://orcid.org/0000-0003-0576-159X" target="_blank" rel="noopener noreferrer" aria-label="ORCID"><Icon icon="simple-icons:orcid" /></a></li>
      <li><a href="https://observablehq.com/@juanfuc" target="_blank" rel="noopener noreferrer" aria-label="Observable"><Icon icon="simple-icons:observable" /></a></li>
    </ul>
  </div>
  <ul class="menu">
    <li><a href="/perfil" aria-current={isActive('/perfil') ? 'page' : undefined}>Perfil</a></li>
    <li><a href="/experiencia" aria-current={isActive('/experiencia') ? 'page' : undefined}>Experiencia</a></li>
    <li><a href="/publicaciones" aria-current={isActive('/publicaciones') ? 'page' : undefined}>Publicaciones</a></li>
    <li><a href="/programacion" aria-current={isActive('/programacion') ? 'page' : undefined}>Programación/HD</a></li>
  </ul>
</nav>

<slot />

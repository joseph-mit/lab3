<script>
  import { base } from "$app/paths";
  import { page } from "$app/stores";

  let pages = [
    { url: "/", title: "Home" },
    { url: "/projects", title: "Projects" },
    { url: "/resume", title: "resume" },
    { url: "/contact", title: "Contact" },
    { url: "https://github.com/joseph-mit", title: "GitHub" }
  ];
</script>

<nav>
  {#each pages as p}
    <a
      href={p.url.startsWith("http") ? p.url : base + p.url}
      target={p.url.startsWith("http") ? "_blank" : null}
      rel={p.url.startsWith("http") ? "noreferrer" : null}
      class:current={
        p.url.startsWith("http")
          ? false
          : p.url === "/"
            ? $page.url.pathname === (base + "/") ||
              $page.url.pathname === base ||
              $page.url.pathname === "/"
            : $page.url.pathname.startsWith(base + p.url)
      }
    >
      {p.title}
    </a>
  {/each}
</nav>

<slot />

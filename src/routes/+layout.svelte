<script>
  import { base } from "$app/paths";
  import { page } from "$app/stores";

  const pages = [
    { url: "/", title: "Home" },
    { url: "/projects", title: "Projects" },
    { url: "/resume", title: "Resume" },
    { url: "/contact", title: "Contact" },
    { url: "https://github.com/joseph-mit", title: "GitHub" }
  ];

  const isExternal = (url) => /^https?:\/\//.test(url);
  const hrefFor = (url) => (isExternal(url) ? url : `${base}${url}`);

  // --- Theme switcher state ---
  let colorScheme = "light dark";
  const storage = globalThis?.localStorage;
  if (storage?.colorScheme) colorScheme = storage.colorScheme;

  $: storage && (storage.colorScheme = colorScheme);
  $: globalThis?.document?.documentElement?.style.setProperty(
    "color-scheme",
    colorScheme
  );
</script>

<!-- DEBUG: remove this line once highlighting works -->
<p style="font-size:12px; color:red;">route.id = {$page.route?.id} | pathname = {$page.url.pathname} | base = {base}</p>

<div class="layout">
  <label class="color-scheme-switch">
    Theme:
    <select bind:value={colorScheme}>
      <option value="light dark">Automatic</option>
      <option value="light">Light</option>
      <option value="dark">Dark</option>
    </select>
  </label>

  <nav aria-label="Primary">
    {#each pages as p}
      {@const active = $page.route?.id === "/" ? "/" : $page.route?.id ?? ""}
      {@const target = isExternal(p.url) ? null : p.url}
      {@const match = target === null
        ? false
        : target === "/"
          ? active === "/"
          : active === target || active.startsWith(target + "/")}

        href={hrefFor(p.url)}
        target={isExternal(p.url) ? "_blank" : null}
        rel={isExternal(p.url) ? "noreferrer" : null}
        class:current={match}
      >
        {p.title}
      </a>
    {/each}
  </nav>

  <slot />
</div>

<style>
  .layout {
    position: relative;
    padding-top: 2.25rem;
  }

  .color-scheme-switch {
    position: absolute;
    top: 0;
    right: 0;
    z-index: 2;
    display: inline-flex;
    gap: 0.5rem;
    align-items: center;
    font-size: 0.9rem;
    padding: 0.25rem 0.5rem;
    border: 1px solid var(--border-gray);
    border-radius: 0.5rem;
    background: canvas;
    color: canvastext;
  }

  .color-scheme-switch select {
    font: inherit;
    background: canvas;
    color: canvastext;
    border: 1px solid var(--border-gray);
    border-radius: 0.4rem;
    padding: 0.15rem 0.35rem;
  }

  nav {
    --border-color: oklch(50% 10% 200 / 40%);
    display: flex;
    justify-content: center;
    gap: clamp(1rem, 4vw, 3.5rem);
    border-bottom: 2px solid var(--border-color);
    margin-bottom: 2rem;
    padding-bottom: 0.3rem;
  }

  nav a {
    display: inline-block;
    padding: 0.5rem 0.75rem;
    text-decoration: none;
    color: inherit;
    border-bottom: 4px solid transparent;
    border-radius: 0.35rem;
  }

  nav a:hover {
    background-color: color-mix(in oklch, var(--color-accent), canvas 85%);
  }

  nav a.current {
    border-bottom-color: var(--border-color);
    font-weight: 600;
  }
</style>

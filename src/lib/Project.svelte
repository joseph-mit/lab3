<script>
  import { base } from "$app/paths";

  export let data = {};
</script>

<article>
  <h2>{data.title}</h2>
  <img
    src={base + data.image}
    alt={data.title}
  />
  <p>{data.description}</p>
</article>

<style>
  article {
    display: grid;
    gap: 0.75rem;
  }

  img {
    display: block;
    width: 100%;
    height: auto;
  }
</style>

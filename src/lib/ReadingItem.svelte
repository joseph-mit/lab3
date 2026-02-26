<script>
  import { base } from "$app/paths";

  export let data = {};
</script>

<div class="reading-item">
  <img
    src={base + data.image}
    alt={data.title}
  />

  <div>
    <h3>{data.title}</h3>
    <p>{data.author}</p>
  </div>
</div>

<style>
  .reading-item {
    display: flex;
    gap: 1rem;
    align-items: flex-start;
  }

  img {
    width: 90px;
    height: auto;
    display: block;
    flex-shrink: 0;
  }

  h3,
  p {
    margin: 0;
  }
</style>

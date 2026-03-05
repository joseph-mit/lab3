<svelte:head>
  <title>Contact</title>
</svelte:head>

<h1>Contact</h1>

<form action="mailto:josephfn@mit.edu" method="GET">
  <label>
    Email:
    <input name="email" type="email" />
  </label>

  <br />

  <label>
    Subject:
    <input name="subject" />
  </label>

  <br />

  <label>
    Message:
    <textarea name="body"></textarea>
  </label>

  <br />

  <button>Send</button>
</form>

<svelte:head>
  <title>Contact</title>
</svelte:head>

<h1>Contact</h1>

<form action="mailto:josephfn@mit.edu" method="GET">
  <label>
    <span>Email:</span>
    <input name="email" type="email" />
  </label>

  <label>
    <span>Subject:</span>
    <input name="subject" />
  </label>

  <label>
    <span>Message:</span>
    <textarea name="body"></textarea>
  </label>

  <button>Send</button>
</form>

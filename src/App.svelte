<script>
  import ChatRoom from "./lib/ChatRoom.svelte";
  import { userName, state } from "./store.js";

  /** @type {HTMLParagraphElement} */
  let warningEl;
  function handleLogin() {
    if ($userName === "") {
      warningEl.textContent = `the username is not valid`;
    } else {
      $state = "Login";
    }
  }
</script>

<!-- <div class="layout">
  <nav>
    <h1>sveltype</h1>
  </nav>
  <main>
    <Game />
  </main>
</div> -->

{#if $state === "Logout"}
  <form class="form" on:submit|preventDefault>
    <label for="user_name"> User Name </label>
    <input type="text" id="user_name" bind:value={$userName} />
    <p bind:this={warningEl} />
    <br />
    <button on:click={handleLogin} style="width:100%" type="submit">Login</button>
  </form>
{/if}

{#if $state === "Login" && $userName !== ""}
  <ChatRoom />
{/if}

<style>
  .form {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    background-color: var(--bg-50);
    padding: 3rem;
    border-radius: 0.3rem;
    gap: 1rem;
  }
  label {
    font-family: "lexend deca", sans-serif;
    color: var(--fg-200);
    letter-spacing: 2px;
  }
  input {
    width: 20rem;
    height: 3rem;
    border-radius: 0.25rem;
    padding: 1rem;
    border: none;
    color: var(--primary);
    font-weight: bold;
    background-color: var(--bg-25);
  }

  input:focus-visible {
    outline: 2px solid var(--fg-100);
  }

  p {
    color: var(--primary);
  }
  /* h1 {
    font-family: "lexend deca", sans-serif;
    font-size: 1.8rem;
    color: var(--fg-200);
    letter-spacing: 2px;
  } */
  /* .layout {
    height: 100%;
    display: grid;
    grid-template-rows: auto 1fr;
    align-items: center;
    padding: 2rem;
  } */
</style>

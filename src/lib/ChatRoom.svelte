<script>
    import { onMount } from "svelte";
    import { userName, state } from "../store";
    import { fly, fade } from "svelte/transition";
    import Game from "./Game.svelte";

    let toast = true;
    onMount(() => {
        setTimeout(() => (toast = false), 1500);
    });

    function handleLogout() {
        $userName = "";
        $state = "Logout";
    }
</script>

{#if toast}
    <p id="toast" in:fly={{ y: -30, duration: 500 }} out:fade>
        Logged in as {$userName}
    </p>
{/if}
    <button on:click={handleLogout} class="logout">Logout</button>
    <Game />

<style>

    .logout{
        position: absolute;
        top:5rem;
    }
    #toast {
        width: 20rem;
        min-width: fit-content;
        height: 4rem;
        font-size: 1.1rem;
        letter-spacing: 0.15rem;
        padding: 1rem;
        color: var(--bg-100);
        background-color: var(--success);
        display: flex;
        align-items: center;
        flex-direction: column;
        border-radius: 0.35rem;
        font-weight: bold;

        position: absolute;
        top: 2rem;
        right: 4rem;
    }
</style>

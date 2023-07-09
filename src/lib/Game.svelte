<script lang="ts">
    import { tweened } from "svelte/motion";
    type Game = "waiting for input" | "in progress" | "game over";
    type Word = string;
    let words: Word[] = "The quick brown fox jumps over a lazy dog. "
        .repeat(2)
        .split(" ");
    let wordIndex = 0;
    let letterIndex = 0;
    let correctLetters = 0;
    let seconds = 30;
    let wordsPerMinute = tweened(0, { delay: 300, duration: 1000 });
    let accuracy = tweened(0, { delay: 1300, duration: 1000 });

    let wordsEl: HTMLDivElement;
    let letterEl: HTMLSpanElement;
    let inputEle: HTMLInputElement;
    let caretEl: HTMLDivElement;

    function getwordsPerMinute() {
        const word = 5;
        const minutes = 0.5;
        return Math.floor(correctLetters / word / minutes);
    }

    function getTotalLetters(words: Word[]) {
        return words.reduce((count, word) => count + word.length, 0);
    }

    function getAccuracy() {
        const totalLetters = getTotalLetters(words);
        return Math.floor((correctLetters / totalLetters) * 100);
    }

    function getResult() {
        $wordsPerMinute = getwordsPerMinute();
        $accuracy = getAccuracy();
    }

    function updateGameState() {
        setLetter();
        checkLetter();
        nextLetter();
        resetLetter();
        updateLine();
        moveCaret();
    }

    function setLetter() {
        const isWordCompleted = letterIndex > words[wordIndex].length - 1;

        if (!isWordCompleted) {
            letterEl = wordsEl.children[wordIndex].children[
                letterIndex
            ] as HTMLSpanElement;
        }
    }

    function checkLetter() {
        const currentLetter = words[wordIndex][letterIndex];

        if (currentLetter === typedLetter) {
            letterEl.dataset.letter = "correct";
            increaseScore();
        }

        if (typedLetter !== currentLetter) {
            letterEl.dataset.letter = "incorrect";
        }
    }

    function increaseScore() {
        correctLetters += 1;
    }

    function nextLetter() {
        letterIndex += 1;
    }

    function resetLetter() {
        typedLetter = "";
    }

    function nextWord() {
        const isNotFristLetter = letterIndex !== 0;
        const isOneLetterWord = words[wordIndex].length === 1;
        const lastLetter =
            wordIndex === words.length - 1 &&
            letterIndex > words[words.length - 1].length;

        if (lastLetter) {
            setGameState("game over");
        }

        if (isNotFristLetter || isOneLetterWord) {
            wordIndex += 1;
            letterIndex = 0;
            increaseScore();
            moveCaret();
            caretEl.style.left = `${caretEl.offsetLeft + 12}px`;
        }
    }

    function moveCaret() {
        caretEl.style.top = `${wordsEl.offsetTop + letterEl.offsetTop}px`;
        caretEl.style.left = `${
            wordsEl.offsetLeft + letterEl.offsetLeft + letterEl.offsetWidth
        }px`;
    }

    function updateLine() {
        const wordEl = wordsEl.children[wordIndex];
        const wordsY = wordsEl.getBoundingClientRect().y;
        const wordY = wordEl.getBoundingClientRect().y;

        if (wordY > wordsY) {
            wordEl.scrollIntoView({ block: "center" });
        }
    }

    function startGame() {
        setGameState("in progress");

        setGameTimer();
    }

    function setGameTimer() {
        function gameTimer() {
            if (seconds > 0) {
                seconds -= 1;
            }

            if (game === "waiting for input" || seconds === 0) {
                clearInterval(intervel);
            }

            if (seconds === 0) {
                setGameState("game over");
                getResult();
            }
        }

        const intervel = setInterval(gameTimer, 1000);
    }

    function setGameState(state: Game) {
        game = state;
    }

    function hanleKeydown(event: KeyboardEvent) {
        if (event.code == "Space") {
            event.preventDefault();
            if (game === "in progress") {
                nextWord();
            }
        }

        if (game === "waiting for input") {
            startGame();
        }
    }

    function resetGame() {
        setGameState("waiting for input");

        seconds = 30;
        typedLetter = "";
        wordIndex = 0;
        letterIndex = 0;
        correctLetters = 0;

        $wordsPerMinute = 0;
        $accuracy = 0;
    }

    let game: Game = "waiting for input";
    let typedLetter = "";
</script>

{#if game !== "game over"}
    <div class="game-arena">
        <div class="game" data-game={game}>
            <div class="time">{seconds}</div>
            <input
                type="text"
                bind:this={inputEle}
                bind:value={typedLetter}
                on:input={updateGameState}
                on:keydown={hanleKeydown}
                class="input"
            />

            <div class="words" bind:this={wordsEl}>
                {#each words as word}
                    <span class="word">
                        {#each word as letter}
                            <span class="letter">{letter}</span>
                        {/each}
                    </span>
                {/each}
            </div>
            <div class="caret" bind:this={caretEl} />
        </div>
        <button on:click={resetGame} style="width: fit-content;"
            >Reset Game</button
        >
    </div>
{/if}

{#if game === "game over"}
    <div class="results">
        <div>
            <p class="title">wpm</p>
            <p class="score">{Math.trunc($wordsPerMinute)}</p>
        </div>

        <div>
            <p class="title">accuracy</p>
            <p class="score">{Math.trunc($accuracy)}%</p>
        </div>

        <button on:click={resetGame}>Play Again</button>
    </div>
{/if}

<style>
    .game-arena {
        display: flex;
        align-items: center;
        justify-content: center;
        flex-direction: column;
    }
    .game[data-game="in progress"] .caret {
        animation: none;
    }
    .game[data-game="in progress"]:focus {
        padding: 2rem;
        border: 1px solid var(--primary);
    }
    .game[data-game="waiting for input"] .caret {
        display: none;
    }

    .game {
        position: relative;
        width: 75%;
    }

    .game[data-game="in progress"] .time {
        opacity: 1;
    }

    .time {
        font-size: 1.5rem;
        color: var(--primary);
        opacity: 0;
        transition: all 0.3s ease;
    }

    input {
        width: 100%;
        position: absolute;
        height: 100%;
        opacity: 0;
        z-index: 4;
        cursor: pointer;
    }
    .caret {
        position: absolute;
        height: 2.2rem;
        top: 0;
        border-right: 1px solid var(--primary);
        animation: caret 1s infinite;
        transition: all 0.2s ease;
    }

    @keyframes caret {
        0%,
        to {
            opacity: 0;
        }
        50% {
            opacity: 1;
        }
    }

    .words {
        --line-height: 0.2rem;
        --lines: 3;

        width: 100%;
        min-width: calc(var(--line-height) * var(--lines) * 1.42);
        display: flex;
        flex-wrap: wrap;
        gap: 0.6rem;
        position: relative;
        font-size: 1.7rem;
        line-height: var(--line-height);
        overflow: hidden;
        user-select: none;

        padding: 2rem;
        padding-left: 0;
        height: 20rem;
        z-index: 3;
    }

    .letter {
        opacity: 0.4;
        transition: all 0.3s ease;
    }

    :global([data-letter="correct"]) {
        opacity: 0.9;
        border-bottom: 2px solid hsla(145, 77%, 70%, 0.815);
    }

    :global([data-letter="incorrect"]) {
        color: var(--primary);
        opacity: 1;
    }

    .results {
        gap: 3rem;
        text-align: center;
    }
    .title {
        font-size: 2rem;
        color: var(--primary);
    }
    .score {
        font-size: 4rem;
        color: var(--primary);
    }
</style>

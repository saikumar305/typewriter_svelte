<script lang="ts">
	import { onMount } from 'svelte';
	import { blur } from 'svelte/transition';
	import { tweened } from 'svelte/motion';

	type Game = 'waiting for input' | 'in progress' | 'game over';
	type Word = String;

	let game: Game = 'waiting for input';
	let seconds: number = 30;
	let typedLetter: String = '';
	let words: Word[] = [];
	let toggleReset = false;

	let wordIndex = 0;
	let letterIndex = 0;
	let correctLetters = 0;
	let wordsPerMinute = tweened(0, { delay: 300, duration: 1000 });
	let accuracy = tweened(0, { delay: 1300, duration: 1000 });

	let wordsEl: HTMLDivElement;
	let letterEl: HTMLSpanElement;
	let inputEl: HTMLInputElement;
	let caretEl: HTMLDivElement;

	function resetGame() {
		toggleReset = !toggleReset;
		setGameState('waiting for input');
		getWords(100);
		typedLetter = '';
		wordIndex = 0;
		letterIndex = 0;
		correctLetters = 0;
		$wordsPerMinute = 0;
		$accuracy;
		inputEl.focus();
	}

	function getWordsPerMinute() {
		const word = 5;
		const minutes = 0.5;
		return Math.floor(correctLetters / word / minutes);
	}
	function getAccuracy() {
		const totalLetters = getTotalLetters(words);
		console.log(words);
		return Math.floor((correctLetters / totalLetters) * 100);
	}

	function getTotalLetters(words: Word[]) {
		return words.reduce((count, word) => count + word.length, 0);
	}

	function getResults() {
		$wordsPerMinute = getWordsPerMinute();
		$accuracy = getAccuracy();
	}

	function setGameState(state: Game) {
		game = state;
	}
	function startGame() {
		console.log('start game');
		setGameState('in progress');
		setGameTimer();
	}

	function setGameTimer() {
		const gameTimer = () => {
			if (seconds > 0) {
				seconds -= 1;
			}
			if (game === 'waiting for input' || seconds === 0) {
				clearInterval(interval);
			}
			if (seconds === 0) {
				setGameState('game over');
				getResults();
			}
		};
		const interval = setInterval(gameTimer, 1000);
	}

	function setLetter() {
		const isWordCompleted = letterIndex > words[wordIndex].length;
		if (!isWordCompleted) {
			letterEl = wordsEl.children[wordIndex].children[letterIndex] as HTMLSpanElement;
		}
	}

	function increaseScore() {
		correctLetters += 1;
	}

	function nextWord() {
		const isNotFirstLetter = letterIndex !== 0;
		const isOneletterWord = words[wordIndex].length === 1;
		if (isOneletterWord || isNotFirstLetter) {
			wordIndex += 1;
			letterIndex = 0;
			increaseScore();
			moveCaret();
		}
	}

	function checkLetter() {
		const currentLetter = words[wordIndex][letterIndex];

		if (typedLetter === currentLetter) {
			letterEl.dataset.letter = 'correct';
			increaseScore();
		}
		if (typedLetter !== currentLetter) {
			letterEl.dataset.letter = 'incorrect';
		}
		console.log({ currentLetter, letterEl });
	}

	function nextLetter() {
		letterIndex += 1;
	}

	function updateLine() {
		const wordEl = wordsEl.children[wordIndex];
		const wordsY = wordsEl.getBoundingClientRect().y;
		const wordY = wordEl.getBoundingClientRect().y;

		if (wordY > wordsY) {
			wordEl.scrollIntoView({ block: 'center' });
		}
	}

	function moveCaret() {
		const { offsetTop, offsetLeft, offsetWidth } = letterEl;
		// added 4 to adjust with the offset of line at the start
		caretEl.style.top = `${offsetTop + 4}px`;
		caretEl.style.left = `${offsetLeft + offsetWidth}px`;
	}

	function resetLetter() {
		typedLetter = '';
	}

	function updateGameState() {
		setLetter();
		checkLetter();
		nextLetter();
		updateLine();
		resetLetter();
		moveCaret();
	}

	function handleKeydown(e: KeyboardEvent) {
		if (e.code === 'Space') {
			e.preventDefault();
			if (game === 'waiting for input') {
				startGame();
			}
			if (game === 'in progress') {
				nextWord();
			}
		}
	}

	async function getWords(limit: number) {
		const response = await fetch(`/api/words?limit=${limit}`);
		words = await response.json();
	}

	onMount(() => {
		getWords(200);
		inputEl.focus();
	});
</script>

{#if game !== 'game over'}
	<div class="game" data-game={game}>
		<input
			bind:this={inputEl}
			bind:value={typedLetter}
			on:input={updateGameState}
			on:keydown={handleKeydown}
			type="text"
			class="input"
		/>
		<div class="timer">{seconds}</div>
		{#key toggleReset}
			<div in:blur|local class="words" bind:this={wordsEl}>
				{#each words as word}
					<span class="word">
						{#each word as letter}
							<span class="letter">{letter}</span>
						{/each}
					</span>
				{/each}
				<div bind:this={caretEl} class="caret" />
			</div>

			<div class="reset">
				<button on:click={resetGame} aria-label="reset">reset</button>
			</div>
		{/key}
	</div>
{/if}

{#if game === 'game over'}
	<div in:blur class="results">
		<div>
			<p class="title">wpm</p>
			<p class="score">{Math.trunc($wordsPerMinute)}</p>
		</div>
		<div>
			<p class="title">accuracy</p>
			<p class="score">{Math.trunc($accuracy)}%</p>
		</div>
		<div on:click={resetGame} class="play">play again</div>
	</div>
{/if}

<style lang="scss">
	.game {
		position: relative;
		display: grid;
		place-items: center;
		width: 100%;
		.input {
			position: absolute;
			opacity: 0;
		}

		.timer {
			position: absolute;
			top: -48px;
			font-size: 1.5rem;
			color: var(--primary);
			opacity: 1;
			transition: all 0.3s ease;

			&[data-game='in progress'] .timer {
				opacity: 1;
			}
		}
		.reset {
			width: 100%;
			display: grid;
			justify-content: center;
			font-size: 1.5rem;
			margin-top: 2rem;
		}
	}

	.words {
		--line-height: 1em;
		--lines: 3;
		width: 100%;
		max-height: calc(var(--line-height) * var(--lines) * 1.42);
		display: flex;
		flex-wrap: wrap;
		gap: 0.6em;
		position: relative;
		font-size: 1.8rem;
		line-height: var(--line-height);
		overflow: hidden;
		user-select: none;

		.letter {
			opacity: 0.4;
			transition: all 0.3 ease;

			&:global([data-letter='correct']) {
				opacity: 0.8;
			}
			&:global([data-letter='incorrect']) {
				opacity: 1;
				color: var(--primary);
			}
		}
	}

	.caret {
		position: absolute;
		height: 1.8rem;
		top: 0;
		border-right: 4px solid var(--primary);
		animation: caret 1s infinite;
		transition: all 0.2s ease;

		@keyframes caret {
			0%,
			to {
				opacity: 0;
			}
			50% {
				opacity: 1;
			}
		}
	}

	.results {
		.title {
			font-size: 2rem;
			color: var(--fg-200);
		}
		.score {
			font-size: 4rem;
			color: var(--primary);
		}
		.play {
			margin-top: 1rem;
			cursor: pointer;
			font-size: 1.5rem;
		}
	}
</style>

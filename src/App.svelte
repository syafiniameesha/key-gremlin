<script lang="ts">
	const durations = [15, 30, 60];

	const wordList = [
		'the', 'quick', 'brown', 'fox', 'jumps', 'over', 'lazy', 'dog',
		'and', 'then', 'runs', 'through', 'quiet', 'forest', 'while',
		'morning', 'sun', 'begins', 'to', 'rise', 'every', 'day',
		'people', 'create', 'new', 'ideas', 'that', 'can', 'change',
		'world', 'with', 'simple', 'actions', 'great', 'things',
		'take', 'time', 'small', 'steps', 'lead', 'to', 'big',
		'results', 'never', 'stop', 'learning', 'because', 'knowledge',
		'makes', 'us', 'stronger', 'than', 'before', 'practice',
		'makes', 'progress', 'patience', 'makes', 'perfect', 'focus',
		'on', 'what', 'matters', 'most', 'today', 'keep', 'moving',
		'forward', 'even', 'when', 'things', 'feel', 'difficult',
		'your', 'future', 'starts', 'with', 'what', 'you', 'do',
		'right', 'now'
	];

	let selectedDuration = 30;
	let words = generateWords(100);
	let typed = '';
	let timeLeft = selectedDuration;
	let started = false;
	let finished = false;
	let interval: ReturnType<typeof setInterval> | null = null;

	function generateWords(count: number): string {
		const result: string[] = [];

		for (let i = 0; i < count; i++) {
			const randomIndex = Math.floor(Math.random() * wordList.length);
			result.push(wordList[randomIndex]);
		}

		return result.join(' ');
	}

	function startTest() {
		if (started || finished) return;

		started = true;

		interval = setInterval(() => {
			timeLeft -= 1;

			if (timeLeft <= 0) {
				finishTest();
			}
		}, 1000);
	}

	function handleInput(event: Event) {
		if (finished) return;

		const target = event.target as HTMLInputElement;

		typed = target.value;

		if (!started && typed.length > 0) {
			startTest();
		}
	}

	function finishTest() {
		if (interval) {
			clearInterval(interval);
			interval = null;
		}

		timeLeft = 0;
		started = false;
		finished = true;
	}

	function restart() {
		if (interval) {
			clearInterval(interval);
			interval = null;
		}

		words = generateWords(100);
		typed = '';
		timeLeft = selectedDuration;
		started = false;
		finished = false;

		setTimeout(() => {
			document.querySelector<HTMLInputElement>('.typing-input')?.focus();
		}, 0);
	}

	function changeDuration(duration: number) {
		selectedDuration = duration;
		restart();
	}

	function calculateCorrectCharacters(): number {
		let correct = 0;

		for (let i = 0; i < typed.length; i++) {
			if (typed[i] === words[i]) {
				correct++;
			}
		}

		return correct;
	}

	function calculateErrors(): number {
		let errors = 0;

		for (let i = 0; i < typed.length; i++) {
			if (typed[i] !== words[i]) {
				errors++;
			}
		}

		return errors;
	}

	function calculateWpm(): number {
		const correctCharacters = calculateCorrectCharacters();

		const elapsedSeconds = finished
			? selectedDuration
			: selectedDuration - timeLeft;

		if (elapsedSeconds <= 0) {
			return 0;
		}

		return Math.round(
			(correctCharacters / 5) / (elapsedSeconds / 60)
		);
	}

	function calculateAccuracy(): number {
		if (typed.length === 0) {
			return 100;
		}

		const correctCharacters = calculateCorrectCharacters();

		return Math.round(
			(correctCharacters / typed.length) * 100
		);
	}

	function getCharacterClass(index: number): string {
		if (index >= typed.length) {
			return '';
		}

		return typed[index] === words[index]
			? 'correct'
			: 'incorrect';
	}
</script>

<svelte:head>
	<title>KeyGremlin — Typing Speed Test</title>
	<meta
		name="description"
		content="Test your typing speed and accuracy with KeyGremlin."
	/>
</svelte:head>

<div class="app">
	<header class="header">
		<div class="brand">
			<img src={`${import.meta.env.BASE_URL}logo.svg`} alt="KeyGremlin" />
			<span>KEYGREMLIN</span>
		</div>

		<div class="duration-selector">
			{#each durations as duration}
				<button
					class:active={selectedDuration === duration}
					onclick={() => changeDuration(duration)}
				>
					{duration}s
				</button>
			{/each}
		</div>
	</header>

	<main class="main">
		{#if !finished}
			<div class="timer">
				{timeLeft}
			</div>

			<div
        class="typing-container"
        onclick={() => document.querySelector<HTMLInputElement>('.typing-input')?.focus()}
      >
				<div class="words">
					{#each words.split('') as character, index}
						<span class={getCharacterClass(index)}>
							{character === ' ' ? '\u00A0' : character}
						</span>
					{/each}

					{#if typed.length < words.length}
						<span class="caret"></span>
					{/if}
				</div>
			</div>

			<input
        class="typing-input"
        bind:value={typed}
        oninput={handleInput}
        onpaste={(event) => event.preventDefault()}
        aria-label="Typing input"
      />

			<div class="live-stats">
				<div class="stat">
					<span class="stat-value">{calculateWpm()}</span>
					<span class="stat-label">WPM</span>
				</div>

				<div class="stat">
					<span class="stat-value">{calculateAccuracy()}%</span>
					<span class="stat-label">ACCURACY</span>
				</div>

				<div class="stat">
					<span class="stat-value">{calculateErrors()}</span>
					<span class="stat-label">ERRORS</span>
				</div>
			</div>
		{:else}
			<div class="result">
				<div class="result-label">
					TEST COMPLETE
				</div>

				<div class="result-wpm">
					{calculateWpm()}
					<span>WPM</span>
				</div>

				<div class="result-stats">
					<div>
						<strong>{calculateAccuracy()}%</strong>
						<span>ACCURACY</span>
					</div>

					<div>
						<strong>{typed.length}</strong>
						<span>CHARACTERS</span>
					</div>

					<div>
						<strong>{calculateErrors()}</strong>
						<span>ERRORS</span>
					</div>
				</div>

				<button
					class="restart-button"
					onclick={restart}
				>
					TRY AGAIN
				</button>
			</div>
		{/if}
	</main>

	<footer class="footer">
		<span>Type fast. Stay focused.</span>

		<button onclick={restart}>
			↻ Restart
		</button>
	</footer>
</div>
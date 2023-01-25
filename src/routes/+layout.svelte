<script lang="ts">
	import '../styles/app.scss';
</script>

<svelte:head>
	<title>Sveltype</title>
</svelte:head>

<div class="layout">
	<nav>
		<h1>Sveltype</h1>
	</nav>
	<main>
		<div class="audio">
			<audio controls autoplay>
				<source src="https://sveltejs.github.io/assets/music/strauss.mp3" />
			</audio>
		</div>
		<slot />
	</main>
</div>

<style lang="scss">
	h1 {
		font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
		font-size: 1.8rem;
		color: var(--fg-200);
		letter-spacing: 2px;
	}
	.layout {
		height: 100%;
		width: 100%;
		display: grid;
		grid-template-rows: auto 1fr;
		align-items: center;
		justify-content: center;
		padding: 2rem;
	}
	.audio {
		opacity: 0.5;
		position: absolute;
		bottom: 0;
		left: 0;
	}
</style>

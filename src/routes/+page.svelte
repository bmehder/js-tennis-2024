<script>
	let isTiebreak = false

	const players = ['Player 1', 'Player 2']

	const gamePointsMap = new Map([
		[0, 15],
		[15, 30],
		[30, 40],
		[40, 0],
	])

	const match = {
		set: 0,
		sets: [
			[5, 5],
			[0, 0],
			[0, 0],
		],
		game: [0, 0],
		tiebreak: [0, 0],
		setWinners: [],
	}

	const resetGameScore = () => {
		match.game[0] = 0
		match.game[1] = 0
		match.tiebreak = [0, 0]
	}

	const checkMatchWinner = () => {
		if (match.set < 2) return
		if (match.setWinners.every(x => x === 0)) console.log('Player 1 wins')
		if (match.setWinners.every(x => x === 1)) console.log('Player 2 wins')
		if (match.setWinners[2] === 0) console.log('Player 1 wins')
		if (match.setWinners[2] === 1) console.log('Player 2 wins')
	}

	const handleTiebreak = x => {
		const winner = x
		const loser = winner === 0 ? 1 : 0

		isTiebreak = true
		match.tiebreak[x]++

		if (
			match.tiebreak[winner] >= 7 &&
			match.tiebreak[loser] <= match.tiebreak[winner] - 2
		) {
			match.sets[match.set][winner]++
			match.set++
			match.setWinners.push(winner)
			checkMatchWinner()
			isTiebreak = false
		}
	}

	const handleSet = winner => {
		const loser = winner === 0 ? 1 : 0
		if (match.sets[match.set].every(x => x === 6)) {
			console.log('Tiebreak!')
			handleTiebreak()
		}

		if (
			(match.sets[match.set][winner] === 7 && match.sets[match.set][loser] <= 5) ||
			(match.sets[match.set][winner] === 6 && match.sets[match.set][loser] <= 4)
		) {
			match.set++
			match.setWinners.push(winner)
			checkMatchWinner()
		}
	}

	const handleClick = x => {
		if (isTiebreak) {
			handleTiebreak(x)
			return
		}

		const isDeuce = match.game[0] === 40 && match.game[1] === 40
		const isAd = match.game.some(x => x === 'Ad')
		const winner = x === 0 ? 0 : 1

		if (isDeuce) {
			match.game[winner] = 'Ad'
			return
		}

		if (isAd) {
			const isPlayer1Ad = match.game[0] === 'Ad'
			const isPlayer2Ad = match.game[1] === 'Ad'

			isPlayer1Ad && winner === 1 && (match.game[0] = 40)
			isPlayer2Ad && winner === 0 && (match.game[1] = 40)

			isPlayer1Ad && winner === 0 && ++match.sets[match.set][0] && resetGameScore()
			isPlayer2Ad && winner === 1 && ++match.sets[match.set][1] && resetGameScore()
			return
		}

		const lastGameScore = match.game[winner]

		match.game[winner] = gamePointsMap.get(lastGameScore)

		if (lastGameScore === 40) {
			++match.sets[match.set][winner]
			resetGameScore()
		}

		handleSet(winner)
	}
</script>

<main>
	<h1>JS Tennis 2024</h1>

	<div class="scoreboard">
		<div></div>
		<div>Set 1</div>
		<div>Set 2</div>
		<div>Set 3</div>
		{#if isTiebreak}
			<div>TB</div>
		{:else}
			<div>Game</div>
		{/if}

		<div>{players[0]}</div>
		{#each match.sets as set}
			<div>{set[0]}</div>
		{/each}
		{#if isTiebreak}
			<div>{match.tiebreak[0]}</div>
		{:else}
			<div>{match.game[0]}</div>
		{/if}

		<div>{players[1]}</div>
		{#each match.sets as set}
			<div>{set[1]}</div>
		{/each}
		{#if isTiebreak}
			<div>{match.tiebreak[1]}</div>
		{:else}
			<div>{match.game[1]}</div>
		{/if}
	</div>

	<div class="buttons">
		<button on:click={() => handleClick(0)}>Player 1</button>
		<button on:click={() => handleClick(1)}>Player 2</button>
	</div>
</main>

<footer>
	<details open>
		<summary>Match State</summary>
		<pre>{JSON.stringify(match, null, 2)}</pre>
	</details>
</footer>

<style>
	:root {
		color-scheme: dark;
	}

	main {
		display: grid;
		gap: 1.5rem;
		text-align: center;

		& .scoreboard {
			max-width: 32rem;
			display: grid;
			grid-template-columns: repeat(5, 1fr);
			gap: 1rem;
			margin-inline: auto;
			text-align: center;
		}

		& .buttons > * {
			padding-block: 0.5rem;
			padding-inline: 1rem;
		}
	}

	footer {
		font-size: 1rem;

		& details {
			margin-block: 1.5rem;
			
			& pre {
				padding: 1.5rem;
			}
		}
	}
</style>

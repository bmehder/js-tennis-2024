<script>
	const scoreboardHeadings = ['', 'Set 1', 'Set 2', 'Set 3', ['TB', 'Game']]

	const gamePointsMappings = new Map([
		[0, 15],
		[15, 30],
		[30, 40],
		[40, 0],
	])

	const matchState = {
		players: ['Player 1', 'Player 2'],
		set: 0,
		sets: [
			[0, 0],
			[0, 0],
			[0, 0],
		],
		game: [0, 0],
		isTiebreak: false,
		tiebreak: [
			[0, 0],
			[0, 0],
			[0, 0],
		],
		setWinners: [],
		matchWinner: null,
	}

	const resetGameScore = () => {
		matchState.game = [0, 0]
	}

	const resetMatch = () => {
		matchState.players = ['Player 1', 'Player 2']
		matchState.set = 0
		matchState.sets = [
			[0, 0],
			[0, 0],
			[0, 0],
		]
		matchState.game = [0, 0]
		matchState.isTiebreak = false
		matchState.tiebreak = [
			[0, 0],
			[0, 0],
			[0, 0],
		]
		matchState.setWinners = []
		matchState.matchWinner = null
	}

	const checkMatchWinner = () => {
		if (matchState.set < 2) return

		const player1 = 0
		const player2 = 1

		const is3rdSetWinner = x => matchState.setWinners[2] === x
		const isWonFirstTwoSets = y => matchState.setWinners.every(x => x === y)

		const setMatchWinner = x => {
			if (is3rdSetWinner(x) || isWonFirstTwoSets(x)) {
				matchState.matchWinner = matchState.players[x]
			}
		}

		;[player1, player2].forEach(setMatchWinner)
	}

	const scoreTiebreak = x => {
		const winner = x
		const loser = winner === 0 ? 1 : 0

		matchState.isTiebreak = true
		matchState.tiebreak[matchState.set][winner]++

		if (
			matchState.tiebreak[matchState.set][winner] >= 7 &&
			matchState.tiebreak[matchState.set][loser] <=
				matchState.tiebreak[matchState.set][winner] - 2
		) {
			matchState.sets[matchState.set][winner]++
			matchState.set++
			matchState.setWinners.push(winner)
			matchState.isTiebreak = false
			checkMatchWinner()
		}
	}

	const scoreSet = x => {
		const isTiebreak = matchState.sets[matchState.set].every(x => x === 6)
		if (isTiebreak) scoreTiebreak()

		const winner = x
		const loser = winner === 0 ? 1 : 0

		const isWinsBy7 =
			matchState.sets[matchState.set][winner] === 7 &&
			matchState.sets[matchState.set][loser] <= 5
		const isWinsBy6 =
			matchState.sets[matchState.set][winner] === 6 &&
			matchState.sets[matchState.set][loser] <= 4

		if (isWinsBy7 || isWinsBy6) {
			matchState.set++
			matchState.setWinners.push(winner)
			checkMatchWinner()
		}
	}

	const scoreGame = x => {
		const isDeuce = matchState.game.every(x => x === 40)
		const isAd = matchState.game.some(x => x === 'Ad')
		const winner = x === 0 ? 0 : 1

		if (isDeuce) {
			matchState.game[winner] = 'Ad'
			return
		}

		if (isAd) {
			const isPlayer1Ad = matchState.game[0] === 'Ad'
			const isPlayer2Ad = matchState.game[1] === 'Ad'

			isPlayer1Ad && winner === 1 && (matchState.game[0] = 40)
			isPlayer2Ad && winner === 0 && (matchState.game[1] = 40)

			isPlayer1Ad &&
				winner === 0 &&
				++matchState.sets[matchState.set][0] &&
				resetGameScore()
			isPlayer2Ad &&
				winner === 1 &&
				++matchState.sets[matchState.set][1] &&
				resetGameScore()
			return
		}

		const lastGameScore = matchState.game[winner]

		if (lastGameScore === 40) {
			++matchState.sets[matchState.set][winner]
			resetGameScore()
		}

		matchState.game[winner] = gamePointsMappings.get(lastGameScore)

		scoreSet(winner)
	}

	const handleClick = x => {
		if (matchState.isTiebreak) {
			scoreTiebreak(x)
			return
		}

		scoreGame(x)
	}
</script>

<div class="js-tennis">
	<header>
		<h1>JS Tennis 2🎾24</h1>
	</header>

	<main>
		<div class="scoreboard">
			{#each scoreboardHeadings as item, idx}
				{@const isLastItem = idx === scoreboardHeadings.length - 1}
				{#if isLastItem && matchState.isTiebreak}
					<div>{item[0]}</div>
				{:else if isLastItem && !matchState.isTiebreak}
					<div>{item[1]}</div>
				{:else}
					<div>{item}</div>
				{/if}
			{/each}

			<div>{matchState.players[0]}</div>
			{#each matchState.sets as set, idx}
				{@const isSetTiebreak =
					matchState.tiebreak[idx].some(x => x !== 0) &&
					typeof matchState.setWinners[idx] === 'number'}
				<div
					class:won={set[0] > matchState.sets[idx][1] &&
						typeof matchState.setWinners[idx] === 'number'}
				>
					{set[0]}
					{#if isSetTiebreak}
						<sup>{matchState.tiebreak[idx][0]}</sup>
					{/if}
				</div>
			{/each}
			{#if matchState.isTiebreak}
				<div>{matchState.tiebreak[matchState.set][0]}</div>
			{:else}
				<div>{matchState.game[0]}</div>
			{/if}

			<div>{matchState.players[1]}</div>
			{#each matchState.sets as set, idx}
				{@const isSetTiebreak =
					matchState.tiebreak[idx].some(x => x !== 0) &&
					typeof matchState.setWinners[idx] === 'number'}
				<div
					class:won={set[1] > matchState.sets[idx][0] &&
						typeof matchState.setWinners[idx] === 'number'}
				>
					{set[1]}
					{#if isSetTiebreak}
						<sup>{matchState.tiebreak[idx][1]}</sup>
					{/if}
				</div>
			{/each}
			{#if matchState.isTiebreak}
				<div>{matchState.tiebreak[matchState.set][1]}</div>
			{:else}
				<div>{matchState.game[1]}</div>
			{/if}
		</div>

		{#if !matchState.matchWinner}
			<div>
				<button on:click={() => handleClick(0)}>Player 1</button>
				<button on:click={() => handleClick(1)}>Player 2</button>
			</div>
		{/if}

		{#if matchState.matchWinner}
			<p>{matchState.matchWinner} wins!</p>
			<button on:click={resetMatch}>Start New Match</button>
		{/if}
	</main>

	<footer>
		<details>
			<summary>Match State</summary>
			<pre>{JSON.stringify(matchState, null, 2)}</pre>
		</details>
	</footer>
</div>

<style>
	.js-tennis {
		display: grid;
		gap: 3rem;
		place-content: center;
		text-align: center;
		font-size: 1.5rem;

		& main {
			display: grid;
			gap: 1.5rem;

			& .scoreboard {
				max-width: 48rem;
				display: grid;
				grid-template-columns: repeat(5, 1fr);
				margin-inline: auto;
				gap: 1.5rem;
				padding: 1.5rem;
				border: 1px solid;

				& sup {
					font-size: 70%;
				}
			}

			& button {
				padding-block: 0.5rem;
				padding-inline: 1rem;
			}
		}

		& footer {
			justify-self: self-start;
			font-size: 1rem;
			text-align: left;

			& details {
				margin-block: 1.5rem;

				& pre {
					padding: 1.5rem;
				}
			}
		}

		& .won {
			color: gold;
		}
	}
</style>

<script>
	const players = ['Player 1', 'Player 2']

  let playerToServe = players[0]

  const gamePointsMap = new Map([
    [0, 15],
    [15, 30],
    [30, 40]
  ])
	
  const match = {
    set1: [0, 0],
		set2: [0, 0],
		set3: [0, 0],
		game: [0, 0],
	}

	const handleClick = x => {
    const winner = x === 'player-1' ? 0 : 1
    const lastGameScore = match.game[winner]

    match.game[winner] = gamePointsMap.get(lastGameScore)

  }
</script>

<pre>{JSON.stringify(match, null, 2)}</pre>

<button on:click={() => handleClick('player-1')}>Player 1</button>
<button on:click={() => handleClick('player-2')}>Player 2</button>

<style>
	:root {
		color-scheme: dark;
	}
</style>

<script>
	export let currentTask = localStorage.getItem('currentTask') || 'nothing'

	export let tasks = JSON.parse(localStorage.getItem('tasks') || '[]') || []

	const formatDate = (d = new Date()) => `${d.getFullYear()}-${d.getMonth()}-${d.getDate()}`

	let savedLines = JSON.parse(localStorage.getItem('savedLines') || '{}') || {};
	let lines = savedLines[formatDate()] || {}

	$: startTime = new Date().getTime() - (lines[currentTask] || 0)

	const formatTime = (interval = 0) => {
		const d = interval || (new Date().getTime() - startTime)
		const h = '' + Math.floor(d / (60*1000*60))
		const m = '' + Math.floor((d - h*60*1000*60) / (1000*60))
		const s = '' + Math.floor((d - h*60*1000*60 - m*1000*60) / 1000)
		return `${h}:${m.padStart(2, '0')}:${s.padStart(2, '0')}`
	}

	export let time = '0:00:00'
	const interval = setInterval(() => {
		time = formatTime()
	}, 1000);

	export let addTaskValue = ''

	export const addTask = () => {
		const newTasks = [...tasks, addTaskValue]
		addTaskValue = ''
		localStorage.setItem('tasks', JSON.stringify(newTasks))
		tasks = newTasks;
	}

	export const changeTask = (currentTaskValue) => {
		if (currentTaskValue === currentTask) return;
		const d = new Date().getTime()
		const timeSpent = d - startTime
		lines[currentTask] = timeSpent
		localStorage.setItem('lines', JSON.stringify(lines))
		localStorage.setItem('currentTask', currentTaskValue)
		currentTask = currentTaskValue
		// startTime = d - (lines[currentTaskValue] || 0)
		time = formatTime(timeSpent)
	}

	const reset = () => {
		localStorage.setItem('lines', '{}')
		lines = {}
	}

	export const save = () => {
		const d = new Date()
		savedLines[formatDate(d)] = lines;
		localStorage.setItem('savedLines', JSON.stringify(savedLines))
	}
</script>

<svelte:head>
	<title>Time Keeper</title>
</svelte:head>

<main>
	<div>
		<h2>You've spent</h2>
		<h1>{time}</h1>
		<h2>on {currentTask}</h2>
	</div>
	<div class="select-task">
		<h2>Select Task</h2>
		{#each tasks as task}
			<div
				class="task"
				class:selected={task === currentTask}
				class:option={task !== currentTask}
				on:click={() => changeTask(task)}
			>
				{task}
			</div>
		{/each}
		<h2>Add Task</h2>
		<input bind:value={addTaskValue} />
		<button on:click={addTask}>Add</button>
	</div>
	<div>
		<h2>Saved Lines</h2>
		<button on:click={save}>Save Lines</button>
		{#each Object.entries(savedLines) as [ date, lines ]}
			<div>{date}</div>
			{#each Object.entries(lines) as entry}
				<div>{entry[0]}: {formatTime(entry[1])}</div>
			{/each}
		{/each}
		<button on:click={reset}>Reset</button>
	</div>
</main>

<style>
	main {
		text-align: center;
	}

	.task {
		display: inline;
		border: solid #ccc 1px;
		padding: 10px;
		margin: 10px;
		border-radius: 10px;
	}

	.task.selected {
		border-color: lightblue;
	}

	.task.option:hover {
		border-color: black;
		cursor: pointer;
	}
</style>
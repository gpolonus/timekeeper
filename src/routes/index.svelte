<script lang="ts">

	// let localStorage = typeof 'localStorage' === 'object'
	// 	? localStorage
	// 	: { getItem: ()=>{}, setItem: ()=>{} }

	let tasks = JSON.parse(localStorage.getItem('tasks') || '[]') || []

	const formatDate = (d = new Date()) => `${d.getFullYear()}-${d.getMonth()}-${d.getDate()}`

	type Line = { start: number, end: number, task: string }
	let savedLines: {[index: string]: Line[]} = JSON.parse(localStorage.getItem('savedLines') || '{}') || {};
	let lines: Line[] = savedLines[formatDate()] || []

	$: line = lines.length && !lines[lines.length - 1].end
			? lines[lines.length - 1]
			: { start: 0, task: '' }
	$: startTime = line.start
	$: currentTask = line.task

	const formatInterval = (startTime: number, endTime = new Date().getTime()) => {
		const d = endTime - startTime
		const h = Math.floor(d / (60*1000*60))
		const m = Math.floor((d - h*60*1000*60) / (1000*60))
		const s = Math.floor((d - h*60*1000*60 - m*1000*60) / 1000)
		return `${h}:${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`
	}

	let time = '0:00:00'
	let totalTime = '0:00:00'
	let intervalId: NodeJS.Timer | null;
	const start = () => {
		intervalId = setInterval(() => {
			time = formatInterval(startTime)
			totalTime = formatInterval(currentTotalStartTime)
		}, 1000);
	}

	let addTaskValue = ''

	const addTask = () => {
		const newTasks = [...tasks, addTaskValue]
		addTaskValue = ''
		localStorage.setItem('tasks', JSON.stringify(newTasks))
		tasks = newTasks;
	}

	const changeTask = (newTaskValue: string) => {
		if (newTaskValue === currentTask) return;
		const d = new Date().getTime()

		if (currentTask) {
			lines[lines.length - 1].end = d
		}

		lines = [
			...lines,
			{ start: d, task: newTaskValue, end: 0 }
		]

		savedLines[formatDate()] = lines;
		localStorage.setItem('savedLines', JSON.stringify(savedLines))

		if (intervalId === null) {
			start()
		}
	}

	const stop = () => {
		time = '0:00:00'
		totalTime = '0:00:00'
		if (currentTask) {
			const d = new Date().getTime()
			lines[lines.length - 1].end = d;
			lines = [ ...lines ]

			savedLines[formatDate()] = lines;
			localStorage.setItem('savedLines', JSON.stringify(savedLines))
		}

		intervalId && clearInterval(intervalId)
		intervalId = null
	}

	function reset() {
		localStorage.setItem('savedLines', '{}')
		savedLines = {}
	}

	const timeSpentOnTask = (ct = currentTask, ls = lines) =>
		ls.filter(l => l.task === ct && l.end).reduce((ac, { start, end } = { start: 0, end: 0, task: '' }) => ac + (end - start), 0)

	$: currentTotalStartTime = (startTime || 0) - timeSpentOnTask(currentTask, lines)
</script>

<svelte:head>
	<title>Time Keeper</title>
</svelte:head>

{(currentTask && start()) || ''}

<main>
	<div>
		{#if currentTask}
			<h2>You've currently spent</h2>
			<h1>{time}</h1>
			<h2>on {currentTask}</h2>
			<h2>and {totalTime} total</h2>
		{:else}
			<h2>You're currently doing nothing</h2>
			<h1>Pick a Task</h1>
		{/if}
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
		{#each Object.entries(savedLines) as [ date, lines ]}
			<div>{date}</div>
			{#each lines as { task, start, end }}
				<div>{task}: </div>
				<div>Difference: {formatInterval(start, end)}</div>
			{/each}
		{/each}
		<button on:click={stop}>Stop</button>
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
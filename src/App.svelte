<script>
	import { quintOut } from "svelte/easing";
	import { crossfade } from "svelte/transition";
	import { flip } from "svelte/animate";

	let time = new Date();
	let hours = 24 - time.getHours();
	let hoursString = hours.toString().padStart(2, "0");

	let minutes = 60 - time.getMinutes();
	let minutesString = minutes.toString().padStart(2, "0");

	let seconds = 60 - time.getSeconds();
	let secondsString = seconds.toString().padStart(2, "0");

	let timeLeft = `${hoursString} : ${minutesString} : ${secondsString}`;

	setInterval(() => {
		time = new Date();
		hours = 24 - time.getHours();
		hoursString = hours.toString().padStart(2, "0");

		minutes = 60 - time.getMinutes();
		minutesString = minutes.toString().padStart(2, "0");

		seconds = 60 - time.getSeconds();
		secondsString = seconds.toString().padStart(2, "0");

		timeLeft = `${hoursString} : ${minutesString} : ${secondsString}`;
	}, 1000);

	const [send, receive] = crossfade({
		fallback(node, params) {
			const style = getComputedStyle(node);
			const transform = style.transform === "none" ? "" : style.transform;

			return {
				duration: 600,
				easing: quintOut,
				css: (t) => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`,
			};
		},
	});

	let todos = [
		{ id: 1, done: false, description: "Docs" },
		{ id: 2, done: false, description: "Learn TS" },
		{ id: 3, done: true, description: "Code" },
		{ id: 4, done: false, description: "OS" },
		{ id: 5, done: false, description: "Python" },
		{ id: 6, done: false, description: "DBMS" },
	];

	let uid = todos.length + 1;

	function add(input) {
		const todo = {
			id: uid++,
			done: false,
			description: input.value,
		};

		if (todo.description) {
			todos = [todo, ...todos];
			input.value = "";
		} else {
			alert("Please enter a todo");
		}

		if (todo.description.length > 25) {
			todo.description = todo.description.slice(0, 25) + "...";
		} else {
			todo.description = todo.description;
		}
	}

	function remove(todo) {
		todos = todos.filter((t) => t !== todo);
	}
</script>

<main>
	<h1 class="time">{timeLeft}</h1>

	<div class="board">
		<input
			class="new-todo"
			placeholder="Any Plans ?"
			on:keydown={(event) => event.key === "Enter" && add(event.target)}
		/>

		<div class="left">
			<h2>Todo</h2>
			{#each todos.filter((t) => !t.done) as todo (todo.id)}
				<label
					in:receive={{ key: todo.id }}
					out:send={{ key: todo.id }}
					animate:flip
				>
					<input type="checkbox" bind:checked={todo.done} />
					{todo.description}
					<button on:click={() => remove(todo)}>x</button>
				</label>
			{/each}
		</div>

		<div class="right">
			<h2>Done</h2>
			{#each todos.filter((t) => t.done) as todo (todo.id)}
				<label
					in:receive={{ key: todo.id }}
					out:send={{ key: todo.id }}
					animate:flip
				>
					<input type="checkbox" bind:checked={todo.done} />
					{todo.description}
					<button on:click={() => remove(todo)}>x</button>
				</label>
			{/each}
		</div>
	</div>
</main>

<style>
	.new-todo {
		font-size: 1.4em;
		width: 100%;
		margin: 2em 0 1em 0;
	}

	.board {
		max-width: 36em;
		margin: 0 auto;
	}

	.left,
	.right {
		float: left;
		width: 50%;
		padding: 0 1em 0 0;
		box-sizing: border-box;
	}

	h2 {
		font-size: 2em;
		font-weight: 200;
		user-select: none;
	}

	label {
		top: 0;
		left: 0;
		display: block;
		font-size: 1em;
		line-height: 1;
		padding: 0.5em;
		margin: 0 auto 0.5em auto;
		border-radius: 2px;
		background-color: rgb(255, 255, 255);
		user-select: none;
		color: black;
		text-align: left;
	}

	input {
		margin: 0;
	}

	.right label {
		background-color: rgb(180, 240, 100);
	}

	button {
		float: right;
		height: 1em;
		box-sizing: border-box;
		padding: 0 0.5em;
		line-height: 1;
		background-color: transparent;
		border: none;
		color: rgb(170, 30, 30);
		opacity: 0;
		transition: opacity 0.2s;
	}

	label:hover button {
		opacity: 1;
	}
</style>

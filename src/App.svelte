<script>
	import DataTable from './DataTable.svelte'
    import {onMount} from "svelte"

	export let name;

	let cols = [
		{id: 'id', label: 'ID', editable: false, visible: false},
		{id: 'name', label: 'Name', editable: false, sortable: true},
		{id: 'email', label: 'Email', editable: false},
		{id: 'jobTitle', label: 'Job Title', editable: true, sortable: true},
		{id: 'checkbox', label: 'Checkbox', editable: true, sortable: true, type: 'checkbox'}
	];

	let data = [
		[1, 'John Smith', 'john.smith@demo.com', 'Director', false],
		[2, 'Hugo Boss', 'hugo.boss@demo.com', 'Manager', false],
		[3, 'Jacinta Kelly', 'jacinta.kelly@demo.com', 'Manager', false],
		[4, 'Groundskeeper Willy', 'bill.scotsman@demo.com', 'Caretaker', false]
	]

	let gridManager;

	let printSelected = (event) => {
		console.log(data.filter(n => n[4] === true));
	}

    onMount(async() => {
		gridManager.on('update', (row, col, updatedValue) => {
			console.log(row);
			console.log(col);
			console.log(updatedValue);
		});

		gridManager.on('sort', (colIndex, column) => {
			console.log(colIndex);
			console.log(column);
		});
	});
</script>

<main>
	<DataTable columns={cols} rows={data} bind:manager={gridManager} />
	<br> 
	<button on:click={() => console.log(gridManager)}>Data to console</button>
	<br>
	<button on:click={printSelected}>Print Selected</button>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
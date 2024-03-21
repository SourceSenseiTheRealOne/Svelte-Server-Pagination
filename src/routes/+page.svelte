<script>
	import { onMount, setContext } from 'svelte';
	import dotenv from 'dotenv';

	dotenv.config();

	let API_URL = process.env.API_URL;

	export let params;

	let stations = [];
	let currentPage = 1;
	let pageSize = 15;
	let totalResults = 0;
	let totalPages = 0;
	let currentServerPage = 1;
	let initialServerPage = 1;

	$: updateInfoFromServer = async () => {
		await fetchStations();
	};

	const fetchStations = async () => {
		try {
			const response = await fetch(`${API_URL}?PageSize=250&Page=${currentServerPage}`);

			const data = await response.json();

			if (data) {
				totalResults = parseInt(response.headers.get('X-Total-Count'));
				totalPages = Math.ceil(totalResults / pageSize);

				if (currentServerPage === 1) {
					stations = data;
				} else {
					data.forEach((result) => {
						if (!stations.find((station) => station.id === result.id)) {
							stations.push(result);
						}
					});
				}

				console.log(stations, 'stations');
				console.log(totalResults, 'total results');
				console.log(response.headers.get('cache-control'), 'cache-control');
			}

			return data;
		} catch (error) {
			console.log(error);
		}
	};
	const goToPage = async (page) => {
		if (page > 0 && page <= totalPages) {
			currentPage = page;
			const serverPageToFetch = Math.ceil((currentPage * pageSize) / 250);
			currentServerPage = serverPageToFetch; // current Server Page

			console.log(currentPage, 'current page');
			console.log(currentServerPage, 'server page');

			if (currentServerPage < initialServerPage || currentServerPage > initialServerPage) {
				await updateInfoFromServer();
				initialServerPage = currentServerPage;
			}
		}
	};

	onMount(async () => {
		await fetchStations();
	});
</script>

<svelte:head>
	<link
		rel="stylesheet"
		href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/css/bootstrap.min.css"
	/>

	<script src="https://cdn.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.slim.min.js"></script>

	<script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.1/dist/umd/popper.min.js"></script>

	<script
		src="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/js/bootstrap.bundle.min.js"
	></script>
</svelte:head>

<div class="container">
	<div class="jumbotron">
		<h1>Paragens</h1>
	</div>

	{#if stations.length > 0}
		{#each stations.slice((currentPage - 1) * pageSize, currentPage * pageSize) as station}
			<div class="card">
				<div class="card-body">
					<h5 class="card-title">{station.name}</h5>
					<p class="card-text">{station.latitude} | {station.longitude}</p>
				</div>
			</div>
		{/each}
	{:else}
		<p>Nenhuma estação encontrada.</p>
	{/if}

	<div class="text-center mt-3">
		Página {currentPage} de {totalPages} (registros {(currentPage - 1) * pageSize + 1} - {Math.min(
			currentPage * pageSize,
			totalResults
		)} de {totalResults} resultados)
	</div>

	<div class="d-flex justify-content-center mt-4">
		<ul class="pagination">
			<li class="page-item">
				<a class="page-link" href="#" on:click|preventDefault={() => goToPage(currentPage - 1)}
					>&laquo;</a
				>
			</li>

			{#if totalPages <= 7}
				{#each Array.from({ length: totalPages }, (_, i) => i + 1) as page}
					<li class="page-item {currentPage === page ? 'active' : ''}">
						<a class="page-link" href="#" on:click|preventDefault={() => goToPage(page)}>{page}</a>
					</li>
				{/each}
			{:else}
				{#if currentPage - 3 > 1}
					<li class="page-item">
						<a class="page-link" href="#" on:click|preventDefault={() => goToPage(1)}>{1}</a>
					</li>
					{#if currentPage - 3 > 2}
						<li class="page-item disabled"><span class="page-link">...</span></li>
					{/if}
				{/if}

				{#each Array.from({ length: 7 }, (_, i) => currentPage - 3 + i).filter((page) => page > 0 && page <= totalPages) as page}
					<li class="page-item {currentPage === page ? 'active' : ''}">
						<a class="page-link" href="#" on:click|preventDefault={() => goToPage(page)}>{page}</a>
					</li>
				{/each}

				{#if currentPage + 3 < totalPages}
					{#if currentPage + 3 < totalPages - 1}
						<li class="page-item disabled"><span class="page-link">...</span></li>
					{/if}
					<li class="page-item">
						<a class="page-link" href="#" on:click|preventDefault={() => goToPage(totalPages)}
							>{totalPages}</a
						>
					</li>
				{/if}
			{/if}

			<li class="page-item">
				<a class="page-link" href="#" on:click|preventDefault={() => goToPage(currentPage + 1)}
					>&raquo;</a
				>
			</li>
		</ul>
	</div>
</div>

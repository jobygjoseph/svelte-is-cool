<script>
	import { each } from "svelte/internal";
	import { Out, query, setupClient } from "svql";
	import Grid from "svelte-grid-responsive";
	import AIMS_FLAVORS from "./aimsFlavors";
	import Row from "./Row.svelte";

	const aimsFlavors = Object.entries(AIMS_FLAVORS);
	let imageID = "rcim160";

	setupClient({
		url: "https://uat.newsdigitalapi.com/",
	});

	const GET_IMAGE_INFO = `
    query getImages($imageID: ID!) {
			CloudinaryImage: image(id: $imageID) {
				id
				url(preferredImageService: CLOUDINARY) {
					primary
				}
			}
			AimsImage: image(id: $imageID) {
				id
				url(preferredImageService: AIMS) {
					primary
				}
			}
		}
  `;

	$: query(GET_IMAGE_INFO, { imageID });
</script>

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

<main>
	<h1>Cloundinary VERSUS AIMS!!</h1>
	<input bind:value={imageID} />
	<Out nostatus from={GET_IMAGE_INFO} let:data>
		<Grid container gutter={6} columns={3}>
			{#each aimsFlavors as flavor}
				<Row {flavor} {data} />
			{/each}
		</Grid>
	</Out>
</main>

<script lang="ts">
	import { GQL_AddStar, type userBestRepoInfo$data } from '$houdini';
	import html2canvas from 'html2canvas';
	import GhImg from '../GhImg/GhImg.svelte';
	import GhRepoLanguages from '../GhRepoLanguages/GhRepoLanguages.svelte';
	import GhStar from '../star/GhStar.svelte';

	export let userBestRepoInfo: userBestRepoInfo$data | null = null;

	async function wrongAdd(id: string) {
		await GQL_AddStar.mutate({
			variables: { id },
			optimisticResponse: {
				addStar: {
					clientMutationId: 'From KitQL',
					starrable: {
						__typename: 'Repository',
						id,
						viewerHasStarred: true,
						stargazers: { totalCount: 999 }
					}
				}
			}
		});
	}

	async function dl() {
		const canvas = await html2canvas(document.querySelector('#card'), {
			backgroundColor: null,
			allowTaint: true,
			useCORS: true,
			logging: true
		});
		canvas.style.display = 'none';
		document.body.appendChild(canvas);
		const image = canvas.toDataURL('image/png').replace('image/png', 'image/octet-stream');
		const a = document.createElement('a');
		a.setAttribute('download', `gh-template.png`);
		a.setAttribute('href', image);
		a.click();
	}
</script>

{#if userBestRepoInfo?.repositories?.nodes?.length > 0}
	<div id="card" class="card">
		<div class="container">
			<div class="row">
				<a
					style="color: inherit;text-decoration: inherit;"
					target="_blanck"
					href={`https://github.com/${userBestRepoInfo.login}/${userBestRepoInfo.repositories.nodes[0].name}`}
					>{userBestRepoInfo.login} / <b>{userBestRepoInfo.repositories.nodes[0].name}</b></a
				>
				<div>
					<GhImg userInfo={userBestRepoInfo} />
				</div>
			</div>
			<div class="desc">
				{userBestRepoInfo.repositories.nodes[0].description ?? 'No description'}
			</div>
			<div class="row">
				<GhStar
					id={userBestRepoInfo.repositories.nodes[0].id}
					starInfo={userBestRepoInfo.repositories.nodes[0]}
				/>
			</div>

			<GhRepoLanguages languagesInfo={userBestRepoInfo.repositories.nodes[0]} />
		</div>
	</div>

	<div class="dlBtn">
		<button on:click={() => wrongAdd(userBestRepoInfo.repositories.nodes[0].id)}
			>Wrong Optimistic UI ADD</button
		>
		<button on:click={dl}>Download</button>
	</div>
{:else}
	No data!
{/if}

<style>
	.container {
		height: 250px;
		padding: 30px 40px 40px 40px;
	}

	.desc {
		font-style: italic;
		font-size: small;
		font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
	}

	.dlBtn {
		margin-top: 40px;
		text-align: center;
		width: 100%;
	}

	.row {
		height: 45%;
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		gap: 40px;
	}

	.card {
		margin: 0px auto 0px auto;
		width: 640px;
		height: 320px;
		border-radius: 15px;
		background-color: #babbbd;
		color: #0d1117;
		font-size: xx-large;
		font-family: 'Courier New', Courier, monospace;
		box-shadow: rgba(186, 187, 189, 0.4) 0px 2px 4px, rgba(186, 187, 189, 0.3) 0px 7px 13px -3px,
			rgba(186, 187, 189, 0.2) 0px -3px 0px inset;
	}
</style>

<script>
	import { currentPage } from './router.js';
	import Cart from './pages/Cart.svelte';
	import Investors from './pages/Investors.svelte';
	
	let products = [];
	let investors = [];

	$: total = products.reduce((sum, product) => 
		sum + (product.sellingPrice * product.quantity), 0);

	function handleInvestorsUpdate(event) {
		investors = event.detail;
	}
</script>

<main>
	<header>
		<h1>Business Management System</h1>
		<nav>
			<button 
				class="nav-btn" 
				class:active={$currentPage === 'cart'} 
				on:click={() => currentPage.set('cart')}
			>
				Cart
			</button>
			<button 
				class="nav-btn" 
				class:active={$currentPage === 'investors'} 
				on:click={() => currentPage.set('investors')}
			>
				Investors
			</button>
		</nav>
	</header>

	<div class="content">
		{#if $currentPage === 'cart'}
			<Cart bind:products bind:total />
		{:else if $currentPage === 'investors'}
			<Investors 
				{products} 
				{total} 
				bind:investors
			/>
		{/if}
	</div>
</main>

<style>
	main {
		max-width: 1200px;
		margin: 0 auto;
		padding: 2em;
		min-height: 100vh;
		background-color: #fff;
	}

	header {
		text-align: center;
		margin-bottom: 3em;
		padding-bottom: 1em;
		border-bottom: 2px solid #ff5e28;
	}

	h1 {
		color: #ff5e28;
		text-transform: uppercase;
		font-size: 2.5em;
		font-weight: 300;
		margin-bottom: 1em;
		letter-spacing: 2px;
	}

	nav {
		display: flex;
		justify-content: center;
		gap: 2em;
	}

	.nav-btn {
		font-size: 1.2em;
		padding: 0.8em 2em;
		background-color: #f9f9f9;
		border: 2px solid #ff5e28;
		border-radius: 8px;
		color: #333;
		cursor: pointer;
		transition: all 0.3s ease;
	}

	.nav-btn:hover {
		background-color: #ff7f50;
		color: white;
		transform: translateY(-2px);
	}

	.nav-btn.active {
		background-color: #ff5e28;
		color: white;
		box-shadow: 0 4px 8px rgba(255, 94, 40, 0.2);
	}

	.content {
		background-color: #ffffff;
		border-radius: 12px;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
		padding: 2em;
	}

	@media (max-width: 768px) {
		main {
			padding: 1em;
		}

		h1 {
			font-size: 2em;
		}

		nav {
			gap: 1em;
		}

		.nav-btn {
			font-size: 1em;
			padding: 0.6em 1.5em;
		}
	}
</style>
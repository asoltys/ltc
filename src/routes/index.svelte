<script>
	import { onMount } from 'svelte';

	import qrcode from 'qrcode-generator-es6';

	let img;
	let buying;
	let selling;
	let address = 'M9rSM2qFg8o81tNKbtvpavBJMEhfYKY6oi';
	let amount = 0.01;
	let received = 0;
	let SATS = 100000000;
	let ws;
	let asset;
	let to = 'AzpuQjLb2GbM37S6MiYM4CBVaLK7drpqqMqFUqwimGoStSrB5SgBGeD4JrTczVPmv4bUjcFmQdavUMh8';
  let txid;

	onMount(() => {
    ws = new WebSocket(`wss://ltc.coinos.io/ws`);
		ws.onopen = () => {
			ws.send(JSON.stringify({ type: 'subscribe', value: address }));
		};

		ws.onmessage = ({ data }) => {
			try {
				let { type, value } = JSON.parse(data);

				if (type === 'payment') {
					buying = false;
					received += value;
				}

				if (type === 'asset') {
					asset = value;
				}

				if (type === 'txid') {
          received = false;
					txid = value;
				}
			} catch (e) {}
		};
	});

	let sell = () => {
		asset = false;
		selling = true;
	};

	let buy = () => {
		const qr = new qrcode(0, 'H');
		selling = false;
		buying = true;
		qr.addData(`litecoin:${address}`);
		qr.make();
		img = qr.createSvgTag({});
	};

	let mint = () => {
		ws.send(JSON.stringify({ type: 'mint' }));
	};

  let send = () => {
    ws.send(JSON.stringify({ type: 'send', value: to, asset }));
  } 
</script>

<main>
	{#if asset}
		<div>Asset created!</div>

		<div>Id: {asset}</div>

		<div style="margin-top: 10px">Ready to sell it?</div>
		<button on:click={sell}>Yes let's go</button>
	{:else if selling}
		<p>How much do you want for it?</p>
		<div>
			<input bind:value={amount} /> LTC
		</div>
		<button on:click={buy}>List it</button>
	{:else if buying}
		<div>Send {amount} LTC to:</div>
		<div class="qr">
			{@html img}
			<div>
				{address}
			</div>
		</div>
	{:else if received}
		<p>
			Received {(received / SATS).toFixed(8)} LTC!
		</p>

		<p>Where do you want the token sent?</p>
		<div>
			<input bind:value={to} />
		</div>
		<button on:click={send}>Send it</button>
  {:else if txid}
    <p>Token has been sent!</p>
    <p>Txid: {txid}</p>
	{:else}
		<img src="/chicken.jpg" alt="Chicken" />
		<div style="margin-top: 10px">Ready to mint?</div>
		<button on:click={mint}>Yes, Mint it</button>
	{/if}
</main>

<style>
	.received {
		margin-top: 10px;
		font-size: 1.2em;
	}

	.qr {
		margin: auto;
		width: 300px;
	}

	img {
		width: 300px;
	}

	main {
		text-align: center;
		padding: 1em;
		margin: 0 auto;
	}

	input {
		padding: 10px;
	}

	button {
		background: none;
		border: 1px solid #ff3e00;
		border-radius: 5px;
		color: #ff3e00;
		cursor: pointer;
		text-transform: uppercase;
		font-size: 1.25rem;
		padding: 10px 20px;
		font-weight: 100;
		line-height: 1.1;
		margin: 4rem auto;
		max-width: 14rem;
	}

	p {
		max-width: 14rem;
		margin: 2rem auto;
		line-height: 1.35;
	}

	@media (min-width: 480px) {
		h1 {
			max-width: none;
		}

		p {
			max-width: none;
		}
	}
</style>

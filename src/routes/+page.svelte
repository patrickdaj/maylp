<script>
	//  ___  ___  _____   _____    _     ______   _____ _               _             
	//  |  \/  | / _ \ \ / / _ \  | |    | ___ \ /  __ \ |             | |            
	//  | .  . |/ /_\ \ V / /_\ \ | |    | |_/ / | /  \/ |__   ___  ___| | _____ _ __ 
	//  | |\/| ||  _  |\ /|  _  | | |    |  __/  | |   | '_ \ / _ \/ __| |/ / _ \ '__|
	//  | |  | || | | || || | | | | |____| |     | \__/\ | | |  __/ (__|   <  __/ |   
	//  \_|  |_/\_| |_/\_/\_| |_/ \_____/\_|      \____/_| |_|\___|\___|_|\_\___|_|                            //                                                               by FamiliarCow
	//
	//Select your pool and input your wallet address to get started
	  
	
	import { onMount, tick } from "svelte";
	  import { fade } from "svelte/transition";
	
	  let pools = [];
	  let selectedPool = "THOR.RUNE";
	  let walletAddress = "";
	  let result = {};
	  let usdtPool = "ETH.USDT-0XDAC17F958D2EE523A2206206994597C13D831EC7";
	
	  onMount(async () => {
		const response = await fetch("https://mayanode.mayachain.info/mayachain/pools");
		const data = await response.json();
		pools = data;
	  });
	
	async function checkPosition() {
	  // asset fetch
	  const response = await  fetch(`https://mayanode.mayachain.info/mayachain/pool/${selectedPool}/liquidity_providers`);
	  const data = await response.json();
	  const poolData = pools.find(pool => pool.asset === selectedPool);
	  const entry = data.find(item => item.asset_address === walletAddress);
		
	  // get decimals
	  const decimals = pools.find(item => item.asset === selectedPool).decimals;
		
	  // pool data fetch
	  const poolsResponse = await fetch("https://mayanode.mayachain.info/mayachain/pools");
	  const currentPoolData = await poolsResponse.json();
	  const usdtPoolData = currentPoolData.find(item => item.asset === usdtPool);
      const assetPoolData = currentPoolData.find(item => item.asset == selectedPool);
	
	  if (entry && usdtPoolData) {
		// asset
		const units = BigInt(entry.units);
		const poolUnits = BigInt(poolData.pool_units);
		const cacaoDepth = BigInt(poolData.balance_cacao);
		const assetDepth = BigInt(poolData.balance_asset);
			
		// current balances
		const cacaoBalance = Number(units * cacaoDepth / poolUnits) / 1e10;
		const assetBalance = Number(units * assetDepth / poolUnits) / (10 ** decimals);
			
		// deposit balance
		const cacaoDepositBalance = Number(BigInt(entry.cacao_deposit_value)) / 1e10;
		const assetDepositBalance = Number(BigInt(entry.asset_deposit_value)) / (10 ** decimals);

		// ratios
		let usdtCacaoRatio = (usdtPoolData.balance_asset / usdtPoolData.balance_cacao);
		const assetCacaoRatio = assetPoolData.balance_asset / assetPoolData.balance_cacao;
		const assetUsdtRatio = usdtCacaoRatio / assetCacaoRatio;
		usdtCacaoRatio *= 100;
			
		// usdt values
		const hodlValue = usdtCacaoRatio * cacaoDepositBalance + assetUsdtRatio * assetDepositBalance;
		const lpValue = usdtCacaoRatio * cacaoBalance + assetUsdtRatio * assetBalance;
        const lpVsHodlPercent = ((lpValue / hodlValue) - 1) * 100;
	
		result = {
				cacao_deposit_value: cacaoDepositBalance.toString(),
				asset_deposit_value: assetDepositBalance.toString(),
				cacao_balance: cacaoBalance.toString(),
				asset_balance: assetBalance.toString(),
				hodl_value: hodlValue.toString(),
				lp_value: lpValue.toString(),
                lp_vs_hodl: lpVsHodlPercent.toString()					 
		};
	  } else {
		result = { error: "Address not found" };
	  }
	}
	
		
		function formatValue(num) {
	  const match = num.toString().match(/^-?\d+(\.\d{0,5})?/);
	  if (match) {
		const [integerPart, decimalPart] = match[0].split('.');
		const formattedInteger = integerPart.toLocaleString('en');
		if (decimalPart) {
		  const paddedDecimal = decimalPart.padEnd(5, '0');
		  return `${formattedInteger}.${paddedDecimal}`;
		} else {
		  return formattedInteger;
		}
	  }
	  return num;
	}
	
	
	</script>
	
	<style>
	  * {
		box-sizing: border-box;
	  }
	
	
	  main {
		background-color: #1c1c1c;
		border-radius: 8px;
		box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
		padding: 24px;
		max-width: 500px;
		width: 100%;
	  }
	
	main {
	  background-color: #1c1c1c;
	  border-radius: 24px;
	  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
	  padding: 24px;
	  max-width: 500px;
	  width: 100%;
	}
	
	.header {
	  display: flex;
	  justify-content: center;
	  align-items: center;
	  margin-bottom: 224px;
	}
	
	.logo {
	  max-width: 150px;
	  height: auto;
	  margin-right: 8px; /* Change the margin-right to 8px */
	}
	
	.liquidity-checker {
	  font-size: 1.5rem;
	  margin: 0;
	}
	
	
	  h1 {
		font-size: 1.5rem;
		margin-bottom: 24px;
		text-align: center;
		color: #fff;
	  }
	
	  label {
		display: block;
		font-size: 0.875rem;
		font-weight: 500;
		margin-bottom: 4px;
		color: #ccc;
	  }
	
	  input, select, button {
		display: block;
		width: 100%;
		font-family: inherit;
		font-size: 1rem;
		border: 1px solid rgba(255, 255, 255, 0.2);
		outline: none;
		background-color: #2c2c2c;
		color: #ccc;
		padding: 8px;
		border-radius: 4px;
		margin-bottom: 16px;
	  }
	
	  button {
		background-color: #2cbe8c;
		color: #ffffff;
		font-weight: 500;
		border: none;
		cursor: pointer;
		transition: background-color 0.2s;
	  }
	
	  button:hover {
		background-color: #3DA17E;
	  }
	
	  .results {
		display: flex;
		flex-wrap: wrap;
		justify-content: space-between;
	  }
	
	  .result-box {
		background-color: #2c2c2c;
		padding: 16px;
		border-radius: 4px;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		width: 48%;
		margin-bottom: 16px;
		text-align: center;
		color: #ccc;
	  }
	
	  .error {
		color: #d93025;
	  }
	  
	  .results {
		display: flex;
		flex-direction: column;
		align-items: center;
	  }
	
	  .result-row {
		display: flex;
		justify-content: space-between;
		width: 100%;
		max-width: 480px;
		gap: 16px;
	  }
	
	  .result-box {
		background-color: #2c2c2c;
		padding: 16px;
		border-radius: 4px;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		flex: 1;
		text-align: center;
		color: #ccc;
	  }
	
	  /* Responsive style for smaller screens */
	  @media (max-width: 480px) {
		.result-row {
		  flex-direction: column;
		}
	  }
		
		.header {
	  text-align: center;
	}
	
	.logo {
	  max-width: 150px;
	  height: auto;
	  display: block;
	  margin: 0 auto px auto; /* Change the bottom margin to 2px */
	}
	
	
	h1 {
	  font-size: 1.5rem;
	  margin: 0;
	  padding-top: 2px;
	  text-align: center;
	}
	
		.header {
	  display: flex;
	  flex-direction: column;
	  align-items: center;
	  margin-bottom: 24px;
	}
	
	.header-wrapper {
	  display: flex;
	  justify-content: center;
	  align-items: center;
	}
	
	.logo {
	  max-width: 150px;
	  height: auto;
	  margin-right: 2px;
	}
	
	.liquidity-checker {
	  font-size: 1.5rem;
	  margin: -2px 0 0 0; /* Move the text up 2 pixels on the y-axis */
	}
	main {
      position: absolute;
      top: 50%;
      left: 50%;
      -moz-transform: translateX(-50%) translateY(-50%);
      -webkit-transform: translateX(-50%) translateY(-50%);
      transform: translateX(-50%) translateY(-50%);
    }
	
	
	</style>
	
	<main>
	  <div class="header">
		<div class="header-wrapper">
		  <img src="https://assets.website-files.com/62a14669b65c6aeed054f32e/62a14669b65c6a622e54f3bc_LogoMaya-p-500.png" alt="Maya Logo" class="logo">
		  <h1 class="liquidity-checker">&nbsp;Liquidity Checker</h1>
		</div>
	  </div>
	  <label for="pool-selector">Select A Pool</label>
	  <select id="pool-selector" bind:value={selectedPool}>
		{#each pools as pool (pool.asset)}
	<option value={pool.asset}>{pool.asset.split('.')[1].split('-')[0]}</option>
	{/each}
	</select>
	<label for="wallet-address">Wallet Address</label>
	<input id="wallet-address" type="text" bind:value={walletAddress} placeholder="Enter {selectedPool.split('.')[1].split('-')[0]} wallet address" />
	<div style="height: 10px"></div>
	<button on:click={checkPosition}>Check Position</button>
	
	 <div class="results">
		{#if result.error}
		  <p class="error">{result.error}</p>
		{/if}
		{#if result.cacao_deposit_value && result.asset_deposit_value && result.asset_balance && result.cacao_balance}
		  <div class="result-row">
			<div class="result-box">
			  <p><b>Deposited Value</b></p>
			  <p>{formatValue(result.asset_deposit_value)} {selectedPool.split('.')[1].split('-')[0]}</p>
			  <p>{formatValue(result.cacao_deposit_value)} CACAO</p>
			</div>
			
			<div class="result-box">
			  <p><b>Current LP Balance</b></p>
			  <p>{formatValue(result.asset_balance)} {selectedPool.split('.')[1].split('-')[0]}</p>
			  <p>{formatValue(result.cacao_balance)} CACAO</p>
			</div>
		  </div>
		  <div class="result-row">
			<div class="result-box">
			  <p><b>HODL Value</b></p>
			  <p>{formatValue(result.hodl_value)} USD</p>
			</div>
			
			<div class="result-box">
			  <p><b>LP Value</b></p>
			  <p>{formatValue(result.lp_value)} USD</p>
              <p style="color: {result.lp_vs_hodl < 0 ? 'red': 'green'}"><bold>{formatValue(result.lp_vs_hodl)} %</bold></p>
			</div>
		  </div>
		{/if}
	  </div>
	</main>

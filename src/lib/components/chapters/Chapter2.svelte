<script lang="ts">
  import { costData } from '$lib/data/sampleData';

  type CostBreakdownItem = { name: string; cost: number };

  let mode: 'car' | 'transit' | 'bike' = 'car';
  let miles: number = 10;
  let totalCost: number = 0;
  let breakdown: CostBreakdownItem[] = [];

  $: calculateCosts(mode, miles);

  function calculateCosts(selectedMode: string, distance: number): void {
    const annualMiles = distance * 2 * 5 * 50;
    const costs = costData[selectedMode as keyof typeof costData];
    
    totalCost = 0;
    breakdown = [];
    
    if (costs && costs.components) {
      costs.components.forEach((component) => {
        let cost = 0;
        if ('fixed' in component && component.fixed) {
          cost = component.fixed;
        } else if ('perMile' in component && component.perMile) {
          cost = component.perMile * annualMiles;
        }
        totalCost += cost;
        breakdown.push({ name: component.name, cost });
      });
    }
  }
</script>

<div class="chapter-container">
  <div class="chapter-header">
    <h1>💰 The Money Drain</h1>
    <p class="subtitle">The True Cost of Your Commute</p>
  </div>

  <div class="calculator">
    <div class="controls">
      <div class="control-group">
        <label for="mode">Commute Mode:</label>
        <select id="mode" bind:value={mode}>
          <option value="car">🚗 Drive Alone (Car)</option>
          <option value="transit">🚇 Public Transit</option>
          <option value="bike">🚲 Bike</option>
        </select>
      </div>

      <div class="control-group">
        <label for="miles">One-way Distance (miles):</label>
        <input 
          id="miles"
          type="number" 
          bind:value={miles}
          min="0" 
          max="100" 
          step="0.5"
        />
      </div>
    </div>

    <div class="cost-breakdown">
      <h2>Annual Cost Breakdown:</h2>
      
      {#each breakdown as item}
        <div class="cost-item">
          <span class="cost-label">{item.name}</span>
          <span class="cost-value">${item.cost.toFixed(0)}</span>
        </div>
      {/each}

      <div class="cost-item total">
        <span class="cost-label">TOTAL ANNUAL COST</span>
        <span class="cost-value">${totalCost.toFixed(0)}</span>
      </div>

      <div class="cost-item">
        <span class="cost-label">Cost Per Commute</span>
        <span class="cost-value">${(totalCost / 250).toFixed(2)}</span>
      </div>
    </div>

    <div class="insight">
      <p>💡 <strong>Did you know?</strong> The average American spends over $9,000 per year on commuting costs. That's equivalent to a nice vacation or several months of rent!</p>
    </div>
  </div>
</div>

<style>
  .chapter-container {
    min-height: 100vh;
    padding: 2rem;
    background: #1a1a2e;
    color: white;
  }

  .chapter-header {
    text-align: center;
    margin-bottom: 3rem;
  }

  .chapter-header h1 {
    font-size: 3rem;
    margin-bottom: 0.5rem;
  }

  .subtitle {
    font-size: 1.5rem;
    opacity: 0.9;
  }

  .calculator {
    max-width: 900px;
    margin: 0 auto;
    background: linear-gradient(135deg, #0f3460 0%, #16213e 100%);
    padding: 2rem;
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.3);
  }

  .controls {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2rem;
    margin-bottom: 2rem;
  }

  .control-group {
    display: flex;
    flex-direction: column;
  }

  label {
    margin-bottom: 0.5rem;
    font-weight: bold;
  }

  select, input {
    padding: 1rem;
    font-size: 1rem;
    border: none;
    border-radius: 8px;
    background: white;
    color: #333;
  }

  .cost-breakdown {
    margin-top: 2rem;
  }

  .cost-breakdown h2 {
    margin-bottom: 1rem;
    text-align: center;
  }

  .cost-item {
    background: rgba(255,255,255,0.1);
    padding: 1rem;
    margin: 0.75rem 0;
    border-radius: 8px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .cost-item.total {
    background: #e94560;
    font-size: 1.2rem;
    font-weight: bold;
    margin-top: 1.5rem;
  }

  .cost-label {
    font-size: 1.1rem;
  }

  .cost-value {
    font-size: 1.5rem;
    font-weight: bold;
    color: #00d4ff;
  }

  .total .cost-value {
    color: white;
  }

  .insight {
    margin-top: 2rem;
    padding: 1.5rem;
    background: rgba(255, 206, 86, 0.2);
    border-left: 4px solid #ffce54;
    border-radius: 8px;
  }

  .insight p {
    margin: 0;
    line-height: 1.6;
  }

  @media (max-width: 768px) {
    .controls {
      grid-template-columns: 1fr;
      gap: 1rem;
    }

    .chapter-header h1 {
      font-size: 2rem;
    }

    .calculator {
      padding: 1rem;
    }
  }
</style>
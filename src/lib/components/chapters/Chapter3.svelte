<script lang="ts">
  import { onMount } from 'svelte';
  import { emissionFactors } from '$lib/data/sampleData';

  type VehicleType = keyof typeof emissionFactors;

  let vehicleType: VehicleType = 'sedan';
  let distance: number = 15;
  let daysPerWeek: number = 5;
  let dataLoaded: boolean = false;

  // PA data (loaded from JSON)
  let paStats = {
    avgCommuteMinutes: 0,
    avgCommuteMiles: 0,
    avgCO2PerPerson: 0,
    totalCommuters: 0
  };

  type MetroData = {
    metro: string;
    avg_commute: number;
    workers: number;
    co2_per_capita: number;
    transit_pct: number;
    median_income: number;
    commute_burden_score: number;
  };

  let metroData: MetroData[] = [];

  $: annualCO2 = calculateEmissions(vehicleType, distance, daysPerWeek);
  $: comparisons = calculateComparisons(annualCO2);
  $: paComparison = paStats.avgCO2PerPerson > 0 ? 
    ((annualCO2 / paStats.avgCO2PerPerson - 1) * 100).toFixed(1) : '0';
  $: isAboveAverage = parseFloat(paComparison) > 0;

  // Convert commute minutes to approximate miles (assuming 0.5 miles per minute average)
  $: userCommuteMinutes = (distance / 0.5);
  $: vsPAAvgCommute = paStats.avgCommuteMinutes > 0 ? 
    ((userCommuteMinutes / paStats.avgCommuteMinutes - 1) * 100).toFixed(1) : '0';

  function calculateEmissions(vehicle: VehicleType, miles: number, days: number): number {
    const dailyMiles = miles * 2;
    const annualMiles = dailyMiles * days * 50;
    return (annualMiles * emissionFactors[vehicle]) / 1000;
  }

  function calculateComparisons(co2Tons: number) {
    return {
      flights: (co2Tons * 1000 / 400).toFixed(1),
      beefPounds: (co2Tons * 1000 / 27).toFixed(0),
      treesNeeded: (co2Tons / 0.021).toFixed(0),
      gallons: (co2Tons * 1000 / 8.89).toFixed(0)
    };
  }

  onMount(async () => {
    try {
      // Load PA summary stats
      const summaryResponse = await fetch('/pa_summary_stats.json');
      const summaryData = await summaryResponse.json();
      
      // Load scenarios to get CO2 per person
      const scenariosResponse = await fetch('/pa_scenarios.json');
      const scenariosData = await scenariosResponse.json();
      
      // Load metro comparison data
      const metroResponse = await fetch('/pa_metro_comparison.json');
      const metroDataResponse = await metroResponse.json();
      
      // Calculate average commute miles from minutes (rough estimate: 0.5 miles per minute)
      const avgCommuteMiles = summaryData.avg_commute_minutes * 0.5;
      
      // Calculate CO2 per person from scenarios
      const totalCommuters = summaryData.total_commuters;
      const annualCO2Tons = scenariosData.current.emissions_tons;
      const avgCO2PerPerson = annualCO2Tons / totalCommuters;
      
      paStats = {
        avgCommuteMinutes: summaryData.avg_commute_minutes,
        avgCommuteMiles: avgCommuteMiles,
        avgCO2PerPerson: avgCO2PerPerson,
        totalCommuters: totalCommuters
      };
      
      metroData = (metroDataResponse || []) as MetroData[];
      
      // Set default distance to match PA average
      distance = Math.round(avgCommuteMiles);
      
      dataLoaded = true;
    } catch (error) {
      console.error('Error loading PA data:', error);
      // Fallback values
      paStats = {
        avgCommuteMinutes: 9.9,
        avgCommuteMiles: 5.0,
        avgCO2PerPerson: 0.35,
        totalCommuters: 6_179_069
      };
      dataLoaded = true;
    }
  });
</script>

<div class="chapter-container">
  <div class="chapter-header">
    <h1>🌍 The Carbon Footprint</h1>
    <p class="subtitle">Your Commute's Environmental Impact</p>
  </div>

  {#if !dataLoaded}
    <div class="loading-state">
      <p>Loading Pennsylvania commute data...</p>
    </div>
  {:else}
  <div class="calculator">
    <div class="controls">
      <div class="control-group">
        <label for="vehicle">Vehicle Type:</label>
        <select id="vehicle" bind:value={vehicleType}>
          <option value="sedan">Sedan (avg)</option>
          <option value="suv">SUV/Truck</option>
          <option value="electric">Electric Vehicle</option>
          <option value="hybrid">Hybrid</option>
          <option value="transit">Public Transit (bus)</option>
          <option value="train">Train/Metro</option>
          <option value="bike">Bicycle</option>
        </select>
      </div>

      <div class="control-group">
        <label for="distance">One-way distance (miles):</label>
        <input 
          id="distance"
          type="number" 
          bind:value={distance}
          min="0" 
          max="100" 
          step="0.5"
        />
      </div>

      <div class="control-group">
        <label for="days">Days per week:</label>
        <input 
          id="days"
          type="number" 
          bind:value={daysPerWeek}
          min="1" 
          max="7"
        />
      </div>
    </div>

    <div class="result-card">
      <h2>Your Annual Commute Emissions:</h2>
      <div class="big-stat">{annualCO2.toFixed(2)}</div>
      <p class="stat-label">metric tons of CO₂</p>
    </div>

    <h2 class="comparison-header">That's Equivalent To:</h2>
    <div class="comparison-grid">
      <div class="comparison-item">
        <div class="icon">✈️</div>
        <div class="comparison-value">{comparisons.flights}</div>
        <p>NYC to LA flights</p>
      </div>

      <div class="comparison-item">
        <div class="icon">🍔</div>
        <div class="comparison-value">{comparisons.beefPounds}</div>
        <p>pounds of beef</p>
      </div>

      <div class="comparison-item">
        <div class="icon">🌳</div>
        <div class="comparison-value">{comparisons.treesNeeded}</div>
        <p>trees needed to offset</p>
      </div>

      <div class="comparison-item">
        <div class="icon">⛽</div>
        <div class="comparison-value">{comparisons.gallons}</div>
        <p>gallons of gasoline</p>
      </div>
    </div>

    {#if dataLoaded && paStats.avgCO2PerPerson > 0}
      <div class="pa-comparison">
        <h3>📊 How You Compare to PA Average</h3>
        <div class="comparison-stats">
          <div class="comparison-stat">
            <span class="stat-label">PA Average Commute</span>
            <span class="stat-value">{paStats.avgCommuteMiles.toFixed(1)} miles</span>
            <span class="stat-note">({paStats.avgCommuteMinutes.toFixed(1)} minutes)</span>
          </div>
          <div class="comparison-stat">
            <span class="stat-label">PA Average CO₂</span>
            <span class="stat-value">{paStats.avgCO2PerPerson.toFixed(2)} tons/year</span>
          </div>
          <div class="comparison-stat highlight" class:above={isAboveAverage} class:below={!isAboveAverage}>
            <span class="stat-label">Your Impact vs PA Average</span>
            <span class="stat-value" class:positive={!isAboveAverage} class:negative={isAboveAverage}>
              {isAboveAverage ? '+' : ''}{paComparison}%
            </span>
            <span class="stat-note">
              {isAboveAverage ? 'above' : 'below'} the PA average
            </span>
          </div>
        </div>
      </div>
    {/if}

    {#if metroData.length > 0}
      <div class="metro-comparison">
        <h3>🏙️ PA Metro Areas Comparison</h3>
        <div class="metro-grid">
          {#each metroData.slice(0, 4) as metro}
            <div class="metro-card">
              <div class="metro-name">{metro.metro}</div>
              <div class="metro-stat">
                <span class="metro-label">Avg Commute:</span>
                <span class="metro-value">{metro.avg_commute.toFixed(1)} min</span>
              </div>
              <div class="metro-stat">
                <span class="metro-label">CO₂ per person:</span>
                <span class="metro-value">{metro.co2_per_capita.toFixed(2)} tons</span>
              </div>
              <div class="metro-stat">
                <span class="metro-label">Transit usage:</span>
                <span class="metro-value">{metro.transit_pct}%</span>
              </div>
            </div>
          {/each}
        </div>
      </div>
    {/if}

    <div class="insight">
      <p>🌱 <strong>Take Action:</strong> Switching to public transit or carpooling just 2 days per week could reduce your carbon footprint by 40%. Small changes make a big difference!</p>
      {#if dataLoaded && paStats.totalCommuters > 0}
        <p style="margin-top: 1rem;">
          💡 <strong>Pennsylvania Context:</strong> With {paStats.totalCommuters.toLocaleString()} commuters across the state, 
          if everyone reduced their commute emissions by just 10%, we'd save over 
          {(paStats.avgCO2PerPerson * paStats.totalCommuters * 0.1 / 1000).toFixed(0)}K tons of CO₂ annually!
        </p>
      {/if}
    </div>
  </div>
  {/if}
</div>

<style>
  .chapter-container {
    min-height: 100vh;
    padding: 2rem;
    background: linear-gradient(to bottom, #2c3e50, #34495e);
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
    max-width: 1000px;
    margin: 0 auto;
    background: rgba(255,255,255,0.1);
    padding: 2rem;
    border-radius: 15px;
    backdrop-filter: blur(10px);
  }

  .controls {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1.5rem;
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
    padding: 0.75rem;
    font-size: 1rem;
    border: none;
    border-radius: 8px;
    background: white;
    color: #333;
  }

  .result-card {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 2rem;
    border-radius: 12px;
    text-align: center;
    margin: 2rem 0;
  }

  .result-card h2 {
    margin-top: 0;
    font-size: 1.5rem;
  }

  .big-stat {
    font-size: 4rem;
    font-weight: bold;
    color: #ffd700;
    margin: 1rem 0;
  }

  .stat-label {
    font-size: 1.5rem;
    margin: 0;
  }

  .comparison-header {
    text-align: center;
    margin: 2rem 0 1rem;
  }

  .comparison-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1.5rem;
    margin: 2rem 0;
  }

  .comparison-item {
    background: rgba(255,255,255,0.15);
    padding: 1.5rem;
    border-radius: 10px;
    text-align: center;
  }

  .icon {
    font-size: 3rem;
    margin-bottom: 0.5rem;
  }

  .comparison-value {
    font-size: 2rem;
    font-weight: bold;
    color: #ffd700;
    margin: 0.5rem 0;
  }

  .comparison-item p {
    margin: 0;
    font-size: 1rem;
  }

  .insight {
    margin-top: 2rem;
    padding: 1.5rem;
    background: rgba(46, 204, 113, 0.2);
    border-left: 4px solid #2ecc71;
    border-radius: 8px;
  }

  .insight p {
    margin: 0;
    line-height: 1.6;
  }

  .loading-state {
    text-align: center;
    padding: 4rem 2rem;
    font-size: 1.5rem;
    opacity: 0.8;
  }

  .pa-comparison {
    margin: 2rem 0;
    padding: 2rem;
    background: rgba(78, 205, 196, 0.1);
    border-left: 4px solid #4ecdc4;
    border-radius: 12px;
  }

  .pa-comparison h3 {
    margin-top: 0;
    margin-bottom: 1.5rem;
    font-size: 1.5rem;
    color: #4ecdc4;
  }

  .comparison-stats {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1.5rem;
  }

  .comparison-stat {
    padding: 1rem;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 8px;
    text-align: center;
  }

  .comparison-stat.highlight {
    background: rgba(255, 255, 255, 0.15);
    border: 2px solid;
  }

  .comparison-stat.highlight.above {
    border-color: #ff6b6b;
  }

  .comparison-stat.highlight.below {
    border-color: #51cf66;
  }

  .stat-label {
    display: block;
    font-size: 0.9rem;
    opacity: 0.8;
    margin-bottom: 0.5rem;
  }

  .stat-value {
    display: block;
    font-size: 2rem;
    font-weight: bold;
    color: #ffd700;
    margin: 0.5rem 0;
  }

  .stat-value.positive {
    color: #51cf66;
  }

  .stat-value.negative {
    color: #ff6b6b;
  }

  .stat-note {
    display: block;
    font-size: 0.85rem;
    opacity: 0.7;
    margin-top: 0.25rem;
  }

  .metro-comparison {
    margin: 2rem 0;
    padding: 2rem;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
  }

  .metro-comparison h3 {
    margin-top: 0;
    margin-bottom: 1.5rem;
    font-size: 1.5rem;
    text-align: center;
  }

  .metro-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1.5rem;
  }

  .metro-card {
    padding: 1.5rem;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 10px;
    border: 1px solid rgba(255, 255, 255, 0.2);
  }

  .metro-name {
    font-size: 1.3rem;
    font-weight: bold;
    margin-bottom: 1rem;
    color: #4ecdc4;
    text-align: center;
  }

  .metro-stat {
    display: flex;
    justify-content: space-between;
    margin: 0.75rem 0;
    padding: 0.5rem 0;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }

  .metro-stat:last-child {
    border-bottom: none;
  }

  .metro-label {
    font-size: 0.9rem;
    opacity: 0.8;
  }

  .metro-value {
    font-size: 1rem;
    font-weight: 600;
    color: #ffd700;
  }

  @media (max-width: 768px) {
    .controls {
      grid-template-columns: 1fr;
    }

    .chapter-header h1 {
      font-size: 2rem;
    }

    .big-stat {
      font-size: 3rem;
    }

    .comparison-grid {
      grid-template-columns: repeat(2, 1fr);
    }
  }
</style>
<script lang="ts">
  import { emissionFactors } from '$lib/data/sampleData';

  type VehicleType = keyof typeof emissionFactors;

  let vehicleType: VehicleType = 'sedan';
  let distance: number = 15;
  let daysPerWeek: number = 5;

  $: annualCO2 = calculateEmissions(vehicleType, distance, daysPerWeek);
  $: comparisons = calculateComparisons(annualCO2);

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
</script>

<div class="chapter-container">
  <div class="chapter-header">
    <h1>🌍 The Carbon Footprint</h1>
    <p class="subtitle">Your Commute's Environmental Impact</p>
  </div>

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

    <div class="insight">
      <p>🌱 <strong>Take Action:</strong> Switching to public transit or carpooling just 2 days per week could reduce your carbon footprint by 40%. Small changes make a big difference!</p>
    </div>
  </div>
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
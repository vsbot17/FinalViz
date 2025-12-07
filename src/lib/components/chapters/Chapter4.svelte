<script lang="ts">
  import { onMount, tick } from 'svelte';

  // PA Baseline data (loaded from JSON)
  let PA_BASELINE = {
    total_commuters: 0,
    current_emissions_tons: 0,
    drive_alone_pct: 0,
    transit_pct: 0,
    wfh_pct: 0
  };

  // Emissions per mode (kg CO2 per commuter per year)
  const EMISSIONS_PER_MODE = {
    drive: 2580,
    transit: 580,
    wfh: 0,
    bike_walk: 0
  };

  let transitShiftPct: number = 0;
  let wfhIncreasePct: number = 0;
  let dataLoaded: boolean = false;
  let predefinedScenarios: any = null;

  $: newDrivePct = PA_BASELINE.drive_alone_pct - transitShiftPct - wfhIncreasePct * 0.75;
  $: newTransitPct = PA_BASELINE.transit_pct + transitShiftPct;
  $: newWfhPct = PA_BASELINE.wfh_pct + wfhIncreasePct;

  $: driveCommuters = PA_BASELINE.total_commuters * (newDrivePct / 100);
  $: transitCommuters = PA_BASELINE.total_commuters * (newTransitPct / 100);
  $: wfhCommuters = PA_BASELINE.total_commuters * (newWfhPct / 100);

  $: newEmissions = (
    driveCommuters * EMISSIONS_PER_MODE.drive +
    transitCommuters * EMISSIONS_PER_MODE.transit +
    wfhCommuters * EMISSIONS_PER_MODE.wfh
  ) / 1000;

  $: emissionsReduced = PA_BASELINE.current_emissions_tons - newEmissions;
  $: pctReduction = (emissionsReduced / PA_BASELINE.current_emissions_tons * 100);
  $: carsEquivalent = Math.round(emissionsReduced / 4.6);

  let chartCanvas: HTMLCanvasElement;
  let chart: any;

  // Load data from JSON files
  onMount(async () => {
    try {
      // Load summary stats for baseline
      const summaryResponse = await fetch('/pa_summary_stats.json');
      const summaryData = await summaryResponse.json();
      
      // Load scenarios data
      const scenariosResponse = await fetch('/pa_scenarios.json');
      const scenariosData = await scenariosResponse.json();
      predefinedScenarios = scenariosData;
      
      // Set baseline from actual data
      PA_BASELINE = {
        total_commuters: summaryData.total_commuters,
        current_emissions_tons: scenariosData.current.emissions_tons,
        drive_alone_pct: scenariosData.current.mode_split.drive_alone_pct,
        transit_pct: scenariosData.current.mode_split.transit_pct,
        wfh_pct: scenariosData.current.mode_split.wfh_pct
      };
      
      dataLoaded = true;
      
      // Wait for DOM to be ready, then initialize chart
      await tick();
      await initializeChart();
    } catch (error) {
      console.error('Error loading PA data:', error);
      // Fallback to default values
      PA_BASELINE = {
        total_commuters: 6_179_069,
        current_emissions_tons: 2_166_258,
        drive_alone_pct: 3.0,
        transit_pct: 47.1,
        wfh_pct: 7.6
      };
      dataLoaded = true;
      
      await tick();
      await initializeChart();
    }
  });
  
  async function initializeChart() {
    if (!chartCanvas || !dataLoaded || chart) return;
    
    try {
      // Import chart.js/auto - Vite should optimize this
      const chartModule = await import('chart.js/auto');
      const Chart = chartModule.default || chartModule;
      
      // Wait for DOM to be ready
      await tick();
      
      if (chartCanvas && !chart) {
        createChart(Chart);
      }
    } catch (error) {
      console.error('Error initializing chart:', error);
    }
  }
  
  // Reactive statement to create chart when both canvas and data are ready
  $: if (chartCanvas && dataLoaded && !chart) {
    initializeChart();
  }

  function createChart(Chart: any) {
    if (!chartCanvas || !dataLoaded) {
      console.warn('Cannot create chart: canvas or data not ready');
      return;
    }
    
    // Destroy existing chart if it exists
    if (chart) {
      chart.destroy();
      chart = null;
    }
    
    const ctx = chartCanvas.getContext('2d');
    if (!ctx) {
      console.warn('Cannot get 2d context from canvas');
      return;
    }
    
    // Calculate current emissions breakdown
    const currentDriveEmissions = PA_BASELINE.total_commuters * (PA_BASELINE.drive_alone_pct / 100) * EMISSIONS_PER_MODE.drive / 1000;
    const currentTransitEmissions = PA_BASELINE.total_commuters * (PA_BASELINE.transit_pct / 100) * EMISSIONS_PER_MODE.transit / 1000;
    const scenarioDriveEmissions = driveCommuters * EMISSIONS_PER_MODE.drive / 1000;
    const scenarioTransitEmissions = transitCommuters * EMISSIONS_PER_MODE.transit / 1000;
    
    console.log('Creating chart with data:', {
      currentDriveEmissions,
      currentTransitEmissions,
      scenarioDriveEmissions,
      scenarioTransitEmissions
    });
    
    try {
      chart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: ['Current', 'Your Scenario'],
          datasets: [
            {
              label: 'Drive Alone',
              data: [currentDriveEmissions, scenarioDriveEmissions],
              backgroundColor: '#ff6b6b'
            },
            {
              label: 'Public Transit',
              data: [currentTransitEmissions, scenarioTransitEmissions],
              backgroundColor: '#4ecdc4'
            },
            {
              label: 'Work From Home',
              data: [0, 0],
              backgroundColor: '#51cf66'
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: true,
          aspectRatio: 2,
          scales: {
            x: { 
              stacked: true,
              ticks: {
                color: 'rgba(255, 255, 255, 0.8)'
              },
              grid: {
                color: 'rgba(255, 255, 255, 0.1)'
              }
            },
            y: {
              stacked: true,
              beginAtZero: true,
              title: {
                display: true,
                text: 'Annual CO₂ Emissions (tons)',
                color: 'rgba(255, 255, 255, 0.8)'
              },
              ticks: {
                color: 'rgba(255, 255, 255, 0.8)'
              },
              grid: {
                color: 'rgba(255, 255, 255, 0.1)'
              }
            }
          },
          plugins: {
            legend: { 
              position: 'bottom',
              labels: {
                color: 'rgba(255, 255, 255, 0.8)',
                padding: 15
              }
            },
            title: {
              display: true,
              text: 'Pennsylvania Commute Emissions by Scenario',
              color: 'rgba(255, 255, 255, 0.9)',
              font: {
                size: 16
              }
            }
          }
        }
      });
      console.log('Chart created successfully');
    } catch (error) {
      console.error('Error creating chart:', error);
    }
  }

  $: if (chart && dataLoaded) {
    const currentDriveEmissions = PA_BASELINE.total_commuters * (PA_BASELINE.drive_alone_pct / 100) * EMISSIONS_PER_MODE.drive / 1000;
    const currentTransitEmissions = PA_BASELINE.total_commuters * (PA_BASELINE.transit_pct / 100) * EMISSIONS_PER_MODE.transit / 1000;
    
    chart.data.datasets[0].data = [
      currentDriveEmissions,
      driveCommuters * EMISSIONS_PER_MODE.drive / 1000
    ];
    chart.data.datasets[1].data = [
      currentTransitEmissions,
      transitCommuters * EMISSIONS_PER_MODE.transit / 1000
    ];
    chart.update();
  }

  // Predefined scenarios - use actual scenario data if available
  function applyScenario(type: 'modest' | 'ambitious' | 'transformative') {
    if (predefinedScenarios) {
      if (type === 'modest') {
        // Use 10pct_transit scenario as modest
        const scenario = predefinedScenarios['10pct_transit'];
        if (scenario) {
          transitShiftPct = 10;
          wfhIncreasePct = 0;
        } else {
          transitShiftPct = 5;
          wfhIncreasePct = 5;
        }
      } else if (type === 'ambitious') {
        // Use 20pct_wfh scenario as ambitious
        const scenario = predefinedScenarios['20pct_wfh'];
        if (scenario) {
          transitShiftPct = 0;
          wfhIncreasePct = 15; // Approximate from the scenario
        } else {
          transitShiftPct = 10;
          wfhIncreasePct = 10;
        }
      } else {
        // Transformative: combine both
        transitShiftPct = 15;
        wfhIncreasePct = 15;
      }
    } else {
      // Fallback to default values
      if (type === 'modest') {
        transitShiftPct = 5;
        wfhIncreasePct = 5;
      } else if (type === 'ambitious') {
        transitShiftPct = 10;
        wfhIncreasePct = 10;
      } else {
        transitShiftPct = 15;
        wfhIncreasePct = 15;
      }
    }
  }

  function resetScenario() {
    transitShiftPct = 0;
    wfhIncreasePct = 0;
  }
</script>

<div class="chapter-container">
  <div class="chapter-header">
    <h1>🔮 What If We Changed?</h1>
    <p class="subtitle">Modeling Pennsylvania's Clean Commute Future</p>
  </div>

  {#if !dataLoaded}
    <div class="loading-state">
      <p>Loading Pennsylvania scenario data...</p>
    </div>
  {:else}
  <div class="content-wrapper">
    <div class="intro-section">
      <p class="intro-text">
        The good news? <strong>Small shifts create massive impact.</strong> 
        Adjust the sliders below to see how different scenarios would reduce 
        Pennsylvania's commute-related carbon emissions.
      </p>
    </div>

    <div class="controls-section">
      <h2>Build Your Scenario</h2>
      
      <div class="slider-group">
        <div class="slider-header">
          <label for="transit-slider">🚇 Shift to Public Transit</label>
          <span class="slider-value">{transitShiftPct}%</span>
        </div>
        <input 
          id="transit-slider"
          type="range" 
          bind:value={transitShiftPct}
          min="0" 
          max="20" 
          step="1"
        />
        <p class="slider-description">
          Move {transitShiftPct}% of drivers to buses, trains, and metro
        </p>
      </div>

      <div class="slider-group">
        <div class="slider-header">
          <label for="wfh-slider">🏠 Increase Remote Work</label>
          <span class="slider-value">{wfhIncreasePct}%</span>
        </div>
        <input 
          id="wfh-slider"
          type="range" 
          bind:value={wfhIncreasePct}
          min="0" 
          max="25" 
          step="1"
        />
        <p class="slider-description">
          Increase work-from-home adoption by {wfhIncreasePct} percentage points
        </p>
      </div>

      <div class="quick-scenarios">
        <p>Quick scenarios:</p>
        <button on:click={() => applyScenario('modest')}>Modest (5% each)</button>
        <button on:click={() => applyScenario('ambitious')}>Ambitious (10% each)</button>
        <button on:click={() => applyScenario('transformative')}>Transformative (15% each)</button>
        <button on:click={resetScenario} class="reset">Reset</button>
      </div>
    </div>

    <div class="results-section">
      <h2>Impact of Your Scenario</h2>
      
      <div class="impact-grid">
        <div class="impact-card primary">
          <div class="impact-icon">🌍</div>
          <div class="impact-label">CO₂ Reduced</div>
          <div class="impact-value">{(emissionsReduced / 1000).toFixed(1)}M</div>
          <div class="impact-unit">tons per year</div>
          <div class="impact-pct">{pctReduction.toFixed(1)}% reduction</div>
        </div>

        <div class="impact-card">
          <div class="impact-icon">🚗</div>
          <div class="impact-label">Equivalent To</div>
          <div class="impact-value">{carsEquivalent.toLocaleString()}</div>
          <div class="impact-unit">cars off the road</div>
        </div>

        <div class="impact-card">
          <div class="impact-icon">🌳</div>
          <div class="impact-label">Trees Needed</div>
          <div class="impact-value">{Math.round(emissionsReduced / 0.021).toLocaleString()}</div>
          <div class="impact-unit">to offset naturally</div>
        </div>

        <div class="impact-card">
          <div class="impact-icon">✈️</div>
          <div class="impact-label">Flight Equivalent</div>
          <div class="impact-value">{Math.round(emissionsReduced * 1000 / 400).toLocaleString()}</div>
          <div class="impact-unit">NYC to LA flights</div>
        </div>
      </div>

      <div class="mode-breakdown">
        <h3>New Mode Split</h3>
        <div class="mode-bars">
          <div class="mode-bar">
            <span class="mode-label">🚗 Drive Alone</span>
            <div class="bar-container">
              <div class="bar drive" style="width: {newDrivePct}%"></div>
            </div>
            <span class="mode-pct">{newDrivePct.toFixed(1)}%</span>
          </div>
          <div class="mode-bar">
            <span class="mode-label">🚇 Public Transit</span>
            <div class="bar-container">
              <div class="bar transit" style="width: {newTransitPct}%"></div>
            </div>
            <span class="mode-pct">{newTransitPct.toFixed(1)}%</span>
          </div>
          <div class="mode-bar">
            <span class="mode-label">🏠 Work From Home</span>
            <div class="bar-container">
              <div class="bar wfh" style="width: {newWfhPct}%"></div>
            </div>
            <span class="mode-pct">{newWfhPct.toFixed(1)}%</span>
          </div>
        </div>
      </div>
    </div>

    <div class="chart-section">
      {#if dataLoaded}
        <canvas bind:this={chartCanvas}></canvas>
      {:else}
        <div class="chart-loading">Loading chart data...</div>
      {/if}
    </div>

    <div class="reality-check">
      <h3>📊 Reality Check: COVID-19 Proved This Works</h3>
      <p>
        During 2020, Pennsylvania's work-from-home rate jumped from <strong>5.6% to 23.7%</strong> 
        as companies rapidly adapted to remote work.
      </p>
      <p>
        This unplanned experiment prevented an estimated <strong>3.2 million tons of CO₂</strong> 
        - proving that large-scale behavior change is not only possible, we've already done it.
      </p>
      <p class="emphasis">
        The question isn't "Can we?" - it's "Will we?"
      </p>
    </div>
  </div>
  {/if}
</div>

<style>
  .chapter-container {
    min-height: 100vh;
    background: linear-gradient(180deg, #16213e 0%, #0f3460 100%);
    color: white;
    padding: 4rem 2rem;
  }

  .chapter-header {
    text-align: center;
    margin-bottom: 3rem;
  }

  .chapter-header h1 {
    font-size: 3.5rem;
    font-weight: 800;
    margin-bottom: 1rem;
  }

  .subtitle {
    font-size: 1.5rem;
    opacity: 0.9;
  }

  .content-wrapper {
    max-width: 1200px;
    margin: 0 auto;
  }

  .intro-section {
    margin: 3rem 0;
    padding: 2rem;
    background: rgba(78, 205, 196, 0.1);
    border-left: 4px solid #4ecdc4;
    border-radius: 12px;
  }

  .intro-text {
    font-size: 1.3rem;
    line-height: 1.8;
    margin: 0;
  }

  .controls-section {
    margin: 4rem 0;
    padding: 3rem;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 15px;
  }

  .controls-section h2 {
    font-size: 2rem;
    margin-bottom: 2rem;
    text-align: center;
  }

  .slider-group {
    margin: 3rem 0;
  }

  .slider-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1rem;
  }

  .slider-header label {
    font-size: 1.3rem;
    font-weight: 600;
  }

  .slider-value {
    font-size: 2rem;
    font-weight: 800;
    color: #4ecdc4;
  }

  input[type="range"] {
    width: 100%;
    height: 10px;
    background: rgba(255, 255, 255, 0.2);
    border-radius: 5px;
    outline: none;
    -webkit-appearance: none;
  }

  input[type="range"]::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    width: 28px;
    height: 28px;
    background: #4ecdc4;
    border-radius: 50%;
    cursor: pointer;
    box-shadow: 0 2px 8px rgba(0,0,0,0.3);
  }

  input[type="range"]::-moz-range-thumb {
    width: 28px;
    height: 28px;
    background: #4ecdc4;
    border-radius: 50%;
    cursor: pointer;
    border: none;
  }

  .slider-description {
    margin-top: 0.5rem;
    font-size: 1rem;
    opacity: 0.8;
  }

  .quick-scenarios {
    margin-top: 3rem;
    text-align: center;
  }

  .quick-scenarios p {
    margin-bottom: 1rem;
    font-size: 1.1rem;
  }

  .quick-scenarios button {
    margin: 0.5rem;
    padding: 0.75rem 1.5rem;
    font-size: 1rem;
    font-weight: 600;
    background: rgba(255, 255, 255, 0.1);
    color: white;
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s;
  }

  .quick-scenarios button:hover {
    background: #4ecdc4;
    border-color: #4ecdc4;
    transform: translateY(-2px);
  }

  .quick-scenarios button.reset {
    background: rgba(255, 107, 107, 0.2);
    border-color: #ff6b6b;
  }

  .quick-scenarios button.reset:hover {
    background: #ff6b6b;
  }

  .results-section {
    margin: 4rem 0;
  }

  .results-section h2 {
    font-size: 2.5rem;
    text-align: center;
    margin-bottom: 3rem;
  }

  .impact-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    margin-bottom: 4rem;
  }

  .impact-card {
    padding: 2rem;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    text-align: center;
    transition: transform 0.3s;
  }

  .impact-card:hover {
    transform: translateY(-5px);
  }

  .impact-card.primary {
    background: rgba(78, 205, 196, 0.15);
    border: 2px solid #4ecdc4;
    grid-column: 1 / -1;
  }

  .impact-icon {
    font-size: 3rem;
    margin-bottom: 1rem;
  }

  .impact-label {
    font-size: 1rem;
    text-transform: uppercase;
    letter-spacing: 1px;
    opacity: 0.7;
    margin-bottom: 0.5rem;
  }

  .impact-value {
    font-size: 3rem;
    font-weight: 800;
    color: #4ecdc4;
    line-height: 1;
  }

  .primary .impact-value {
    font-size: 4rem;
  }

  .impact-unit {
    font-size: 1rem;
    opacity: 0.8;
    margin-top: 0.5rem;
  }

  .impact-pct {
    margin-top: 1rem;
    font-size: 1.5rem;
    font-weight: 600;
    color: #51cf66;
  }

  .mode-breakdown {
    margin: 3rem 0;
    padding: 2rem;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
  }

  .mode-breakdown h3 {
    font-size: 1.8rem;
    margin-bottom: 2rem;
    text-align: center;
  }

  .mode-bars {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  .mode-bar {
    display: grid;
    grid-template-columns: 200px 1fr 80px;
    align-items: center;
    gap: 1rem;
  }

  .mode-label {
    font-size: 1.1rem;
    font-weight: 600;
  }

  .bar-container {
    height: 40px;
    background: rgba(255, 255, 255, 0.1);
    border-radius: 20px;
    overflow: hidden;
  }

  .bar {
    height: 100%;
    transition: width 0.5s ease;
    border-radius: 20px;
  }

  .bar.drive {
    background: linear-gradient(90deg, #ff6b6b, #ff8e53);
  }

  .bar.transit {
    background: linear-gradient(90deg, #4ecdc4, #44a3d5);
  }

  .bar.wfh {
    background: linear-gradient(90deg, #51cf66, #37b24d);
  }

  .mode-pct {
    font-size: 1.3rem;
    font-weight: 700;
    text-align: right;
  }

  .chart-section {
    margin: 4rem 0;
    padding: 2rem;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    min-height: 400px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .chart-section canvas {
    max-width: 100%;
    width: 100% !important;
    height: 400px !important;
  }

  .chart-loading {
    text-align: center;
    padding: 2rem;
    font-size: 1.2rem;
    opacity: 0.8;
  }

  .reality-check {
    margin: 4rem 0;
    padding: 3rem;
    background: rgba(255, 215, 0, 0.1);
    border-left: 4px solid #ffd93d;
    border-radius: 12px;
  }

  .reality-check h3 {
    font-size: 2rem;
    margin-bottom: 1.5rem;
    color: #ffd93d;
  }

  .reality-check p {
    font-size: 1.2rem;
    line-height: 1.8;
    margin: 1rem 0;
  }

  .reality-check .emphasis {
    font-size: 1.5rem;
    font-weight: 700;
    text-align: center;
    margin-top: 2rem;
    color: #ffd93d;
  }

  .loading-state {
    text-align: center;
    padding: 4rem 2rem;
    font-size: 1.5rem;
    opacity: 0.8;
  }

  @media (max-width: 768px) {
    .chapter-header h1 {
      font-size: 2rem;
    }

    .impact-grid {
      grid-template-columns: 1fr;
    }

    .impact-card.primary {
      grid-column: 1;
    }

    .mode-bar {
      grid-template-columns: 1fr;
      gap: 0.5rem;
    }

    .bar-container {
      height: 30px;
    }
  }
</style>
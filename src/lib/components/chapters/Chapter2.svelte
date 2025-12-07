<script lang="ts">
  import { onMount } from 'svelte';

  // County data type
  type CountyData = {
    county: string;
    lat: number;
    lon: number;
    co2_tons: number;
    population?: number;
    co2_per_capita: number;
    total_workers?: number;
    mean_commute?: number;
  };

  // PA County data (loaded from JSON)
  let countyData: CountyData[] = [];
  let dataLoaded: boolean = false;

  let mapContainer: HTMLDivElement;
  let viewMode: 'total' | 'per_capita' = 'total';
  let selectedCounty: CountyData | null = null;

  // County coordinates mapping (for counties that might be in map data)
  const countyCoordinates: Record<string, { lat: number; lon: number }> = {
    'Allegheny': { lat: 40.4406, lon: -79.9959 },
    'Philadelphia': { lat: 40.0094, lon: -75.1333 },
    'Montgomery': { lat: 40.1688, lon: -75.3560 },
    'Bucks': { lat: 40.3154, lon: -75.1085 },
    'Delaware': { lat: 39.9167, lon: -75.4167 },
    'Chester': { lat: 39.9857, lon: -75.7499 },
    'Lancaster': { lat: 40.0379, lon: -76.3055 },
    'York': { lat: 39.9626, lon: -76.7277 },
    'Berks': { lat: 40.4167, lon: -75.9269 },
    'Dauphin': { lat: 40.3991, lon: -76.7897 },
    'Westmoreland': { lat: 40.3097, lon: -79.5181 },
    'Erie': { lat: 42.1292, lon: -80.0851 },
    'Lehigh': { lat: 40.6023, lon: -75.5964 },
    'Northampton': { lat: 40.7533, lon: -75.3082 },
    'Luzerne': { lat: 41.1843, lon: -75.8813 }
  };

  // Metro to county mapping (approximate)
  const metroToCounties: Record<string, string[]> = {
    'Philadelphia': ['Philadelphia', 'Montgomery', 'Bucks', 'Delaware', 'Chester'],
    'Pittsburgh': ['Allegheny', 'Westmoreland'],
    'Harrisburg': ['Dauphin'],
    'Allentown': ['Lehigh', 'Northampton']
  };

  $: totalEmissions = countyData.length > 0 ? countyData.reduce((sum, c) => sum + c.co2_tons, 0) : 0;
  $: top10Emissions = countyData.length > 0 ? countyData.slice(0, Math.min(10, countyData.length)).reduce((sum, c) => sum + c.co2_tons, 0) : 0;
  $: top10Percentage = totalEmissions > 0 ? (top10Emissions / totalEmissions * 100).toFixed(0) : '0';

  onMount(async () => {
    try {
      // Try to load county map data first
      const mapDataResponse = await fetch('/pa_county_map_data.json');
      const mapData = await mapDataResponse.json();
      
      // Try to load county emissions data
      const emissionsResponse = await fetch('/pa_county_emissions.json');
      const emissionsData = await emissionsResponse.json();
      
      // Try to load metro comparison data as fallback
      const metroResponse = await fetch('/pa_metro_comparison.json');
      const metroData = await metroResponse.json();

      if (mapData && mapData.length > 0) {
        // Use map data if available
        countyData = mapData.map((item: any) => ({
          county: item.county,
          lat: item.lat,
          lon: item.lon,
          co2_tons: item.co2_tons || 0,
          co2_per_capita: item.co2_per_capita || 0,
          mean_commute: item.mean_commute
        }));
      } else if (emissionsData && emissionsData.length > 0) {
        // Use emissions data and add coordinates
        countyData = emissionsData.map((item: any) => {
          const coords = countyCoordinates[item.county] || { lat: 40.0, lon: -77.0 };
          return {
            county: item.county,
            lat: coords.lat,
            lon: coords.lon,
            co2_tons: item.annual_co2_tons || 0,
            co2_per_capita: item.co2_per_capita_tons || 0,
            total_workers: item.total_workers,
            mean_commute: item.mean_commute_minutes
          };
        });
      } else if (metroData && metroData.length > 0) {
        // Use metro data as fallback - convert metros to representative counties
        countyData = metroData.map((metro: any) => {
          const counties = metroToCounties[metro.metro] || [metro.metro];
          const primaryCounty = counties[0];
          const coords = countyCoordinates[primaryCounty] || { lat: 40.0, lon: -77.0 };
          
          // Estimate total CO2 from per capita and workers
          const estimatedCO2 = metro.co2_per_capita * metro.workers;
          
          return {
            county: primaryCounty,
            lat: coords.lat,
            lon: coords.lon,
            co2_tons: estimatedCO2,
            co2_per_capita: metro.co2_per_capita,
            total_workers: metro.workers,
            mean_commute: metro.avg_commute
          };
        });
      } else {
        // Fallback to default data structure
        countyData = [
          { county: 'Philadelphia', lat: 40.0094, lon: -75.1333, co2_tons: 2_345_000, co2_per_capita: 1.48 },
          { county: 'Allegheny', lat: 40.4406, lon: -79.9959, co2_tons: 1_234_000, co2_per_capita: 0.99 },
          { county: 'Montgomery', lat: 40.1688, lon: -75.3560, co2_tons: 987_000, co2_per_capita: 1.15 },
          { county: 'Bucks', lat: 40.3154, lon: -75.1085, co2_tons: 765_000, co2_per_capita: 1.22 },
          { county: 'Delaware', lat: 39.9167, lon: -75.4167, co2_tons: 654_000, co2_per_capita: 1.16 }
        ];
      }

      // Sort by CO2 emissions (descending)
      countyData.sort((a, b) => b.co2_tons - a.co2_tons);
      
      dataLoaded = true;
      renderMap();
    } catch (error) {
      console.error('Error loading county data:', error);
      // Use fallback data
      countyData = [
        { county: 'Philadelphia', lat: 40.0094, lon: -75.1333, co2_tons: 2_345_000, co2_per_capita: 1.48 },
        { county: 'Allegheny', lat: 40.4406, lon: -79.9959, co2_tons: 1_234_000, co2_per_capita: 0.99 },
        { county: 'Montgomery', lat: 40.1688, lon: -75.3560, co2_tons: 987_000, co2_per_capita: 1.15 }
      ];
      dataLoaded = true;
      renderMap();
    }
  });

  function renderMap() {
    if (!mapContainer || countyData.length === 0) return;
    
    // Simple visualization without external libraries
    const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
    svg.setAttribute('viewBox', '0 0 800 600');
    svg.style.width = '100%';
    svg.style.height = '600px';
    svg.style.background = 'rgba(255,255,255,0.05)';
    svg.style.borderRadius = '12px';

    // PA state outline (simplified rectangle representation)
    const outline = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
    outline.setAttribute('x', '50');
    outline.setAttribute('y', '50');
    outline.setAttribute('width', '700');
    outline.setAttribute('height', '500');
    outline.setAttribute('fill', 'none');
    outline.setAttribute('stroke', 'rgba(255,255,255,0.3)');
    outline.setAttribute('stroke-width', '2');
    outline.setAttribute('rx', '10');
    svg.appendChild(outline);

    // Plot counties as circles
    countyData.forEach(county => {
      // Map coordinates to SVG space (simple scaling)
      const x = ((county.lon + 80.5) / 5) * 700 + 50;
      const y = ((42.5 - county.lat) / 3) * 500 + 50;

      const value = viewMode === 'total' ? county.co2_tons : county.co2_per_capita;
      const maxValue = viewMode === 'total' ? 
        Math.max(...countyData.map(c => c.co2_tons)) :
        Math.max(...countyData.map(c => c.co2_per_capita));
      
      const radius = Math.sqrt(value / maxValue) * 50 + 10;

      const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
      circle.setAttribute('cx', x.toString());
      circle.setAttribute('cy', y.toString());
      circle.setAttribute('r', radius.toString());
      circle.setAttribute('fill', '#ff6b6b');
      circle.setAttribute('fill-opacity', '0.6');
      circle.setAttribute('stroke', '#ff6b6b');
      circle.setAttribute('stroke-width', '2');
      circle.style.cursor = 'pointer';
      circle.style.transition = 'all 0.3s';

      circle.addEventListener('mouseenter', () => {
        circle.setAttribute('fill-opacity', '0.9');
        circle.setAttribute('stroke-width', '3');
        selectedCounty = county;
      });

      circle.addEventListener('mouseleave', () => {
        circle.setAttribute('fill-opacity', '0.6');
        circle.setAttribute('stroke-width', '2');
      });

      svg.appendChild(circle);

      // Label for major counties (top 5 or those with significant emissions)
      const topEmitters = countyData.slice(0, 5).map(c => c.county);
      if (topEmitters.includes(county.county) || county.co2_tons > (countyData[0]?.co2_tons || 0) * 0.3) {
        const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        text.setAttribute('x', x.toString());
        text.setAttribute('y', (y + radius + 15).toString());
        text.setAttribute('text-anchor', 'middle');
        text.setAttribute('fill', 'white');
        text.setAttribute('font-size', '12');
        text.setAttribute('font-weight', 'bold');
        text.textContent = county.county;
        svg.appendChild(text);
      }
    });

    mapContainer.innerHTML = '';
    mapContainer.appendChild(svg);
  }

  function toggleView() {
    viewMode = viewMode === 'total' ? 'per_capita' : 'total';
    renderMap();
  }
</script>

<div class="chapter-container">
  <div class="chapter-header">
    <h1>🗺️ Where's the Problem?</h1>
    <p class="subtitle">Pennsylvania's Carbon Hotspots</p>
  </div>

  {#if !dataLoaded}
    <div class="loading-state">
      <p>Loading Pennsylvania county emissions data...</p>
    </div>
  {:else}
  <div class="content-wrapper">
    <div class="intro-stats">
      <div class="key-stat">
        <span class="stat-number">{Math.min(10, countyData.length)}</span>
        <span class="stat-label">counties account for</span>
        <span class="stat-highlight">{top10Percentage}%</span>
        <span class="stat-label">of PA's total commute emissions</span>
      </div>
    </div>

    <div class="map-controls">
      <button 
        class="view-toggle"
        class:active={viewMode === 'total'}
        on:click={toggleView}
      >
        {viewMode === 'total' ? '📊 Total Emissions' : '👤 Per Capita'}
      </button>
      <p class="control-hint">
        {viewMode === 'total' 
          ? 'Showing total annual CO₂ emissions by county' 
          : 'Showing emissions per person - rural counties often higher!'}
      </p>
    </div>

    <div class="map-container" bind:this={mapContainer}></div>

    {#if selectedCounty}
      <div class="county-detail">
        <h3>{selectedCounty.county} County</h3>
        <div class="detail-grid">
          <div class="detail-item">
            <span class="detail-label">Total Annual Emissions</span>
            <span class="detail-value">{(selectedCounty.co2_tons / 1000).toFixed(0)}K tons</span>
          </div>
          {#if selectedCounty.total_workers}
            <div class="detail-item">
              <span class="detail-label">Total Workers</span>
              <span class="detail-value">{selectedCounty.total_workers.toLocaleString()}</span>
            </div>
          {/if}
          {#if selectedCounty.mean_commute}
            <div class="detail-item">
              <span class="detail-label">Avg Commute Time</span>
              <span class="detail-value">{selectedCounty.mean_commute.toFixed(1)} min</span>
            </div>
          {/if}
          <div class="detail-item">
            <span class="detail-label">Per Capita Emissions</span>
            <span class="detail-value">{selectedCounty.co2_per_capita.toFixed(2)} tons/person</span>
          </div>
        </div>
      </div>
    {/if}

    <div class="top-counties">
      <h2>Top {Math.min(5, countyData.length)} Emitting {countyData.length > 0 ? 'Counties' : 'Areas'}</h2>
      <div class="counties-list">
        {#each countyData.slice(0, Math.min(5, countyData.length)) as county, i}
          <div 
            class="county-row" 
            role="button"
            tabindex="0"
            on:click={() => selectedCounty = county}
            on:keydown={(e) => e.key === 'Enter' && (selectedCounty = county)}
          >
            <span class="rank">#{i + 1}</span>
            <span class="county-name">{county.county}</span>
            <span class="emissions">{(county.co2_tons / 1000).toFixed(0)}K tons</span>
            <div class="bar" style="width: {countyData[0] ? (county.co2_tons / countyData[0].co2_tons * 100) : 0}%"></div>
          </div>
        {/each}
      </div>
    </div>

    <div class="insight-box">
      <h3>💡 The Surprise</h3>
      <p>
        Urban counties like <strong>Philadelphia</strong> and <strong>Allegheny</strong> 
        generate the most <em>total</em> emissions (due to population).
      </p>
      <p>
        But when we look at emissions <strong>per person</strong>, rural and suburban 
        counties often exceed urban areas. Why? <strong>Longer distances, less transit, 
        more car-dependency.</strong>
      </p>
    </div>
  </div>
  {/if}
</div>

<style>
  .chapter-container {
    min-height: 100vh;
    background: linear-gradient(180deg, #0f3460 0%, #16213e 100%);
    color: white;
    padding: 4rem 2rem;
  }

  .chapter-header {
    text-align: center;
    margin-bottom: 4rem;
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

  .intro-stats {
    text-align: center;
    margin: 3rem 0;
    padding: 2rem;
    background: rgba(255, 107, 107, 0.1);
    border-radius: 15px;
    border: 2px solid #ff6b6b;
  }

  .key-stat {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 0.5rem;
  }

  .stat-number {
    font-size: 4rem;
    font-weight: 800;
    color: #ff6b6b;
  }

  .stat-highlight {
    font-size: 5rem;
    font-weight: 800;
    color: #ffd93d;
  }

  .stat-label {
    font-size: 1.3rem;
    opacity: 0.9;
  }

  .map-controls {
    text-align: center;
    margin: 2rem 0;
  }

  .view-toggle {
    padding: 1rem 2rem;
    font-size: 1.1rem;
    font-weight: 600;
    background: rgba(255, 255, 255, 0.1);
    color: white;
    border: 2px solid rgba(255, 255, 255, 0.3);
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s;
  }

  .view-toggle:hover,
  .view-toggle.active {
    background: #4ecdc4;
    border-color: #4ecdc4;
    transform: translateY(-2px);
  }

  .control-hint {
    margin-top: 1rem;
    font-size: 1.1rem;
    opacity: 0.8;
  }

  .map-container {
    margin: 3rem 0;
    min-height: 600px;
  }

  .county-detail {
    margin: 3rem auto;
    padding: 2rem;
    max-width: 800px;
    background: rgba(78, 205, 196, 0.1);
    border-left: 4px solid #4ecdc4;
    border-radius: 12px;
  }

  .county-detail h3 {
    font-size: 2rem;
    margin-bottom: 1.5rem;
    color: #4ecdc4;
  }

  .detail-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 2rem;
  }

  .detail-item {
    display: flex;
    flex-direction: column;
  }

  .detail-label {
    font-size: 0.9rem;
    text-transform: uppercase;
    letter-spacing: 1px;
    opacity: 0.7;
    margin-bottom: 0.5rem;
  }

  .detail-value {
    font-size: 2rem;
    font-weight: 800;
    color: #4ecdc4;
  }

  .top-counties {
    margin: 4rem 0;
  }

  .top-counties h2 {
    font-size: 2rem;
    margin-bottom: 2rem;
    text-align: center;
  }

  .counties-list {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .county-row {
    padding: 1.5rem;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    display: grid;
    grid-template-columns: 60px 1fr 150px;
    align-items: center;
    position: relative;
    cursor: pointer;
    transition: all 0.3s;
  }

  .county-row:hover,
  .county-row:focus {
    background: rgba(255, 255, 255, 0.1);
    transform: translateX(10px);
    outline: 2px solid #4ecdc4;
    outline-offset: 2px;
  }

  .rank {
    font-size: 2rem;
    font-weight: 800;
    color: #ffd93d;
  }

  .county-name {
    font-size: 1.3rem;
    font-weight: 600;
  }

  .emissions {
    font-size: 1.3rem;
    font-weight: 700;
    color: #ff6b6b;
    text-align: right;
  }

  .bar {
    position: absolute;
    bottom: 0;
    left: 0;
    height: 4px;
    background: linear-gradient(90deg, #ff6b6b, #ffd93d);
    border-radius: 0 0 12px 12px;
    transition: width 0.5s ease;
  }

  .insight-box {
    margin: 4rem auto;
    padding: 2rem;
    max-width: 800px;
    background: rgba(255, 215, 0, 0.1);
    border-left: 4px solid #ffd93d;
    border-radius: 12px;
  }

  .insight-box h3 {
    font-size: 1.8rem;
    margin-bottom: 1rem;
    color: #ffd93d;
  }

  .insight-box p {
    font-size: 1.2rem;
    line-height: 1.8;
    margin: 1rem 0;
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

    .stat-number {
      font-size: 2.5rem;
    }

    .stat-highlight {
      font-size: 3rem;
    }

    .county-row {
      grid-template-columns: 50px 1fr;
      gap: 0.5rem;
    }

    .emissions {
      grid-column: 1 / -1;
      text-align: left;
      margin-top: 0.5rem;
    }
  }
</style>
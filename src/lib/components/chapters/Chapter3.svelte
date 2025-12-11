<script lang="ts">
  import { onMount } from 'svelte';

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
  let dataLoaded: boolean = false;
  let selectedMetro: MetroData | null = null;
  let chartContainer: HTMLDivElement;

  onMount(async () => {
    try {
      const metroResponse = await fetch('/pa_metro_comparison.json');
      const metroDataResponse = await metroResponse.json();
      metroData = (metroDataResponse || []) as MetroData[];
      dataLoaded = true;
      
      // Render chart after data loads
      setTimeout(() => renderChart(), 100);
    } catch (error) {
      console.error('Error loading metro data:', error);
      dataLoaded = true;
    }
  });

  function selectMetro(metro: MetroData) {
    selectedMetro = selectedMetro?.metro === metro.metro ? null : metro;
    renderChart();
  }

  function renderChart() {
    if (!chartContainer || metroData.length === 0) return;

    const svgWidth = 900;
    const svgHeight = 500;
    const margin = { top: 60, right: 50, bottom: 100, left: 80 };
    const chartWidth = svgWidth - margin.left - margin.right;
    const chartHeight = svgHeight - margin.top - margin.bottom;
    const barWidth = chartWidth / (metroData.length * 2);
    const barSpacing = barWidth * 0.3;

    const svg = document.createElementNS('http://www.w3.org/2000/svg', 'svg');
    svg.setAttribute('viewBox', `0 0 ${svgWidth} ${svgHeight}`);
    svg.style.width = '100%';
    svg.style.height = `${svgHeight}px`;
    svg.style.background = 'rgba(255,255,255,0.05)';
    svg.style.borderRadius = '12px';

    // Create group for chart
    const chartGroup = document.createElementNS('http://www.w3.org/2000/svg', 'g');
    chartGroup.setAttribute('transform', `translate(${margin.left},${margin.top})`);

    // Get max values for scaling
    const maxCommute = Math.max(...metroData.map(m => m.avg_commute));
    const maxCO2 = Math.max(...metroData.map(m => m.co2_per_capita));
    const maxTransit = Math.max(...metroData.map(m => m.transit_pct));

    // Draw axes
    const xAxis = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    xAxis.setAttribute('x1', '0');
    xAxis.setAttribute('y1', String(chartHeight));
    xAxis.setAttribute('x2', String(chartWidth));
    xAxis.setAttribute('y2', String(chartHeight));
    xAxis.setAttribute('stroke', 'rgba(255,255,255,0.5)');
    xAxis.setAttribute('stroke-width', '2');
    chartGroup.appendChild(xAxis);

    const yAxis = document.createElementNS('http://www.w3.org/2000/svg', 'line');
    yAxis.setAttribute('x1', '0');
    yAxis.setAttribute('y1', '0');
    yAxis.setAttribute('x2', '0');
    yAxis.setAttribute('y2', String(chartHeight));
    yAxis.setAttribute('stroke', 'rgba(255,255,255,0.5)');
    yAxis.setAttribute('stroke-width', '2');
    chartGroup.appendChild(yAxis);

    // Y-axis ticks and labels for Commute Time
    const commuteTickCount = 5;
    for (let i = 0; i <= commuteTickCount; i++) {
      const value = (maxCommute / commuteTickCount) * i;
      const y = chartHeight - (value / maxCommute) * chartHeight;
      
      const tick = document.createElementNS('http://www.w3.org/2000/svg', 'line');
      tick.setAttribute('x1', '-5');
      tick.setAttribute('y1', String(y));
      tick.setAttribute('x2', '0');
      tick.setAttribute('y2', String(y));
      tick.setAttribute('stroke', 'rgba(255,255,255,0.4)');
      tick.setAttribute('stroke-width', '1');
      chartGroup.appendChild(tick);
      
      const label = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      label.setAttribute('x', '-10');
      label.setAttribute('y', String(y + 4));
      label.setAttribute('text-anchor', 'end');
      label.setAttribute('fill', 'rgba(255,255,255,0.8)');
      label.setAttribute('font-size', '12');
      label.textContent = value.toFixed(0);
      chartGroup.appendChild(label);
    }

    // Draw bars for each metro
    metroData.forEach((metro, index) => {
      const x = (index * chartWidth / metroData.length) + (chartWidth / metroData.length / 2) - (barWidth / 2);
      const isSelected = selectedMetro?.metro === metro.metro;

      // Bar 1: Average Commute Time
      const commuteHeight = (metro.avg_commute / maxCommute) * chartHeight;
      const commuteBar = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
      commuteBar.setAttribute('x', String(x));
      commuteBar.setAttribute('y', String(chartHeight - commuteHeight));
      commuteBar.setAttribute('width', String(barWidth - barSpacing));
      commuteBar.setAttribute('height', String(commuteHeight));
      commuteBar.setAttribute('fill', isSelected ? '#4ecdc4' : '#4ecdc4');
      commuteBar.setAttribute('fill-opacity', isSelected ? '0.9' : '0.6');
      commuteBar.setAttribute('stroke', '#4ecdc4');
      commuteBar.setAttribute('stroke-width', isSelected ? '3' : '1');
      commuteBar.style.cursor = 'pointer';
      commuteBar.addEventListener('click', () => selectMetro(metro));
      chartGroup.appendChild(commuteBar);

      // Bar 2: CO2 per Capita
      const co2Height = (metro.co2_per_capita / maxCO2) * chartHeight;
      const co2Bar = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
      co2Bar.setAttribute('x', String(x + barWidth - barSpacing));
      co2Bar.setAttribute('y', String(chartHeight - co2Height));
      co2Bar.setAttribute('width', String(barWidth - barSpacing));
      co2Bar.setAttribute('height', String(co2Height));
      co2Bar.setAttribute('fill', isSelected ? '#ff6b6b' : '#ff6b6b');
      co2Bar.setAttribute('fill-opacity', isSelected ? '0.9' : '0.6');
      co2Bar.setAttribute('stroke', '#ff6b6b');
      co2Bar.setAttribute('stroke-width', isSelected ? '3' : '1');
      co2Bar.style.cursor = 'pointer';
      co2Bar.addEventListener('click', () => selectMetro(metro));
      chartGroup.appendChild(co2Bar);

      // Metro name label
      const nameLabel = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      nameLabel.setAttribute('x', String(x + barWidth / 2));
      nameLabel.setAttribute('y', String(chartHeight + 25));
      nameLabel.setAttribute('text-anchor', 'middle');
      nameLabel.setAttribute('fill', isSelected ? '#ffd93d' : 'rgba(255,255,255,0.9)');
      nameLabel.setAttribute('font-size', '14');
      nameLabel.setAttribute('font-weight', isSelected ? 'bold' : '600');
      nameLabel.style.cursor = 'pointer';
      nameLabel.addEventListener('click', () => selectMetro(metro));
      nameLabel.textContent = metro.metro;
      chartGroup.appendChild(nameLabel);

      // Value labels on bars
      const commuteLabel = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      commuteLabel.setAttribute('x', String(x + (barWidth - barSpacing) / 2));
      commuteLabel.setAttribute('y', String(chartHeight - commuteHeight - 5));
      commuteLabel.setAttribute('text-anchor', 'middle');
      commuteLabel.setAttribute('fill', '#ffffff');
      commuteLabel.setAttribute('font-size', '11');
      commuteLabel.setAttribute('font-weight', '600');
      commuteLabel.textContent = metro.avg_commute.toFixed(1);
      chartGroup.appendChild(commuteLabel);

      const co2Label = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      co2Label.setAttribute('x', String(x + barWidth - barSpacing + (barWidth - barSpacing) / 2));
      co2Label.setAttribute('y', String(chartHeight - co2Height - 5));
      co2Label.setAttribute('text-anchor', 'middle');
      co2Label.setAttribute('fill', '#ffffff');
      co2Label.setAttribute('font-size', '11');
      co2Label.setAttribute('font-weight', '600');
      co2Label.textContent = metro.co2_per_capita.toFixed(2);
      chartGroup.appendChild(co2Label);
    });

    // Y-axis title (centered in left margin, rotated -90 degrees)
    const yAxisTitle1 = document.createElementNS('http://www.w3.org/2000/svg', 'text');
    const yAxisTitleX = -40; // Middle of left margin (80px / 2)
    const yAxisTitleY = chartHeight / 2; // Middle of chart height
    yAxisTitle1.setAttribute('x', String(yAxisTitleX));
    yAxisTitle1.setAttribute('y', String(yAxisTitleY));
    yAxisTitle1.setAttribute('text-anchor', 'middle');
    yAxisTitle1.setAttribute('fill', 'rgba(255,255,255,0.9)');
    yAxisTitle1.setAttribute('font-size', '13');
    yAxisTitle1.setAttribute('font-weight', '600');
    yAxisTitle1.setAttribute('transform', `rotate(-90, ${yAxisTitleX}, ${yAxisTitleY})`);
    yAxisTitle1.textContent = 'Average Commute Time (min) / CO₂ per Capita (tons)';
    chartGroup.appendChild(yAxisTitle1);

    // Legend
    const legendY = -30;
    const legendItems = [
      { color: '#4ecdc4', label: 'Commute Time (min)', x: 20 },
      { color: '#ff6b6b', label: 'CO₂ per Capita (tons)', x: 200 }
    ];
    legendItems.forEach(item => {
      const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
      rect.setAttribute('x', String(item.x));
      rect.setAttribute('y', String(legendY - 8));
      rect.setAttribute('width', '15');
      rect.setAttribute('height', '15');
      rect.setAttribute('fill', item.color);
      chartGroup.appendChild(rect);
      
      const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
      text.setAttribute('x', String(item.x + 20));
      text.setAttribute('y', String(legendY + 4));
      text.setAttribute('fill', 'rgba(255,255,255,0.9)');
      text.setAttribute('font-size', '12');
      text.textContent = item.label;
      chartGroup.appendChild(text);
    });

    svg.appendChild(chartGroup);
    chartContainer.innerHTML = '';
    chartContainer.appendChild(svg);
  }

  $: if (dataLoaded && chartContainer) {
    renderChart();
  }
</script>

<div class="chapter-container">
  <div class="chapter-header">
    <h1>🏙️ The Metro Divide</h1>
    <p class="subtitle">Pennsylvania's Four Major Cities Tell Different Stories</p>
  </div>

  {#if !dataLoaded}
    <div class="loading-state">
      <p>Loading Pennsylvania metro comparison data...</p>
    </div>
  {:else}
  <div class="content-wrapper">
    <div class="intro-section">
      <p class="intro-text">
        Pennsylvania's four major metros reveal a surprising pattern: <strong>longer commutes don't always mean more pollution.</strong>
        Transit infrastructure makes all the difference.
      </p>
    </div>

    <div class="chart-section">
      <div class="chart-container" bind:this={chartContainer}></div>
    </div>

    <div class="metro-comparison-grid">
      {#each metroData as metro}
        <div 
          class="metro-card"
          class:selected={selectedMetro?.metro === metro.metro}
          on:click={() => selectMetro(metro)}
          role="button"
          tabindex="0"
          on:keydown={(e) => e.key === 'Enter' && selectMetro(metro)}
        >
          <h3>{metro.metro}</h3>
          <div class="metro-stats">
            <div class="stat-item">
              <span class="stat-label">Avg Commute</span>
              <span class="stat-value">{metro.avg_commute.toFixed(1)} min</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">Transit Use</span>
              <span class="stat-value">{metro.transit_pct}%</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">CO₂ per Person</span>
              <span class="stat-value">{metro.co2_per_capita.toFixed(2)} tons</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">Median Income</span>
              <span class="stat-value">${metro.median_income.toLocaleString()}</span>
            </div>
          </div>
          {#if selectedMetro?.metro === metro.metro}
            <div class="burden-score">
              <span class="burden-label">Commute Burden Score</span>
              <span class="burden-value">{metro.commute_burden_score.toFixed(1)}</span>
              <p class="burden-note">Time vs. Income ratio (lower is better)</p>
            </div>
          {/if}
        </div>
      {/each}
    </div>

    <div class="story-section">
      <h2>The Story</h2>
      <div class="story-content">
        <div class="story-point">
          <h3>🏛️ Philadelphia</h3>
          <p>
            <strong>{metroData.find(m => m.metro === 'Philadelphia')?.avg_commute.toFixed(1)} min</strong> average commute, 
            <strong>{metroData.find(m => m.metro === 'Philadelphia')?.transit_pct}%</strong> transit use, 
            <strong>{metroData.find(m => m.metro === 'Philadelphia')?.co2_per_capita.toFixed(2)} tons</strong> CO₂/person
          </p>
          <p>
            Philadelphia has the <strong>longest commutes</strong> but <strong>lowest per-capita emissions</strong> - 
            thanks to its robust transit system. The SEPTA network keeps cars off the road.
          </p>
        </div>

        <div class="story-point">
          <h3>🏭 Pittsburgh</h3>
          <p>
            <strong>{metroData.find(m => m.metro === 'Pittsburgh')?.avg_commute.toFixed(1)} min</strong> average commute, 
            <strong>{metroData.find(m => m.metro === 'Pittsburgh')?.transit_pct}%</strong> transit use, 
            <strong>{metroData.find(m => m.metro === 'Pittsburgh')?.co2_per_capita.toFixed(2)} tons</strong> CO₂/person
          </p>
          <p>
            Pittsburgh's shorter commutes are offset by <strong>lower transit adoption</strong>. 
            More car-dependency means higher emissions per person despite shorter distances.
          </p>
        </div>

        <div class="story-point">
          <h3>🏛️ Harrisburg</h3>
          <p>
            <strong>{metroData.find(m => m.metro === 'Harrisburg')?.avg_commute.toFixed(1)} min</strong> average commute, 
            <strong>{metroData.find(m => m.metro === 'Harrisburg')?.transit_pct}%</strong> transit use, 
            <strong>{metroData.find(m => m.metro === 'Harrisburg')?.co2_per_capita.toFixed(2)} tons</strong> CO₂/person
          </p>
          <p>
            The state capital has the <strong>shortest commutes</strong> but limited transit options. 
            Lower emissions reflect shorter distances, but there's room for transit growth.
          </p>
        </div>

        <div class="story-point">
          <h3>🏭 Allentown</h3>
          <p>
            <strong>{metroData.find(m => m.metro === 'Allentown')?.avg_commute.toFixed(1)} min</strong> average commute, 
            <strong>{metroData.find(m => m.metro === 'Allentown')?.transit_pct}%</strong> transit use, 
            <strong>{metroData.find(m => m.metro === 'Allentown')?.co2_per_capita.toFixed(2)} tons</strong> CO₂/person
          </p>
          <p>
            Allentown shows moderate commute times with <strong>minimal transit infrastructure</strong>. 
            This metro area represents the typical PA pattern: car-dependent suburban sprawl.
          </p>
        </div>
      </div>

      <div class="insight-box">
        <h3>💡 The Key Insight</h3>
        <p>
          <strong>Philadelphia has the longest commutes but lowest per-capita emissions</strong> - thanks to its transit system. 
          Other PA metros drive much more, even though commutes are shorter. This proves that <strong>transit infrastructure, 
          not just commute distance, determines environmental impact.</strong>
        </p>
      </div>
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

  .chart-section {
    margin: 4rem 0;
    padding: 2rem;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 15px;
  }

  .chart-container {
    min-height: 500px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .metro-comparison-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 2rem;
    margin: 4rem 0;
  }

  .metro-card {
    padding: 2rem;
    background: rgba(255, 255, 255, 0.08);
    border-radius: 12px;
    border: 2px solid rgba(255, 255, 255, 0.2);
    cursor: pointer;
    transition: all 0.3s;
  }

  .metro-card:hover,
  .metro-card:focus {
    background: rgba(255, 255, 255, 0.12);
    transform: translateY(-5px);
    border-color: #4ecdc4;
    outline: 2px solid #4ecdc4;
    outline-offset: 2px;
  }

  .metro-card.selected {
    background: rgba(78, 205, 196, 0.15);
    border-color: #4ecdc4;
    box-shadow: 0 4px 20px rgba(78, 205, 196, 0.3);
  }

  .metro-card h3 {
    font-size: 1.8rem;
    margin: 0 0 1.5rem 0;
    color: #4ecdc4;
    text-align: center;
  }

  .metro-stats {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .stat-item {
    display: flex;
    justify-content: space-between;
    padding: 0.75rem 0;
    border-bottom: 1px solid rgba(255, 255, 255, 0.1);
  }

  .stat-item:last-child {
    border-bottom: none;
  }

  .stat-label {
    font-size: 0.9rem;
    opacity: 0.8;
  }

  .stat-value {
    font-size: 1.1rem;
    font-weight: 700;
    color: #ffd93d;
  }

  .burden-score {
    margin-top: 1.5rem;
    padding: 1rem;
    background: rgba(255, 215, 0, 0.15);
    border-radius: 8px;
    text-align: center;
  }

  .burden-label {
    display: block;
    font-size: 0.85rem;
    opacity: 0.8;
    margin-bottom: 0.5rem;
  }

  .burden-value {
    display: block;
    font-size: 2rem;
    font-weight: 800;
    color: #ffd93d;
    margin-bottom: 0.5rem;
  }

  .burden-note {
    font-size: 0.8rem;
    opacity: 0.7;
    margin: 0;
  }

  .story-section {
    margin: 4rem 0;
  }

  .story-section h2 {
    font-size: 2.5rem;
    text-align: center;
    margin-bottom: 3rem;
    color: #ffd93d;
  }

  .story-content {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 2rem;
    margin-bottom: 3rem;
  }

  .story-point {
    padding: 1.5rem;
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    border-left: 4px solid #4ecdc4;
  }

  .story-point h3 {
    font-size: 1.5rem;
    margin: 0 0 1rem 0;
    color: #4ecdc4;
  }

  .story-point p {
    font-size: 1.1rem;
    line-height: 1.7;
    margin: 0.75rem 0;
  }

  .insight-box {
    margin: 3rem auto;
    padding: 2rem;
    max-width: 800px;
    background: rgba(255, 215, 0, 0.1);
    border-left: 4px solid #ffd93d;
    border-radius: 12px;
  }

  .insight-box h3 {
    font-size: 1.8rem;
    margin: 0 0 1rem 0;
    color: #ffd93d;
  }

  .insight-box p {
    font-size: 1.2rem;
    line-height: 1.8;
    margin: 0;
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

    .metro-comparison-grid {
      grid-template-columns: 1fr;
    }

    .story-content {
      grid-template-columns: 1fr;
    }

    .chart-container {
      min-height: 400px;
    }
  }
</style>

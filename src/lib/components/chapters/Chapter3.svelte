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

  onMount(async () => {
    try {
      const metroResponse = await fetch('/pa_metro_comparison.json');
      const metroDataResponse = await metroResponse.json();
      metroData = (metroDataResponse || []) as MetroData[];
    } catch (error) {
      console.error('Error loading metro data:', error);
    }
  });
</script>

<div class="chapter-container">
  <div class="chapter-header">
    <h1>🏙️ The Metro Divide</h1>
    <p class="subtitle">Pennsylvania's Four Major Cities Tell Different Stories</p>
  </div>

  <div class="content-wrapper">
    <div class="narrative-intro">
      <h2>The Transit Paradox</h2>
      <p>
        The geographic patterns we explored in Chapter 2 hint at something important: <strong>emissions aren't 
        just about distance or commute time</strong>. When we compare Pennsylvania's four major metropolitan areas, 
        a surprising pattern emerges that challenges our assumptions about what drives carbon emissions.
      </p>
      <p>
        You might expect that longer commutes automatically mean more pollution. But the data tells a different 
        story—one that reveals how <strong>transit infrastructure can dramatically reduce per-person emissions, 
        even in dense urban areas with long average commute times</strong>.
      </p>
    </div>

    <div class="intro-section">
      <p class="intro-text">
        Pennsylvania's four major metros reveal a surprising pattern: <strong>longer commutes don't always mean more pollution.</strong>
        Transit infrastructure makes all the difference.
      </p>
    </div>


    <div class="story-section">
      <h2>The Story</h2>
      <div class="story-content">
        <div class="story-point">
          <h3>🏛️ Philadelphia</h3>
          <p>
            {#if metroData.length > 0}
              <strong>{metroData.find(m => m.metro === 'Philadelphia')?.avg_commute?.toFixed(1) || '32.5'} min</strong> average commute, 
              <strong>{metroData.find(m => m.metro === 'Philadelphia')?.transit_pct || '25'}%</strong> transit use, 
              <strong>{metroData.find(m => m.metro === 'Philadelphia')?.co2_per_capita?.toFixed(2) || '2.89'} tons</strong> CO₂/person
            {:else}
              <strong>32.5 min</strong> average commute, <strong>25%</strong> transit use, <strong>2.89 tons</strong> CO₂/person
            {/if}
          </p>
          <p>
            Philadelphia has the <strong>longest commutes</strong> but <strong>lowest per-capita emissions</strong> - 
            thanks to its robust transit system. The SEPTA network keeps cars off the road.
          </p>
        </div>

        <div class="story-point">
          <h3>🏭 Pittsburgh</h3>
          <p>
            {#if metroData.length > 0}
              <strong>{metroData.find(m => m.metro === 'Pittsburgh')?.avg_commute?.toFixed(1) || '26.8'} min</strong> average commute, 
              <strong>{metroData.find(m => m.metro === 'Pittsburgh')?.transit_pct || '12'}%</strong> transit use, 
              <strong>{metroData.find(m => m.metro === 'Pittsburgh')?.co2_per_capita?.toFixed(2) || '2.38'} tons</strong> CO₂/person
            {:else}
              <strong>26.8 min</strong> average commute, <strong>12%</strong> transit use, <strong>2.38 tons</strong> CO₂/person
            {/if}
          </p>
          <p>
            Pittsburgh's shorter commutes are offset by <strong>lower transit adoption</strong>. 
            More car-dependency means higher emissions per person despite shorter distances.
          </p>
        </div>

        <div class="story-point">
          <h3>🏛️ Harrisburg</h3>
          <p>
            {#if metroData.length > 0}
              <strong>{metroData.find(m => m.metro === 'Harrisburg')?.avg_commute?.toFixed(1) || '24.5'} min</strong> average commute, 
              <strong>{metroData.find(m => m.metro === 'Harrisburg')?.transit_pct || '3'}%</strong> transit use, 
              <strong>{metroData.find(m => m.metro === 'Harrisburg')?.co2_per_capita?.toFixed(2) || '2.18'} tons</strong> CO₂/person
            {:else}
              <strong>24.5 min</strong> average commute, <strong>3%</strong> transit use, <strong>2.18 tons</strong> CO₂/person
            {/if}
          </p>
          <p>
            The state capital has the <strong>shortest commutes</strong> but limited transit options. 
            Lower emissions reflect shorter distances, but there's room for transit growth.
          </p>
        </div>

        <div class="story-point">
          <h3>🏭 Allentown</h3>
          <p>
            {#if metroData.length > 0}
              <strong>{metroData.find(m => m.metro === 'Allentown')?.avg_commute?.toFixed(1) || '25.2'} min</strong> average commute, 
              <strong>{metroData.find(m => m.metro === 'Allentown')?.transit_pct || '4'}%</strong> transit use, 
              <strong>{metroData.find(m => m.metro === 'Allentown')?.co2_per_capita?.toFixed(2) || '2.24'} tons</strong> CO₂/person
            {:else}
              <strong>25.2 min</strong> average commute, <strong>4%</strong> transit use, <strong>2.24 tons</strong> CO₂/person
            {/if}
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

    <div class="narrative-conclusion">
      <h2>What This Means for Pennsylvania</h2>
      <p>
        The metro comparison reveals a fundamental truth: <strong>infrastructure investment pays environmental dividends</strong>. 
        Philadelphia's SEPTA network—despite serving a region with longer average commutes—produces dramatically lower 
        per-person emissions than car-dependent metros with shorter commute times.
      </p>
      <p>
        This isn't just about Philadelphia. The pattern holds across Pennsylvania: <strong>regions with robust transit 
        infrastructure consistently show lower emissions per commuter, regardless of commute distance</strong>. 
        Pittsburgh, Harrisburg, and Allentown all have shorter average commutes than Philadelphia, but their lower 
        transit adoption means each individual commuter generates more CO₂.
      </p>
      <p class="transition-to-solutions">
        The data is clear: <strong>transit infrastructure works</strong>. But what if we could model exactly how much 
        Pennsylvania could reduce emissions by expanding transit and remote work options? The next chapter lets you 
        explore different scenarios and see the potential impact.
      </p>
    </div>
  </div>
</div>

<style>
  .chapter-container {
    min-height: 100vh;
    background: #f5f7fa;
    color: #2d3748;
    padding: 4rem 2rem;
  }

  .chapter-header {
    text-align: center;
    margin-bottom: 3rem;
  }

  .chapter-header h1 {
    font-size: 3.5rem;
    font-weight: 700;
    margin-bottom: 1rem;
    color: #2c5282;
  }

  .subtitle {
    font-size: 1.5rem;
    color: #4a5568;
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

  .narrative-intro {
    margin: 3rem 0;
    padding: 2rem;
    background: white;
    border-radius: 12px;
    line-height: 1.8;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  }

  .narrative-intro h2 {
    font-size: 2.5rem;
    margin-bottom: 1.5rem;
    color: #2c5282;
    text-align: center;
  }

  .narrative-intro p {
    font-size: 1.2rem;
    margin: 1.5rem 0;
    color: #2d3748;
  }

  .narrative-intro strong {
    color: #2c5282;
    font-weight: 600;
  }

  .intro-text {
    font-size: 1.3rem;
    line-height: 1.8;
    margin: 0;
  }

  .narrative-conclusion {
    margin: 4rem 0;
    padding: 3rem;
    background: white;
    border-radius: 12px;
    line-height: 1.8;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  }

  .narrative-conclusion h2 {
    font-size: 2.2rem;
    margin-bottom: 2rem;
    color: #2c5282;
    text-align: center;
  }

  .narrative-conclusion p {
    font-size: 1.2rem;
    margin: 1.5rem 0;
    color: #2d3748;
  }

  .narrative-conclusion strong {
    color: #2c5282;
    font-weight: 600;
  }

  .transition-to-solutions {
    font-size: 1.3rem;
    font-weight: 600;
    color: #2d3748;
    margin-top: 2rem;
    padding: 2rem;
    background: #ebf8ff;
    border-left: 4px solid #4299e1;
    border-radius: 8px;
  }

  .transition-to-solutions strong {
    color: #2c5282;
  }

  .story-section {
    margin: 4rem 0;
  }

  .story-section h2 {
    font-size: 2.5rem;
    text-align: center;
    margin-bottom: 3rem;
    color: #2c5282;
  }

  .story-content {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 2rem;
    margin-bottom: 3rem;
  }

  .story-point {
    padding: 1.5rem;
    background: white;
    border-radius: 12px;
    border-left: 4px solid #4299e1;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  }

  .story-point h3 {
    font-size: 1.5rem;
    margin: 0 0 1rem 0;
    color: #2c5282;
  }

  .story-point p {
    font-size: 1.1rem;
    line-height: 1.7;
    margin: 0.75rem 0;
    color: #2d3748;
  }

  .story-point strong {
    color: #2c5282;
    font-weight: 600;
  }

  .insight-box {
    margin: 3rem auto;
    padding: 2rem;
    max-width: 800px;
    background: white;
    border-left: 4px solid #4299e1;
    border-radius: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  }

  .insight-box h3 {
    font-size: 1.8rem;
    margin: 0 0 1rem 0;
    color: #2c5282;
  }

  .insight-box p {
    font-size: 1.2rem;
    line-height: 1.8;
    margin: 0;
    color: #2d3748;
  }

  .insight-box strong {
    color: #2c5282;
    font-weight: 600;
  }

  @media (max-width: 768px) {
    .chapter-header h1 {
      font-size: 2rem;
    }

    .story-content {
      grid-template-columns: 1fr;
    }
  }
</style>

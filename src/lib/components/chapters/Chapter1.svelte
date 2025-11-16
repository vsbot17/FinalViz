<script lang="ts">
  import { fade } from 'svelte/transition';
  
  let commuteTime: number = 30;
  
  $: hoursPerYear = (commuteTime * 2 * 5 * 50) / 60;
  $: fullDays = hoursPerYear / 24;
  $: books = Math.floor(hoursPerYear / 8);
  $: movies = Math.floor(hoursPerYear / 2);
  $: marathons = Math.floor(hoursPerYear / 4);
  $: lifetimeYears = (hoursPerYear * 40) / (24 * 365);
  $: showResults = commuteTime > 0;
</script>

<div class="chapter-container">
  <div class="chapter-header">
    <h1>⏰ The Time Thief</h1>
    <p class="subtitle">How Commutes Steal Your Life</p>
  </div>

  <div class="calculator">
    <h2>How Much Time Does Your Commute Steal?</h2>
    <p>Enter your one-way commute time in minutes:</p>
    
    <div class="input-group">
      <input 
        type="number" 
        bind:value={commuteTime}
        placeholder="30" 
        min="0" 
        max="240"
      />
      <span class="unit">minutes</span>
    </div>
    
    {#if showResults}
      <div class="results" transition:fade>
        <div class="metric">
          <strong>Per Year:</strong>
          <span class="big-number">{hoursPerYear.toFixed(0)}</span>
          <p>hours spent commuting</p>
        </div>
        
        <div class="metric">
          <strong>That's equivalent to:</strong>
          <span class="big-number">{fullDays.toFixed(1)}</span>
          <p>full 24-hour days</p>
        </div>
        
        <div class="metric">
          <strong>You could instead:</strong>
          <ul class="alternatives">
            <li>📚 Read <strong>{books}</strong> books</li>
            <li>🎬 Watch <strong>{movies}</strong> movies</li>
            <li>🏃 Run <strong>{marathons}</strong> marathons</li>
          </ul>
        </div>
        
        <div class="metric lifetime">
          <strong>Over 40-year career:</strong>
          <span class="big-number">{lifetimeYears.toFixed(1)}</span>
          <p>YEARS of your life</p>
        </div>
      </div>
    {/if}
  </div>
</div>

<style>
  .chapter-container {
    min-height: 100vh;
    padding: 2rem;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
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
    max-width: 800px;
    margin: 0 auto;
    background: rgba(255,255,255,0.15);
    padding: 2rem;
    border-radius: 15px;
    backdrop-filter: blur(10px);
  }

  .calculator h2 {
    text-align: center;
    margin-bottom: 1rem;
  }

  .input-group {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin: 2rem 0;
  }

  input {
    width: 200px;
    padding: 1rem;
    font-size: 2rem;
    border: none;
    border-radius: 8px;
    text-align: center;
  }

  .unit {
    font-size: 1.2rem;
    font-weight: bold;
  }

  .results {
    margin-top: 2rem;
  }

  .metric {
    background: rgba(255,255,255,0.2);
    padding: 1.5rem;
    border-radius: 8px;
    margin: 1rem 0;
  }

  .metric strong {
    display: block;
    font-size: 1.2rem;
    margin-bottom: 0.5rem;
  }

  .big-number {
    display: block;
    font-size: 3rem;
    font-weight: bold;
    color: #ffd700;
    margin: 0.5rem 0;
  }

  .metric p {
    font-size: 1.1rem;
    margin: 0;
  }

  .alternatives {
    list-style: none;
    padding: 0;
    margin: 1rem 0;
  }

  .alternatives li {
    padding: 0.5rem 0;
    font-size: 1.1rem;
  }

  .lifetime {
    background: rgba(255, 206, 86, 0.3);
    border: 2px solid #ffd700;
  }

  @media (max-width: 768px) {
    .chapter-header h1 {
      font-size: 2rem;
    }

    .calculator {
      padding: 1rem;
    }

    input {
      width: 150px;
      font-size: 1.5rem;
    }

    .big-number {
      font-size: 2rem;
    }
  }
</style>
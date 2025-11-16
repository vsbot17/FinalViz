<script lang="ts">
  import { onMount } from 'svelte';
  import { currentChapter } from '$lib/stores/navigation';
  import Navigation from '$lib/components/Navigation.svelte';
  import Chapter1 from '$lib/components/chapters/Chapter1.svelte';
  import Chapter2 from '$lib/components/chapters/Chapter2.svelte';
  import Chapter3 from '$lib/components/chapters/Chapter3.svelte';
  // import Chapter4 from '$lib/components/chapters/Chapter4.svelte';
  // import Chapter5 from '$lib/components/chapters/Chapter5.svelte';
  // import Chapter6 from '$lib/components/chapters/Chapter6.svelte';
  // import Chapter7 from '$lib/components/chapters/Chapter7.svelte';
  // import Chapter8 from '$lib/components/chapters/Chapter8.svelte';
  // import Chapter9 from '$lib/components/chapters/Chapter9.svelte';
  // import Chapter10 from '$lib/components/chapters/Chapter10.svelte';

  let chapterElements: (HTMLElement | null)[] = [];
  let scrolling: boolean = false;

  onMount(() => {
    // Smooth scroll to chapter on load
    if ($currentChapter > 1) {
      scrollToChapter($currentChapter - 1);
    }

    // Add scroll listener to update current chapter
    const handleScroll = (): void => {
      if (scrolling) return;

      const scrollPosition = window.scrollY + window.innerHeight / 2;
      
      for (let i = 0; i < chapterElements.length; i++) {
        const element = chapterElements[i];
        if (element) {
          const rect = element.getBoundingClientRect();
          const elementTop = rect.top + window.scrollY;
          const elementBottom = elementTop + rect.height;
          
          if (scrollPosition >= elementTop && scrollPosition < elementBottom) {
            currentChapter.set(i + 1);
            break;
          }
        }
      }
    };

    window.addEventListener('scroll', handleScroll, { passive: true });
    return () => window.removeEventListener('scroll', handleScroll);
  });

  function scrollToChapter(index: number): void {
    if (chapterElements[index]) {
      scrolling = true;
      chapterElements[index]!.scrollIntoView({ 
        behavior: 'smooth', 
        block: 'start' 
      });
      
      setTimeout(() => {
        scrolling = false;
      }, 1000);
    }
  }

  function handleNavigate(chapterId: number): void {
    scrollToChapter(chapterId - 1);
  }
</script>

<svelte:head>
  <title>The Hidden Cost of Commutes - Interactive Visualization</title>
  <meta name="description" content="Explore how commute patterns affect well-being, productivity, and sustainability through interactive data visualizations." />
</svelte:head>

<div class="app">
  <Navigation onNavigate={handleNavigate} />
  
  <main class="main-content">
    <!-- Hero Section -->
    <section class="hero">
      <div class="hero-content">
        <h1 class="hero-title">The Hidden Cost of Commutes</h1>
        <p class="hero-subtitle">
          An interactive exploration of how our daily journeys shape our lives, 
          our planet, and our future
        </p>
        <button class="cta-button" on:click={() => scrollToChapter(0)}>
          Start Exploring ↓
        </button>
      </div>
    </section>

    <!-- Chapter Sections -->
    <section bind:this={chapterElements[0]} class="chapter-section">
      <Chapter1 />
    </section>

    <section bind:this={chapterElements[1]} class="chapter-section">
      <Chapter2 />
    </section>

    <section bind:this={chapterElements[2]} class="chapter-section">
      <Chapter3 />
    </section>

    <!-- <section bind:this={chapterElements[3]} class="chapter-section">
      <Chapter4 />
    </section>

    <section bind:this={chapterElements[4]} class="chapter-section">
      <Chapter5 />
    </section>

    <section bind:this={chapterElements[5]} class="chapter-section">
      <Chapter6 />
    </section>

    <section bind:this={chapterElements[6]} class="chapter-section">
      <Chapter7 />
    </section>

    <section bind:this={chapterElements[7]} class="chapter-section">
      <Chapter8 />
    </section>

    <section bind:this={chapterElements[8]} class="chapter-section">
      <Chapter9 />
    </section>

    <section bind:this={chapterElements[9]} class="chapter-section">
      <Chapter10 />
    </section> -->

    <!-- Footer -->
    <footer class="footer">
      <div class="footer-content">
        <h3>Take Action</h3>
        <p>
          Now that you understand the true cost of commuting, it's time to make a change.
          Small decisions can lead to big impacts on your life and the planet.
        </p>
        <div class="footer-links">
          <button on:click={() => scrollToChapter(0)}>
            Revisit the Journey
          </button>
          <a href="https://data.census.gov/" target="_blank" rel="noopener noreferrer">
            Explore the Data
          </a>
        </div>
        <p class="footer-credit">
          Data sources: U.S. Census Bureau ACS, EPA, LEHD, BLS
        </p>
      </div>
    </footer>
  </main>
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
    overflow-x: hidden;
  }

  .app {
    position: relative;
  }

  .main-content {
    margin-top: 120px; /* Account for fixed navigation */
  }

  .hero {
    min-height: calc(100vh - 120px);
    display: flex;
    align-items: center;
    justify-content: center;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    text-align: center;
    padding: 2rem;
    position: relative;
  }

  .hero::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: url("data:image/svg+xml,%3Csvg width='60' height='60' viewBox='0 0 60 60' xmlns='http://www.w3.org/2000/svg'%3E%3Cg fill='none' fill-rule='evenodd'%3E%3Cg fill='%23ffffff' fill-opacity='0.05'%3E%3Cpath d='M36 34v-4h-2v4h-4v2h4v4h2v-4h4v-2h-4zm0-30V0h-2v4h-4v2h4v4h2V6h4V4h-4zM6 34v-4H4v4H0v2h4v4h2v-4h4v-2H6zM6 4V0H4v4H0v2h4v4h2V6h4V4H6z'/%3E%3C/g%3E%3C/g%3E%3C/svg%3E");
    opacity: 0.1;
  }

  .hero-content {
    position: relative;
    z-index: 1;
    max-width: 900px;
  }

  .hero-title {
    font-size: 4rem;
    margin-bottom: 1rem;
    font-weight: bold;
    line-height: 1.2;
  }

  .hero-subtitle {
    font-size: 1.5rem;
    margin-bottom: 2rem;
    opacity: 0.95;
    line-height: 1.6;
  }

  .cta-button {
    padding: 1rem 3rem;
    font-size: 1.2rem;
    background: white;
    color: #667eea;
    border: none;
    border-radius: 50px;
    cursor: pointer;
    font-weight: bold;
    transition: transform 0.3s, box-shadow 0.3s;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  }

  .cta-button:hover {
    transform: translateY(-3px);
    box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
  }

  .chapter-section {
    scroll-margin-top: 120px;
  }

  .footer {
    background: #2c3e50;
    color: white;
    padding: 4rem 2rem;
    text-align: center;
  }

  .footer-content {
    max-width: 800px;
    margin: 0 auto;
  }

  .footer-content h3 {
    font-size: 2rem;
    margin-bottom: 1rem;
  }

  .footer-content p {
    font-size: 1.1rem;
    line-height: 1.6;
    margin-bottom: 2rem;
  }

  .footer-links {
    display: flex;
    gap: 2rem;
    justify-content: center;
    margin-bottom: 2rem;
  }

  .footer-links button,
  .footer-links a {
    color: white;
    text-decoration: none;
    padding: 0.75rem 1.5rem;
    background: rgba(255, 255, 255, 0.1);
    border: none;
    border-radius: 8px;
    transition: background 0.3s;
    cursor: pointer;
    font-size: 1rem;
  }

  .footer-links button:hover,
  .footer-links a:hover {
    background: rgba(255, 255, 255, 0.2);
  }

  .footer-credit {
    font-size: 0.9rem;
    opacity: 0.7;
    margin-top: 2rem;
  }

  @media (max-width: 768px) {
    .main-content {
      margin-top: 100px;
    }

    .hero {
      min-height: calc(100vh - 100px);
    }

    .hero-title {
      font-size: 2.5rem;
    }

    .hero-subtitle {
      font-size: 1.2rem;
    }

    .cta-button {
      padding: 0.75rem 2rem;
      font-size: 1rem;
    }

    .footer-links {
      flex-direction: column;
      gap: 1rem;
    }
  }
</style>
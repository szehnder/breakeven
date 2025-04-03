<script lang="ts">
  import RelocationCalculator from './routes/RelocationCalculator.svelte';
  import InvestmentCalculator from './routes/InvestmentCalculator.svelte';

  let currentPage: 'home' | 'relocation' | 'investment' = 'home';
</script>

<style>
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  .hero {
    text-align: center;
    padding: 4rem 2rem;
    background: linear-gradient(135deg, #1e293b 0%, #0f172a 100%);
    color: white;
    border-radius: 1rem;
    margin-bottom: 3rem;
  }

  .hero h1 {
    font-size: 3rem;
    margin-bottom: 1rem;
  }

  .hero p {
    font-size: 1.25rem;
    opacity: 0.9;
    max-width: 600px;
    margin: 0 auto;
  }

  .calculator-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 2rem;
    margin-top: 2rem;
  }

  .calculator-card {
    background: white;
    border-radius: 1rem;
    padding: 2rem;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition: transform 0.2s, box-shadow 0.2s;
    cursor: pointer;
    text-align: center;
  }

  .calculator-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
  }

  .calculator-card h2 {
    color: #1e293b;
    margin-bottom: 1rem;
  }

  .calculator-card p {
    color: #64748b;
    margin-bottom: 1.5rem;
  }

  .button {
    display: inline-block;
    padding: 0.75rem 1.5rem;
    background-color: #3b82f6;
    color: white;
    border-radius: 0.5rem;
    text-decoration: none;
    font-weight: 500;
    transition: background-color 0.2s;
  }

  .button:hover {
    background-color: #2563eb;
  }

  .back-button {
    display: inline-block;
    padding: 0.5rem 1rem;
    background-color: #64748b;
    color: white;
    border-radius: 0.5rem;
    text-decoration: none;
    margin-bottom: 2rem;
  }

  .back-button:hover {
    background-color: #475569;
  }
</style>

<div class="container">
  {#if currentPage === 'home'}
    <div class="hero">
      <h1>Financial Calculators</h1>
      <p>Choose a calculator to help with your financial decisions</p>
    </div>

    <div class="calculator-grid">
      <div class="calculator-card" on:click={() => currentPage = 'relocation'}>
        <h2>Relocation Calculator</h2>
        <p>Calculate break-even points and potential profits for your home sale with employer relocation benefits.</p>
        <a class="button" on:click|stopPropagation={() => currentPage = 'relocation'}>Open Calculator</a>
      </div>

      <div class="calculator-card" on:click={() => currentPage = 'investment'}>
        <h2>Property Investment Calculator</h2>
        <p>Calculate potential returns and analyze investment scenarios.</p>
        <a class="button" on:click|stopPropagation={() => currentPage = 'investment'}>Open Calculator</a>
      </div>
    </div>
  {:else}
    <a class="back-button" on:click={() => currentPage = 'home'}>← Back to Home</a>
    
    {#if currentPage === 'relocation'}
      <RelocationCalculator />
    {:else if currentPage === 'investment'}
      <InvestmentCalculator />
    {/if}
  {/if}
</div>



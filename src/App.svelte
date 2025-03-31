<script lang="ts">
  // User inputs:
  let bma1: number = 550_000;
  let bma2: number = 560_000;
  let originalPurchasePrice: number = 563_000;
  let commissionRate: number = 6; // Employer covers up to 6%
  let otherClosingCosts: number = 2_000; // E.g., title, escrow, etc.
  let showOptionalFields: boolean = false;

  // Derived values
  $: difference = Math.abs(bma1 - bma2);
  $: averageBMA = (bma1 + bma2) / 2;

  // Always use the midpoint as the recommended BMA
  $: recommendedBMA = averageBMA;

  // Maximum price your relocation policy will allow (103% of recommended BMA).
  $: maxListPrice = recommendedBMA * 1.03;

  // Set desired sale price to match the recommended BMA
  $: desiredSalePrice = recommendedBMA;

  // Validation for hypothetical sale price
  $: isOverMaxPrice = desiredSalePrice > maxListPrice;
  $: priceWarning = isOverMaxPrice 
    ? `Warning: Your hypothetical sale price exceeds the maximum allowed price of ${maxListPrice.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}`
    : '';

  // Compute net proceeds.
  // Employer covers up to 6% commission. If user sets commission > 6%, user pays the difference.
  $: effectiveCommission = commissionRate <= 6 
    ? 0 
    : desiredSalePrice * ((commissionRate - 6) / 100);

  // Basic net proceeds: sale price - user-paid portion of commission - other costs
  $: netProceeds = desiredSalePrice - effectiveCommission - otherClosingCosts;

  // Calculate profit/loss with employer covering commission
  $: profitWithEmployerCoverage = netProceeds - originalPurchasePrice;

  // Calculate what it would be if you had to pay full commission yourself
  $: fullCommission = desiredSalePrice * (commissionRate / 100);
  $: netProceedsWithoutEmployerCoverage = desiredSalePrice - fullCommission - otherClosingCosts;
  $: profitWithoutEmployerCoverage = netProceedsWithoutEmployerCoverage - originalPurchasePrice;

  // Calculate break-even prices
  // Without employer coverage (paying full commission)
  $: breakEvenPriceWithoutCoverage = (originalPurchasePrice + otherClosingCosts) / (1 - (commissionRate / 100));
  // With employer coverage (only paying commission above 6%)
  $: breakEvenPriceWithCoverage = originalPurchasePrice + otherClosingCosts;

  // Tooltip visibility states
  let showNetProceedsTooltip = false;
  let showNetProceedsWithoutTooltip = false;
  let showBreakEvenTooltip = false;
  let showBreakEvenWithoutTooltip = false;
</script>

<style>
  .container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  .grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2rem;
  }

  .card {
    background: white;
    border-radius: 0.5rem;
    padding: 1.5rem;
    box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  }

  .input-group {
    margin-bottom: 1rem;
  }

  .label {
    display: block;
    font-weight: 600;
    margin-bottom: 0.5rem;
  }

  .input {
    width: 100%;
    padding: 0.5rem;
    border: 1px solid #d1d5db;
    border-radius: 0.25rem;
  }

  .help-text {
    font-size: 0.875rem;
    color: #6b7280;
    margin-top: 0.25rem;
  }

  .result-item {
    margin-bottom: 1rem;
  }

  .result-item strong {
    display: block;
    margin-bottom: 0.25rem;
  }

  .divider {
    border: none;
    border-top: 1px solid #e5e7eb;
    margin: 1rem 0;
  }

  .profit {
    color: #059669;
  }

  .loss {
    color: #dc2626;
  }

  .accordion-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0.5rem;
    background-color: #f9fafb;
    border: 1px solid #e5e7eb;
    border-radius: 0.25rem;
    cursor: pointer;
    margin-bottom: 1rem;
  }

  .accordion-header:hover {
    background-color: #f3f4f6;
  }

  .accordion-content {
    padding: 1rem;
    border: 1px solid #e5e7eb;
    border-top: none;
    border-radius: 0 0 0.25rem 0.25rem;
    margin-bottom: 1rem;
  }

  .chevron {
    transition: transform 0.2s;
  }

  .chevron.open {
    transform: rotate(180deg);
  }

  .highlighted-input {
    background-color: #f0f9ff;
    border: 2px solid #bae6fd;
    padding: 1rem;
    border-radius: 0.5rem;
    margin: 1rem 0;
  }

  .warning {
    color: #dc2626;
    font-size: 0.875rem;
    margin-top: 0.5rem;
    padding: 0.5rem;
    background-color: #fee2e2;
    border-radius: 0.25rem;
  }

  .info-icon {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 1.25rem;
    height: 1.25rem;
    border-radius: 50%;
    background-color: #e5e7eb;
    color: #6b7280;
    font-size: 0.75rem;
    margin-left: 0.5rem;
    cursor: help;
  }

  .tooltip {
    position: relative;
    display: inline-block;
  }

  .tooltip-content {
    position: absolute;
    bottom: 100%;
    left: 50%;
    transform: translateX(-50%);
    padding: 0.5rem;
    background-color: #1f2937;
    color: white;
    border-radius: 0.25rem;
    font-size: 0.875rem;
    white-space: nowrap;
    z-index: 10;
    margin-bottom: 0.5rem;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .tooltip-content::after {
    content: '';
    position: absolute;
    top: 100%;
    left: 50%;
    transform: translateX(-50%);
    border-width: 0.5rem;
    border-style: solid;
    border-color: #1f2937 transparent transparent transparent;
  }

  .results-group {
    margin-bottom: 1.5rem;
    background-color: #f8fafc;
    border-radius: 0.5rem;
    padding: 1rem;
  }

  .results-group-title {
    font-weight: 600;
    margin-bottom: 1rem;
    color: #1e293b;
    padding-bottom: 0.5rem;
    border-bottom: 1px solid #e2e8f0;
  }

  .result-item {
    margin-bottom: 1rem;
    padding-left: 0.5rem;
  }

  .result-item:last-child {
    margin-bottom: 0;
  }
</style>

<div class="container">
  <h1>House Sale Break-Even Calculator</h1>

  <div class="grid">
    <!-- Input Section -->
    <div class="card">
      <div class="input-group">
        <label class="label">BMA 1</label>
        <input
          class="input"
          type="number"
          bind:value={bma1}
        />
      </div>

      <div class="input-group">
        <label class="label">BMA 2</label>
        <input
          class="input"
          type="number"
          bind:value={bma2}
        />
      </div>

      <div class="input-group">
        <label class="label">Original Purchase Price</label>
        <input
          class="input"
          type="number"
          bind:value={originalPurchasePrice}
        />
      </div>

      <div class="highlighted-input">
        <div class="input-group">
          <label class="label">Hypothetical Sale Price</label>
          <input
            class="input"
            type="number"
            bind:value={desiredSalePrice}
          />
          {#if priceWarning}
            <div class="warning">{priceWarning}</div>
          {/if}
        </div>
      </div>

      <div class="accordion-header" on:click={() => showOptionalFields = !showOptionalFields}>
        <span>Optional Fields</span>
        <span class="chevron {showOptionalFields ? 'open' : ''}">▼</span>
      </div>

      {#if showOptionalFields}
        <div class="accordion-content">
          <div class="input-group">
            <label class="label">Commission Rate (Employer pays up to 6%)</label>
            <input
              class="input"
              type="number"
              bind:value={commissionRate}
              step="0.01"
            />
            <small class="help-text">If you choose more than 6%, you pay the difference</small>
          </div>

          <div class="input-group">
            <label class="label">Other Closing Costs (est.)</label>
            <input
              class="input"
              type="number"
              bind:value={otherClosingCosts}
            />
          </div>
        </div>
      {/if}
    </div>

    <!-- Results Section -->
    <div class="card">
      <div class="result-item">
        <strong>Difference Between BMA1 & BMA2</strong>
        {difference.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
      </div>

      <div class="result-item">
        <strong>Recommended BMA</strong>
        {recommendedBMA.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
      </div>

      <div class="result-item">
        <strong>Max List Price (103% of Recommended BMA)</strong>
        {maxListPrice.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
      </div>

      <hr class="divider" />

      <!-- With Employer Coverage Group -->
      <div class="results-group">
        <div class="results-group-title">With Employer Commission Coverage</div>
        
        <div class="result-item">
          <strong>
            Net Proceeds
            <span class="tooltip">
              <span class="info-icon" on:mouseenter={() => showNetProceedsTooltip = true} on:mouseleave={() => showNetProceedsTooltip = false}>i</span>
              {#if showNetProceedsTooltip}
                <span class="tooltip-content">After covered commission and other costs</span>
              {/if}
            </span>
          </strong>
          {netProceeds.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
        </div>

        <div class="result-item">
          <strong>Profit/Loss</strong>
          {profitWithEmployerCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
          <span class={profitWithEmployerCoverage >= 0 ? 'profit' : 'loss'}>
            ({profitWithEmployerCoverage >= 0 ? 'Profit' : 'Loss'})
          </span>
        </div>

        <div class="result-item">
          <strong>
            Break-Even Price
            <span class="tooltip">
              <span class="info-icon" on:mouseenter={() => showBreakEvenTooltip = true} on:mouseleave={() => showBreakEvenTooltip = false}>i</span>
              {#if showBreakEvenTooltip}
                <span class="tooltip-content">Minimum price needed to break even with employer covering commission</span>
              {/if}
            </span>
          </strong>
          {breakEvenPriceWithCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
        </div>
      </div>

      <!-- Without Employer Coverage Group -->
      <div class="results-group">
        <div class="results-group-title">Without Employer Commission Coverage</div>
        
        <div class="result-item">
          <strong>
            Net Proceeds
            <span class="tooltip">
              <span class="info-icon" on:mouseenter={() => showNetProceedsWithoutTooltip = true} on:mouseleave={() => showNetProceedsWithoutTooltip = false}>i</span>
              {#if showNetProceedsWithoutTooltip}
                <span class="tooltip-content">After paying full commission and other costs</span>
              {/if}
            </span>
          </strong>
          {netProceedsWithoutEmployerCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
        </div>

        <div class="result-item">
          <strong>Profit/Loss</strong>
          {profitWithoutEmployerCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
          <span class={profitWithoutEmployerCoverage >= 0 ? 'profit' : 'loss'}>
            ({profitWithoutEmployerCoverage >= 0 ? 'Profit' : 'Loss'})
          </span>
        </div>

        <div class="result-item">
          <strong>
            Break-Even Price
            <span class="tooltip">
              <span class="info-icon" on:mouseenter={() => showBreakEvenWithoutTooltip = true} on:mouseleave={() => showBreakEvenWithoutTooltip = false}>i</span>
              {#if showBreakEvenWithoutTooltip}
                <span class="tooltip-content">Minimum price needed to break even when paying full commission</span>
              {/if}
            </span>
          </strong>
          {breakEvenPriceWithoutCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
        </div>
      </div>
    </div>
  </div>
</div>



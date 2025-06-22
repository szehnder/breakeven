<script lang="ts">
  // User inputs:
  let bma1: number = 550_000;
  let bma2: number = 560_000;
  let originalPurchasePrice: number = 563_000;
  let commissionRate: number = 6; // Employer covers up to 6%
  let otherClosingCosts: number = 2_000; // E.g., title, escrow, etc.
  let capitalGainsTaxRate: number = 20; // Default 20% capital gains tax
  let showOptionalFields: boolean = false;

  // Derived values
  $: difference = Math.abs(bma1 - bma2);
  $: averageBMA = (bma1 + bma2) / 2;
  $: bmaSpreadPercent = ((Math.max(bma1, bma2) - Math.min(bma1, bma2)) / averageBMA) * 100;

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

  // Calculate capital gains tax
  $: capitalGainsTaxWithCoverage = profitWithEmployerCoverage > 0 
    ? profitWithEmployerCoverage * (capitalGainsTaxRate / 100) 
    : 0;
  $: afterTaxProfitWithCoverage = profitWithEmployerCoverage - capitalGainsTaxWithCoverage;

  $: capitalGainsTaxWithoutCoverage = profitWithoutEmployerCoverage > 0 
    ? profitWithoutEmployerCoverage * (capitalGainsTaxRate / 100) 
    : 0;
  $: afterTaxProfitWithoutCoverage = profitWithoutEmployerCoverage - capitalGainsTaxWithoutCoverage;

  // Tooltip visibility states
  let showNetProceedsTooltip = false;
  let showNetProceedsWithoutTooltip = false;
  let showBreakEvenTooltip = false;
  let showBreakEvenWithoutTooltip = false;
  let showMaxListPriceTooltip = false;
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

  .spread-high {
    color: #dc2626;
  }

  .spread-low {
    color: #059669;
  }

  /* Print-specific styles */
  @media print {
    /* Reset everything for print */
    * {
      box-sizing: border-box !important;
    }

    .container {
      max-width: none !important;
      margin: 0 !important;
      padding: 0.25rem !important;
      font-size: 0.6875rem !important;
      line-height: 1.2 !important;
    }

    h1 {
      font-size: 1rem !important;
      margin: 0 0 0.375rem 0 !important;
      text-align: center !important;
      font-weight: bold !important;
    }

    .grid {
      display: block !important;
      grid-template-columns: none !important;
      gap: 0 !important;
    }

    .card {
      display: inline-block !important;
      vertical-align: top !important;
      width: 48% !important;
      margin: 0 1% 0.375rem 0 !important;
      padding: 0.375rem !important;
      border: 1px solid #000 !important;
      box-shadow: none !important;
      page-break-inside: avoid !important;
    }

    .input-group {
      margin-bottom: 0.25rem !important;
      display: block !important;
    }

    .label {
      font-size: 0.6875rem !important;
      margin: 0 0 0.125rem 0 !important;
      font-weight: 600 !important;
      display: block !important;
      width: 100% !important;
    }

    .input {
      padding: 0.125rem !important;
      font-size: 0.6875rem !important;
      height: 1.25rem !important;
      min-height: 1.25rem !important;
      width: 100% !important;
      display: block !important;
    }

    .help-text {
      font-size: 0.5625rem !important;
      margin: 0.0625rem 0 0 0 !important;
      display: block !important;
      color: #666 !important;
    }

    .result-item {
      margin-bottom: 0.25rem !important;
      padding-left: 0 !important;
      display: flex !important;
      justify-content: space-between !important;
      align-items: center !important;
    }

    .result-item strong {
      font-size: 0.6875rem !important;
      margin: 0 !important;
      font-weight: 600 !important;
    }

    .divider {
      margin: 0.25rem 0 !important;
      border-top: 1px solid #000 !important;
    }

    .accordion-header {
      display: none !important;
    }

    .accordion-content {
      display: block !important;
      border: none !important;
      padding: 0 !important;
      margin: 0 !important;
    }

    .highlighted-input {
      background-color: transparent !important;
      border: 1px solid #000 !important;
      padding: 0.25rem !important;
      margin: 0.25rem 0 !important;
    }

    .warning {
      font-size: 0.5625rem !important;
      margin: 0.125rem 0 !important;
      padding: 0.125rem !important;
      background-color: transparent !important;
      border: 1px solid #000 !important;
    }

    .info-icon {
      display: none !important;
    }

    .tooltip {
      display: inline !important;
    }

    .tooltip-content {
      display: none !important;
    }

    .results-group {
      margin-bottom: 0.375rem !important;
      background-color: transparent !important;
      border: 1px solid #000 !important;
      padding: 0.25rem !important;
    }

    .results-group-title {
      font-size: 0.6875rem !important;
      margin-bottom: 0.25rem !important;
      padding-bottom: 0.125rem !important;
      border-bottom: 1px solid #000 !important;
      font-weight: bold !important;
    }

    /* Ensure page breaks don't occur in the middle of important sections */
    .card:first-child {
      page-break-after: avoid !important;
    }

    .card:last-child {
      page-break-before: avoid !important;
    }

    /* Hide any remaining interactive elements */
    input[type="number"] {
      -webkit-appearance: none !important;
      appearance: none !important;
      border: 1px solid #000 !important;
      background: transparent !important;
    }

    /* Ensure text is readable in print */
    * {
      color: #000 !important;
      background: transparent !important;
    }

    .profit {
      color: #000 !important;
      font-weight: bold !important;
    }

    .loss {
      color: #000 !important;
      font-weight: bold !important;
    }

    .spread-high {
      color: #000 !important;
      font-weight: bold !important;
    }

    .spread-low {
      color: #000 !important;
      font-weight: bold !important;
    }

    /* Force single page layout */
    @page {
      size: A4 !important;
      margin: 0.25in !important;
    }

    /* Hide any overflow */
    body {
      overflow: hidden !important;
    }

    /* Compact spacing for all elements */
    .card > * {
      margin-bottom: 0.25rem !important;
    }

    .card > *:last-child {
      margin-bottom: 0 !important;
    }

    /* Make result items more compact */
    .result-item > * {
      margin: 0 !important;
      padding: 0 !important;
    }
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

          <div class="input-group">
            <label class="label">Capital Gains Tax Rate (%)</label>
            <input
              class="input"
              type="number"
              bind:value={capitalGainsTaxRate}
              step="0.1"
            />
            <small class="help-text">Tax rate applied to profits from the sale</small>
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
        <strong class={bmaSpreadPercent >= 5 ? 'spread-high' : 'spread-low'}>BMA Spread</strong>
        <span class={bmaSpreadPercent >= 5 ? 'spread-high' : 'spread-low'}>
          {bmaSpreadPercent.toFixed(1)}%
        </span>
      </div>

      <div class="result-item">
        <strong>Average BMA</strong>
        {recommendedBMA.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
      </div>

      <div class="result-item">
        <strong>
          Max List Price
          <span class="tooltip">
            <span class="info-icon" on:mouseenter={() => showMaxListPriceTooltip = true} on:mouseleave={() => showMaxListPriceTooltip = false}>i</span>
            {#if showMaxListPriceTooltip}
              <span class="tooltip-content">Maximum price allowed by relocation policy (103% of Avg BMA)</span>
            {/if}
          </span>
        </strong>
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

        {#if profitWithEmployerCoverage > 0}
          <div class="result-item">
            <strong>Capital Gains Tax ({capitalGainsTaxRate}%)</strong>
            {capitalGainsTaxWithCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
          </div>

          <div class="result-item">
            <strong>After-Tax Profit</strong>
            {afterTaxProfitWithCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
            <span class="profit">(Profit)</span>
          </div>
        {/if}

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

        {#if profitWithoutEmployerCoverage > 0}
          <div class="result-item">
            <strong>Capital Gains Tax ({capitalGainsTaxRate}%)</strong>
            {capitalGainsTaxWithoutCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
          </div>

          <div class="result-item">
            <strong>After-Tax Profit</strong>
            {afterTaxProfitWithoutCoverage.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
            <span class="profit">(Profit)</span>
          </div>
        {/if}

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
<script lang="ts">
  import { onMount } from 'svelte';
  import PaymentBreakdownChart from '../components/PaymentBreakdownChart.svelte';

  // User inputs
  let homePrice: number = 400_000;
  let downPayment: number = 80_000;
  let downPaymentPercent: number = 20; // percentage of home price
  let loanTermYears: number = 30; // in years
  let loanTerm: number = loanTermYears * 12; // converted to months
  let interestRate: number = 6.5; // annual interest rate
  let propertyTaxRate: number = 1.2; // annual property tax rate
  let homeownersInsurance: number = 1_500; // annual insurance cost
  let hoaFees: number = 0; // monthly HOA fees
  let pmiRate: number = 0.5; // annual PMI rate (if applicable)
  let additionalPayment: number = 0; // additional monthly payment

  // Computed values
  $: monthlyInterestRate = interestRate / 100 / 12;
  $: requiresPMI = downPaymentPercent < 20;
  $: monthlyPMI = requiresPMI ? (loanAmount * (pmiRate / 100)) / 12 : 0;
  $: monthlyPropertyTax = (homePrice * (propertyTaxRate / 100)) / 12;
  $: monthlyInsurance = homeownersInsurance / 12;
  $: totalFinanced = loanAmount;
  $: loanAmount = homePrice - downPayment;

  // Handle down payment changes
  function updateDownPaymentFromAmount() {
    if (homePrice > 0) {
      downPaymentPercent = (downPayment / homePrice) * 100;
    }
  }

  function updateDownPaymentFromPercent() {
    if (homePrice > 0) {
      downPayment = (downPaymentPercent / 100) * homePrice;
    }
  }

  // Handle home price changes
  function handleHomePriceChange(event: Event) {
    const input = event.target as HTMLInputElement;
    // Remove all non-numeric characters and parse the value
    const value = parseInt(input.value.replace(/[^0-9]/g, '')) || 0;
    homePrice = value;
    // Update down payment amount based on current percentage
    updateDownPaymentFromPercent();
  }

  // Format currency for display
  function formatCurrency(value: number): string {
    return value.toLocaleString('en-US', {
      style: 'currency',
      currency: 'USD',
      minimumFractionDigits: 0,
      maximumFractionDigits: 0
    });
  }

  // Format percentage for display
  function formatPercentage(value: number): string {
    return `${value}%`;
  }

  // Handle currency input changes
  function handleCurrencyInput(event: Event) {
    const input = event.target as HTMLInputElement;
    const value = input.value.replace(/[^0-9]/g, '');
    downPayment = parseInt(value) || 0;
    updateDownPaymentFromAmount();
  }

  // Handle percentage input changes
  function handlePercentageInput(event: Event) {
    const input = event.target as HTMLInputElement;
    const value = input.value.replace(/[^0-9]/g, '');
    const numValue = parseInt(value) || 0;
    if (numValue <= 100) {
      downPaymentPercent = numValue;
      updateDownPaymentFromPercent();
    }
  }

  // Monthly payment calculation using the loan amortization formula
  $: monthlyPrincipalAndInterest = totalFinanced * 
    (monthlyInterestRate * Math.pow(1 + monthlyInterestRate, loanTerm)) / 
    (Math.pow(1 + monthlyInterestRate, loanTerm) - 1);

  // Total monthly payment including all components
  $: monthlyPayment = monthlyPrincipalAndInterest + monthlyPMI + monthlyPropertyTax + monthlyInsurance + hoaFees;

  // Total interest paid over the life of the loan
  $: totalInterest = (monthlyPrincipalAndInterest * loanTerm) - totalFinanced;

  // Total PMI paid (if applicable)
  $: totalPMI = requiresPMI ? monthlyPMI * loanTerm : 0;

  // Total cost of the home including all fees and interest
  $: totalCostWithInterest = homePrice + totalInterest + totalPMI + (monthlyPropertyTax * loanTerm) + (monthlyInsurance * loanTerm) + (hoaFees * loanTerm);

  // Amortization schedule
  $: amortizationSchedule = Array.from({ length: loanTerm }, (_, i) => {
    const month = i + 1;
    let remainingBalance = totalFinanced;
    let totalPrincipalPaid = 0;
    let totalInterestPaid = 0;
    
    // Calculate payments for each month up to current month
    for (let m = 1; m <= month; m++) {
      const interestPayment = remainingBalance * monthlyInterestRate;
      const principalPayment = monthlyPrincipalAndInterest - interestPayment;
      remainingBalance -= principalPayment;
      totalPrincipalPaid += principalPayment;
      totalInterestPaid += interestPayment;
    }
    
    // Check if PMI should be removed (when LTV reaches 78%)
    const currentLTV = (remainingBalance / homePrice) * 100;
    const pmiRemoved = requiresPMI && currentLTV <= 78;
    
    return {
      month,
      payment: monthlyPayment,
      principalAndInterest: monthlyPrincipalAndInterest,
      principal: monthlyPrincipalAndInterest - (remainingBalance * monthlyInterestRate),
      interest: remainingBalance * monthlyInterestRate,
      pmi: pmiRemoved ? 0 : monthlyPMI,
      propertyTax: monthlyPropertyTax,
      insurance: monthlyInsurance,
      hoa: hoaFees,
      remainingBalance,
      ltv: currentLTV
    };
  });

  // Chart data
  $: chartData = {
    months: amortizationSchedule.map(row => row.month),
    principal: amortizationSchedule.map(row => row.principal),
    interest: amortizationSchedule.map(row => row.interest),
    pmi: amortizationSchedule.map(row => row.pmi),
    propertyTax: amortizationSchedule.map(row => row.propertyTax),
    insurance: amortizationSchedule.map(row => row.insurance),
    hoa: amortizationSchedule.map(row => row.hoa)
  };

  // Amortization schedule with additional payments
  $: acceleratedAmortizationSchedule = Array.from({ length: loanTerm }, (_, i) => {
    const month = i + 1;
    let remainingBalance = totalFinanced;
    let totalPrincipalPaid = 0;
    let totalInterestPaid = 0;
    let monthsToPayoff = loanTerm;
    
    // Calculate payments for each month up to current month
    for (let m = 1; m <= month; m++) {
      const interestPayment = remainingBalance * monthlyInterestRate;
      const principalPayment = monthlyPrincipalAndInterest - interestPayment + additionalPayment;
      remainingBalance -= principalPayment;
      totalPrincipalPaid += principalPayment;
      totalInterestPaid += interestPayment;
      
      // Check if loan is paid off early
      if (remainingBalance <= 0 && monthsToPayoff === loanTerm) {
        monthsToPayoff = m;
        remainingBalance = 0;
      }
    }
    
    // Check if PMI should be removed (when LTV reaches 78%)
    const currentLTV = (remainingBalance / homePrice) * 100;
    const pmiRemoved = requiresPMI && currentLTV <= 78;
    
    return {
      month,
      payment: monthlyPayment + additionalPayment,
      principalAndInterest: monthlyPrincipalAndInterest + additionalPayment,
      principal: monthlyPrincipalAndInterest - (remainingBalance * monthlyInterestRate) + additionalPayment,
      interest: remainingBalance * monthlyInterestRate,
      pmi: pmiRemoved ? 0 : monthlyPMI,
      propertyTax: monthlyPropertyTax,
      insurance: monthlyInsurance,
      hoa: hoaFees,
      remainingBalance,
      ltv: currentLTV,
      monthsToPayoff
    };
  });

  // Calculate savings from additional payments
  $: acceleratedMonthsToPayoff = acceleratedAmortizationSchedule[acceleratedAmortizationSchedule.length - 1].monthsToPayoff;
  $: monthsSaved = loanTerm - acceleratedMonthsToPayoff;
  $: yearsSaved = monthsSaved / 12;
  $: totalInterestWithAdditional = acceleratedAmortizationSchedule.reduce((sum, row) => sum + row.interest, 0);
  $: interestSaved = totalInterest - totalInterestWithAdditional;
  
  // Calculate PMI savings
  $: originalPMIMonths = requiresPMI ? loanTerm : 0;
  $: acceleratedPMIMonths = requiresPMI ? acceleratedMonthsToPayoff : 0;
  $: pmiMonthsSaved = originalPMIMonths - acceleratedPMIMonths;
  $: pmiSaved = monthlyPMI * pmiMonthsSaved;
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

  .down-payment-container {
    background-color: #f8fafc;
    border-radius: 0.5rem;
    padding: 1rem;
    margin-bottom: 0.5rem;
  }

  .down-payment-input {
    margin-bottom: 0.5rem;
  }

  .down-payment-input:last-child {
    margin-bottom: 0;
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

  .summary-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1rem;
    margin-bottom: 2rem;
  }

  .summary-item {
    background: #f8fafc;
    padding: 1rem;
    border-radius: 0.5rem;
    text-align: center;
  }

  .summary-item strong {
    display: block;
    color: #1e293b;
    margin-bottom: 0.5rem;
  }

  .summary-item .value {
    font-size: 1.5rem;
    font-weight: 600;
    color: #3b82f6;
  }

  .amortization-table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
  }

  .amortization-table th,
  .amortization-table td {
    padding: 0.75rem;
    text-align: right;
    border-bottom: 1px solid #e2e8f0;
  }

  .amortization-table th {
    background: #f8fafc;
    font-weight: 600;
  }

  .amortization-table th:first-child,
  .amortization-table td:first-child {
    text-align: left;
  }

  .highlight {
    color: #3b82f6;
    font-weight: 600;
  }

  .chart-container {
    width: 100%;
    height: 400px;
    margin-top: 2rem;
  }

  .pmi-warning {
    background-color: #fef3c7;
    color: #92400e;
    padding: 0.5rem;
    border-radius: 0.25rem;
    margin-top: 0.5rem;
    font-size: 0.875rem;
  }
</style>

<div class="container">
  <h1>Home Loan Calculator</h1>

  <div class="grid">
    <!-- Input Section -->
    <div class="card">
      <div class="input-group">
        <label class="label">Home Price</label>
        <input
          class="input"
          type="text"
          value={formatCurrency(homePrice)}
          on:input={handleHomePriceChange}
          min="0"
          step="10000"
        />
      </div>

      <div class="input-group">
        <label class="label">Down Payment</label>
        <div class="down-payment-container">
          <div class="down-payment-input">
            <input
              class="input"
              type="text"
              value={formatCurrency(downPayment)}
              on:input={handleCurrencyInput}
              min="0"
            />
            <small class="help-text">Amount</small>
          </div>
          <div class="down-payment-input">
            <input
              class="input"
              type="text"
              value={formatPercentage(downPaymentPercent)}
              on:input={handlePercentageInput}
              min="0"
              max="100"
            />
            <small class="help-text">Percent</small>
          </div>
        </div>
        {#if requiresPMI}
          <div class="pmi-warning">
            ⚠️ PMI will be required (less than 20% down payment)
          </div>
        {/if}
      </div>

      <div class="input-group">
        <label class="label">Loan Term (years)</label>
        <input
          class="input"
          type="number"
          bind:value={loanTermYears}
          min="15"
          max="30"
          step="5"
        />
        <small class="help-text">Common terms: 15, 20, or 30 years</small>
      </div>

      <div class="input-group">
        <label class="label">Interest Rate (%)</label>
        <input
          class="input"
          type="number"
          bind:value={interestRate}
          min="0"
          max="30"
          step="0.1"
        />
      </div>

      <div class="input-group">
        <label class="label">Property Tax Rate (%)</label>
        <input
          class="input"
          type="number"
          bind:value={propertyTaxRate}
          min="0"
          max="5"
          step="0.1"
        />
        <small class="help-text">Annual property tax rate</small>
      </div>

      <div class="input-group">
        <label class="label">Homeowners Insurance (annual)</label>
        <input
          class="input"
          type="number"
          bind:value={homeownersInsurance}
          min="0"
          step="100"
        />
      </div>

      <div class="input-group">
        <label class="label">HOA Fees (monthly)</label>
        <input
          class="input"
          type="number"
          bind:value={hoaFees}
          min="0"
          step="50"
        />
      </div>

      {#if requiresPMI}
        <div class="input-group">
          <label class="label">PMI Rate (%)</label>
          <input
            class="input"
            type="number"
            bind:value={pmiRate}
            min="0"
            max="2"
            step="0.1"
          />
          <small class="help-text">Annual PMI rate (typically 0.5-1%)</small>
        </div>
      {/if}
    </div>

    <!-- Results Section -->
    <div class="card">
      <div class="summary-grid">
        <div class="summary-item">
          <strong>Monthly Payment</strong>
          <div class="value">
            {monthlyPayment.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
          </div>
        </div>

        <div class="summary-item">
          <strong>Total Cost</strong>
          <div class="value">
            {totalCostWithInterest.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
          </div>
        </div>

        <div class="summary-item">
          <strong>Total Interest</strong>
          <div class="value">
            {totalInterest.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}
          </div>
        </div>
      </div>

      <div class="result-item">
        <strong>Loan Details</strong>
        <div>Loan Amount: {loanAmount.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
        <div>Down Payment: {downPayment.toLocaleString('en-US', { style: 'currency', currency: 'USD' })} ({downPaymentPercent.toFixed(1)}%)</div>
        {#if requiresPMI}
          <div>Monthly PMI: {monthlyPMI.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
          <div>Total PMI: {totalPMI.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
        {/if}
        <div>Monthly Property Tax: {monthlyPropertyTax.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
        <div>Monthly Insurance: {monthlyInsurance.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
        <div>Monthly HOA: {hoaFees.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
      </div>

      <div class="result-item">
        <strong>Amortization Schedule</strong>
        <table class="amortization-table">
          <thead>
            <tr>
              <th>Month</th>
              <th>Payment</th>
              <th>Principal</th>
              <th>Interest</th>
              <th>PMI</th>
              <th>Balance</th>
            </tr>
          </thead>
          <tbody>
            {#each amortizationSchedule.slice(0, 12) as row}
              <tr>
                <td>{row.month}</td>
                <td>{row.payment.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</td>
                <td>{row.principal.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</td>
                <td>{row.interest.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</td>
                <td>{row.pmi.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</td>
                <td>{row.remainingBalance.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</td>
              </tr>
            {/each}
          </tbody>
        </table>
        {#if loanTerm > 12}
          <div class="help-text">Showing first 12 months of {loanTerm} total months</div>
        {/if}
      </div>
    </div>
  </div>

  <!-- Chart Section -->
  <div class="card">
    <PaymentBreakdownChart
      title="Monthly Payment Breakdown Over Time"
      data={chartData}
    />
  </div>

  <!-- Additional Payment Section -->
  <div class="card">
    <div class="input-group">
      <label class="label">Additional Monthly Payment</label>
      <input
        class="input"
        type="text"
        value={formatCurrency(additionalPayment)}
        on:input={(e) => {
          const input = e.target as HTMLInputElement;
          const value = input.value.replace(/[^0-9]/g, '');
          additionalPayment = parseInt(value) || 0;
        }}
        min="0"
      />
      <small class="help-text">Extra payment applied to principal each month</small>
    </div>

    {#if additionalPayment > 0}
      <div class="summary-grid" style="margin-top: 1rem; background-color: #f0fdf4; padding: 1rem; border-radius: 0.5rem;">
        <div class="summary-item">
          <strong>Months Saved</strong>
          <div class="value" style="color: #10b981;">
            {monthsSaved}
          </div>
        </div>

        <div class="summary-item">
          <strong>Years Saved</strong>
          <div class="value" style="color: #10b981;">
            {yearsSaved.toFixed(1)}
          </div>
        </div>

        <div class="summary-item">
          <strong>Interest Saved</strong>
          <div class="value" style="color: #10b981;">
            {formatCurrency(interestSaved)}
          </div>
        </div>

        {#if requiresPMI}
          <div class="summary-item">
            <strong>PMI Months Saved</strong>
            <div class="value" style="color: #10b981;">
              {pmiMonthsSaved}
            </div>
          </div>

          <div class="summary-item">
            <strong>PMI Saved</strong>
            <div class="value" style="color: #10b981;">
              {formatCurrency(pmiSaved)}
            </div>
          </div>
        {/if}
      </div>
    {/if}
  </div>

  <!-- Legal Disclaimer -->
  <div class="card" style="margin-top: 2rem; background-color: #fef2f2; border: 1px solid #fee2e2;">
    <div style="color: #991b1b; font-size: 0.875rem;">
      <strong>⚠️ IMPORTANT DISCLAIMER:</strong>
      <p>This calculator is provided for informational and educational purposes only. It was created for personal use and should not be considered as financial, investment, tax, or legal advice. The calculations and results are estimates and may not reflect actual financial outcomes.</p>
      <p>By using this calculator, you acknowledge that you are using it at your own risk and should not rely solely on the results for making financial decisions. Always consult with qualified financial professionals before making any financial decisions.</p>
    </div>
  </div>
</div> 
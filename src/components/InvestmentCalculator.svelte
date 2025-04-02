<script lang="ts">
    // --- Input Properties ---
    let purchasePrice: number = 570000;
    let downPaymentPercentage: number = 20; // %
    let loanInterestRate: number = 6.625; // Annual %
    let loanTermYears: number = 30;
    let annualPropertyTax: number = 3500;
    let annualHomeInsurance: number = 1000;
    let monthlyRent: number = 2750;
    let propertyManagementFeePercentage: number = 12; // % of monthly rent
    let annualOtherExpenses: number = 1500; // Maintenance, repairs, vacancy buffer, HOA, etc.
    let buildingValuePercentage: number = 80; // % of purchase price allocated to building (for depreciation)
  
    // --- Calculated Values (Reactive) ---
  
    // Loan & Down Payment
    $: downPaymentAmount = purchasePrice * (downPaymentPercentage / 100);
    $: loanAmount = purchasePrice - downPaymentAmount;
  
    // Monthly Mortgage Principal & Interest (P&I)
    $: monthlyInterestRate = loanInterestRate / 100 / 12;
    $: numberOfPayments = loanTermYears * 12;
    $: monthlyPrincipalAndInterest =
      loanAmount <= 0 || monthlyInterestRate <= 0 || numberOfPayments <= 0
        ? 0
        : loanAmount *
          (monthlyInterestRate * Math.pow(1 + monthlyInterestRate, numberOfPayments)) /
          (Math.pow(1 + monthlyInterestRate, numberOfPayments) - 1);
  
    // Monthly Escrow (Taxes & Insurance)
    $: monthlyPropertyTax = annualPropertyTax / 12;
    $: monthlyHomeInsurance = annualHomeInsurance / 12;
  
    // Total Monthly Housing Payment (PITI)
    $: monthlyPITI = monthlyPrincipalAndInterest + monthlyPropertyTax + monthlyHomeInsurance;
  
    // Monthly Operating Expenses
    $: monthlyPropertyManagementFee = monthlyRent * (propertyManagementFeePercentage / 100);
    $: monthlyOtherExpenses = annualOtherExpenses / 12;
    $: totalMonthlyExpenses = monthlyPITI + monthlyPropertyManagementFee + monthlyOtherExpenses;
  
    // Cash Flow
    $: monthlyGrossIncome = monthlyRent;
    $: monthlyNetCashFlow = monthlyGrossIncome - totalMonthlyExpenses;
    $: annualNetCashFlow = monthlyNetCashFlow * 12;
  
    // Investment Performance
    $: totalCashInvested = downPaymentAmount; // Simplified: Add closing costs for real analysis
    $: cashOnCashReturn =
      totalCashInvested <= 0 ? 0 : (annualNetCashFlow / totalCashInvested) * 100;
  
    // Tax Considerations (Approximations)
    $: buildingValueForDepreciation = purchasePrice * (buildingValuePercentage / 100);
    $: annualDepreciation =
      buildingValueForDepreciation > 0 ? buildingValueForDepreciation / 27.5 : 0; // Straight line over 27.5 years
  
    // Approximation of interest paid in the first year
    // A more accurate calculation requires an amortization schedule
    $: estimatedAnnualInterestPaidYear1 = 0
    $: {
      let remainingBalance = loanAmount;
      let totalInterest = 0;
      if (loanAmount > 0 && monthlyInterestRate > 0 && numberOfPayments > 0) {
        for (let i = 0; i < 12 && i < numberOfPayments; i++) {
            const interestPayment = remainingBalance * monthlyInterestRate;
            const principalPayment = monthlyPrincipalAndInterest - interestPayment;
            totalInterest += interestPayment;
            remainingBalance -= principalPayment;
        }
        estimatedAnnualInterestPaidYear1 = totalInterest;
      } else {
        estimatedAnnualInterestPaidYear1 = 0;
      }
    }
  
  
    // --- Helper Functions for Formatting ---
    const formatCurrency = (value: number): string => {
      return value.toLocaleString("en-US", { style: "currency", currency: "USD" });
    };
  
    const formatPercent = (value: number): string => {
      return value.toFixed(2) + "%";
    };
  
  </script>
  
  <div class="calculator-container">
    <h1>Investment Property Calculator</h1>
  
    <form on:submit|preventDefault>
      <section>
        <h2>Property & Loan Details</h2>
        <div class="input-group">
          <label for="purchasePrice">Purchase Price:</label>
          <input id="purchasePrice" type="number" bind:value={purchasePrice} min="0" />
        </div>
        <div class="input-group">
          <label for="downPaymentPercentage">Down Payment (%):</label>
          <input id="downPaymentPercentage" type="number" bind:value={downPaymentPercentage} min="0" max="100" step="0.1" />
          <span>({formatCurrency(downPaymentAmount)})</span>
        </div>
         <p>Loan Amount: {formatCurrency(loanAmount)}</p>
        <div class="input-group">
          <label for="loanInterestRate">Interest Rate (%):</label>
          <input id="loanInterestRate" type="number" bind:value={loanInterestRate} min="0" step="0.01" />
        </div>
        <div class="input-group">
          <label for="loanTermYears">Loan Term (Years):</label>
          <input id="loanTermYears" type="number" bind:value={loanTermYears} min="1" />
        </div>
      </section>
  
      <section>
        <h2>Recurring Expenses</h2>
         <div class="input-group">
          <label for="annualPropertyTax">Annual Property Tax:</label>
          <input id="annualPropertyTax" type="number" bind:value={annualPropertyTax} min="0" />
        </div>
         <div class="input-group">
          <label for="annualHomeInsurance">Annual Home Insurance:</label>
          <input id="annualHomeInsurance" type="number" bind:value={annualHomeInsurance} min="0" />
        </div>
         <div class="input-group">
          <label for="propertyManagementFeePercentage">Property Management Fee (% of Rent):</label>
          <input id="propertyManagementFeePercentage" type="number" bind:value={propertyManagementFeePercentage} min="0" max="100" step="0.1" />
        </div>
         <div class="input-group">
          <label for="annualOtherExpenses">Est. Annual Other Expenses (Maintenance, Vacancy etc.):</label>
          <input id="annualOtherExpenses" type="number" bind:value={annualOtherExpenses} min="0" />
        </div>
      </section>
  
      <section>
        <h2>Income</h2>
        <div class="input-group">
          <label for="monthlyRent">Monthly Rent:</label>
          <input id="monthlyRent" type="number" bind:value={monthlyRent} min="0" />
        </div>
      </section>
  
       <section>
        <h2>Depreciation Basis</h2>
         <div class="input-group">
          <label for="buildingValuePercentage">Building Value (% of Purchase Price):</label>
          <input id="buildingValuePercentage" type="number" bind:value={buildingValuePercentage} min="0" max="100" step="1" />
           <span>(Used for depreciation calc: {formatCurrency(buildingValueForDepreciation)})</span>
        </div>
      </section>
    </form>
  
    <section class="results">
      <h2>Calculated Results</h2>
  
      <div class="result-item">
        <strong>Monthly P&I Payment:</strong>
        <span>{formatCurrency(monthlyPrincipalAndInterest)}</span>
      </div>
       <div class="result-item">
        <strong>Monthly Taxes & Insurance (Escrow):</strong>
        <span>{formatCurrency(monthlyPropertyTax + monthlyHomeInsurance)} ({formatCurrency(monthlyPropertyTax)} Tax + {formatCurrency(monthlyHomeInsurance)} Ins)</span>
      </div>
       <div class="result-item">
        <strong>Total Monthly Payment (loan+taxes+insurance):</strong>
        <span>{formatCurrency(monthlyPITI)}</span>
      </div>
       <div class="result-item">
        <strong>Monthly Property Management Fee:</strong>
        <span>{formatCurrency(monthlyPropertyManagementFee)}</span>
      </div>
       <div class="result-item">
        <strong>Other Monthly Expenses:</strong>
        <span>{formatCurrency(monthlyOtherExpenses)}</span>
      </div>
      <hr/>
       <div class="result-item summary">
        <strong>Total Monthly Expenses:</strong>
        <span>{formatCurrency(totalMonthlyExpenses)}</span>
      </div>
       <div class="result-item summary">
        <strong>Monthly Gross Income (Rent):</strong>
        <span>{formatCurrency(monthlyGrossIncome)}</span>
      </div>
       <div class="result-item summary cashflow" class:positive={monthlyNetCashFlow > 0} class:negative={monthlyNetCashFlow < 0}>
        <strong>Monthly Net Cash Flow:</strong>
        <span>{formatCurrency(monthlyNetCashFlow)}</span>
      </div>
       <div class="result-item summary cashflow" class:positive={annualNetCashFlow > 0} class:negative={annualNetCashFlow < 0}>
        <strong>Annual Net Cash Flow:</strong>
        <span>{formatCurrency(annualNetCashFlow)}</span>
      </div>
      <hr/>
      <div class="result-item">
          <strong>Total Cash Invested (Down Payment):</strong>
          <span>{formatCurrency(totalCashInvested)}</span>
          <small>(Note: Does not include closing costs)</small>
      </div>
      <div class="result-item summary">
          <strong>Cash-on-Cash Return:</strong>
          <span>{formatPercent(cashOnCashReturn)}</span>
      </div>
       <hr/>
      <h3>Tax Considerations (Estimates)</h3>
       <div class="result-item">
          <strong>Estimated Annual Depreciation Deduction:</strong>
          <span>{formatCurrency(annualDepreciation)}</span>
           <small>(Based on {buildingValuePercentage}% building value / 27.5 yrs)</small>
      </div>
       <div class="result-item">
          <strong>Estimated Interest Paid (Year 1):</strong>
          <span>{formatCurrency(estimatedAnnualInterestPaidYear1)}</span>
           <small>(Approximate value for tax deduction)</small>
      </div>
       <p><small>Disclaimer: These calculations are estimates for informational purposes only. Consult with a financial advisor, accountant, and real estate professional before making any investment decisions. Tax implications can be complex.</small></p>
  
    </section>
  
  </div>
  
  <style>
    .calculator-container {
      font-family: sans-serif;
      max-width: 700px;
      margin: 2rem auto;
      padding: 1.5rem;
      border: 1px solid #ccc;
      border-radius: 8px;
      background-color: #f9f9f9;
    }
  
    h1, h2, h3 {
      color: #333;
      border-bottom: 1px solid #eee;
      padding-bottom: 0.5rem;
      margin-bottom: 1rem;
    }
  
    section {
      margin-bottom: 2rem;
    }
  
    .input-group {
      margin-bottom: 1rem;
      display: flex;
      flex-wrap: wrap;
      align-items: center;
    }
  
    .input-group label {
      flex-basis: 250px; /* Adjust as needed */
      margin-right: 10px;
      font-weight: bold;
      color: #555;
    }
  
    .input-group input[type="number"] {
      flex-grow: 1;
      padding: 8px;
      border: 1px solid #ccc;
      border-radius: 4px;
      min-width: 100px;
    }
     .input-group span {
       margin-left: 10px;
       font-size: 0.9em;
       color: #666;
     }
  
    .results {
      margin-top: 2rem;
      background-color: #fff;
      padding: 1.5rem;
      border-radius: 6px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }
  
    .result-item {
      display: flex;
      justify-content: space-between;
      padding: 0.6rem 0;
      border-bottom: 1px dashed #eee;
      align-items: center;
    }
    .result-item:last-child {
        border-bottom: none;
    }
  
    .result-item strong {
      color: #444;
    }
    .result-item span {
        font-weight: bold;
        text-align: right;
    }
    .result-item small {
        display: block;
        width: 100%;
        font-size: 0.8em;
        color: #777;
        margin-top: 2px;
        text-align: right;
    }
  
  
    .summary {
        font-size: 1.1em;
        font-weight: bold;
    }
  
    .cashflow span {
        padding: 2px 6px;
        border-radius: 4px;
    }
    .cashflow.positive span {
        background-color: #dff0d8; /* light green */
        color: #3c763d; /* dark green */
    }
    .cashflow.negative span {
         background-color: #f2dede; /* light red */
        color: #a94442; /* dark red */
    }
  
    hr {
        border: none;
        border-top: 1px solid #ddd;
        margin: 1rem 0;
    }
  
    p small {
        font-style: italic;
        color: #888;
        line-height: 1.4;
    }
  
    /* Basic responsiveness */
     @media (max-width: 600px) {
       .input-group label {
         flex-basis: 100%;
         margin-bottom: 5px;
       }
       .result-item {
         flex-direction: column;
         align-items: flex-start;
       }
        .result-item span {
            margin-top: 4px;
            text-align: left;
        }
         .result-item small {
             text-align: left;
         }
     }
  
  
  </style>
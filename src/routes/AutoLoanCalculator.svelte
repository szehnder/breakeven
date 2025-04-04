<script lang="ts">
  import { onMount } from 'svelte';
  import * as echarts from 'echarts';

  // User inputs
  let vehiclePrice: number = 30_000;
  let downPayment: number = 5_000;
  let loanTerm: number = 60; // in months
  let interestRate: number = 5.5; // annual interest rate
  let salesTax: number = 8.5; // sales tax rate
  let titleAndFees: number = 500; // estimated title and registration fees
  let additionalPayment: number = 0; // additional monthly payment

  // Chart element reference
  let chartContainer: HTMLElement;
  let chart: echarts.ECharts;

  // Computed values
  $: loanAmount = vehiclePrice - downPayment;
  $: monthlyInterestRate = interestRate / 100 / 12;
  $: salesTaxAmount = vehiclePrice * (salesTax / 100);
  $: totalCost = vehiclePrice + salesTaxAmount + titleAndFees;
  $: totalFinanced = loanAmount + salesTaxAmount + titleAndFees;

  // Monthly payment calculation using the loan amortization formula
  $: monthlyPayment = totalFinanced * 
    (monthlyInterestRate * Math.pow(1 + monthlyInterestRate, loanTerm)) / 
    (Math.pow(1 + monthlyInterestRate, loanTerm) - 1);

  // Total interest paid over the life of the loan
  $: totalInterest = (monthlyPayment * loanTerm) - totalFinanced;

  // Total cost of the vehicle including all fees and interest
  $: totalCostWithInterest = totalCost + totalInterest;

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
      const principalPayment = monthlyPayment - interestPayment + additionalPayment;
      remainingBalance -= principalPayment;
      totalPrincipalPaid += principalPayment;
      totalInterestPaid += interestPayment;
      
      // Check if loan is paid off early
      if (remainingBalance <= 0 && monthsToPayoff === loanTerm) {
        monthsToPayoff = m;
        remainingBalance = 0;
      }
    }
    
    return {
      month,
      payment: monthlyPayment + additionalPayment,
      principal: monthlyPayment - (remainingBalance * monthlyInterestRate) + additionalPayment,
      interest: remainingBalance * monthlyInterestRate,
      remainingBalance,
      monthsToPayoff
    };
  });

  // Calculate savings from additional payments
  $: acceleratedMonthsToPayoff = acceleratedAmortizationSchedule[acceleratedAmortizationSchedule.length - 1].monthsToPayoff;
  $: monthsSaved = loanTerm - acceleratedMonthsToPayoff;
  $: totalInterestWithAdditional = acceleratedAmortizationSchedule.reduce((sum, row) => sum + row.interest, 0);
  $: interestSaved = totalInterest - totalInterestWithAdditional;

  // Format currency for display
  function formatCurrency(value: number): string {
    return value.toLocaleString('en-US', {
      style: 'currency',
      currency: 'USD',
      minimumFractionDigits: 0,
      maximumFractionDigits: 0
    });
  }

  // Chart data
  $: chartData = {
    months: acceleratedAmortizationSchedule.slice(0, acceleratedMonthsToPayoff).map(row => row.month),
    principal: acceleratedAmortizationSchedule.slice(0, acceleratedMonthsToPayoff).map(row => row.principal),
    interest: acceleratedAmortizationSchedule.slice(0, acceleratedMonthsToPayoff).map(row => row.interest)
  };

  // Initialize and update chart
  $: if (chart && chartData) {
    const option = {
      title: {
        text: 'Principal vs Interest Payments Over Time',
        left: 'center'
      },
      tooltip: {
        trigger: 'axis',
        formatter: function(params: any) {
          const month = params[0].dataIndex + 1;
          const principal = params[0].value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const interest = params[1].value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const total = (params[0].value + params[1].value).toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const principalPercent = ((params[0].value / (params[0].value + params[1].value)) * 100).toFixed(1);
          const interestPercent = ((params[1].value / (params[0].value + params[1].value)) * 100).toFixed(1);
          return `Month ${month}<br/>
                  Total Payment: ${total}<br/>
                  Principal: ${principal} (${principalPercent}%)<br/>
                  Interest: ${interest} (${interestPercent}%)`;
        }
      },
      legend: {
        data: ['Principal', 'Interest'],
        bottom: 0
      },
      grid: {
        left: '3%',
        right: '4%',
        bottom: '15%',
        containLabel: true
      },
      xAxis: {
        type: 'category',
        data: chartData.months,
        name: 'Month'
      },
      yAxis: {
        type: 'value',
        name: 'Amount ($)',
        axisLabel: {
          formatter: (value: number) => value.toLocaleString('en-US', { style: 'currency', currency: 'USD' })
        }
      },
      series: [
        {
          name: 'Principal',
          type: 'line',
          stack: 'total',
          areaStyle: {
            opacity: 0.8,
            color: '#3b82f6'
          },
          data: chartData.principal,
          smooth: true,
          lineStyle: {
            width: 3
          },
          itemStyle: {
            color: '#3b82f6'
          }
        },
        {
          name: 'Interest',
          type: 'line',
          stack: 'total',
          areaStyle: {
            opacity: 0.8,
            color: '#ef4444'
          },
          data: chartData.interest,
          smooth: true,
          lineStyle: {
            width: 3
          },
          itemStyle: {
            color: '#ef4444'
          }
        }
      ]
    };
    chart.setOption(option);
  }

  // Initialize chart on mount
  onMount(() => {
    chart = echarts.init(chartContainer);
    window.addEventListener('resize', () => chart.resize());
    return () => {
      chart.dispose();
      window.removeEventListener('resize', () => chart.resize());
    };
  });
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
</style>

<div class="container">
  <h1>Auto Loan Calculator</h1>

  <div class="grid">
    <!-- Input Section -->
    <div class="card">
      <div class="input-group">
        <label class="label">Vehicle Price</label>
        <input
          class="input"
          type="number"
          bind:value={vehiclePrice}
          min="0"
          step="1000"
        />
      </div>

      <div class="input-group">
        <label class="label">Down Payment</label>
        <input
          class="input"
          type="number"
          bind:value={downPayment}
          min="0"
          step="500"
        />
        <small class="help-text">Recommended: 20% of vehicle price</small>
      </div>

      <div class="input-group">
        <label class="label">Loan Term (months)</label>
        <input
          class="input"
          type="number"
          bind:value={loanTerm}
          min="12"
          max="84"
          step="12"
        />
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
        <label class="label">Sales Tax Rate (%)</label>
        <input
          class="input"
          type="number"
          bind:value={salesTax}
          min="0"
          max="15"
          step="0.1"
        />
      </div>

      <div class="input-group">
        <label class="label">Title & Registration Fees</label>
        <input
          class="input"
          type="number"
          bind:value={titleAndFees}
          min="0"
          step="100"
        />
      </div>
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
        <div>Sales Tax: {salesTaxAmount.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
        <div>Title & Fees: {titleAndFees.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
        <div>Total Financed: {totalFinanced.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</div>
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
              <th>Balance</th>
            </tr>
          </thead>
          <tbody>
            {#each acceleratedAmortizationSchedule.slice(0, 12) as row}
              <tr>
                <td>{row.month}</td>
                <td>{row.payment.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</td>
                <td>{row.principal.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</td>
                <td>{row.interest.toLocaleString('en-US', { style: 'currency', currency: 'USD' })}</td>
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
    <div bind:this={chartContainer} class="chart-container"></div>
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
          <strong>Interest Saved</strong>
          <div class="value" style="color: #10b981;">
            {formatCurrency(interestSaved)}
          </div>
        </div>
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
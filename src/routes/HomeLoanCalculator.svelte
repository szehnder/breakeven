<script lang="ts">
  import { onMount } from 'svelte';
  import * as echarts from 'echarts';

  // User inputs
  let homePrice: number = 400_000;
  let downPayment: number = 80_000;
  let loanTermYears: number = 30; // in years
  let loanTerm: number = loanTermYears * 12; // converted to months
  let interestRate: number = 6.5; // annual interest rate
  let propertyTaxRate: number = 1.2; // annual property tax rate
  let homeownersInsurance: number = 1_500; // annual insurance cost
  let hoaFees: number = 200; // monthly HOA fees
  let pmiRate: number = 0.5; // annual PMI rate (if applicable)

  // Chart element reference
  let chartContainer: HTMLElement;
  let chart: echarts.ECharts;

  // Computed values
  $: loanAmount = homePrice - downPayment;
  $: monthlyInterestRate = interestRate / 100 / 12;
  $: downPaymentPercent = (downPayment / homePrice) * 100;
  $: requiresPMI = downPaymentPercent < 20;
  $: monthlyPMI = requiresPMI ? (loanAmount * (pmiRate / 100)) / 12 : 0;
  $: monthlyPropertyTax = (homePrice * (propertyTaxRate / 100)) / 12;
  $: monthlyInsurance = homeownersInsurance / 12;
  $: totalFinanced = loanAmount;

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

  // Initialize and update chart
  $: if (chart && chartData) {
    const option = {
      title: {
        text: 'Monthly Payment Breakdown Over Time',
        left: 'center'
      },
      tooltip: {
        trigger: 'axis',
        formatter: function(params: any) {
          const month = params[0].dataIndex + 1;
          const principal = params[0].value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const interest = params[1].value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const pmi = params[2].value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const propertyTax = params[3].value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const insurance = params[4].value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const hoa = params[5].value.toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          const total = params.reduce((sum: number, param: any) => sum + param.value, 0)
            .toLocaleString('en-US', { style: 'currency', currency: 'USD' });
          
          return `Month ${month}<br/>
                  Total Payment: ${total}<br/>
                  Principal: ${principal}<br/>
                  Interest: ${interest}<br/>
                  PMI: ${pmi}<br/>
                  Property Tax: ${propertyTax}<br/>
                  Insurance: ${insurance}<br/>
                  HOA: ${hoa}`;
        }
      },
      legend: {
        data: ['Principal', 'Interest', 'PMI', 'Property Tax', 'Insurance', 'HOA'],
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
        },
        {
          name: 'PMI',
          type: 'line',
          stack: 'total',
          areaStyle: {
            opacity: 0.8,
            color: '#f59e0b'
          },
          data: chartData.pmi,
          smooth: true,
          lineStyle: {
            width: 3
          },
          itemStyle: {
            color: '#f59e0b'
          }
        },
        {
          name: 'Property Tax',
          type: 'line',
          stack: 'total',
          areaStyle: {
            opacity: 0.8,
            color: '#10b981'
          },
          data: chartData.propertyTax,
          smooth: true,
          lineStyle: {
            width: 3
          },
          itemStyle: {
            color: '#10b981'
          }
        },
        {
          name: 'Insurance',
          type: 'line',
          stack: 'total',
          areaStyle: {
            opacity: 0.8,
            color: '#8b5cf6'
          },
          data: chartData.insurance,
          smooth: true,
          lineStyle: {
            width: 3
          },
          itemStyle: {
            color: '#8b5cf6'
          }
        },
        {
          name: 'HOA',
          type: 'line',
          stack: 'total',
          areaStyle: {
            opacity: 0.8,
            color: '#ec4899'
          },
          data: chartData.hoa,
          smooth: true,
          lineStyle: {
            width: 3
          },
          itemStyle: {
            color: '#ec4899'
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
          type="number"
          bind:value={homePrice}
          min="0"
          step="10000"
        />
      </div>

      <div class="input-group">
        <label class="label">Down Payment</label>
        <input
          class="input"
          type="number"
          bind:value={downPayment}
          min="0"
          step="10000"
        />
        <small class="help-text">Recommended: 20% of home price</small>
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
    <div bind:this={chartContainer} class="chart-container"></div>
  </div>
</div> 
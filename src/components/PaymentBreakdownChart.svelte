<script lang="ts">
  import { onMount } from 'svelte';
  import * as echarts from 'echarts';

  export let title: string;
  export let data: {
    months: number[];
    principal: number[];
    interest: number[];
    pmi: number[];
    propertyTax: number[];
    insurance: number[];
    hoa: number[];
  };

  let chartContainer: HTMLElement;
  let chart: echarts.ECharts;

  onMount(() => {
    chart = echarts.init(chartContainer);
    window.addEventListener('resize', () => chart.resize());
    return () => {
      chart.dispose();
      window.removeEventListener('resize', () => chart.resize());
    };
  });

  $: if (chart && data) {
    const option = {
      title: {
        text: title,
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
        data: data.months,
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
          data: data.principal,
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
          data: data.interest,
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
          data: data.pmi,
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
          data: data.propertyTax,
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
          data: data.insurance,
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
          data: data.hoa,
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
</script>

<style>
  .chart-container {
    width: 100%;
    height: 400px;
    margin-top: 2rem;
  }
</style>

<div bind:this={chartContainer} class="chart-container"></div> 
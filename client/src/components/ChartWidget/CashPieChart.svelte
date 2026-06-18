<script lang="ts">
  import { Card, Chart } from 'flowbite-svelte'
  import { _ } from 'svelte-i18n'
  import Caption from './../Caption.svelte'
  import { genCashPieOptions } from '../../helper/chart'
  import type { ApexOptions } from 'apexcharts'

  export let cashEquivalentTotal = 0
  export let otherAssetsTotal = 0
  export let currencySymbol = ''

  const options: ApexOptions | any = genCashPieOptions('light')

  $: roundedCashEquivalent = Math.round(cashEquivalentTotal)
  $: roundedOtherAssets = Math.round(otherAssetsTotal)

  $: {
    const total = cashEquivalentTotal + otherAssetsTotal
    if (total > 0) {
      options.series = [
        Number(((cashEquivalentTotal / total) * 100).toFixed(2)),
        Number(((otherAssetsTotal / total) * 100).toFixed(2)),
      ]
    } else {
      options.series = [0, 0]
    }
    options.labels = [$_('cashEquivalent'), $_('otherAssets')]
    options.tooltip.y.formatter = function (value, { seriesIndex }) {
      const amount = seriesIndex === 0 ? roundedCashEquivalent : roundedOtherAssets
      return `${value}% (${currencySymbol}${amount})`
    }
  }
</script>

<Card size="xl" class="h-fit shadow-none md:p-4">
  <div class="mb-4 flex flex-row gap-3 sm:flex-row sm:items-start sm:justify-between">
    <Caption title={$_('cashEquivalentPercentage')} subtitle={$_('currentAssetStatus')}></Caption>
  </div>

  <Chart {options}></Chart>

  <div class="my-6 flex flex-wrap items-center justify-around pt-4 sm:pt-6">
    <div class="flex items-center space-x-2">
      <div class="h-3 w-3 rounded-full bg-[#2edfa3]"></div>
      <span class="text-sm font-medium text-gray-700">{$_('cashEquivalent')}</span>
      <span class="text-sm font-bold text-blue">
        {options.series[0]}% ({currencySymbol}{roundedCashEquivalent})
      </span>
    </div>
    <div class="flex items-center space-x-2">
      <div class="h-3 w-3 rounded-full bg-[#E5E7EB]"></div>
      <span class="text-sm font-medium text-gray-700">{$_('otherAssets')}</span>
      <span class="text-sm font-bold text-blue">
        {options.series[1]}% ({currencySymbol}{roundedOtherAssets})
      </span>
    </div>
  </div>
</Card>

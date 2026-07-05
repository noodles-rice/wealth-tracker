<script lang="ts">
  import { Card, Chart } from 'flowbite-svelte'
  import { _ } from 'svelte-i18n'
  import Caption from './../Caption.svelte'
  import { genCashPieOptions } from '../../helper/chart'
  import type { ApexOptions } from 'apexcharts'

  export let cashEquivalentTotal = 0
  export let otherAssetsTotal = 0
  export let currencySymbol = ''

  const HEALTHY_THRESHOLD = 10
  const TIGHT_THRESHOLD = 5

  // Keep these class strings in sync with the i18n legend copy in lang/*.json.
  const STATUS_STYLES: Record<string, { dotClass: string; textClass: string; rowClass: string }> = {
    healthy: {
      dotClass: 'bg-[#2edfa3]',
      textClass: 'text-[#2edfa3]',
      rowClass: 'bg-[#2edfa3]/10',
    },
    tight: {
      dotClass: 'bg-[#f59e0b]',
      textClass: 'text-[#f59e0b]',
      rowClass: 'bg-[#f59e0b]/10',
    },
    dangerous: {
      dotClass: 'bg-[#ff4582]',
      textClass: 'text-[#ff4582]',
      rowClass: 'bg-[#ff4582]/10',
    },
  }

  const STATUS_LABEL_KEYS: Record<string, string> = {
    healthy: 'cashFlowStatusHealthy',
    tight: 'cashFlowStatusTight',
    dangerous: 'cashFlowStatusDangerous',
  }

  const options: ApexOptions | any = genCashPieOptions('light')

  $: roundedCashEquivalent = Math.round(cashEquivalentTotal)
  $: roundedOtherAssets = Math.round(otherAssetsTotal)

  $: cashEquivalentPercentage =
    cashEquivalentTotal + otherAssetsTotal > 0
      ? Number(((cashEquivalentTotal / (cashEquivalentTotal + otherAssetsTotal)) * 100).toFixed(2))
      : 0

  $: cashFlowStatus =
    cashEquivalentPercentage > HEALTHY_THRESHOLD
      ? 'healthy'
      : cashEquivalentPercentage > TIGHT_THRESHOLD
        ? 'tight'
        : 'dangerous'

  $: cashFlowStatusConfig = {
    ...STATUS_STYLES[cashFlowStatus],
    label: $_(STATUS_LABEL_KEYS[cashFlowStatus]),
  }

  $: statusLegendList = [
    {
      ...STATUS_STYLES.healthy,
      label: $_('cashFlowStatusHealthyLegend', { values: { threshold: HEALTHY_THRESHOLD } }),
    },
    {
      ...STATUS_STYLES.tight,
      label: $_('cashFlowStatusTightLegend', {
        values: { min: TIGHT_THRESHOLD, max: HEALTHY_THRESHOLD },
      }),
    },
    {
      ...STATUS_STYLES.dangerous,
      label: $_('cashFlowStatusDangerousLegend', { values: { threshold: TIGHT_THRESHOLD } }),
    },
  ]

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
    <Caption
      title={$_('cashEquivalentPercentage')}
      subtitle={$_('currentAssetStatus')}
      itemsAlign="start">
      <div class="flex flex-col items-center gap-2">
        <div class="flex items-center space-x-2 rounded-full bg-gray-100 px-3 py-1">
          <div class="h-2.5 w-2.5 rounded-full {cashFlowStatusConfig.dotClass}"></div>
          <span class="text-sm font-bold {cashFlowStatusConfig.textClass}">
            {cashFlowStatusConfig.label}
          </span>
        </div>
        <div class="rounded-lg border border-gray-200 bg-gray-50 p-2">
          <div class="flex flex-col items-start gap-1.5">
            {#each statusLegendList as item}
              <span
                class="flex w-full items-center justify-start gap-2 rounded-full {item.rowClass} px-2 py-0.5 text-xs font-medium {item.textClass}">
                <span class="h-1.5 w-1.5 rounded-full {item.dotClass}"></span>
                <span>{item.label}</span>
              </span>
            {/each}
          </div>
        </div>
      </div>
    </Caption>
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

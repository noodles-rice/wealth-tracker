<script>
  import { Heading } from 'flowbite-svelte'
  export let title = ''
  export let subtitle = ''
  export let itemsAlign = 'center'
</script>

<div
  class="flex w-full justify-between md:flex-col md:items-start md:justify-start md:space-y-2"
  class:items-center={itemsAlign === 'center'}
  class:items-start={itemsAlign === 'start'}>
  <div class="flex flex-col">
    <Heading tag="h2" class="text-xl">{title}</Heading>
    {#if subtitle}
      <p class="w-full text-base font-light text-grey">
        {subtitle}
      </p>
    {/if}
  </div>
  <slot />
</div>

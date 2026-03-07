<script lang="ts">
  import * as noUiSlider from "nouislider";
  import "nouislider/dist/nouislider.css";
  import { onMount } from "svelte";

  interface Props {
    minValue?: number;
    maxValue?: number;
    currentMin?: any;
    currentMax?: any;
    onset?: (data: [number, number]) => void;
    onupdate?: (data: [number, number]) => void;
  }

  let {
    minValue = 1, // Start at 1 for log scale
    maxValue = 1000,
    currentMin = $bindable(minValue),
    currentMax = $bindable(maxValue),
    onset,
    onupdate,
  }: Props = $props();

  let sliderContainer: any = $state();

  onMount(() => {
    const updateValues = (values: string[]) => {
      currentMin = parseFloat(values[0]);
      currentMax = parseFloat(values[1]);
      onupdate?.([currentMin, currentMax]);
    };

    noUiSlider.create(sliderContainer, {
      start: [currentMin, currentMax],
      connect: true,
      range: {
        min: [Math.log10(minValue)],
        max: [Math.log10(maxValue)],
      },
      // Use a logarithmic scale
      scale: [
        [0, Math.log10(minValue)],
        [100, Math.log10(maxValue)],
      ],
      // Format values for display
      format: {
        to: (value: number) => Math.pow(10, value).toFixed(0),
        from: (value: number) => Math.log10(value),
      },
    });

    sliderContainer.noUiSlider.on("update", (values: string[]) => {
      const min = parseFloat(values[0]);
      const max = parseFloat(values[1]);
      currentMin = Math.pow(10, min);
      currentMax = Math.pow(10, max);
      onupdate?.([currentMin, currentMax]);
    });

    sliderContainer.noUiSlider.on("set", () => {
      onset?.([currentMin, currentMax]);
    });
  });
</script>

<div class="my-4" id="slider" bind:this={sliderContainer}></div>

<style lang="postcss">
  @reference "tailwindcss";
  @reference "../../../css/app.css";

  :global(.noUi-horizontal) {
    height: 6px;
  }
  :global(.noUi-connect) {
    @apply bg-primary;
  }

  :global(.noUi-horizontal .noUi-handle) {
    @apply rounded-full w-7 h-7 -top-3;
  }

  :global(.noUi-handle::before) {
    @apply content-none;
  }

  :global(.noUi-handle::after) {
    @apply content-none;
  }
</style>

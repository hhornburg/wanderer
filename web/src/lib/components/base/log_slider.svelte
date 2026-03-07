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
    minValue = 0,
    maxValue = 1000,
    currentMin = minValue,
    currentMax = maxValue,
    onset,
    onupdate,
  }: Props = $props();

  let sliderContainer: any = $state();
  const offset = 1; // To avoid log(0)

  // Convert value to log scale with offset
  const toLogScale = (value: number) => Math.log10(value + offset);
  // Convert log scale back to value with offset
  const fromLogScale = (value: number) => Math.pow(10, value) - offset;

  onMount(() => {
    const slider = noUiSlider.create(sliderContainer, {
      start: [toLogScale(currentMin), toLogScale(currentMax)],
      connect: true,
      range: {
        min: toLogScale(minValue),
        max: toLogScale(maxValue),
      },
      format: {
        to: (value: number) => fromLogScale(value).toFixed(0),
        from: (value: number) => toLogScale(value),
      },
    });

    slider.on("update", (values: string[]) => {
      const min = parseFloat(values[0]);
      const max = parseFloat(values[1]);
      currentMin = fromLogScale(min);
      currentMax = fromLogScale(max);
      onupdate?.([currentMin, currentMax]);
    });

    slider.on("set", () => {
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

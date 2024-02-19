<script>
  import * as d3 from "d3";
  import { sharedXDomain } from "../store.js";
  import Tooltip from "./Tooltip.svelte";
  export let dimensions;
  export let numVideoPerMonth, importantDates, tomCount, yoyoCount, yawCount;

  // Configures scales
  $: x = d3
    .scaleTime()
    .domain(d3.extent(numVideoPerMonth, d => d.date))
    .range([0, dimensions.boundedWidth]);
  $: y = d3
    .scaleLinear()
    .domain([0, d3.max(numVideoPerMonth, d => +d.count)])
    .range([dimensions.boundedHeight, 0]);

  // Configure axes
  let gx, gy;
  $: d3.select(gx).call(d3.axisBottom(x));
  $: d3.select(gy).call(d3.axisLeft(y));

  // Generate area
  $: area = d3
    .area()
    .x(d => x(d.date))
    .y0(dimensions.boundedHeight)
    .y1(d => y(d.count));

  // Interactivity
  let hoveredData = null;
  function handleMouseOver(event, dataPoint) {
    hoveredData = {
      ...dataPoint,
      recorded_mouse_position: { x: event.pageX, y: event.pageY },
    };
  }

  function handleMouseOut() {
    hoveredData = null;
  }

  function handleClick(dataPoint) {
    window.open(dataPoint.link, "_blank");
  }
</script>

<div class="area-plot-wrapper">
  <svg
    class="area-plot"
    width={dimensions.width}
    height={dimensions.height}
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
    style="max-width: 100%; height: auto;"
  >
    <!-- The Data Points -->
    <g
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      <!-- The Area Chart! -->
      <path
        d={area(numVideoPerMonth)}
        fill="#cce5df"
        stroke="#69b3a2"
        stroke-width="1.5"
      />

      <!-- Important Dates -->
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <!-- svelte-ignore a11y-mouse-events-have-key-events -->
      <!-- svelte-ignore a11y-click-events-have-key-events -->

      {#each importantDates as data, i}
        <circle
          id={i}
          cx={x(data.date)}
          cy={y(-5)}
          r="5"
          fill="red"
          style="cursor: {'pointer'}"
          on:mouseover={event => handleMouseOver(event, data)}
          on:mouseout={handleMouseOut}
          on:click={() => handleClick(data)}
        >
        </circle>
      {/each}
    </g>

    <!-- The Axes -->
    <g
      bind:this={gx}
      transform={`translate(${dimensions.margin.left}, ${
        dimensions.boundedHeight + dimensions.margin.top
      })`}
    />
    <g
      bind:this={gy}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      <text
        transform="rotate(-90)"
        x={0 - dimensions.height / 2}
        y={0 - dimensions.margin.left + 20}
        text-anchor="middle"
        style="fill: black;"
      >
        Video Count
      </text>
    </g>
  </svg>

  <!-- Tooltip  -->
  {#if hoveredData}
    <Tooltip {hoveredData} chart_type={"area"} />
  {/if}
</div>

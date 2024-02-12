<script>
  import * as d3 from "d3";
  export let num_video_per_month, important_dates;
  export let dimensions;

  // $: console.log(dimensions.boundedWidth);
  // Configures scales
  $: x = d3
    .scaleTime()
    .domain(d3.extent(num_video_per_month, d => d.month))
    .range([0, dimensions.boundedWidth]);
  $: y = d3
    .scaleLinear()
    .domain([0, d3.max(num_video_per_month, d => +d.count)])
    .range([dimensions.boundedHeight, 0]);

  // Generate area
  $: area = d3
    .area()
    .x(d => x(d.month))
    .y0(dimensions.boundedHeight)
    .y1(d => y(d.count));

  // Configure axes
  let gx, gy;
  $: d3.select(gx).call(d3.axisBottom(x));
  $: d3.select(gy).call(d3.axisLeft(y));
</script>

<div class="area-plot">
  <svg
    width={dimensions.width}
    height={dimensions.height}
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
    style="max-width: 100%; height: auto;"
  >
    <!-- The Data Points -->
    <g
      role="img"
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      <!-- The Area Chart! -->
      <path
        d={area(num_video_per_month)}
        fill="#cce5df"
        stroke="#69b3a2"
        stroke-width="1.5"
      />

      <!-- The Important Dates -->
      {#each important_dates as event, i}
        <circle id={i} cx={x(event.date)} cy={y(0)} r="3" fill="red"></circle>
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
    />
  </svg>
</div>

<style>
</style>

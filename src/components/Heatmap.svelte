<script>
  import * as d3 from "d3";
  import Tooltip from "./Tooltip.svelte";
  export let dimensions;
  export let tagCount;

  let myGroup;
  let myVars;
  let uniqueGroup;
  let uniqueGroupArray;
  let uniqueVar;
  let uniqueVarArray;

  $: myGroup = tagCount.map(function (d) {
    return d.group;
  });
  $: myVars = tagCount.map(function (d) {
    return d.variable;
  });
  $: uniqueGroup = new Set(myGroup);
  $: uniqueGroupArray = [...uniqueGroup];
  $: uniqueVar = new Set(myVars);
  $: uniqueVarArray = [...uniqueVar];

  // Configures scales
  $: x = d3
    .scaleBand()
    .range([0, dimensions.boundedWidth])
    .domain(myGroup)
    .padding(0);
  $: y = d3
    .scaleBand()
    .range([dimensions.boundedHeight, 0])
    .domain(uniqueVarArray)
    .padding(0);

  // Configure colors
  $: myColor = d3
    .scaleSequential()
    .interpolator(d3.interpolatePuBu)
    .domain([0, 200]);

  // Configure axes
  $: xAxis = d3
    .scaleTime()
    .domain(d3.extent(tagCount, d => d.group))
    .range([0, dimensions.boundedWidth]);

  let gx, gy;
  $: d3.select(gx).call(d3.axisBottom(xAxis));
  $: d3.select(gy).call(d3.axisLeft(y));

  // Interactivity

  // Tooltip
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
</script>

<div class="heatmap-plot-wrapper">
  <svg
    id="heatmap-plot"
    width={dimensions.width}
    height={dimensions.height}
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
    style="max-width: 100%; height: auto;"
  >
    <!-- Tags Data  -->
    <g
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <!-- svelte-ignore a11y-mouse-events-have-key-events -->
      {#each tagCount as data, i}
        <rect
          id={i}
          x={x(data.group)}
          y={y(data.variable)}
          width={x.bandwidth()}
          height={y.bandwidth()}
          fill={myColor(data.value)}
          rx={0}
          ry={0}
          on:mouseover={event => handleMouseOver(event, data)}
          on:mouseout={handleMouseOut}
        />
      {/each}
    </g>

    <!-- Axes  -->
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

  <!-- Tooltip  -->
  {#if hoveredData}
    <Tooltip {hoveredData} chart_type={"heatmap"} />
  {/if}
</div>

<style>
  .heatmap-plot-wrapper {
    padding-top: 2em;
    padding-left: 2em;
    padding-bottom: 2em;
  }
</style>

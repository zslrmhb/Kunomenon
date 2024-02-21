<script>
  import * as d3 from "d3";
  import { sharedXDomain, isZoom } from "../store.js";
  import Tooltip from "./Tooltip.svelte";

  export let dimensions, curDataset, activeMetric;

  

  // Configures scales
  $: x = d3
    .scaleTime()
    .domain(d3.extent(curDataset, d => d.date))
    .range([0, dimensions.boundedWidth]);

  $: y = d3
    .scaleLinear()
    .domain(d3.extent(curDataset, d => +d.count))
    .range([dimensions.boundedHeight, 0]);

  // Configure axes
  let gx, gy;
  $: d3.select(gx).call(d3.axisBottom(x));
  $: d3.select(gy).call(d3.axisLeft(y));

  // Axes Labels
  const datasetLabels = {
    playCount: "View Count",
    likeCount: "Like Count",
    reviewCount: "Review Count",
    favoriteCount: "Favorite Count",
    danmakuCount: "Danmaku Count",
    durationCount: "Duration Count (minutes)",
  };
  let yAxisLabel = "";
  $: yAxisLabel = datasetLabels[activeMetric];

  // Interacticity

  // Tooltip
  let hoveredData = null;
  function handleMouseOver(event, dataPoint) {
    if (isInTopN(dataPoint)) {
      hoveredData = {
        ...dataPoint,
        recorded_mouse_position: { x: event.pageX, y: event.pageY },
        activeMetric: yAxisLabel,
      };
    }
  }

  function handleMouseOut() {
    hoveredData = null;
  }

  function handleClick(dataPoint) {
    if (isInTopN(dataPoint)) {
      window.open(dataPoint.arcurl, "_blank");
    }
  }

  // Top N points
  let N = 1;
  let topNData;
  $: topNData = [...curDataset]
    .sort((a, b) => d3.descending(a.count, b.count))
    .slice(0, N);

  function isInTopN(dataPoint) {
    return topNData.some(topPoint => topPoint?.aid === dataPoint?.aid);
  }

  // Slider
  function handleSliderChange(event) {
    N = +event.target.value;
    topNData = [...curDataset]
      .sort((a, b) => d3.descending(a.count, b.count))
      .slice(0, N);
    d3.select("#scatter-plot")
      .selectAll("circle")
      .data(curDataset)
      .attr("fill", d => (isInTopN(d) ? "#4574cc" : "gray"))
      .style("cursor", d => (isInTopN(d) ? "pointer" : "default"));
  }

  // Brush
  $: brush = d3
    .brushX()
    .extent([
      [0, 0],
      [dimensions.boundedWidth, dimensions.boundedHeight],
    ])
    .on("end", updateChart);
  let brushGroup;
  $: if (brushGroup) {
    d3.select(brushGroup).call(brush);
    d3.select(brushGroup).on("dblclick", resetChart);
  }

  function updateChart(event) {
    const selection = event.selection;
    if (selection) {
      const newDomain = [x.invert(selection[0]), x.invert(selection[1])];
      sharedXDomain.set(newDomain);
      d3.select(brushGroup).call(brush.move, null);
      isZoom.set(true);
    }
  }
  function resetChart() {
    sharedXDomain.set(d3.extent(curDataset, d => d.date));
    isZoom.set(false);
  }

  // Sync Both the Area and Scatter plot
  $: if ($sharedXDomain) {
    x.domain($sharedXDomain);
    d3.select(gx).transition().duration(500).call(d3.axisBottom(x));
    d3.select("#scatter-plot")
      .selectAll("circle")
      .data(curDataset)
      .transition()
      .duration(500)
      .attr("cx", d => x(d.date))
      .attr("cy", d => y(d.count));
  }
</script>

<!-- Slider -->
<div class="slider">
  <label for="top-n-slider">Top {N}:</label>
  <input
    type="range"
    id="top-n-slider"
    min="1"
    max="100"
    value={N}
    on:input={handleSliderChange}
    orient="vertical"
  />
</div>

<div class="scatter-plot-wrapper">
  <svg
    id="scatter-plot"
    width={dimensions.width}
    height={dimensions.height}
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
    style="max-width: 100%; height: auto;"
    ><defs>
      <marker
        id="arrowhead"
        markerWidth="10"
        markerHeight="7"
        refX="0"
        refY="3.5"
        orient="auto"
      >
        <polygon points="0 0, 10 3.5, 0 7" fill="black" />
      </marker>
    </defs>
    <!-- Brush  -->
    <g
      bind:this={brushGroup}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    ></g>
    <g
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      <!-- The Data Points -->
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <!-- svelte-ignore a11y-mouse-events-have-key-events -->
      <!-- svelte-ignore a11y-click-events-have-key-events -->
      {#each curDataset as data, i}
        <circle
          id={i}
          cx={x(data.date)}
          cy={y(data.count)}
          r="5"
          fill={isInTopN(data) ? "#4574cc" : "gray"}
          style="cursor: {isInTopN(data) ? 'pointer' : 'default'}"
          on:mouseover={event => handleMouseOver(event, data)}
          on:mouseout={handleMouseOut}
          on:click={() => handleClick(data)}
        ></circle>
        {#if !$isZoom}
          {#if topNData[0]?.aid === data?.aid}
            <line
              class="annotation-line"
              x1={x(data.date) + 60}
              y1={y(data.count)}
              x2={x(data.date) + 20}
              y2={y(data.count)}
              marker-end="url(#arrowhead)"
            />
            <text
              class="anotation-text"
              x={x(data.date) + 65}
              y={y(data.count) + 5}
              style="fill: black;"
            >
              Click me!
            </text>
          {/if}
        {/if}
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
    >
      <text
        transform="rotate(-90)"
        x={0 - dimensions.height / 2}
        y={0 - dimensions.margin.left + 20}
        text-anchor="middle"
        style="fill: black;"
      >
        {yAxisLabel}
      </text>
    </g>
  </svg>

  <!-- Tooltip  -->
  {#if hoveredData}
    <Tooltip {hoveredData} chart_type={"scatter"} />
  {/if}
</div>

<style>
  .scatter-plot-wrapper {
    padding: 2em;
    display: flex;
    align-items: flex-start;
    /* vertical-align: bottom; */
  }

  .slider {
    text-align: center;
    /* margin-bottom: -0.8em; */
    /*  */
    /* width: 2em; */
  }

  .annotation-line {
    stroke: rgb(0, 0, 0);
    stroke-dasharray: 5, 5;
    opacity: 0.5;
  }
</style>

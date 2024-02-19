<script>
  import * as d3 from "d3";
  export let dimensions, curDataset, activeMetric;
  import Tooltip from "./Tooltip.svelte";

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
    d3.select(".scatter-plot-wrapper .scatter-plot")
      .selectAll("circle")
      .data(curDataset)
      .attr("fill", d => (isInTopN(d) ? "#4574cc" : "gray"))
      .style("cursor", d => (isInTopN(d) ? "pointer" : "default"));
  }
</script>

<div class="scatter-plot-wrapper">
  <svg
    class="scatter-plot"
    width={dimensions.width}
    height={dimensions.height}
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
    style="max-width: 100%; height: auto;"
  >
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
          r="7"
          fill={isInTopN(data) ? "#4574cc" : "gray"}
          style="cursor: {isInTopN(data) ? 'pointer' : 'default'}"
          on:mouseover={event => handleMouseOver(event, data)}
          on:mouseout={handleMouseOut}
          on:click={() => handleClick(data)}
        ></circle>{/each}
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

  <!-- Slider  -->
  <div class="slider">
    <label for="top-n-slider">Top N: {N}</label>
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
</div>

<style>
  .scatter-plot-wrapper {
    display: inline-block;
    vertical-align: bottom;
  }

  .slider {
    height: 200px;
  }
</style>

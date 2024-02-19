<script>
  import * as d3 from "d3";
  import { sharedXDomain } from "../store.js";
  import Tooltip from "./Tooltip.svelte";
  export let dimensions;
  export let cur_dataset, activeDataset;

  // Configures scales
  $: x = d3
    .scaleTime()
    .domain(d3.extent(cur_dataset, d => d.date))
    .range([0, dimensions.boundedWidth]);

  $: y = d3
    .scaleLinear()
    .domain(d3.extent(cur_dataset, d => +d.count))
    .range([dimensions.boundedHeight, 0]);

  // Configure axes
  let gx, gy;
  $: d3.select(gx).call(d3.axisBottom(x));
  $: d3.select(gy).call(d3.axisLeft(y));

  // Axes Labels
  const datasetLabels = {
    play_count: "View Count",
    like_count: "Like Count",
    review_count: "Review Count",
    danmaku_count: "Danmaku Count",
    duration_count: "Duration Count (minutes)",
  };
  let yAxisLabel = "";
  $: yAxisLabel = datasetLabels[activeDataset];

  // $: console.log(cur_dataset);

  // Interactivity
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
    }
  }

  function resetChart() {
    sharedXDomain.set(d3.extent(cur_dataset, d => d.date));
  }

  // Sync Both the Area and Scatter plot
  $: if ($sharedXDomain) {
    x.domain($sharedXDomain);
    d3.select(gx).transition().duration(500).call(d3.axisBottom(x));
    d3.select("#scatter-plot")
      .selectAll("circle")
      .data(cur_dataset)
      .transition()
      .duration(500)
      .attr("cx", d => x(d.date))
      .attr("cy", d => y(d.count));
  }

  // Sort and get top N poionts
  let N = 1;
  let topNData;

  $: topNData = [...cur_dataset]
    .sort((a, b) => d3.descending(a.count, b.count))
    .slice(0, N);

  function isInTopN(dataPoint) {
    return topNData.includes(dataPoint);
  }

  function handleSliderChange(event) {
    N = +event.target.value;
    topNData = [...cur_dataset]
      .sort((a, b) => d3.descending(a.count, b.count))
      .slice(0, N);
    d3.select("#scatter-plot")
      .selectAll("circle")
      .data(cur_dataset)
      .attr("fill", d => (isInTopN(d) ? "#4574cc" : "gray"));
  }

  // Tooltip
  let hoveredData = null;
  function handleMouseOver(event, dataPoint) {
    // console.log(data);
    if (isInTopN(dataPoint)) {
      console.log("THIS!");
      hoveredData = {
        ...dataPoint,
        recorded_mouse_position: { x: event.pageX, y: event.pageY },
      };
    }
  }
  // $: console.log(hoveredData);

  function handleMouseOut() {
    hoveredData = null;
  }

  function handleClick(dataPoint) {
    if (isInTopN(dataPoint)) {
      window.open(dataPoint.arcurl, "_blank");
    }
  }
</script>

<div class="scatter-plot-wrapper">
  <svg
    id="scatter-plot"
    width={dimensions.width}
    height={dimensions.height}
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
    style="max-width: 100%; height: auto;"
  >
    <g
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <!-- svelte-ignore a11y-mouse-events-have-key-events -->
      {#each cur_dataset as data, i}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <circle
          id={i}
          cx={x(data.date)}
          cy={y(data.count)}
          r="7"
          fill={isInTopN(data) ? "#4574cc" : "gray"}
          on:mouseover={event => handleMouseOver(event, data)}
          on:mouseout={handleMouseOut}
          on:click={() => handleClick(data)}
        ></circle>
      {/each}
    </g>

    <!-- Axes -->
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

    <!-- Brush  -->
    <g
      bind:this={brushGroup}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    ></g>
  </svg>
</div>
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

  <!-- Tooltip  -->
  {#if hoveredData}
    <Tooltip {hoveredData} chart_type={"scatter"} />
  {/if}
  <!-- https://www.bilibili.com/video/BV1o7421K7pY/?spm_id_from=333.1007.tianma.1-3-3.click&vd_source=b9984a0b10edc46995ca14a3500b11a3 -->

  <!-- //player.bilibili.com/player.html?aid=1350175617 -->
</div>

<!-- <div class="controls-container"> -->
<!-- <div class="top-n-filter">
    <label for="top-n"> Top N Points</label>
    <input type="number" id="top-n" min="1" max="1000" bind:value={N} />
  </div> -->

<style>
  .scatter-plot-wrapper {
    display: inline-block;
    vertical-align: bottom;
  }

  .slider {
    /* width: 20px;
    height: 200px; /* Adjust as per your graph's height */
    /* padding: 20px 0; */
    /* display: flex; */
    /* flex-direction: column; */
    /* justify-content: center; */
    /* align-items: center; */
  }

  circle {
    cursor: pointer;
  }
</style>

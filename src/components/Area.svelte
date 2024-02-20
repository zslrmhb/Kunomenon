<script>
  import * as d3 from "d3";
  import { sharedXDomain } from "../store.js";
  import Tooltip from "./Tooltip.svelte";
  export let dimensions;
  export let numVideoPerMonth, importantDates, tomCount, yoyoCount, yawCount;

  let activeDatasets = {
    Tom: false,
    Yoyo: false,
    Yaw: false,
  };

  // Configures scales
  $: x = d3
    .scaleTime()
    .domain(d3.extent(numVideoPerMonth, d => d.pubmonth))
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
    .x(d => x(d.pubmonth))
    .y0(dimensions.boundedHeight)
    .y1(d => y(d.count));

  // Interactivity

  // Top 3 content creators

  let combinedData = getCombinedData();

  function updateSmallChart() {
    combinedData = getCombinedData();
  }
  // $: console.log(combinedData);
  function getCombinedData() {
    let data = numVideoPerMonth.map(d => ({
      pubmonth: d.pubmonth,
      count:
        0 +
        (activeDatasets.Tom ? getCountOnDate(tomCount, d.pubmonth) : 0) +
        (activeDatasets.Yoyo ? getCountOnDate(yoyoCount, d.pubmonth) : 0) +
        (activeDatasets.Yaw ? getCountOnDate(yawCount, d.pubmonth) : 0),
    }));
    return data;
  }
  function getCountOnDate(dataset, month) {
    let entry = dataset.find(d => +d.pubmonth === +month);
    return entry ? entry.count : 0;
  }

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
    }
  }

  function resetChart() {
    sharedXDomain.set(d3.extent(numVideoPerMonth, d => d.pubmonth));
  }

  // Sync Both the Area and Scatter plot
  $: if ($sharedXDomain) {
    x.domain($sharedXDomain);
    d3.select(gx).transition().duration(500).call(d3.axisBottom(x));
    d3.select("#area-plot")
      .select(".main-plot")
      .datum(numVideoPerMonth) // Re-bind data
      .transition()
      .duration(500)
      .attr("d", area);
    d3.select("#area-plot")
      .select(".combined-plot")
      .datum(combinedData) // Re-bind data
      .transition()
      .duration(500)
      .attr("d", area);
    d3.select("#area-plot")
      .selectAll("circle")
      .data(importantDates)
      .transition()
      .duration(500)
      .attr("cx", d => x(d.pubmonth))
      .attr("cy", y(-8));
  }
</script>

<div class="area-plot-wrapper">
  <svg
    id="area-plot"
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
        class="main-plot"
        d={area(numVideoPerMonth)}
        fill="#cce5df"
        stroke="#69b3a2"
        stroke-width="1.5"
      />
      <path
        class="combined-plot"
        d={area(combinedData)}
        fill="blue"
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
          cy={y(-8)}
          r="7"
          fill="red"
          style="cursor: {'pointer'}"
          on:mouseover={event => handleMouseOver(event, data)}
          on:mouseout={handleMouseOut}
          on:click={() => handleClick(data)}
        >
        </circle>
      {/each}
    </g>

    <!-- The Brush -->
    <g
      bind:this={brushGroup}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    ></g>

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

  <div class="buttons-container">
    <h3>Top 3 Content Creators:</h3>
    {#each Object.keys(activeDatasets) as datasetName}
      <div class="button-row">
        <button
          class:active={activeDatasets[datasetName]}
          on:click={() => {
            activeDatasets[datasetName] = !activeDatasets[datasetName];
            updateSmallChart();
          }}
        >
          {datasetName}
        </button>
      </div>
    {/each}
  </div>

  <!-- Tooltip  -->
  {#if hoveredData}
    <Tooltip {hoveredData} chart_type={"area"} />
  {/if}
</div>

<style>
  .buttons-container button {
    /* Basic styling */
    margin: 0.5em;
    padding: 0.5em;
    border: 1px solid #ccc;
    background-color: white;
    cursor: pointer;
  }

  .buttons-container button.active {
    /* Active button styling */
    color: white;
    background-color: #4574cc;
    border-color: #4574cc;
  }

  .button-row {
    display: flex;
    align-items: center;
    /* margin-bottom: 10px; Space between rows */
  }
</style>

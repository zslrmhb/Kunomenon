<script>
  import * as d3 from "d3";
  import { sharedXDomain, isZoom } from "../store.js";
  import Tooltip from "./Tooltip.svelte";
  export let dimensions;
  export let numVideoPerMonth, importantDates, tomCount, yoyoCount, yawCount;

  let activeDatasets = {
    ProfessorTom: false,
    BossChen: false,
    YawFitness: false,
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
    .y1(d => y(d.count))
    .curve(d3.curveMonotoneX);

  // Interactivity

  // Top 3 content creators

  let combinedData = getCombinedData();

  function updateSmallChart() {
    combinedData = getCombinedData();
  }

  function getCombinedData() {
    let data = numVideoPerMonth.map(d => ({
      pubmonth: d.pubmonth,
      count:
        0 +
        (activeDatasets.ProfessorTom
          ? getCountOnDate(tomCount, d.pubmonth)
          : 0) +
        (activeDatasets.BossChen ? getCountOnDate(yoyoCount, d.pubmonth) : 0) +
        (activeDatasets.YawFitness ? getCountOnDate(yawCount, d.pubmonth) : 0),
    }));
    return data;
  }
  function getCountOnDate(dataset, month) {
    let entry = dataset.find(d => +d.pubmonth === +month);
    return entry ? entry.count : 0;
  }

  function interpolateYCoordinate(date) {
    let i = 0;
    while (
      i < numVideoPerMonth.length - 1 &&
      numVideoPerMonth[i].pubmonth < date
    ) {
      i++;
    }
    if (i === 0) return y(numVideoPerMonth[0].count);
    if (i === numVideoPerMonth.length - 1) return y(numVideoPerMonth[i].count);

    const start = numVideoPerMonth[i - 1];
    const end = numVideoPerMonth[i];
    const proportion =
      (date - start.pubmonth) / (end.pubmonth - start.pubmonth);
    return y(start.count + proportion * (end.count - start.count));
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
      isZoom.set(true);
    }
  }

  function resetChart() {
    sharedXDomain.set(d3.extent(numVideoPerMonth, d => d.pubmonth));
    isZoom.set(false);
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
    d3.select(".area-plot-wrapper #area-plot")
      .selectAll("circle")
      .data(importantDates)
      .transition()
      .duration(500)
      .attr("cx", d => x(d.date))
      .attr("cy", d => interpolateYCoordinate(d.date));
  }
</script>

<div class="area-plot-wrapper">
  <svg
    id="area-plot"
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

    <!-- The Brush -->
    <g
      bind:this={brushGroup}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    ></g>
    <!-- The Data Points -->
    <g
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      <!-- The Area Chart! -->
      <path class="main-plot" d={area(numVideoPerMonth)} />
      <path class="combined-plot" d={area(combinedData)} />

      <!-- Important Dates -->
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <!-- svelte-ignore a11y-mouse-events-have-key-events -->
      <!-- svelte-ignore a11y-click-events-have-key-events -->

      {#each importantDates as data, i}
        <circle
          id={i}
          cx={x(data.date)}
          cy={interpolateYCoordinate(data.date)}
          r="5"
          fill="red"
          style="cursor: {'pointer'}"
          on:mouseover={event => handleMouseOver(event, data)}
          on:mouseout={handleMouseOut}
          on:click={() => handleClick(data)}
        >
        </circle>
        {#if !$isZoom}
          {#if i == 2}
            <line
              class="annotation-line"
              x1={x(data.date) + 60}
              y1={interpolateYCoordinate(data.date)}
              x2={x(data.date) + 20}
              y2={interpolateYCoordinate(data.date)}
              marker-end="url(#arrowhead)"
            />
            <text
              class="anotation-text"
              x={x(data.date) + 65}
              y={interpolateYCoordinate(data.date) + 5}
              style="fill: black;"
            >
              Click me!
            </text>
          {/if}
        {/if}
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

    <g
      class="buttons-group"
      transform={`translate(${dimensions.margin.left + 20}, ${
        dimensions.margin.top + 20
      })`}
    >
      <text class="creator-title" style="font-size: 15px; fill: black"
        >Top 3 content creators</text
      >

      {#each Object.keys(activeDatasets) as datasetName, index}
        <foreignObject x="-10" y={index * 25} width="200" height="30">
          <button
            xmlns="http://www.w3.org/1999/xhtml"
            class:active={activeDatasets[datasetName]}
            on:click={() => {
              activeDatasets[datasetName] = !activeDatasets[datasetName];
              updateSmallChart();
            }}
          >
            {datasetName}
          </button></foreignObject
        >
      {/each}
    </g>
  </svg>

  <!-- Tooltip  -->
  {#if hoveredData}
    <Tooltip {hoveredData} chart_type={"area"} />
  {/if}
</div>

<style>
  .area-plot-wrapper {
    padding-top: 2em;
    padding-left: 2em;
  }

  .main-plot {
    fill: #cce5df; /* Light green fill */
    stroke: #69b3a2; /* Darker green border */
    stroke-width: 0.15em; /* Line thickness */
  }

  .combined-plot {
    fill: #fdf8f6;
    stroke: #d86e41;
    stroke-width: 0.15em;
  }

  .buttons-group button {
    padding: 0.05em 0.6em;
    border: 0.05em solid #8a5959;
    margin: 0.7em;
    background-color: rgb(241, 233, 233);
    cursor: pointer;
    border-radius: 0.5em;
  }

  .buttons-group button.active {
    color: white;
    background-color: #d86e41;
    border: none;
    transform: scale(1);
  }

  .annotation-line {
    stroke: rgb(0, 0, 0);
    stroke-dasharray: 5, 5;
    opacity: 0.5;
  }
</style>

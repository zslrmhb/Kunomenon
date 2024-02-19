<script>
  import * as d3 from "d3";
  import { sharedXDomain } from "../store.js";
  import Tooltip from "./Tooltip.svelte";
  export let num_video_per_month,
    important_dates,
    tom_count,
    yoyo_count,
    yaw_count;
  export let dimensions;

  // Configures scales
  $: x = d3
    .scaleTime()
    .domain(d3.extent(num_video_per_month, d => d.date))
    .range([0, dimensions.boundedWidth]);
  $: y = d3
    .scaleLinear()
    .domain([0, d3.max(num_video_per_month, d => +d.count)])
    .range([dimensions.boundedHeight, 0]);

  // Generate area
  $: area = d3
    .area()
    .x(d => x(d.date))
    .y0(dimensions.boundedHeight)
    .y1(d => y(d.count));

  // Configure axes
  let gx, gy;
  $: d3.select(gx).call(d3.axisBottom(x));
  $: d3.select(gy).call(d3.axisLeft(y));

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
    sharedXDomain.set(d3.extent(num_video_per_month, d => d.date));
  }

  // Sync Both the Area and Scatter plot
  $: if ($sharedXDomain) {
    x.domain($sharedXDomain);
    d3.select(gx).transition().duration(500).call(d3.axisBottom(x));
    d3.select("#area-plot")
      .select("path")
      .datum(num_video_per_month) // Re-bind data
      .transition()
      .duration(500)
      .attr("d", area);
    d3.select("#area-plot")
      .selectAll("circle")
      .data(important_dates)
      .transition()
      .duration(500)
      .attr("cx", d => x(d.date))
      .attr("cy", y(0));
  }

  // Tooltip
  let hovered = -1;
  let recorded_mouse_position = { x: 0, y: 0 };
  let tooltip_content = "44444";
  // $: console.log(hovered);

  // Top 3 Content Creators
  let showTomCount = false;
  let showYoyoCount = false;
  let showYawCount = false;

  function handleClickCircle(circleIndex) {
    if (circleIndex === 5) {
      showTomCount = true;
      showYoyoCount = false;
      showYawCount = false;
    } else if (circleIndex === 6) {
      showYoyoCount = true;
      showTomCount = true;
      showYawCount = false;
    } else if (circleIndex === 7) {
      showYawCount = true;
      showTomCount = true;
      showYoyoCount = true;
    }
  }

  function calculateCumulativeCount() {
    // Assuming all arrays have the same length and corresponding dates
    return num_video_per_month.map((d, i) => {
      let count = d.count;
      if (showTomCount) count += tom_count[i].count;
      if (showYoyoCount) count += yoyo_count[i].count;
      if (showYawCount) count += yaw_count[i].count;
      return { date: d.date, count };
    });
  }

  $: cumulativeCount = calculateCumulativeCount();
  // console.log(calculateCumulativeCount);
</script>

<div class="area-plot">
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
        d={area(num_video_per_month)}
        fill="#cce5df"
        stroke="#69b3a2"
        stroke-width="1.5"
      />
      <path
        d={area(cumulativeCount)}
        fill="yellow"
        stroke="#69b3a2"
        stroke-width="1.5"
      />

      <!-- The Important Dates -->
      <!-- svelte-ignore a11y-no-static-element-interactions -->
      <!-- svelte-ignore a11y-mouse-events-have-key-events -->
      {#each important_dates as event, i}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <circle
          id={i}
          cx={x(event.date)}
          cy={y(-5)}
          r="5"
          fill="red"
          on:mouseover={event => {
            hovered = 1;
            recorded_mouse_position = {
              x: event.pageX,
              y: event.pageY,
            };
          }}
          on:mouseout={event => (hovered = -1)}
          on:click={() => handleClickCircle(i)}
        />
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
    <!-- The Brush -->
    <g
      bind:this={brushGroup}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    ></g>
  </svg>

  <!-- Tooltip -->
  {#if hovered !== -1}
    <Tooltip
      {tooltip_content}
      {recorded_mouse_position}
      chart_type={"area"}
      link="null"
    />
  {/if}

  <!-- <div class="legend">
    {#if showTomCount}
      <p>Tom Count</p>
    {:else if showYoyoCount}
      <p>Tom Count + Yoyo Count</p>
    {:else if showYawCount}
      <p>Tom Count + Yoyo Count + Yaw Count</p>
    {/if}
  </div> -->
</div>

<!-- <iframe src="//player.bilibili.com/player.html?aid=1350175617&bvid=BV1oB421678P&cid=1427854872&p=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe> -->

<style>
</style>

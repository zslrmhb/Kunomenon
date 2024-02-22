<script>
  export let hoveredData;
  export let chart_type;
  import { onMount } from "svelte";
  import * as d3 from "d3";

  const formatTime = d3.utcFormat("%Y-%m-%d");

  let tooltipWidth = 10;
  let tooltipHeight = 10;
  let tooltipElement;

  onMount(() => {
    tooltipWidth = tooltipElement.offsetWidth;
    tooltipHeight = tooltipElement.offsetHeight;
  });

  const tooltipAction = node => {
    tooltipElement = node;
  };
</script>

{#if chart_type === "scatter"}
  <div
    class="tooltip-scatter"
    use:tooltipAction
    style="left: {hoveredData.recorded_mouse_position.x - tooltipWidth / 2}px; 
           top: {hoveredData.recorded_mouse_position.y -
      tooltipHeight -
      15}px;"
  >
    <p class="tootltip-title">{hoveredData.title}</p>
    <p class="tootltip-stats">
      {hoveredData.activeMetric}: {hoveredData.count} | {formatTime(
        hoveredData.date
      )}
    </p>
    <p></p>
    <iframe
      src={"//player.bilibili.com/player.html?aid=" + hoveredData.aid}
      width="200"
      height="150"
      title=""
      border="0"
      frameborder="no"
    ></iframe>
  </div>
{:else if chart_type === "area"}
  <div
    class="tooltip-area"
    use:tooltipAction
    style="left: {hoveredData.recorded_mouse_position.x - tooltipWidth / 2}px; 
           top: {hoveredData.recorded_mouse_position.y -
      tooltipHeight -
      15}px;"
  >
    <p>{formatTime(hoveredData.date)}</p>
    <p>{hoveredData.title}</p>
  </div>
{:else if chart_type === "heatmap"}
  <div
    class="tooltip-heatmap"
    use:tooltipAction
    style="left: {hoveredData.recorded_mouse_position.x - tooltipWidth / 2}px; 
           top: {hoveredData.recorded_mouse_position.y -
      tooltipHeight -
      15}px;"
  >
    Tag {hoveredData.variable}: {hoveredData.value}
  </div>
{:else if chart_type === "radar"}
  <div
    class="tooltip-heatmap"
    use:tooltipAction
    style="left: {hoveredData.recorded_mouse_position.x - tooltipWidth / 2}px; 
           top: {hoveredData.recorded_mouse_position.y -
      tooltipHeight -
      15}px;"
  >
    HELP!
  </div>
{/if}

<style>
  .tooltip-scatter {
    position: absolute;
    background-color: rgba(255, 255, 255, 0.9);
    color: black;
    visibility: visible;
    font: 1em sans-serif;
    font-family: "Nunito", sans-serif;
    width: 20%;
    padding: 1em;
    border: 1px solid #ccc; /* Light grey border */
    border-radius: 5px;
  }

  .tooltip-scatter iframe {
    width: 100%;
    height: auto;
  }

  .tooltip-area {
    position: absolute;
    background-color: rgba(255, 255, 255, 0.9);
    color: black;
    visibility: visible;
    font: 1em sans-serif;
    font-family: "Nunito", sans-serif;
    width: 10%;
    padding: 1em;
    border: 1px solid #ccc; /* Light grey border */
    border-radius: 5px;
  }

  .tooltip-heatmap {
    font: 15px sans-serif;
    font-family: "Nunito", sans-serif;
    visibility: visible;
    background-color: ghostwhite;
    border-radius: 5px;
    width: 100px;
    color: black;
    position: absolute;
    padding: 10px;
  }
</style>

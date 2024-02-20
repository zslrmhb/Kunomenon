<script>
  export let hoveredData;
  export let chart_type;
</script>

{#if chart_type === "scatter"}
  <div
    class="tooltip-scatter"
    style="left: {hoveredData.recorded_mouse_position
      .x}px; bottom: {hoveredData.recorded_mouse_position.y}px"
  >
    <iframe
      src={"//player.bilibili.com/player.html?aid=" + hoveredData.aid}
      width="200"
      height="150"
      title=""
      border="0"
      frameborder="no"
    ></iframe>
    <p>Title: {hoveredData.title}</p>
    <p>Date: {hoveredData.date}</p>
    <p>Count: {hoveredData.count}</p>
  </div>
{:else if chart_type === "area"}
  <div
    class="tooltip-area"
    style="left: {hoveredData.recorded_mouse_position.x}px; top: {hoveredData
      .recorded_mouse_position.y}px"
  >
    {hoveredData.title}!
  </div>
{:else if chart_type === "heatmap"}
  <div
    class="tooltip-heatmap"
    style="left: {hoveredData.recorded_mouse_position.x +
      40}px; top: {hoveredData.recorded_mouse_position.y + 40}px"
  >
    Tag {hoveredData.variable}: {hoveredData.value}
  </div>
{/if}

<style>
  .tooltip-scatter,
  .tooltip-area {
    font: 15px sans-serif;
    font-family: "Nunito", sans-serif;
    visibility: visible;
    position: absolute;
    width: 100px;
    color: black;
    /* background-color: #cce5df; */
    padding: 10px;
    /* border: 1px solid black; */
    border-radius: 5px;
    /* pointer-events: none; */
    opacity: 1;
    /* transition: opacity 0.3s; */
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

<script>
  export let hoveredData;
  export let chart_type;
  // export let y;
</script>

{#if chart_type === "scatter"}
  <div
    class="tooltip"
    style="left: {hoveredData.recorded_mouse_position.x}px; top: {hoveredData
      .recorded_mouse_position.y}px"
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
    <!-- <iframe
        src={link}
        width="250"
        height="200"
        title=""
        border="0"
        scrolling="no"
        frameborder="no"
        allowfullscreen="false"
        loading="lazy"
      ></iframe> -->
  </div>
{/if}

<!-- {#if chart_type === "area"}
  <div
    class="tooltip"
    style="left: {recorded_mouse_position.x}px; top: {recorded_mouse_position.y}px"
  >
    {tooltip_content}
   </div> -->
<!-- {/if} -->

<style>
  .tooltip {
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
  /* .tooltip.svelte-fade {
    opacity: 1;
  } */
</style>

<script>
  export let tooltip_content = "";
  export let recorded_mouse_position;
  export let chart_type;
  export let link;
  // export let y;
</script>

{#if chart_type === "scatter"}
  <div
    class="tooltip"
    style="left: {recorded_mouse_position.x}px; top: {recorded_mouse_position.y}px"
  >
    {tooltip_content}
    {#if link}
      <iframe
        src={link}
        width="250"
        height="200"
        title=""
        border="0"
        scrolling="no"
        frameborder="no"
        allowfullscreen="false"
        loading="lazy"
      ></iframe>
    {/if}
  </div>
{/if}

<!-- 
<div
  class="tooltip"
  style="left: {recorded_mouse_position.x -
    50}px; top: {recorded_mouse_position.y - 100}px"
> -->
<!-- {tooltip_content}
</div> -->

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
    opacity: 0.8;
    /* transition: opacity 0.3s; */
  }
  /* .tooltip.svelte-fade {
    opacity: 1;
  } */
</style>

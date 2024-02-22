<script>
  import Scatter from "./Scatter.svelte";
  import Heatmap from "./Heatmap.svelte";
  import Radar from "./Radar.svelte";
  import Tooltip from "./Tooltip.svelte";
  export let dimensions, radarDim;
  export let videoInfo, tagCount, typeCount;

  let activeMetric = "playCount";
  $: curDataset = videoInfo.map(d => ({
    date: d.pubdate,
    count: +d[activeMetric],
    arcurl: d.arcurl,
    aid: d.aid,
    title: d.title,
  }));

  // Tooltip

  let showTooltip = false;
  let recorded_mouse_position = { x: 0, y: 0 };

  function handleMouseOver(event) {
    showTooltip = true;
    recorded_mouse_position = { x: event.pageX, y: event.pageY };
  }

  function handleMouseOut() {
    showTooltip = false;
  }
</script>

<!-- svelte-ignore a11y-mouse-events-have-key-events -->
<!-- svelte-ignore a11y-mouse-events-have-key-events -->
<div class="dataset-controls">
  {#each ["Play", "Like", "Favorite", "Review", "Duration", "Danmaku", "Tag", "Type"] as metric}
    <button
      on:click={() => (activeMetric = metric.toLowerCase() + "Count")}
      class:active={activeMetric === metric.toLowerCase() + "Count"}
    >
      {metric}
    </button>
  {/each}
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <span
    class="help-icon"
    on:mouseover={event => handleMouseOver(event)}
    on:mouseout={handleMouseOut}>?</span
  >

  {#if showTooltip}
    <div class="tooltip">
      <p>Danmaku: bullet curtain style comment that flow in a video</p>
      <p>Tag: Top 5 trending tags associated with the videos</p>
      <p>Type: Top 5 genres</p>
    </div>
  {/if}
</div>
{#if activeMetric === "tagCount"}
  <Heatmap {dimensions} {tagCount} />
{:else if activeMetric === "typeCount"}
  <Radar {radarDim} {typeCount} />
{:else}
  <Scatter {dimensions} {curDataset} {activeMetric} />
{/if}

<style>
  .dataset-controls {
    /* display: flex; */
    text-align: center;
    padding: 2em;
    margin-top: -0.9em;
    /* vertical-align: left; */
    /* display:flex; */
  }

  .dataset-controls button {
    /* display: inline-block; */
    margin: 0.5em;
    padding: 0.5em;
    border: none;
    background-color: white;
    cursor: pointer;
  }

  .dataset-controls button.active {
    font-weight: bold;
    color: #4574cc;
    /* border-color: #4574cc; */
  }

  .help-icon {
    display: inline-block;
    margin-left: 1em;
    width: 20px;
    height: 20px;
    line-height: 20px;
    text-align: center;
    background-color: #4574cc;
    color: white;
    border-radius: 50%;
    cursor: pointer;
  }

  .tooltip {
    position: absoulte;
    background-color: rgba(255, 255, 255, 0.9);
    color: black;
    visibility: visible;
    font: 1em sans-serif;
    font-family: "Nunito", sans-serif;
    width: 20%;
    padding: 1em;
    border: 1px solid #ccc; /* Light grey border */
    border-radius: 5px;
    text-align: left;
  }
  /* .tooltip {
    position: absolute;
    left: 50%; /* Adjust as needed */
  /* transform: translateX(-50%); Center the tooltip */
  /* bottom: 100%; */
  /* margin-bottom: 10px; Distance from icon */
  /* background-color: black; */
  /* color: white; */
  /* padding: 0.5em; */
  /* border-radius: 4px; */
  /* white-space: nowrap; */
  /* } */
</style>

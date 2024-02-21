<script>
  import Scatter from "./Scatter.svelte";
  import Heatmap from "./Heatmap.svelte";
  import Radar from "./Radar.svelte";
  export let dimensions;
  export let videoInfo, tagCount, typeCount;

  let activeMetric = "playCount";

  $: curDataset = videoInfo.map(d => ({
    date: d.pubdate,
    count: +d[activeMetric],
    arcurl: d.arcurl,
    aid: d.aid,
    title: d.title,
  }));
</script>

<div class="dataset-controls">
  {#each ["Play", "Like", "Favorite", "Review", "Duration", "Danmaku", "Tag", "Type"] as metric}
    <button
      on:click={() => (activeMetric = metric.toLowerCase() + "Count")}
      class:active={activeMetric === metric.toLowerCase() + "Count"}
    >
      {metric}
    </button>
  {/each}
</div>
{#if activeMetric === "tagCount"}
  <Heatmap {dimensions} {tagCount} />
{:else if activeMetric === "typeCount"}
  <Radar {typeCount} />
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
</style>

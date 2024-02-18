<script>
  import Scatter from "./Scatter.svelte";
  import Heatmap from "./Heatmap.svelte";

  export let play_count,
    like_count,
    review_count,
    danmaku_count,
    duration_count,
    tag_count;
  export let dimensions;

  let activeDataset = "play_count";

  $: cur_dataset =
    activeDataset === "play_count"
      ? play_count
      : activeDataset === "like_count"
        ? like_count
        : activeDataset === "review_count"
          ? review_count
          : activeDataset === "danmaku_count"
            ? danmaku_count
            : activeDataset === "duration_count"
              ? duration_count
              : tag_count;
</script>

<div class="dataset-controls">
  {#each ["Play", "Like", "Review", "Duration", "Danmaku", "Tag"] as dataset}
    <button
      on:click={() => (activeDataset = dataset.toLowerCase() + "_count")}
      class:active={activeDataset === dataset.toLowerCase() + "_count"}
    >
      {dataset}
    </button>
  {/each}
</div>

{#if activeDataset !== "tag_count"}
  <Scatter {dimensions} {cur_dataset} {activeDataset}/>
{:else}
  <Heatmap {dimensions} {tag_count} />
{/if}

<style>
  .dataset-controls {
    text-align: left;
  }

  .dataset-controls button {
    display: inline-block;
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

  .dataset-controls {
    display: inline;
    vertical-align: bottom;
  }
</style>

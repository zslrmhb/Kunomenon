<script>
  import * as d3 from "d3";
  export let dimensions;
  export let play_count,
    like_count,
    review_count,
    danmaku_count,
    duration_count;

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
            : duration_count;

  // Sort and get top N poionts
  const N = 10;

  $: topNData = [...cur_dataset]
    .sort((a, b) => d3.descending(a.count, b.count))
    .slice(0, N);

  function isInTopN(dataPoint) {
    return topNData.includes(dataPoint);
  }

  // Configures scales
  $: x = d3
    .scaleTime()
    .domain(d3.extent(cur_dataset, d => d.date))
    .range([0, dimensions.boundedWidth]);

  $: y = d3
    .scaleLinear()
    .domain(d3.extent(cur_dataset, d => +d.count))
    .range([dimensions.boundedHeight, 0]);

  // Configure axes
  let gx, gy;
  $: d3.select(gx).call(d3.axisBottom(x));
  $: d3.select(gy).call(d3.axisLeft(y));
</script>

<div class="scatter-plot">
  <svg
    width={dimensions.width}
    height={dimensions.height}
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
    style="max-width: 100%; height: auto;"
  >
    <g
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      {#each cur_dataset as data, i}
        <circle
          id={i}
          cx={x(data.date)}
          cy={y(data.count)}
          r="3"
          fill={isInTopN(data) ? "#4574cc" : "gray"}
        ></circle>{/each}
    </g>
    <g
      bind:this={gx}
      transform={`translate(${dimensions.margin.left}, ${
        dimensions.boundedHeight + dimensions.margin.top
      })`}
    />
    <g
      bind:this={gy}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    />
  </svg>
</div>
<div class="dataset-controls">
  {#each ["Play", "Like", "Review", "Duration", "Danmaku"] as dataset}
    <button
      on:click={() => (activeDataset = dataset.toLowerCase() + "_count")}
      class:active={activeDataset === dataset.toLowerCase() + "_count"}
    >
      {dataset}
    </button>
  {/each}
</div>

<style>
  .dataset-controls {
    text-align: left;
    padding-right: 4px;
    
  }

  .dataset-controls button {
    /* padding: 5px 10px; */
    display: block;
    margin: 5px;
    padding: 4px;
    border: none;
    background-color: white;
    cursor: pointer;
  }

  .dataset-controls button.active {
    font-weight: bold;
    color: #4574cc;
    /* border-color: #4574cc; */
  }

  .scatter-plot,
  .dataset-controls {
    display: inline-block;
    vertical-align: bottom;
  }
</style>

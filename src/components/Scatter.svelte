<script>
  import * as d3 from "d3";
  export let dimensions;
  export let rank_score_count,
    play_count,
    like_count,
    review_count,
    danmaku_count,
    duration_count;

  let cur_dataset = review_count;

  // Sort and get top N poionts
  const N = 10;
  let topNData = [...cur_dataset]
    .sort((a, b) => d3.descending(a.count, b.count))
    .slice(0, N);

  function isInTopN(dataPoint) {
    return topNData.includes(dataPoint);
  }

  console.log(topNData);
  // Configures scales
  $: x = d3
    .scaleTime()
    .domain(d3.extent(cur_dataset, d => d.date))
    .range([0, dimensions.boundedWidth]);

  $: y = d3
    .scaleLinear()
    .domain(d3.extent(cur_dataset, d => d.count))
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
    {#each cur_dataset as data, i}
      <circle
        id={i}
        cx={x(data.date)}
        cy={y(data.count)}
        r="3"
        fill={isInTopN(data) ? "#4574cc" : "lightgrey"}
      ></circle>{/each}
    <g
      bind:this={gx}
      transform={`translate(${dimensions.margin.left}, ${dimensions.boundedHeight + dimensions.margin.top})`}
    />
    <g
      bind:this={gy}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    />
  </svg>
</div>

<style>
  .scatter-plot {
      padding-top: 10px;
  }
</style>

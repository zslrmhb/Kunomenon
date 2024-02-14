<script>
    import * as d3 from "d3";
    import { sharedXDomain } from "./store.js";
    export let tag_count;
    export let dimensions;

    let data = [
      [10, 20, 30],
      [40, 50, 60],
      [70, 80, 90]
    ];
  
    const margin = { top: 20, right: 30, bottom: 30, left: 40 };
    const width = 600 - margin.left - margin.right;
    const height = 400 - margin.top - margin.bottom;
  
    const colorScale = d3.scaleSequential(d3.interpolateViridis)
      .domain([0, d3.max(data, d => d3.max(d))]);
  
    let heatmapData = [];
  </script>
  
  <svg width={width + margin.left + margin.right} height={height + margin.top + margin.bottom}>
    <g transform={`translate(${margin.left}, ${margin.top})`}>
      {#each heatmapData as { row, column, value }}
        <rect
          x={(column * width) / data[0].length}
          y={(row * height) / data.length}
          width={width / data[0].length}
          height={height / data.length}
          fill={colorScale(value)}
          stroke="white"
        />
      {/each}
    </g>
  </svg>
  
  <style>
    rect {
      stroke: #fff;
    }
  </style>
  
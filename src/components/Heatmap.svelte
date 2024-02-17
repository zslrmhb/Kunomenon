<script>
  import * as d3 from "d3";
  // import { sharedXDomain } from "./store.js";
  export let dimensions;
  export let tag_count;

  // $: console.log(tag_count);
  // let svg = d3
  //   .select("heatmap")
  //   .append("svg")
  //   .attr("width", dimensions.width)
  //   .attr("height", dimensions.height);

  // svg
  //   .append("g")
  //   .attr(
  //     "transform",
  //     "translate(" + dimensions.margin.left + "," + dimensions.margin.top + ")"
  //   );

  // Keep constant scales
  // $: x = d3
  // .scaleTime()
  // .domain(d3.extent(tag_count, d => d.date))
  // .range([0, dimensions.boundedWidth]);

  let myGroup = tag_count.map(function(d) {
      return d.group;
    });
  $: console.log(myGroup)

  const uniqueGroup = new Set(myGroup)
  const uniqueGroupArray = [...uniqueGroup]
  $: console.log(uniqueGroupArray)

  let myVars = tag_count
    .map(function (d) {
      return d.variable;
    });
  const uniqueVar = new Set(myVars)
  const uniqueVarArray = [...uniqueVar]
  $: console.log(uniqueVarArray)

  $: x = d3
    .scaleBand()
    .range([0, dimensions.width])
    .domain(uniqueGroupArray)
    .padding(0.05);
  $: y = d3
    .scaleBand()
    .range([dimensions.height, 0])
    .domain(uniqueVarArray)
    .padding(0.05);

  // $: console.log("x: " + x.bandwidth());
  // $: console.log("y " + y.bandwidth());
  $: myColor = d3
    .scaleSequential()
    .interpolator(d3.interpolateInferno)
    .domain([0, 200]);

  // Configure axes
  // let gx, gy;
  // $: d3.select(gx).call(d3.axisBottom(x));
  // $: d3.select(gy).call(d3.axisLeft(y));

  // $: {svg.selectAll()
  //   .data(tag_count, function(d){return d.group+":"+d.variable;})
  //   .enter()
  //   .append('rect')
  //     .attr("x", function(d){return x(d.group)})
  //     .attr("y", function(d){return y(d.variable)})
  //     .attr("rx", 10)
  //     .attr("ry", 10)
  //     .attr("width", x.bandwidth() )
  //     .attr("height", y.bandwidth() )
  //     .style("fill", function(d) { return myColor(d.value)} )
  //     .style("stroke-width", 4)
  //     .style("stroke", "none")
  //     .style("opacity", 0.8)
  // }
</script>

<div class="heatmap-wrapper">
  <svg
    id="heatmap-plot"
    width={dimensions.width}
    height={dimensions.height}
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}
    style="max-width: 100%; height: auto;"
  >
    <g
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}
    >
      {#each tag_count as d, i}
        <rect
          id={i}
          x={x(d.group)}
          y={y(d.variable)}
          width={x.bandwidth()}
          height={y.bandwidth()}
          fill={myColor(d.value)}
          rx={10}
          ry={10}
        />
      {/each}

      <!-- <rect
        x={function (tag_count) {
          return x(tag_count.group);
        }}
        y={function (tag_count) {
          return y(tag_count.variable);
        }}
      /> -->
    </g>

    <!-- Axes -->
    <!-- <g
      bind:this={gx}
      transform={`translate(${dimensions.margin.left}, ${
        dimensions.boundedHeight + dimensions.margin.top
      })`}
    /> -->
    <!-- <g
      bind:this={gy}
      transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`} -->
    <!-- /> -->
  </svg>
</div>

<!-- <div class="heatmap"></div> -->

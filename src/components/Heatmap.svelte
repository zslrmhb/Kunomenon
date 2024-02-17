<script>
  import * as d3 from "d3";
  // import { sharedXDomain } from "./store.js";
  export let dimensions;
  export let tag_count;

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

  const xBan = 7.042612419700214; 

  $: x = d3
    .scaleBand()
    .range([0, dimensions.width])
    .domain(myGroup)
    .padding(0.05);

  $: console.log(x.bandwidth())

  $: y = d3
    .scaleBand()
    .range([dimensions.height, 0])
    .domain(uniqueVarArray)
    .padding(0.05);

  // $: console.log("x: " + x.bandwidth());
  // $: console.log("y " + y.bandwidth());
  $: myColor = d3
    .scaleSequential()
    .interpolator(d3.interpolatePuBu)
    .domain([0, 200]);

  // Configure axes
  $: xAxis = d3
    .scaleTime()
    .domain(d3.extent(tag_count, d => d.group))
    .range([0, dimensions.width]);

  let gx, gy;
  $: d3.select(gx).call(d3.axisBottom(xAxis));
  $: d3.select(gy).call(d3.axisLeft(y));
  // interaction
  // function handleFocus(event, d){
  //   handleMouseOver(event, d);
  // }

  // function handleBlur(){
  //   handleMouseOut();
  // }

  const mouseover = function(event,d) {
    tooltip
      .style("opacity", 1)
    d3.select(this)
      .style("stroke", "black")
      .style("opacity", 1)
  }
  const mousemove = function(event,d) {
    tooltip
      .html("The exact value of<br>this cell is: " + d.value)
      .style("left", (event.x)/2 + "px")
      .style("top", (event.y)/2 + "px")
  }
  const mouseleave = function(event,d) {
    tooltip
      .style("opacity", 0)
    d3.select(this)
      .style("stroke", "none")
      .style("opacity", 0.8)
  }
  // let svg = d3.select('#heatmap-plot');
  // svg.selectAll()
  //   .on('mouseover', mouseover)
  //   .on('mousemove', mousemove)
  //   .on('mouseleave', mouseleave)
  let svg; 
  let tooltip = null;
  function onPointerMove(event){
    const x0 = x.invert(d3.pointer(event)[0]);
    tooltip = tag_count[x0];
  }

  function onPointerLeave(event){
    tooltip = null;
  }

  $: d3.select(svg)
    .on("pointerenter pointermove", onPointerMove)
    .on("pointerleave", onPointerLeave)

  $: console.log(tooltip)
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
          width={xBan+10}
          height={y.bandwidth()}
          fill={myColor(d.value)}
          rx={0}
          ry={0}
          />
      {/each}
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

  {#if tooltip}
  <g transform="translate({x(tooltip.date)},{y(tooltip.value)})">
    <text font-weight="bold">{tooltip.value}</text>
  </g>
  {/if}
  </svg>
</div>

<!-- <div class="heatmap"></div> -->

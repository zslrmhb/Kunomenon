<script>
  import * as d3 from "d3";
  // import { sharedXDomain } from "./store.js";
  export let dimensions;
  export let tag_count;
<<<<<<< HEAD
  $: console.log(tag_count)
  let myGroup, uniqueGroup, uniqueGroupArray; 
  $: myGroup = tag_count.map(function(d) {
      return d.group;
    });
=======
  // $: console.log("Tag Count:" + tag_count);
>>>>>>> b2bdd4970d455ea774fc5cf98900e278ff9b34e1

  let myGroup;
  $: myGroup = tag_count.map(function (d) {
    return d.group;
  });

<<<<<<< HEAD
  $: uniqueGroup = new Set(myGroup)
  $: uniqueGroupArray = [...uniqueGroup]
  $: console.log(uniqueGroupArray)

  $: myVars = tag_count
    .map(function (d) {
      return d.variable;
    });
  
  $: uniqueVar = new Set(myVars)
  $: uniqueVarArray = [...uniqueVar]
  $: console.log(uniqueVarArray)
=======
  // $: console.log("My Group: " + myGroup);

  let uniqueGroup;
  $: uniqueGroup = new Set(myGroup);

  let uniqueGroupArray;
  $: uniqueGroupArray = [...uniqueGroup];
  $: console.log("Unique Group Array: " + uniqueGroupArray);

  let myVars;
  $: myVars = tag_count.map(function (d) {
    return d.variable;
  });

  let uniqueVar;
  $: uniqueVar = new Set(myVars);
  let uniqueVarArray;
  $: uniqueVarArray = [...uniqueVar];
  // $: console.log(uniqueVarArray);
>>>>>>> b2bdd4970d455ea774fc5cf98900e278ff9b34e1

  $: x = d3
    .scaleBand()
    .range([0, dimensions.width])
    .domain(myGroup)
    .padding(0.05);

  // $: console.log(x.bandwidth());

  $: y = d3
    .scaleBand()
    .range([dimensions.boundedHeight, 0])
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

  // Interactivity
  let hovered = -1;

  let recorded_mouse_poition = {
    x: 0,
    y: 0,
  };
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
          rx={0}
          ry={0}
          on:mouseover={event => {
            hovered = i;
            recorded_mouse_poition = {
              x: event.pageX,
              y: event.pageY,
            };
          }}
          on:mouseout={event => (hovered = -1)}
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

    <!-- {#if tooltip}
  <g transform="translate({x(tooltip.date)},{y(tooltip.value)})">
    <text font-weight="bold">{tooltip.value}</text>
  </g>
  {/if} -->
  </svg>
  <div
    class={hovered === -1 ? "tooltip-hidden" : "tooltip-visible"}
    style="left: {recorded_mouse_poition.x +
      40}px; top: {recorded_mouse_poition.y + 40}px"
  >
    {#if hovered !== -1}
      Tag {tag_count[hovered].variable}: {tag_count[hovered].value}
    {/if}
  </div>
</div>

<style>
  /* dynamic classes for the tooltip */
  .tooltip-hidden {
    visibility: hidden;
    font-family: "Nunito", sans-serif;
    width: 200px;
    position: absolute;
  }

  .tooltip-visible {
    font: 15px sans-serif;
    font-family: "Nunito", sans-serif;
    visibility: visible;
    background-color: ghostwhite;
    border-radius: 5px;
    width: 100px;
    color: black;
    position: absolute;
    padding: 10px;
  }
</style>

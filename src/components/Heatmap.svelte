<script>
  import * as d3 from "d3";
  import { sharedXDomain } from "./store.js";
  export let dimensions;
  export let tag_count;

  let svg = d3.select("heatmap")
    .append("svg")
    .attr("width",dimensions.width)
    .attr('height',dimensions.height)
  svg.append('g')
    .attr('transform', "translate("+dimensions.margin.left+',' + dimensions.margin.top+")");

  // Keep constant scales
  // $: x = d3
  // .scaleTime()
  // .domain(d3.extent(tag_count, d => d.date))
  // .range([0, dimensions.boundedWidth]);

  $: myGroup = d3.map(tag_count, function(d){return d.group;}).keys()
  $: myVars = d3.map(tag_count, function(d){return d.variable}).keys()

  $: x = d3.scaleBand()
    .range([0, dimensions.width])
    .domain(myGroup)
    .padding(0.05)

  $: y = d3.scaleBand()
    .range([height, 0])
    .domain(myVars)
  
  $: myColor = d3.scaleSequential()
    .interpolator(d3.interpolateInferno)
    .domain([0,200])
  
  $: {svg.selectAll()
    .data(tag_count, function(d){return d.group+":"+d.variable;})
    .enter()
    .append('rect')
      .attr("x", function(d){return x(d.group)})
      .attr("y", function(d){return y(d.variable)})
      .attr("rx", 10)
      .attr("ry", 10)
      .attr("width", x.bandwidth() )
      .attr("height", y.bandwidth() )
      .style("fill", function(d) { return myColor(d.value)} )
      .style("stroke-width", 4)
      .style("stroke", "none")
      .style("opacity", 0.8)
  }
</script>

<!-- <div class='heatmap'>
  <svg 
    id = "heatmap-plot",
    width = {dimensions.width},
    height = {dimensions.height}, 
    viewBox={`0 0 ${dimensions.width} ${dimensions.height}`}, 
    style = "max-width: 100%; height: auto;"
  >
    <g transform={`translate(${dimensions.margin.left}, ${dimensions.margin.top})`}>
      <rect
        x={function(tag_count){return x(tag_count.group)}}
        y={function(tag_count){return y(tag_count.variable)}}
        />
    </g>
  </svg>
</div> -->
<div class='heatmap'></div>
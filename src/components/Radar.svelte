<script>
  import * as d3 from "d3";
  export let typeCount;
  export let radarDim;

  //set dimensions
  let yearVal;
  let uniqueYear;
  let filterYear;
  let uniqueYearArray;

  $: yearVal = 2017;
  $: filterYear = typeCount.filter(d => {
    return d.year === yearVal || d.year === yearVal + 1;
  });

  // get the data
  let data = [];
  let yearList = [];

  let dictionary = {};
  let dictionary2 = {};

  $: filterYear.forEach(item => {
    if (item.year === yearVal) {
      dictionary[item.type] = item.count;
    } else {
      dictionary2[item.type] = item.count;
    }
  });

  $: yearList = filterYear.map(d => String(d.year));
  $: uniqueYear = new Set(yearList);
  $: uniqueYearArray = [...uniqueYear];

  data[0] = dictionary;
  data[1] = dictionary2;

  //Scale the figure
  $: radialScale = d3.scaleLinear().domain([0, 75]).range([0, 200]);

  let ticks = [];
  $: ticks = [15, 30, 45, 60, 75];

  function angleToCoordinate(angle, value) {
    let x = Math.cos(angle) * radialScale(value);
    let y = Math.sin(angle) * radialScale(value);
    return { x: radarDim.width / 2 + x, y: radarDim.height / 2 + y };
  }

  //get features
  $: features = ["Autotune", "Comedy", "Daily", "Entertainment", "Theater"];

  $: featureData = features.map((f, i) => {
    let angle = Math.PI / 2 + (2 * Math.PI * i) / features.length;
    return {
      name: f,
      angle: angle,
      line_coord: angleToCoordinate(angle, 75),
      label_coord: angleToCoordinate(angle, 75 + 5),
      text_angle: ((angle * 180) / Math.PI + 90) % 360,
    };
  });

  $: line = d3
    .line()
    .x(d => d.x)
    .y(d => d.y)
    .curve(d3.curveLinearClosed);

  //plot the data
  $: color = ["#d86e41", "#69b3a2"];

  function getPathCoordinate(data_point) {
    let coordinates = [];
    if (data_point) {
      for (var i = 0; i < features.length; i++) {
        let ft_name = features[i];
        let angle = Math.PI / 2 + (2 * Math.PI * i) / features.length;
        coordinates.push(angleToCoordinate(angle, data_point[ft_name]));
      }
    }
    return coordinates;
  }

  // Silder
  function handleSliderChange(event) {
    yearVal = +event.target.value;
    filterYear = [...typeCount].filter(d => {
      // const year = +d.year;
      // console.log('year', year);
      return d.year === yearVal || d.year === yearVal + 1;
    });
    let data = [];
    let yearList;
    let uniqueYear;
    let uniqueYearArray;

    let dictionary = {};
    let dictionary2 = {};

    filterYear.forEach(item => {
      if (item.year === yearVal) {
        dictionary[item.type] = item.count;
      } else {
        dictionary2[item.type] = item.count;
      }
    });
    data.unshift(dictionary2);
    data.unshift(dictionary);

    yearList = filterYear.map(d => String(d.year));
    uniqueYear = new Set(yearList);
    uniqueYearArray = [...uniqueYear];

    d3.select("#radar-plot")
      .selectAll("path")
      .data(data)
      .attr("d", d => line(getPathCoordinate(d)));
  }
</script>

<div class="slider">
  <label for="year-slider">Year {yearVal} and {yearVal + 1}</label>
  <input
    type="range"
    id="year-sliderr"
    min="2017"
    max="2023"
    value={yearVal}
    on:input={handleSliderChange}
    orient="horizontal"
  />
</div>

<div class="radar-plot-wrapper">
  <svg
    id="radar-plot"
    width={radarDim.width}
    height={radarDim.height}
    viewBox={`0 0 ${radarDim.width} ${radarDim.height}`}
    style="max-width: 100%; height: auto;"
  >
    {#each data as d, i}
      <path
        d={line(getPathCoordinate(d))}
        stroke-width={3}
        stroke={color[i]}
        fill={color[i]}
        stroke-opacity={1}
        opacity={0.5}
      >
      </path>
    {/each}
    {#each ticks as data, i}
      <circle
        id={i}
        cx={radarDim.boundedWidth / 2}
        cy={radarDim.boundedHeight / 2}
        r={radialScale(data)}
        fill="none"
        stroke="black"
      >
      </circle>
      <text
        x={radarDim.width / 2 - 10}
        y={radarDim.height / 2 - radialScale(data) - 9}
        dy="0.35em"
      >
        {data}{"%"}
      </text>
    {/each}
    {#each featureData as data, i}
      <line
        id={i}
        x1={radarDim.boundedWidth / 2 }
        y1={radarDim.boundedHeight / 2}
        x2={data.line_coord.x}
        y2={data.line_coord.y}
        stroke="black"
      >
      </line>
      <text
        x={data.label_coord.x + 10}
        y={data.label_coord.y + 5}
        text-anchor="middle"
        transform={`rotate(${
          data.text_angle > 90 && data.text_angle < 270
            ? data.text_angle + 180
            : data.text_angle
        }, ${data.label_coord.x}, ${data.label_coord.y})`}
      >
        {data.name}
      </text>
    {/each}

    Legend
    <g class="legend" transform="translate(20, 20)">
      {#each uniqueYearArray as d, i}
        <text id={i} x={0} y={i * 30} dt={"0.35em"}>{d}</text>
        <rect x="50" y={i * 30 - 13} width="16" height="16" fill={color[i]}>
        </rect>
      {/each}
    </g>
  </svg>
</div>

<style>
  .radar-plot-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
  }

  .slider {
    text-align: center;
  }
</style>

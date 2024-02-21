<script>
    import * as d3 from "d3";
    export let typeCount;

    //$: console.log('typeCount', typeCount);

    //set dimensions
    let width = 600;
    let height = 600;
    let yearVal;
    let uniqueYear; 
    let filterYear;
    let uniqueYearArray; 

    $: yearVal = 2017;
    $: filterYear = typeCount.filter(d => {
        return d.year === yearVal || d.year === yearVal+1;
    });


    //console.log('filterYear', filterYear);

    // get the data
    let data = [];
    let yearList = []; 

    let dictionary = {};
    let dictionary2 = {};  

    $: filterYear.forEach(item => {
        if(item.year===yearVal){
            dictionary[item.type] = item.count
        }else{
            dictionary2[item.type] = item.count
        }
    });

    $: yearList = filterYear.map(d => String(d.year));
    $: uniqueYear = new Set(yearList);
    $: uniqueYearArray = [...uniqueYear];

    data[0] = dictionary;
    data[1] = dictionary2;


    console.log('data', data);

    //$: console.log('yearList', uniqueYearArray);

    //Scale the figure
    $: radialScale = d3.scaleLinear()
    .domain([0, 75])
    .range([0, 200]);

    let ticks = [];
    $: ticks = [15, 30, 45, 60, 75];

    function angleToCoordinate(angle, value){
        let x = Math.cos(angle) * radialScale(value);
        let y = Math.sin(angle) * radialScale(value);
        return {'x': width/2+x, 'y': height/2+y};
    }

    //get features 
    $: features = ['Autotune', 'Comedy', 'Daily', 'Entertainment', 'Theater']

    $: featureData = features.map((f, i) => {
        let angle = (Math.PI/2)+(2*Math.PI*i/features.length);
        return{
            'name': f,
            'angle': angle,
            'line_coord': angleToCoordinate(angle, 75), 
            'label_coord': angleToCoordinate(angle, 75+5),
            'text_angle':(angle*180/Math.PI+90)%360
        }
    })

    $: line = d3.line()
        .x(d => d.x)
        .y(d => d.y)
        .curve(d3.curveLinearClosed);

    //plot the data
    $: color = ['darkorange', 'green'];
    
    function getPathCoordinate(data_point){
        console.log('data_point', data_point);
        console.log('type: data_point', typeof(data_point));

        for (let key in data_point){
            console.log('key: data_point', key);
        }

        let coordinates = [];
        if (data_point){
            for (var i=0; i<features.length; i++){
                let ft_name = features[i];
                console.log('ft_name', ft_name);
                let angle = (Math.PI/2) + (2*Math.PI*i/features.length);
                //console.log('angle', angle);
                console.log('data_point[ft_name]', data_point["Autotune"]);
                coordinates.push(angleToCoordinate(angle, data_point[ft_name]));
            }
        }
        console.log('coordinates', coordinates);
        return coordinates;
    }

    // Silder
    function handleSliderChange(event) {
        yearVal = +event.target.value;
        filterYear = [...typeCount].filter(d => {
            // const year = +d.year;
            // console.log('year', year);
            return d.year === yearVal || d.year === yearVal+1;
        });
        let data = [];
        let yearList;
        let uniqueYear;
        let uniqueYearArray;

        let dictionary = {};
        let dictionary2 = {};  

        filterYear.forEach(item => {
            if(item.year===yearVal){
                dictionary[item.type] = item.count
            }else{
                dictionary2[item.type] = item.count
            }
        });
        data.unshift(dictionary2);
        data.unshift(dictionary);
        //console.log('filterYear', filterYear);
        //console.log('dataChange', data);

        yearList = filterYear.map(d => String(d.year));
        uniqueYear = new Set(yearList);
        uniqueYearArray = [...uniqueYear];

        d3.select("#radar-plot")
            .selectAll("path")
            .data(data)
            .attr('d', d => line(getPathCoordinate(d)));
    }



</script>

<div class = "radar-wrapper">
    <svg 
        id="radar-plot"
        width={width}
        height={height}>
        {#each ticks as data, i}
            <circle 
                id = {i}
                cx = {width / 2}
                cy = {height / 2}
                r = {radialScale(data)}
                fill = "none"
                stroke = "black">
            </circle>
            <text 
                x = {width / 2}
                y = {height/2 -radialScale(data)}
                dy='0.35em'>
                {data}{'%'}
            </text>
        {/each}
        {#each featureData as data, i}
        <line 
            id={i}
            x1 = {width / 2}
            y1 = {height / 2}
            x2 = {data.line_coord.x}
            y2 = {data.line_coord.y}
            stroke = "black">
        </line>
        <text
            x = {data.label_coord.x+10}
            y = {data.label_coord.y+5}
            text-anchor = "middle"
            transform = {`rotate(${data.text_angle > 90 && data.text_angle < 270 ? data.text_angle + 180 : data.text_angle}, ${data.label_coord.x}, ${data.label_coord.y})`}
            > 
            {data.name}
        </text>
        {/each}
        {#each data as d, i}
            <path
                d = {line(getPathCoordinate(d))}
                stroke-width = {3}
                stroke = {color[i]}
                fill = {color[i]}
                stroke-opacity = {1}
                opacity = {0.5}>
            </path>
        {/each}

        <!-- Legend -->
        <g 
            class = "legend"
            transform = "translate(20, 20)">  
            {#each uniqueYearArray as d, i}
                <text 
                    id = {i}
                    x= {0}
                    y = {i*30}
                    dt = {'0.35em'}>{d}</text>
                <rect 
                    x="50" 
                    y={i * 30 - 13} 
                    width="16" 
                    height="16" 
                    fill={color[i]}>
                </rect>
            {/each}
        </g>
    </svg>
    <div class="slider">
        <label for="year-slider">From year {yearVal} and {yearVal+1}</label>
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
</div>
<script>
    import * as d3 from "d3";
    //export let dimensions;
    export let initial;
    export let kunNBA;
    export let legal;
    export let shenZ_debut;
    export let shenZ_parody;
    export let profTom_dance;
    export let yaw;
    export let kunAbor;

    let curr_dataset = kunAbor;

    console.log('curr_dataset', curr_dataset);

    // get the data
    let data = [];

    let dictionary = {}; 

    curr_dataset.forEach(item => {
     dictionary[item.feature] = item.count;
    });

    data.unshift(dictionary);

    console.log('data', data);

    //set dimensions
    let width = 600;
    let height = 600;

    //Scale the figure

    let max = d3.max(curr_dataset, (d) => Math.abs(d.count));
    let roundMax = Math.ceil(max/5)*5;


    let radialScale = d3.scaleLinear()
    .domain([0, roundMax])
    .range([0, 200]);

    function getTicks(max){
        console.log('max', max);
        let roundMax = Math.ceil(max/5)*5;
        let tick = [];
        console.log('roundMax', roundMax);
        for(var i = 1; i <= 5; i++){
            console.log(roundMax/5*i);
            tick.push(roundMax/5*i);
        }
        return tick;
    }

    let ticks = getTicks(max);
    console.log('newTicks', ticks);
    

    function angleToCoordinate(angle, value){
        let x = Math.cos(angle) * radialScale(value);
        let y = Math.sin(angle) * radialScale(value);
        return {'x': width/2+x, 'y': height/2+y};
    }

    //Extract features 
    let features;
    features = curr_dataset.map(function(d){
      return d.feature;
    });
    
    $: console.log('feature ' +typeof(features));

    let featureData = features.map((f, i) => {
        let angle = (Math.PI/2)+(2*Math.PI*i/features.length);
        return{
            'name': f,
            'angle': angle,
            'line_coord': angleToCoordinate(angle, roundMax), 
            'label_coord': angleToCoordinate(angle, roundMax+0.5),
            'text_angle':(angle*180/Math.PI+90)%360
        }
    })

    let line = d3.line()
        .x(d => d.x)
        .y(d => d.y)
        .curve(d3.curveLinearClosed);

    //plot the data
    let color = 'green';

    function getPathCoordinate(data_point){
        let coordinates = [];
        for (var i=0; i<features.length; i++){
            let ft_name = features[i];
            let angle = (Math.PI/2) + (2*Math.PI*i/features.length);
            coordinates.push(angleToCoordinate(angle, data_point[ft_name]));
        }
        return coordinates;
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
                {data}
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
            {data.name}S
        </text>
        {/each}
        {#each data as d, i}
            <path
                d = {line(getPathCoordinate(d))}
                stroke-width = {3}
                stroke = {color}
                fill = {color}
                stroke-opacity = {0.5}
                opacity = {0.5}>
            </path>
        {/each}
    </svg>
</div>
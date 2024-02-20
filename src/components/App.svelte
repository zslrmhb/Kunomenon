<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  // import  zoomState  from "interactive.js";
  import Area from "./Area.svelte";
  import Scatter from "./Scatter.svelte";
  import Heatmap from "./Heatmap.svelte";
  import Radar from "./Radar.svelte";
  //   import color from "../../static/config/color.json";

  // Data Container
  let num_video_per_month = [];
  let important_dates = [];
  let play_count = [];
  let like_count = [];
  let review_count = [];
  let danmaku_count = [];
  let duration_count = [];
  let tag_count = [];
  let initial = [];
  let kunNBA = [];
  let legal = [];
  let shenZ_debut = [];
  let shenZ_parody = [];
  let profTom_dance = [];
  let yaw = [];
  let kunAbor = [];

  // Configures dimensions
  let dimensions = {
    width: 1000,
    height: 500,
    margin: {
      top: 20,
      right: 80,
      bottom: 20,
      left: 80,
    },
  };

  dimensions.boundedWidth =
    dimensions.width - dimensions.margin.left - dimensions.margin.right;
  dimensions.boundedHeight =
    dimensions.height - dimensions.margin.top - dimensions.margin.bottom;

  // Load Data
  onMount(async () => {
    const dataset = await d3.csv("video_count_per_month.csv");
    const dataset2 = await d3.json("important_dates.json");
    const dataset3 = await d3.csv("play.csv");
    const dataset4 = await d3.csv("like.csv");
    const dataset5 = await d3.csv("review.csv");
    const dataset6 = await d3.csv("danmaku.csv");
    const dataset7 = await d3.csv("duration.csv");
    const dataset8 = await d3.csv('tags.csv');
    const dataset9 = await d3.csv('initial_debut.csv');
    const dataset10 = await d3.csv('kunNBA.csv');
    const dataset11 = await d3.csv('legalIssue.csv');
    const dataset12 = await d3.csv('shenZ_debut.csv');
    const dataset13 = await d3.csv('shenZ_parody.csv');
    const dataset14 = await d3.csv('profTom_dance.csv');
    const dataset15 = await d3.csv('yaw.csv');
    const dataset16 = await d3.csv('cxk_abr.csv');

    num_video_per_month = dataset.map(d => ({
      month: d3.timeParse("%Y-%m")(d.month),
      count: +d.count,
    }));
    important_dates = dataset2.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      title: d.title,
    }));
    play_count = dataset3.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      count: +d.play,
    }));
    like_count = dataset4.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      count: +d.like,
    }));
    review_count = dataset5.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      count: +d.review,
    }));
    danmaku_count = dataset6.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      count: +d.danmaku,
    }));
    duration_count = dataset7.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      count: +d.duration_seconds,
    }));
    tag_count = dataset8.map(d => ({
      group: d3.timeParse("%Y-%m")(d.group),
      variable: d.variable,
      value: d.value
    }));
    initial = dataset9.map(d => ({
      feature : d.typename, 
      count : d.count
    }));
    kunNBA = dataset10.map(d => ({
      feature : d.typename,
      count : d.count
    }));
    legal = dataset11.map(d => ({
      feature : d.typename,
      count : d.count
    }));
    shenZ_debut = dataset12.map(d => ({
      feature : d.typename,
      count : d.count
    }));
    shenZ_parody = dataset13.map(d => ({
      feature : d.typename,
      count : d.count
    }));
    profTom_dance = dataset14.map(d => ({
      feature : d.typename,
      count : d.count
    }));
    yaw = dataset15.map(d => ({
      feature : d.typename,
      count : d.count
    }));
    kunAbor = dataset16.map(d => ({
      feature : d.typename,
      count : d.count
    }));
    window.addEventListener("resize", updateSize);
    updateSize();
  });

  // Reponsive Chart
  function updateSize() {
    dimensions.width = window.innerWidth * 0.9;
    dimensions.height = window.innerHeight * 0.35;
    dimensions.boundedWidth =
      dimensions.width - dimensions.margin.left - dimensions.margin.right;
    dimensions.boundedHeight =
      dimensions.height - dimensions.margin.top - dimensions.margin.bottom;
  }
</script>

<main>
  <div class="heading">
    <h1>
      <a
        href="https://github.com/zslrmhb/Kunomenon"
        target="_blank"
        id="bilibili">KUNomenon</a
      >
    </h1>
    <h2>
      The Trend Behind Top 3000+ <a
        href="https://technode.com/2019/04/15/bilibili-threatened-with-lawsuit-about-videos-mocking-chinese-idol/"
        target="_blank"
        id="bilibili">Cai Xukun Videos</a
      >
      on
      <a href="http://bilibili.com" target="_blank" id="bilibili">Bilibili</a>
      #Chinese-Internet-Culture
    </h2>
  </div>
  <!-- <Area {dimensions} {num_video_per_month} {important_dates} /> -->
  <!-- <Scatter
    {dimensions}
    {play_count}
    {like_count}
    {review_count}
    {danmaku_count}
    {duration_count}
    {tag_count}
  /> -->
  <!-- <Heatmap {dimensions} {tag_count} /> -->
  <Radar
    {dimensions} 
    {initial} 
    {kunNBA} 
    {legal} 
    {shenZ_debut} 
    {shenZ_parody} 
    {profTom_dance} 
    {yaw}
    {kunAbor}
  />

</main>

<style>
  * {
    font-family: "Trebuchet MS";
  }

  h1,
  h2 {
    text-align: left;
    margin-left: 3%;
  }

  h1 {
    font-size: x-large;
  }
  h2 {
    font-size: large;
  }

  #bilibili {
    color: #4574cc;
  }

  /* .heading {
    background-color: #c8f6e1;
    margin: 0px;
    padding: 0px;
    width: 100%;
  } */
</style>

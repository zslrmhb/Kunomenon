<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  // import  zoomState  from "interactive.js";
  import Area from "./Area.svelte";
  import Scatter from "./Scatter.svelte";
  import Heatmap from "./Heatmap.svelte";
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

  // Configures dimensions
  let dimensions = {
    width: 1000,
    height: 500,
    margin: {
      top: 20,
      right: 20,
      bottom: 20,
      left: 90,
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
    const dataset8 = await d3.csv("tags.csv");
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
      count: +d.duration,
    }));
    tag_count = dataset8.map(d => ({
      date: d3.timeParse("%Y-%m")(d.group),
      tag: d.variable,
      count: +d.value,
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
  <Area {dimensions} {num_video_per_month} {important_dates} />
  <Scatter
    {dimensions}
    {play_count}
    {like_count}
    {review_count}
    {danmaku_count}
    {duration_count}
    {tag_count}
  />
  <Heatmap {dimensions} {tag_count} />
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

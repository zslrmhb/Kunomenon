<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import Area from "./Area.svelte";
  import Scatter from "./Scatter.svelte";
  //   import color from "../../static/config/color.json";

  // Data Container
  let num_video_per_month = [];
  let important_dates = [];
  let play_count = [];
  let like_count = [];
  let review_count = [];
  let danmaku_count = [];
  let duration_count = [];
  let rank_score_count = [];

  let showScatterPlot = 1;

  // Configures dimensions
  let dimensions = {
    width: 1000,
    height: 500,
    margin: {
      top: 20,
      right: 10,
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
    const dataset = await d3.csv("/data/video_count_per_month.csv");
    const dataset2 = await d3.json("/data/important_dates.json");
    const dataset3 = await d3.csv("/data/play.csv");
    const dataset4 = await d3.csv("/data/like.csv");
    const dataset5 = await d3.csv("/data/review.csv");
    const dataset6 = await d3.csv("/data/danmaku.csv");
    const dataset7 = await d3.csv("/data/duration.csv");
    const dataset8 = await d3.csv("/data/rank_score.csv");
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
    rank_score_count = dataset8.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      count: +d.rank_score,
    }));
    window.addEventListener("resize", updateSize);
    updateSize();
  });

  // Reponse Chart
  function updateSize() {
    dimensions.width = window.innerWidth * 0.9;
    dimensions.height = window.innerHeight * 0.4;
    dimensions.boundedWidth =
      dimensions.width - dimensions.margin.left - dimensions.margin.right;
    dimensions.boundedHeight =
      dimensions.height - dimensions.margin.top - dimensions.margin.bottom;
  }
</script>

<main>
  <h1>KUNomenon</h1>
  <h2>
    The Trend Behind 3500+ <a
      href="https://technode.com/2019/04/15/bilibili-threatened-with-lawsuit-about-videos-mocking-chinese-idol/"
      target="_blank">Cai Xukun Parody Videos</a
    >
    on <strong id="bilibili">Bilibili</strong>
    #Chinese-Internet-Culture
  </h2>
  <Area {dimensions} {num_video_per_month} {important_dates} />

  {#if showScatterPlot}
    <Scatter
      {dimensions}
      {rank_score_count}
      {play_count}
      {like_count}
      {review_count}
      {danmaku_count}
      {duration_count}
    />
  {/if}
</main>

<style>
  h1,
  h2 {
    text-align: left;
  }

  #bilibili {
    color: #4574cc;
  }
</style>

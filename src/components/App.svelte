<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import Area from "./Area.svelte";
  import PlotContainer from "./PlotContainer.svelte";

  // import { GoogleTranslate } from "@candidosales/svelte-google-translate";

  // Data Container
  let num_video_per_month = [];
  let important_dates = [];
  let play_count = [];
  let like_count = [];
  let review_count = [];
  let danmaku_count = [];
  let duration_count = [];
  let tag_count = [];

  // The Top 3 Content Creators
  let tom_count = [];
  let yoyo_count = [];
  let yaw_count = [];

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
    const dataset9 = await d3.csv("tom.csv");
    const dataset10 = await d3.csv("yaw.csv");
    const dateset11 = await d3.csv("yoyo.csv");
    num_video_per_month = dataset.map(d => ({
      date: d3.timeParse("%Y-%m")(d.pubmonth),
      count: +d.video_count,
    }));
    important_dates = dataset2.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      title: d.title,
    }));
    play_count = dataset3.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.pubdate),
      title: d.title,
      count: +d.play,
      arcurl: d.arcurl,
      aid: d.aid,
    }));
    like_count = dataset4.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.pubdate),
      title: d.title,
      count: +d.like,
      arcurl: d.arcurl,
      aid: d.aid,
    }));
    review_count = dataset5.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.pubdate),
      title: d.title,
      count: +d.review,
      arcurl: d.arcurl,
      aid: d.aid,
    }));
    danmaku_count = dataset6.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.pubdate),
      title: d.title,
      count: +d.danmaku,
      arcurl: d.arcurl,
      aid: d.aid,
    }));
    duration_count = dataset7.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.pubdate),
      title: d.title,
      count: +d.duration,
      arcurl: d.arcurl,
      aid: d.aid,
    }));
    tag_count = dataset8.map(d => ({
      group: d3.timeParse("%Y-%m")(d.group),
      variable: d.variable,
      value: d.value,
    }));
    tom_count = dataset9.map(d => ({
      date: d3.timeParse("%Y-%m")(d.pubmonth),
      count: +d.video_count
    }))
    yoyo_count = dataset10.map(d => ({
      date: d3.timeParse("%Y-%m")(d.pubmonth),
      count: +d.video_count
    }))
    yaw_count = dateset11.map(d => ({
      date: d3.timeParse("%Y-%m")(d.pubmonth),
      count: +d.video_count
    }))
    // console.log(yaw_count)
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
    <!-- <GoogleTranslate
      elementId={"google-translate-element"}
      options={{ pageLanguage: 'pt', includedLanguages: 'pt,en,zh-CN' }}
    /> -->
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
    <Area {dimensions} {num_video_per_month} {important_dates} {tom_count} {yoyo_count} {yaw_count}/>
    <PlotContainer
      {dimensions}
      {play_count}
      {like_count}
      {review_count}
      {danmaku_count}
      {duration_count}
      {tag_count}
    />
  </div>
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

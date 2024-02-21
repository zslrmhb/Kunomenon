<script>
  import * as d3 from "d3";
  import { onMount } from "svelte";
  import Area from "./Area.svelte";
  import PlotContainer from "./PlotContainer.svelte";

  // import { GoogleTranslate } from "@candidosales/svelte-google-translate";

  // Data Container
  let numVideoPerMonth = [];
  let importantDates = [];
  let videoInfo = [];
  let tagCount = [];

  // The Top 3 Content Creators
  let tomCount = [];
  let yoyoCount = [];
  let yawCount = [];

  // Configures dimensions
  let dimensions = {
    width: 1000,
    height: 500,
    margin: {
      top: 20,
      right: 0,
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
    const videoDataset = await d3.csv("video_count_per_month.csv");
    const importantDatesDataset = await d3.json("important_dates.json");
    const tomDataset = await d3.csv("tom.csv");
    const yoyoDataset = await d3.csv("yoyo.csv");
    const yawDataset = await d3.csv("yaw.csv");
    const videoInfoDataset = await d3.csv("video_info.csv");
    const tagDataset = await d3.csv("tags.csv");

    numVideoPerMonth = videoDataset.map(d => ({
      pubmonth: d3.timeParse("%Y-%m")(d.pubmonth),
      count: +d.video_count,
    }));
    importantDates = importantDatesDataset.map(d => ({
      date: d3.timeParse("%Y-%m-%d")(d.date),
      title: d.title,
      link: d.url,
    }));
    videoInfo = videoInfoDataset.map(d => ({
      pubdate: d3.timeParse("%Y-%m-%d")(d.pubdate),
      aid: d.aid,
      arcurl: d.arcurl,
      title: d.title,
      playCount: +d.play,
      reviewCount: +d.review,
      favoriteCount: +d.favorites,
      danmakuCount: +d.danmaku,
      likeCount: +d.like,
      durationCount: +d.duration_minutes,
    }));

    tomCount = tomDataset.map(d => ({
      pubmonth: d3.timeParse("%Y-%m")(d.pubmonth),
      count: +d.video_count,
    }));
    yoyoCount = yoyoDataset.map(d => ({
      pubmonth: d3.timeParse("%Y-%m")(d.pubmonth),
      count: +d.video_count,
    }));
    yawCount = yawDataset.map(d => ({
      pubmonth: d3.timeParse("%Y-%m")(d.pubmonth),
      count: +d.video_count,
    }));
    tagCount = tagDataset.map(d => ({
      group: d3.timeParse("%Y-%m")(d.group),
      variable: d.variable,
      value: +d.value,
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
  <div class="content-container">
    <div class="heading-container">
      <h1>
        <a
          href="https://github.com/zslrmhb/Kunomenon"
          target="_blank"
          id="bilibili">KUNomenon</a
        >
      </h1>
      <h2>
        The Trend Behind Top 3000+ <a
          href="https://www.quora.com/Why-is-chicken-you-too-beautiful-%E9%B8%A1%E4%BD%A0%E5%A4%AA%E7%BE%8E-so-popular-in-China-recently"
          target="_blank"
          id="bilibili">Cai Xukun Videos</a
        >
        on
        <a href="http://bilibili.com" target="_blank" id="bilibili">Bilibili</a
        >
        #Chinese-Internet-Culture
      </h2>
    </div>
    <p class="context-description">
      Cai Xukun, alias KUN, is a Chinese pop idol who rose to fame on Bilibili
      (video sharing site) with his song "Because You Are So Beautiful," known
      for its misheard lyrics, his amusing yet iconic dance with the
      basketball, the outfit, and the hairstyle. His prominence grew in the
      controversy with him being selected as the Chinese NBA ambassador in
      early 2019, and subsequently (for you to explore) on 2022. Up until
      today, he still serve as a prominant figure in Chinese internet culture,
      and the top parody source.
    </p>
  </div>

  <Area
    {dimensions}
    {numVideoPerMonth}
    {importantDates}
    {tomCount}
    {yoyoCount}
    {yawCount}
  />
  <PlotContainer {dimensions} {videoInfo} {tagCount} />
</main>

<style>
  * {
    font-family: "Trebuchet MS";
  }

  h1,
  h2 {
    margin-left: 1%;
  }

  h1 {
    font-size: x-large;
    color: white;
  }

  h2 {
    font-size: large;
    color: white;
  }

  p {
    font-size: medium;
    color: white;
  }
  .content-container {
    padding-top: 1em;
    padding-left: 7em;
    padding-right: 5em;
    padding-bottom: 1em;
    margin: -1em;
    margin-bottom: 0.5em;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: rgb(202, 135, 135);
  }

  .heading-container {
    flex: 0 0 50%; 
    margin: 0.5em; 
  }

  .context-description {
    flex: 0 0 50%; 
    text-align: left;
  }

  #bilibili {
    color: #4574cc;
  }

  @media screen and (max-width: 768px) {
    .content-container {
      flex-direction: column;
    }

    .heading-container,
    .context-description {
      flex-basis: 100%;
    }
  }
</style>

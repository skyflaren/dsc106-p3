<script>
  import * as d3 from "d3";
  import * as topojson from "topojson-client";
	import {onMount} from 'svelte';
  import { base } from '$app/paths';

  let slider_label = "Year";
  let slider_time = 0;
  let world_data = {};

  let innerWidth = 0
	let innerHeight = 0

  // Slider for the Years (2002 - 2022)
  const sliderTimeScale = d3
    .scaleLinear()
    .domain([0, 20])
    .rangeRound([2002, 2022]);

  function filterYears(slider_time) {
    let value = sliderTimeScale(slider_time); //converts slider to Year
    console.log("value: " + value);
  }

  const loadDataAndRenderMap = async() => {
    load_data().then(countryData => {
    load_choropleth(countryData);
    });
};
  
  onMount(()=>{
    load_map();
    loadDataAndRenderMap();

    document.getElementById("governance-select").addEventListener("change", () => {
      loadDataAndRenderMap(); 
    });
  })

  // Loads the base world map from topojson file

	const load_map = async() => {
    const svg = d3.select("#my_dataviz")
      .attr("width", innerWidth)
      .attr("height", innerHeight-201)
      .call(d3.zoom().on("zoom", (event) => {   
        g.attr("transform", event.transform); 
      }))

    const width = svg.attr("width");
    const height = svg.attr("height");
    

    const projection = d3.geoMercator()
      .scale(130)
      .center([0,20])
      .translate([width / 2, height / 2]);

    const path = d3.geoPath().projection(projection);
    const g = svg.append("g");

    d3.json(`${base}/world-custom.json`).then(function(world) {
      g.selectAll("path")
        .data(topojson.feature(world, world.objects.countries).features)
        .enter().append("path")
        .attr("d", path)
        .style("stroke", "white") 
        .style("stroke-width", "0.25px") 
        .style("fill", "#aab5bf");
      load_choropleth(world_data);
    });
	}
// Loads the data for governance from the csv file
  const load_data = () => {
    return new Promise((resolve, reject) => {
      const countryData = {};

      d3.csv(`${base}/world_gov_indicators.csv`, function(data) {
        const selectedVal = document.getElementById("governance-select").value;
        countryData[`${data['Country']}, ${data['Year']}`] = parseFloat(data[selectedVal]);
      }).then(() => {
        world_data = countryData;
        resolve(countryData);
      });
    });
  }

// Adds the governance data onto map as choropleth
  const load_choropleth = async(countries) => {
    const filtered_year = await load_filtered(countries);
    const colorScale = d3.scaleSequential([-2.5, 2.5], d3.interpolateBlues);
    d3.selectAll("path")
      .style("fill", d => {
      const countryData = filtered_year[d.properties.name];
      if (!isNaN(countryData)) {
        return colorScale(countryData);
      } else {
        return "#aab5bf";
      }
    })
    .attr("class", d => {
      return d.properties.name
    })
  }

  const load_filtered = async(countries) => {
    const filtered_year = {}
    for (const country in countries){
      if (country.includes("" + sliderTimeScale(slider_time))){
        filtered_year[`${country.substring(0, country.length-6)}`] = countries[country]
      }
    }
    return filtered_year;
  }

$: {
  if (Object.keys(world_data).length !== 0){
    load_choropleth(world_data);
  }
	slider_label = sliderTimeScale(slider_time);
  console.log("uopdate")
}

const resizeWindow = () => {
  innerWidth = window.innerWidth
  innerHeight = window.innerHeight

  d3.select("#my_dataviz")
    .attr("width", innerWidth)
    .attr("height", innerHeight)
}

</script>

<svelte:window on:resize={resizeWindow} bind:innerWidth bind:innerHeight/>

<main>
  <div class="top-half">
    <div>
      <h1>World Governance Indicators Choropleth</h1>
      <hr class="top-hr">
    </div>
    <div class="row-wrap">
      <div class="row-elem">
        <label class="query-label" for="governance"><p class="vert-center">Governance Indicator:</p></label>
        <span class="query-label"><p class="vert-center">Year:</p></span>
      </div>
      <div class="overlay row-elem">
        <select id="governance-select" name="governance">
          <option value="COC" selected="selected">Control of Corruption</option>
          <option value="VAA">Voice And Accountability</option>
          <option value="GE">Government Effectiveness</option>
          <option value="PI">Political Instability</option>
          <option value="RQ">Regulatory Quality</option>
          <option value="ROL">Rule of Law</option>
        </select>
        <div class="row-wrap">
          <input
            id="slider"
            type="range"
            min="0"
            max="20"
            bind:value={slider_time}
          />
          <label id="year-label">{slider_label}</label>
        </div>
      </div>
    </div>
  </div>
  <div class="choropleth" width="{innerWidth}">
    <svg id="my_dataviz" width="600" height="200"></svg>
  </div>
</main>

<style>
  /* @import 'styles.css'; */
  @import url('https://fonts.googleapis.com/css2?family=Outfit:wght@100..900&family=Roboto+Mono:ital,wght@0,100..700;1,100..700&display=swap');

  :global(body) {
    margin: 0; padding: 0;
  }

  main {
    font-family: "Roboto Mono", monospace;
    display: flex;
    flex-direction: column;
    align-items: center;
    overflow: hidden !important;
    overflow-y: hidden !important;
  }

  h1{
    font-size: 2em;
    color: #161c2e;
    margin: 30px 15px 0px;
  }

  .top-half{
    height: 200px;
  }

  .top-hr{
    border: 0.75px #eee solid;
  }

  #governance-select{
    font-family: "Roboto Mono", monospace;
  }

  .query-label{
    font-size: 0.9em;
    display: flex;
    align-items: center;
    justify-content: end;
    text-align: right;

    color: #777;
  }

  .row-wrap {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    margin-top: 0px;
  }

  .row-elem{
    padding: 5px 15px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    margin-left: auto;
  }

  .row-elem > *{
    margin: 5px 0px;
    height: 35px;
  }

  .overlay {
    justify-content: start;
    margin-left: 0px;
    margin-right: auto;
  }

  .overlay > * {
    background-color: #ddd;
    padding: 0px 10px;
    border-radius: 2.5px;
    border: 1px #bbb solid;

    color: #486b99;
  }

  .vert-center{
    margin: 0px;
  }

  label{
    font-size: 0.9em;
    
  }

  #year-label{
    padding: 4px 6px;
    border-radius: 2.5px;
    background-color: #cecece;
  }

  .choropleth {
    border-top: 1px solid #bbb;
    /* width: 100%; */
  }
  
  #my_dataviz{
    display:block;
  }

  @media (min-width: 200px) {
    h1 {
      font-size: 15px;
    }
    .query-label{
      font-size: 9px;
    }
    .overlay > * { 
      padding: 0px 5px;
      font-size: 10px;
    }
  }

  @media (min-width: 500px) {
    h1 {
      font-size: 20px;
    }
    .query-label{
      font-size: 15px;
    }
  }

  @media (min-width: 800px) {
    h1 {
      font-size: 2em;
    }
    .query-label{
      font-size: 0.9em;
    }
    .overlay > * { 
      font-size: 0.7em;
      padding: 0px 10px;
    }
  }
</style>
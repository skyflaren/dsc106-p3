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

  // For dark mode, use
  // let border_col = "#222";
  let border_col = "white";

  
  // Slider for the Years (2002 - 2022)
  const sliderTimeScale = d3
    .scaleLinear()
    .domain([0, 20])
    .rangeRound([2002, 2022]);
  

  function filterYears(slider_time) {
    let value = sliderTimeScale(slider_time); //converts slider to Year
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
      .scale(175)
      .center([0,20])
      .translate([width / 2, height / 2]);

    const path = d3.geoPath().projection(projection);
  
    const g = svg.append("g");

    d3.json(`${base}/world-custom.json`).then(function(world) {
      g.selectAll("path")
        .data(topojson.feature(world, world.objects.countries).features)
        .enter().append("path")
        .attr("d", path)
        .style("stroke", border_col) 
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

    // Tooltip
    d3.select('body')
      .append('div')
      .attr('id', 'tooltip')
      .attr('style', 'position: absolute; opacity: 0; background-color: white; color: black; padding: 2px; border: 1px solid black;')
    
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
      .on("mouseover", d => {
        const countryName = d.srcElement.getAttribute("class")
        d3.select('#tooltip').transition().duration(200).style('opacity', 1).text(`${countryName}: ${filtered_year[countryName]}`)
      })
      .on('mouseout', function() {
        d3.select('#tooltip').style('opacity', 0)
      })
      .on('mousemove', function(event) {
        d3.select('#tooltip')
          .style('left', (event.pageX + 10) + 'px')
          .style('top', (event.pageY + 10) + 'px')
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

<main class="">
  <div class="top-half">
    <div>
      <h1>Governance Quality Across the World</h1>
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
  <span class="data-cred">Data Source: <a href="https://www.worldbank.org/en/publication/worldwide-governance-indicators/interactive-data-access">The World Bank</a></span>
  
  <!-- <div class="theme-switch-wrapper">
    <em>Dark mode?</em>
    <label class="theme-switch" for="checkbox">
          <input type="checkbox" id="checkbox" />
          <div class="slider round"></div>
    </label>
  </div> -->
</main>

<style>
  /* @import 'styles.css'; */
  @import url('https://fonts.googleapis.com/css2?family=Outfit:wght@100..900&family=Roboto+Mono:ital,wght@0,100..700;1,100..700&display=swap');

  :global(body) {
    margin: 0;
    padding: 0;

    /* light mode */
    --background-color: #fff;
    --primary: #486b99;
    --title: #161c2e;
    --drop-shadow: rgba(255, 255, 255, 0.2);
    --heading-label: #777;
    --text-label: #cecece;
    --prompt-border: #bbb;
    --year-background-color: #ddd;
    --divider: #eee;
  }

  .dark-mode {
    --background-color: #222;
    --primary: #5e85b8;
    --title: #e1eaf5;
    --drop-shadow: rgba(0, 0, 0, 0.15);
    --heading-label: #888;
    --text-label: #282828;
    --prompt-border: #0d0d0d;
    --year-background-color: #1d1d1d;
    --divider: #1c1c1c;
  }

  main {
    font-family: "Roboto Mono", monospace;
    display: flex;
    flex-direction: column;
    align-items: center;
    overflow: hidden !important;
    overflow-y: hidden !important;

    background-color: var(--background-color);
  }

  h1{
    font-size: 2em;
    color: var(--title);
    margin: 30px 15px 0px;
  }

  .top-half{
    height: 200px;
  }

  .top-hr{
    border: 0.75px var(--divider) solid;
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

    color: var(--heading-label);
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
    background-color: var(--year-background-color);
    padding: 0px 10px;
    border-radius: 2.5px;
    border: 1px var(--prompt-border) solid;

    color: var(--primary);
  }

  .vert-center{
    margin: 0px;
  }

  label{
    font-size: 0.9em;
  }

  #year-label{
    padding: 4px 6px;
    margin-left: 5px;
    border-radius: 2.5px;
    background-color: var(--text-label);
  }

  .choropleth {
    border-top: 1px solid var(--prompt-border);
  }
  
  #my_dataviz{
    display: block;
  }

  .data-cred{
    position: absolute;
    bottom: 0;
    right: 0;
    padding: 5px 10px 5px 5px;
    font-size: 12px;
    box-shadow: 0px 0px 15px 10px var(--drop-shadow);
    border-top-left-radius: 5px;
    background-color: var(--background-color);
    border: 1px var(--year-background-color) solid;
    color: var(--title);
  }

  a {
    color: var(--primary);
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
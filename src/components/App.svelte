<script>
  import * as d3 from "d3";
  import * as topojson from "topojson-client";
	import {onMount} from 'svelte';
  import { base } from '$app/paths';

  let slider_label = "Year";
  let slider_time = 0;
  let world_data = {};
  const governance_key = {
    "COC": "Control of Corruption",
    "VAA" : "Voice and Accountability",
    "GE": "Government of Effectiveness",
    "PI": "Political Stability",
    "RQ": "Regulatory Quality",
    "ROL": "Rule of Law"
  }

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

  const loadDataAndRenderMap = async() => {
    load_data().then(countryData => {
    load_choropleth(countryData);
    });
};

  const loadLegend = () => {
    let svg = d3.select("#legend");
      svg.selectAll("*").remove();

    legend({
      color: d3.scaleSequential([-2.5, 2.5], d3.interpolateBlues),
      title: governance_key[document.getElementById("governance-select").value],
      tickFormat: ".1f"
    })

  // Code adapted from observablehq.com, since d3-legend-swatch has outdated documentation
    function legend({
  color,
  title,
  tickSize = 6,
  width = 320,
  height = 44 + tickSize,
  marginTop = 18,
  marginRight = 0,
  marginBottom = 16 + tickSize,
  marginLeft = 0,
  ticks = width / 64,
  tickFormat,
  tickValues
} = {}) {
  const svg = d3.select("#legend")
    .attr("width", width)
    .attr("height", height)
    .attr("viewBox", [0, 0, width, height])
    .style("overflow", "visible")
    .style("display", "block");

  let tickAdjust = g => g.selectAll(".tick line").attr("y1", marginTop + marginBottom - height);
  let x;

  // Continuous
  if (color.interpolate) {
    const n = Math.min(color.domain().length, color.range().length);

    x = color.copy().rangeRound(d3.quantize(d3.interpolate(marginLeft, width - marginRight), n));

    svg.append("image")
      .attr("x", marginLeft)
      .attr("y", marginTop)
      .attr("width", width - marginLeft - marginRight)
      .attr("height", height - marginTop - marginBottom)
      .attr("preserveAspectRatio", "none")
      .attr("xlink:href", ramp(color.copy().domain(d3.quantize(d3.interpolate(0, 1), n))).toDataURL());
  }

  // Sequential
  else if (color.interpolator) {
    x = Object.assign(color.copy()
      .interpolator(d3.interpolateRound(marginLeft, width - marginRight)), {
        range() {
          return [marginLeft, width - marginRight];
        }
      });

    svg.append("image")
      .attr("x", marginLeft)
      .attr("y", marginTop)
      .attr("width", width - marginLeft - marginRight)
      .attr("height", height - marginTop - marginBottom)
      .attr("preserveAspectRatio", "none")
      .attr("xlink:href", ramp(color.interpolator()).toDataURL());

    // scaleSequentialQuantile doesn’t implement ticks or tickFormat.
    if (!x.ticks) {
      if (tickValues === undefined) {
        const n = Math.round(ticks + 1);
        tickValues = d3.range(n).map(i => d3.quantile(color.domain(), i / (n - 1)));
      }
      if (typeof tickFormat !== "function") {
        tickFormat = d3.format(tickFormat === undefined ? ",f" : tickFormat);
      }
    }
  }

  // Threshold
  else if (color.invertExtent) {
    const thresholds = color.thresholds ? color.thresholds() // scaleQuantize
      :
      color.quantiles ? color.quantiles() // scaleQuantile
      :
      color.domain(); // scaleThreshold

    const thresholdFormat = tickFormat === undefined ? d => d :
      typeof tickFormat === "string" ? d3.format(tickFormat) :
      tickFormat;

    x = d3.scaleLinear()
      .domain([-1, color.range().length - 1])
      .rangeRound([marginLeft, width - marginRight]);

    svg.append("g")
      .selectAll("rect")
      .data(color.range())
      .join("rect")
      .attr("x", (d, i) => x(i - 1))
      .attr("y", marginTop)
      .attr("width", (d, i) => x(i) - x(i - 1))
      .attr("height", height - marginTop - marginBottom)
      .attr("fill", d => d);

    tickValues = d3.range(thresholds.length);
    tickFormat = i => thresholdFormat(thresholds[i], i);
  }

  // Ordinal
  else {
    x = d3.scaleBand()
      .domain(color.domain())
      .rangeRound([marginLeft, width - marginRight]);

    svg.append("g")
      .selectAll("rect")
      .data(color.domain())
      .join("rect")
      .attr("x", x)
      .attr("y", marginTop)
      .attr("width", Math.max(0, x.bandwidth() - 1))
      .attr("height", height - marginTop - marginBottom)
      .attr("fill", color);

    tickAdjust = () => {};
  }

  svg.append("g")
    .attr("transform", `translate(0,${height - marginBottom})`)
    .call(d3.axisBottom(x)
      .ticks(ticks, typeof tickFormat === "string" ? tickFormat : undefined)
      .tickFormat(typeof tickFormat === "function" ? tickFormat : undefined)
      .tickSize(tickSize)
      .tickValues(tickValues))
    .call(tickAdjust)
    .call(g => g.select(".domain").remove())
    .call(g => g.append("text")
      .attr("x", marginLeft)
      .attr("y", marginTop + marginBottom - height - 6)
      .attr("fill", "currentColor")
      .attr("text-anchor", "start")
      .attr("font-weight", "bold")
      .text(title));

  return svg.node();
}

function ramp(color, n = 256) {
  var canvas = document.createElement('canvas');
  canvas.width = n;
  canvas.height = 1;
  const context = canvas.getContext("2d");
  for (let i = 0; i < n; ++i) {
    context.fillStyle = color(i / (n - 1));
    context.fillRect(i, 0, 1, 1);
  }
  return canvas;
}
}

  
  onMount(()=>{
    load_map();
    loadDataAndRenderMap();
    loadLegend();

    document.getElementById("governance-select").addEventListener("change", () => {
      loadDataAndRenderMap(); 
      loadLegend();
    });

    document.getElementById("play").addEventListener("click", () => {
      cycle_years();
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
        .style("fill", "#6b6b6b");
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
        return "#6b6b6b";
      }
    })
      .attr("class", d => {
        return d.properties.name
      })
      .on("mouseover", d => {
        const countryName = d.srcElement.getAttribute("class")
        d3.select('#tooltip').transition().duration(200).style('opacity', 1).text(`${countryName}: ${filtered_year[countryName] || 'undefined'}`)
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

  const cycle_years = async() => {
    d3.select("#play").attr("pointer-events", "none");
    for (let i = 0; i < 21; i++) {
      slider_time = i;
      await new Promise(r => setTimeout(r, 400));
    }
    d3.select("#play").attr("pointer-events", "auto");
  }

$: {
  if (Object.keys(world_data).length !== 0){
    load_choropleth(world_data);
  }
	slider_label = sliderTimeScale(slider_time);
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
        <span class="query-label"><p class="vert-center">Legend:</p></span>
      </div>
      <div class="overlay row-elem">
        <select id="governance-select" name="governance">
          <option value="COC" selected="selected">Control of Corruption</option>
          <option value="VAA">Voice And Accountability</option>
          <option value="GE">Government Effectiveness</option>
          <option value="PI">Political Stability</option>
          <option value="RQ">Regulatory Quality</option>
          <option value="ROL">Rule of Law</option>
        </select>
        <div class="row-wrap">
          <svg id="play" width="100" height="100" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
            <circle cx="50" cy="50" r="48" stroke="black" stroke-width="4" fill="none" />
            <polygon points="40,30 70,50 40,70" fill="black" />
          </svg>
          <input
            id="slider"
            type="range"
            min="0"
            max="20"
            bind:value={slider_time}
          />
          <label id="year-label">{slider_label}</label>
        </div>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/d3/6.2.0/d3.min.js"></script>
        <svg id="legend"></svg>
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

    font-family: "Roboto Mono", monospace;

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
    height: auto;
  }

  .top-hr{
    border: 0.75px var(--divider) solid;
  }

  #governance-select{
    font-family: "Roboto Mono", monospace;
  }

  #slider{
    min-width: 80%;
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
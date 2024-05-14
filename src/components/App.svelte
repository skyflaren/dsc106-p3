<script>
  import * as d3 from "d3";
  import * as topojson from "topojson-client";
	import {onMount} from 'svelte';
  import { base } from '$app/paths';

  let slider_label = "Year";
  let slider_time = 0;
  let world_data = {};

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
      .attr("width", 1200)
      .attr("height", 600)
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
    console.log("first loaded map")
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
    console.log("started chloro")
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
    console.log(filtered_year);
    return filtered_year;
  }

$: {
  console.log(sliderTimeScale(slider_time));
  console.log(world_data);
  if (Object.keys(world_data).length !== 0){
    console.log(Object.keys(world_data).length);
    load_choropleth(world_data);
  }
	slider_label = sliderTimeScale(slider_time);
}
</script>

<main>
  <h1>World Governance Indicators Choropleth</h1>
  <label for="governance">Choose a Governance Indicator</label>
  <select id="governance-select" name="governance">
    <option value="COC" selected="selected">Control of Corruption</option>
    <option value="VAA">Voice And Accountability</option>
    <option value="GE">Government Effectiveness</option>
    <option value="PI">Political Instability</option>
    <option value="RQ">Regulatory Quality</option>
    <option value="ROL">Rule of Law</option>
  </select>
  <div class="overlay">	
		<label>{slider_label}</label>
		<input
			id="slider"
			type="range"
			min="0"
			max="20"
			bind:value={slider_time}
		/>
	</div>
  <div class="choropleth">
    <svg id="my_dataviz" width="600" height="600"></svg>
  </div>
</main>

<style>
  main {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .choropleth {
    border: 1px solid black;
  }
</style>
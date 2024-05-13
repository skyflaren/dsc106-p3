<script>
  import * as d3 from "d3";
  import * as topojson from "topojson-client";
	import {onMount} from 'svelte';

  const loadDataAndRenderMap = () => {
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

	const load_map = () => {
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

    d3.json("/world-custom.json").then(function(world) {
      g.selectAll("path")
        .data(topojson.feature(world, world.objects.countries).features)
        .enter().append("path")
        .attr("d", path)
        .style("stroke", "white") 
        .style("stroke-width", "0.25px") 
        .style("fill", "#aab5bf");
    });

	}

  const load_data = () => {
    return new Promise((resolve, reject) => {
      const countryData = {};

      d3.csv("/world_gov_indicators.csv", function(data) {
        const selectedVal = document.getElementById("governance-select").value;
        countryData[`${data['Country']}, ${data['Year']}`] = parseFloat(data[selectedVal]);
      }).then(() => {
        resolve(countryData);
      });
    });
  }

  const load_choropleth = (countries) => {
    // Only shows 2002 data til we implement time scale
    const only_2002 = {}
    for (const country in countries){
      if (country.includes("2002")){
        only_2002[`${country.substring(0, country.length-6)}`] = countries[country]
      }
    }
    const colorScale = d3.scaleSequential([-2.5, 2.5], d3.interpolateBlues);
    d3.selectAll("path")
      .style("fill", d => {
      const countryData = only_2002[d.properties.name];
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

</script>

<main>
  <h1>World Governance Indicators Choropleth</h1>
  <label for="governance">Choose a Governance Indicator</label>
  <select id="governance-select" name="governance">
    <option value="COC">Control of Corruption</option>
    <option value="VAA">Voice And Accountability</option>
    <option value="GE">Government Effectiveness</option>
    <option value="PI">Political Instability</option>
    <option value="RQ">Regulatory Quality</option>
    <option value="ROL">Rule of Law</option>
  </select>
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
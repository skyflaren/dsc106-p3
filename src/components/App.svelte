<script>
	// original: https://observablehq.com/@d3/choropleth/2
	// svelte version: Sebastian Lammers https://vis.social/@seblammers
	import Choropleth from './Chloro.svelte';
	// import data from './assets/data.csv';

  import * as d3 from "d3";
	import {onMount} from 'svelte';
  let ldata;
  
  onMount(()=>{
		load();
    createGraph();
    ldata = load()
	})
	
	const load = ()=>{
    ret = {}
    d3.csv("/data.csv", function(data) {
      // console.log(data);
      // d3.select('body').append('p').text(data['Country Name'])
        
      // console.log(data['Country Name'])
      // tabulate(data, ["Country Name", "Country Code"])
      return ret.push(data);
    });
    console.log(ret)
    return ret
	}
</script>

<Choropleth {ldata} />
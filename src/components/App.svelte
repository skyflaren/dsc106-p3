<script>
  import * as d3 from "d3";
	import {onMount} from 'svelte';
  
  onMount(()=>{
		load();
	})
	
	const load = ()=>{
    d3.csv("/data.csv", function(data) {
      // console.log(data);
      d3.select('body').append('p').text(data['Country Name'])
      // console.log(data['Country Name'])
    });
	}

</script>

<main>
  <h1>Svelte template</h1>

  <p>Write your HTML here</p>
</main>

<style>
  /* Write your CSS here */
</style>
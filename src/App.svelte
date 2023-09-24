<script lang="ts">
  // :)
  import { onMount } from 'svelte';
  import L from 'leaflet';
  import "leaflet/dist/leaflet.css";
  import buildings from "./buildings.json?raw"
  import printers from "./building-printers.json";

  let search;


  let map
  onMount(() => {
  map = L.map('map').setView([45.50959185530315, -73.57638495216982], 15);
  
  L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
  }).addTo(map);

  // add buildings.json features to map
  L.geoJSON(JSON.parse(buildings),
  {
    style: function (feature) {
      return {
        color: "#000",
        weight: 1,
        fillOpacity: 0.5,
        fillColor: "lightcoral"
      };
    }}
  ).bindPopup(function (layer) {
    const buildingPrinters = printers.filter(a => a['Building Name'] == layer.feature.properties.name);
    return `
    ${layer.feature.properties.name}
    <br/>
    
    ${buildingPrinters.map(printer => `<strong>${printer["Room"]}</strong>`).join("<br/>")}
    <br/>
    <hr/>
    ${buildingPrinters.length} printers
    `
    
}).addTo(map).on('click', function(e) {
  // scroll to table row
 search = e.layer.feature.properties.name;
}).on("popupclose", function(e) {
  // remove search filter
  search = "";
});
});
</script>

<main>
  <div id="map"/>
  <div>
  <div class="rhs">
    <div class="title">
      <h1><span>McGill</span> Printers</h1> 
      <hr/>
      <h2>map/search uprint printers; note that not all are guaranteed to be public access</h2>
    </div>
  <input type="text" bind:value="{search}" placeholder="Filter building/department"/>
  <table class="search-results">
    <thead>
      <th>Building</th>
      <th>Room</th>
      <th>Department</th>
    </thead>
    {#each printers as printer (printer["Serial Number"])}
    <tr>
      {#if search && (printer["Building Name"].toLowerCase().includes(search.toLowerCase()) || printer["Department"].toLowerCase().includes(search.toLowerCase()))}
          <td on:click={()=>{
            map.eachLayer(function (layer) {
              if (layer?.feature?.properties?.name == printer["Building Name"]) {
                map.setView(layer.getBounds().getCenter(), 17, {duration: 0.5, animate: true});
                // click on layer
                map.openPopup(layer.getPopup());
              }
            });
          }}>{printer["Building Name"]}</td>
          <td>{printer["Room"]}</td>
          <td>{printer["Department"]}</td>
        {:else if !search}
          <td on:click={()=>{
            map.eachLayer(function (layer) {
              if (layer.feature && layer.feature.properties.name == printer["Building Name"]) {
                map.setView(layer.getBounds().getCenter(), 17, {duration: 0.5, animate: true});
                // click on layer
                map.openPopup(layer.getPopup());
              }
            });
          }}
          >{printer["Building Name"]}</td>
          <td>{printer["Room"]}</td>
          <td>{printer["Department"]}</td>
      {/if}
    {/each}
    </table>
    <div class="footer">
      <span>Not associated with McGill</span>
    </div>
  </div>
  
</div>
</main>

<style>
  main {
    text-align: center;
    margin: 0 auto;
    height: 100vh;

    display: grid;
    grid-template-columns: 1fr 1.5fr;
    overflow-y: hidden;
  }

  #map {
    width: 500px;
    height: 100vh;
    overflow-y: hidden;
  }

  .search-results {
    /* make table more compact */
    border-collapse: collapse;
    /* fixed width */
    width: 90%;
    margin-top: 1em;
    /* margin-bottom: em; */

  }

  .search-results th,
  .search-results td {
    padding: 0.5rem;
    border: 1px solid black;

    background-color: lightgray;
  }

  .search-results th {
    background-color: lightcoral;
  }

  .rhs {
    /* make scrollable while keeping other half static */
    overflow-y: scroll;
    height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1em;
  }

  .rhs input {
    width: 33%;
    /* margin-bottom: 1em; */
    padding: 0.5em;
    border-radius: 0.5em;
    border: 2px solid #ccc;
    /* position: fixed; */
    /* margin-top: 5em; */
  }

  em {
/* underline */
  text-decoration: underline;
  }
  .title {
    /* margin-top: 1em; */
    font-style: italic;
  }
  .title h1 {
    font-size: 1.4em;
    /* text-decoration: dotted overline; */
  }
  .title h2 {
    font-size: 0.8em;
    /* color:  */
  }
  .title span {
    color: lightcoral;
    /* text-decoration: dotted underline; */

  }

  .footer {
    /* position: sticky; */
    /* bottom: 0; */
    /* margin-top: -4em; */
    color: lightcoral;
    font-size: 0.8em;
    font-style: italic;
    text-align: center;
    /* width: 100%; */
  }

  hr {
    width: 15%;
    border: 1px solid lightcoral;
  }
</style>

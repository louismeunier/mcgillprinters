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

  // const arr1 = JSON.parse(buildings)["features"].map(a => a.properties.name)
  //   // console.log([... new Set(names_buildings)])
  //   const arr2 = printers.map(a=>a['Building Name'])
  //   // console.log([... new Set(names_printers)])

  //   const diff = arr1.filter(x => !arr2.includes(x))
  //                       .concat(arr2.filter(x => !arr1.includes(x)));

  //                       console.log([... new Set(diff)])
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

console.log([... new Set(printers.map(a => a.Model))].map(a => {
  return [a, printers.filter(b => b.Model == a).length]
}))

</script>

<main>
  <div id="map"/>
  <div>
  <div class="rhs">
    <div class="title">
  <h1><span>McGill</span> Printers</h1> 
      <hr/>
      <span>Up-to-date: <a href="https://mcgill.service-now.com/itportal?id=kb_article_view&sysparm_article=KB0010984&sys_kb_id=3e3698f497fa9d10200579cfe153af71&spa=1" target="_blank">November 2023</a>. </span>
      <span>Not associated with McGill.</span>
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
          <td>{printer["Building Name"]}</td>
          <td>{printer["Room"]}</td>
          <td>{printer["Department"]}</td>
        {:else if !search}
          <td >{printer["Building Name"]}</td>
          <td>{printer["Room"]}</td>
          <td>{printer["Department"]}</td>
      {/if}
    {/each}
    </table>

    <!-- <div>
      <p>Total: {printers.length}</p>
      <p>Most Popular Model: {[... new Set(printers.map(a => a.Model))].map(a => {
        return [a, printers.filter(b => b.Model == a).length]
      })[0][0]} ({[... new Set(printers.map(a => a.Model))].map(a => {
        return [a, printers.filter(b => b.Model == a).length]
      })[0][1]})</p>
      <!-- <p>Printer-est Building: </p> -->
    <!-- </div> --> -->
    <!-- <div class="footer"> -->
      
    <!-- </div> -->
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
    border-radius: 2em;
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
    position: fixed;
    bottom: 0;
    /* margin-top: -4em; */
    color: lightcoral;
    font-size: 0.8em;
    font-style: italic;
    text-align: center;
    margin-bottom: 1em;
    /* width: 100%; */
  }

  hr {
    width: 15%;
    border: 1px solid lightcoral;
  }

  .title> span {
    font-size: 0.8em;
    color:black;
  }
</style>

<!-- on:click={()=>{
            map.eachLayer(function (layer) {
              if (layer.feature && layer.feature.properties.name == printer["Building Name"]) {
                map.setView(layer.getBounds().getCenter(), 17, {duration: 0.5, animate: true});
                // click on layer
                map.openPopup(layer.getPopup());
              }
            }); -->
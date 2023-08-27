<script>
  let allHouses = [
    {
      name: 'Astoran',
      cracked: true,
      techs: ['Deep Space Missiles', 'Sentries'],
      complexity: 2
    },
    {
      name: 'Belitan',
      cracked: false,
      techs: ['Data Refinery', 'Targeting'],
      complexity: 1
    },
    {
      name: 'Cortozaar',
      cracked: false,
      techs: ['Starbases','Torpedoes'],
      complexity: 1
    },
    {
      name: 'Dunlork',
      cracked: true,
      techs: ['Energy Cells','Orbital Docks'],
      complexity: 1
    },
    {
      name: 'Fenrax',
      cracked: false,
      techs: ['Carriers','Central Surveillance'],
      complexity: 4
    },
    {
      name: 'Kradmor',
      cracked: false,
      techs: ['Purifier','Salvage Scanner'],
      complexity: 4
    },
    {
      name: 'Marqualos',
      cracked: false,
      techs: ['Trade Nexus','Autonomous Drones'],
      complexity: 3
    },
    {
      name: 'Nervo',
      cracked: false,
      techs: ['Ark Ships','Robotics'],
      complexity: 3 
    },
    {
      name: 'Novaris',
      cracked: false,
      techs: ['Cybernetics','Combat Replicators'],
      complexity: 4
    },
    {
      name: 'Shiveus',
      cracked: true,
      techs: ['Dreadnoughts','Decontamination Chambers'],
      complexity: 3
    },
    {
      name: 'Thegwyn',
      cracked: false,
      techs: ['Neural Matrix','Terraforming'],
      complexity: 2
    },
    {
      name: 'Valnis',
      cracked: false,
      techs: ['Escape Pods','Shields'],
      complexity: 1
    },
    {
      name: 'Yarvek',
      cracked: true,
      techs: ['Hyperdrive','Tactical Transports'],
      complexity: 2
    },
    {
      name: 'Zenor',
      cracked: true,
      techs: ['Cloning','Destroyers'],
      complexity: 2
    },
  ]
  let houses = [...allHouses];
  let crackedHouses = houses.filter(h => h.cracked);
  let normalHouses = houses.filter(h => !h.cracked);
  let fallenHouses = [];
  let techs = [];

  function toggleAdd(h){
    if(playerHouses.includes(h)){
      playerHouses = playerHouses.filter(ho => ho.name != h.name);
    }else{
      playerHouses = [...playerHouses, h]; 
      if(playerHouses.length > 4) playerHouses = playerHouses.slice(1);
    }
  }

  let playerHouses = [];
  let chosenHouses = [];

  function randomElements(array, howMany = 1){
    let chosen = [];
    for(let times = 0; times < howMany; times++){
      let sel = array.filter(a => !chosen.includes(a));
      let max = sel.length;
      let i = Math.floor(Math.random() * max)
      chosen = [...chosen, sel[i]];
      console.log(array, i, chosen)
    }
    return chosen;
  }

  function shuffle(array) {
    let currentIndex = array.length,  randomIndex;

    // While there remain elements to shuffle.
    while (currentIndex > 0) {

      // Pick a remaining element.
      randomIndex = Math.floor(Math.random() * currentIndex);
      currentIndex--;

      // And swap it with the current element.
      [array[currentIndex], array[randomIndex]] = [
        array[randomIndex], array[currentIndex]];
    }

    return array;
  }

  let players = 1;

  function randomPH(){
    let ph = [...houses];
    shuffle(ph);
    playerHouses = ph.slice(0, players);
  }

  function getTechs(h){
    return `Technologies: ${h.techs.join(', ')}`;
  }

  function getHouses(){
    chosenHouses = [];
    let availableCracked = crackedHouses.filter(h => !playerHouses.includes(h));
    let availableNormal = normalHouses.filter(h => !playerHouses.includes(h));
    let availableAll = [...availableCracked, ...availableNormal];
    
    chosenHouses = [...chosenHouses, ...randomElements(availableCracked), ...randomElements(availableNormal)];
    availableAll = availableAll.filter(h => !chosenHouses.includes(h));
    chosenHouses = [...chosenHouses, ...randomElements(availableAll, 2)];
    fallenHouses = chosenHouses;
    shuffle(fallenHouses);

    // Set up techs
    let availableTechs = fallenHouses.flatMap(h => h.techs);
    shuffle(availableTechs);
    let slicing = 3;
    if(players == 2) slicing = 2;
    if(players < 3){
      let noInfTechs = availableTechs.slice(0, slicing);
      let infTechs = availableTechs.slice(slicing);
      techs = [
        ...infTechs.map(t => {
          return {
            name: t,
            inf:  players == 1 ? `+4 Influence card` : `both cards`,
            class:  players == 1 ? 'inf' : 'both'
          }
        }),
        ...noInfTechs.map(t => {
          return {
            name: t,
            inf: '0 Influence card',
            class: 'no_inf'
          }
        })
      ];
    }else{
      techs = availableTechs.map(t => {
        return {
          name: t,
          inf: `both cards`,
          class: 'both'
        }
      })
    }
  }

  let topComplexity = 4;
  $: houses = allHouses.filter(h => h.complexity <= topComplexity);
</script>

<main>
    <h1>Voidfall house randomiser</h1>
    {#if fallenHouses.length == 0}
    <div class="row">
      Player count: 
      {#each ['Solo', '2 players', '3 players', '4 players'] as p, i}
        <span on:click={e => {players = i + 1}} class="card_info" class:selected={players == i + 1}>{p}</span>
      {/each}
    </div>
    <div>
      <div class="row">
        Select player houses: <button style="font-size: 3vmin; padding: 1vmin;" on:click={randomPH}>Choose at random</button>
        Max complexity: 
        <select style="font-size: 3vmin; padding: 1vmin;" bind:value={topComplexity}>
          <option value={1}>1</option>
          <option value={2}>2</option>
          <option value={3}>3</option>
          <option value={4}>4</option>
        </select>
      </div>
      <div class="row" style="display: flex; flex-wrap: wrap;"> 
        {#each houses as h}
          <span on:click={e => {toggleAdd(h)}} class="card_info" title={getTechs(h)} class:selected={playerHouses.includes(h)} style="cursor: pointer;">{h.name}{#if h.cracked}&nbsp;💥{/if}</span>
        {/each}
      </div>
    </div>
    <hr />
    <div>
        <button style="font-size: 4vmin; padding: 1vmin;" on:click={getHouses}>Generate!</button>
    </div>
    {:else}
      <div>Player houses:</div>
      <div class="row" style="display: flex; flex-wrap: wrap;"> 
        {#each playerHouses as h}
          <span class="card_info selected">{h.name}{#if h.cracked}&nbsp;💥{/if}</span>
        {/each}
      </div>
      <div>
        Fallen houses (in random order – use as many as needed for scenario):
        <div class="row" style="display: flex; flex-wrap: wrap;"> 
          {#each fallenHouses as h}
            <span class="card_info">{h.name}{#if h.cracked}&nbsp;💥{/if}</span>
          {/each}
          
        </div>
        Technologies available {#if players < 3}(with Basic cards to use/remove){/if}:
        <table>
          <tr>
            <th>House</th>
              {#if players == 1}
                <th>Just 0 Influence card</th>
                <th>Just +4 Influence card</th>
              {:else if players == 2}
                <th>Just 0 Influence card</th>
                <th>Both cards</th>
              {:else}
                <th>Cards to use</th>
              {/if}
          </tr>
          {#each [...fallenHouses.sort((a,b) => a.name.localeCompare(b.name))] as h}
            <tr>
              <td class="house_name">{h.name}</td>
              {#if players < 3}<td class="no_inf">{techs.filter(t => h.techs.includes(t.name)).filter(t => t.class == 'no_inf').map(t => t.name).join(', ')}</td>{/if}
              {#if players == 1}
                <td class="inf">{techs.filter(t => h.techs.includes(t.name)).filter(t => t.class == 'inf').map(t => t.name).join(', ')}</td>
              {:else}
                <td class="both">{techs.filter(t => h.techs.includes(t.name)).filter(t => t.class == 'both').map(t => t.name).join(', ')}</td>
              {/if}
            </tr>
          {/each}
        </table>
        <hr />
        <div>
          <button style="font-size: 4vmin; padding: 1vmin;" on:click={getHouses}>Generate again!</button>
          <button style="font-size: 4vmin; padding: 1vmin;" on:click={e => {fallenHouses = []; playerHouses = [];}}>Reset!</button>
        </div>
      </div>
    {/if}
</main>

<style>
  .row{
    padding: 1vmin;
  }

  .card_info {
    padding-left: 10px;
    padding-right: 10px;
    padding-top: 5px;
    padding-bottom: 5px;
    margin-top: 3px;
    margin-bottom: 3px;
    margin-left: 5px;
    margin-right: 5px;
    font-weight: bold;
    color: white;
    border: 0.5vmin solid white;
    background-color: darkmagenta;
    cursor: pointer;
  }

  .selected {
    background-color: yellow;
    color: black;
  }

  .bottom { 
    position: fixed;
    bottom: 0;
    width: 100%;
  }

  .middle {
    overflow-y: auto;
    height: 50vh;
  }

  .house_name {
    font-weight: bold;
  }
  /*.inf {
    color: lightgreen;
  }
  
  .no_inf { 
    color: red;
  }

  .both {
    color: lightblue;
  }*/

  table, tr, th, td {
    border: 2px solid white;
    border-collapse: collapse;
  }

  th {
    background-color: slategray;
  }
</style>

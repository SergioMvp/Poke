<script setup>
import axios from 'axios';
import { ref, onMounted, computed } from 'vue';





let imagen = ref([]);
let num = ref([]);
let nombre = ref([]);
let peso = ref([]);
let altura = ref([]);
let tipo = ref([]);
let hp = ref("");
let attack = ref("");
let defense = ref("");
let special_attack = ref("");
let special_defense = ref("")
let speed = ref("");
const home = ref(1);
let mostrar = ref(true);
let mostrardos = ref(false);
let mostrartres = ref(true);
let pokemons = ref([]);
let selectedPokemon = ref(null);
let searchQuery = ref("");
let tipoFiltrado = ref(""); 
let cantidadPokemonesCargados = ref(50);
const tiposDisponibles = ref([])
const botonesHeader = ref([]); 
const listaPokemon = ref([]); 
const availableTypes = [
  "fire",
  "grass",
  "electric",
  "water",
  "ground",
  "rock",
  "fairy",
  "poison",
  "bug",
  "dragon",
  "psychic",
  "flying",
  "fighting",
  "normal",
  "ice",
  "dark",
  "ghost",
];

let selectedType = ref("");

const filteredPokemonData = computed(() => {
  let filteredData = pokemonData.value;

  if (selectedType.value) {
    filteredData = filteredData.filter((pokemon) =>
      pokemon.tipo_pk.includes(selectedType.value)
    );
  }
})



const handleButtonClick = (botonId) => {
  listaPokemon.value = [];

  for (let i = 1; i <= 151; i++) {
    fetch(URL + i)
      .then((response) => response.json())
      .then((data) => {
        if (botonId === "ver-todos") {
          mostrarPokemon(data);
        } else {
          const tipos = data.types.map((type) => type.type.name);
          if (tipos.some((tipo) => tipo.includes(botonId))) {
            mostrarPokemon(data);
          }
        }
      });
  }
};

function volverALista() {
  mostrardos.value = null;
  mostrar.value = true; 
}

function filtrado() {
  tiposDisponibles.value = [];
  
  for (const pokemon of pokemons.value) {
    for (const tipo of pokemon.tipos) {
      if (!tiposDisponibles.value.includes(tipo.name)) {
        tiposDisponibles.value.push(tipo.name);
      }
    }
  }
}


// function filtrado() {
//   tiposDisponibles.value = computed(() => {
//     const tipos = new Set();

//     for (const pokemon of pokemons.value) {
//       for (const tipo of pokemon.tipos) {
//         tipos.add(tipo.name);
//       }
//     }

//     return ["", ...Array.from(tipos)]; // Agrega una opción vacía al inicio aquí agregamos todos los tipos de pokemones 
//   });
// }

function cargarMasPokemones() {
  cantidadPokemonesCargados.value += 50; 
  obtenerPokemons(); 
}

async function obtenerPokemons() {
  let offset = 0; 

  if (cantidadPokemonesCargados.value > 50) {
    
    offset = cantidadPokemonesCargados.value - 50;
  }

  let pokemonsResponse = await axios.get(
    `https://pokeapi.co/api/v2/pokemon?limit=50&offset=${offset}`
  );

  let pokemonUrls = pokemonsResponse.data.results.map((pokemon) => pokemon.url);

  for (const url of pokemonUrls) {
    await obtenerurlpk(url);
  }
}

function buscarPokemon() {
  if (!searchQuery.value) {
    return;
  }

  const searchTerm = searchQuery.value.toLowerCase();

  
  let foundPokemon = pokemons.value.filter((pokemon) =>
    pokemon.nombre.toLowerCase() === searchTerm || pokemon.numero.toString() === searchTerm
  );

  
  if (tipoFiltrado.value) {
    const tipoSeleccionado = tipoFiltrado.value.toLowerCase();

    foundPokemon = foundPokemon.filter((pokemon) =>
      pokemon.tipos.some((tipo) => tipo.name === tipoSeleccionado)
    );
  }

  if (foundPokemon.length > 0) {
    seleccionarPokemon(foundPokemon[0]);
  } else {
    console.log("Pokémon no encontrado");
  }
}


// function buscarPokemon() {
//   if (!searchQuery.value) {
//     return; 
//   }

//   const searchTerm = searchQuery.value.toLowerCase();

//   const foundPokemon = pokemons.value.find((pokemon) => {
    
//     return (
//       pokemon.nombre.toLowerCase() === searchTerm ||
//       pokemon.numero.toString() === searchTerm
//     );
//   });

//   if (foundPokemon) {
    
//     seleccionarPokemon(foundPokemon);
//   } else {
    
//     console.log("Pokémon no encontrado");
//   }
// // FILTRADO
//   if (tipoFiltrado.value) {
//     // Si se ha seleccionado un tipo, filtra por ese tipo
//     const tipoSeleccionado = tipoFiltrado.value.toLowerCase();

//     foundPokemon = pokemons.value.filter((pokemon) => {
//       return (
//         (pokemon.nombre.toLowerCase() === searchTerm ||
//           pokemon.numero.toString() === searchTerm) &&
//         pokemon.tipos.some((tipo) => tipo.name === tipoSeleccionado)
//       );
//     });
//   }


//   if (foundPokemon.length > 0) {
//     seleccionarPokemon(foundPokemon[0]);
//   } else {
//     console.log("Pokémon no encontrado");
//   }


// }






onMounted(() => {
  filtrado(); // Llamar a la función filtrado aquí
  obtenerPokemons();
});



async function obtenerurlpk(url) {

	try {
		let r = await axios.get(url);
		console.log(r);

		const tipos = ref([]);
		const debilidades = ref([])

		for (let i = 0; i < r.data.types.length; i++) {
			const tipoActual = r.data.types[i].type;
			const tipopoke = r.data.types[i].type.name.toLowerCase();

			tipos.value.push(tipoActual);


		}
		pokemons.value.push({
			imagen: r.data.sprites.other['official-artwork'].front_default,
			nombre: r.data.name,
			numero: r.data.id,
			peso:r.data.weight,
			altura:r.data.height,
			tipos: tipos,
			debilidades: debilidades,
			hp: r.data.stats[0].base_stat,
			attack: r.data.stats[1].base_stat,
			defense: r.data.stats[2].base_stat,
			special_attack: r.data.stats[3].base_stat,
			special_defense: r.data.stats[4].base_stat,
			speed: r.data.stats[5].base_stat,
		});
	} catch (error) {
		console.error("Error al obtener los datos de un Pokémon:", error);
	}

	let r = await axios.get(url)
	imagen.value = r.data.sprites.other['official-artwork'].front_default
	num.value = r.data.id
	nombre.value = r.data.name
	peso.value = r.data.weight
	altura.value = r.data.height
	tipo.value = []
	for (let i = 0; i < r.data.types.length; i++) {
		tipo.value.push(r.data.types[i].type);
	}
	for (let j = 0; j < r.data.stats.length; j++) {
		hp.value = r.data.stats[0].base_stat
		attack.value = r.data.stats[1].base_stat
		defense.value = r.data.stats[2].base_stat
		special_attack.value = r.data.stats[3].base_stat
		special_defense.value = r.data.stats[4].base_stat
		speed.value = r.data.stats[5].base_stat
	}

	console.log(r)
	console.log(imagen.value)
	console.log(num.value)
	console.log(nombre.value)
	console.log(peso.value)
	console.log(altura.value)
}

async function seleccionarPokemon(pokemon) {
  mostrardos.value = pokemon;
  mostrar.value = false; // Oculta la lista de Pokémon // Muestra los detalles del Pokémon seleccionado
  mostrartres.value = false
}



</script>

<template>
	<div>
		

		<div v-if="mostrar">
			
			<header v-if="home === 1" class="container">

				<nav class="nav">
					<a href="/" class="logo">
						<img src="./assets/Pokédex_logo.png" alt="Logo Pokedex" />
					</a>

					<div class="container-filter container">
					    <div class="icon-filter">
						    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
							   stroke="currentColor" class="icon">
							   <path stroke-linecap="round" stroke-linejoin="round"
								d="M10.5 6h9.75M10.5 6a1.5 1.5 0 11-3 0m3 0a1.5 1.5 0 10-3 0M3.75 6H7.5m3 12h9.75m-9.75 0a1.5 1.5 0 01-3 0m3 0a1.5 1.5 0 00-3 0m-3.75 0H7.5m9-6h3.75m-3.75 0a1.5 1.5 0 01-3 0m3 0a1.5 1.5 0 00-3 0m-9.75 0h9.75" />
						   </svg>
						   <span>Filtrar</span>
					    </div>
				    </div>

					<ul class="nav-list">
	                <li class="nav-item"><button class="btn btn-header" id="ver-todos">Ver todos</button></li> 
	                <li class="nav-item"><button class="btn btn-header normal" id="normal">Normal</button></li>
	                <li class="nav-item"><button class="btn btn-header fire" id="fire">Fire</button></li>
	                <li class="nav-item"><button class="btn btn-header water" id="water">Water</button></li>
	                <li class="nav-item"><button class="btn btn-header grass" id="grass">Grass</button></li>
	                <li class="nav-item"><button class="btn btn-header electric" id="electric">Electric</button></li>
	                <li class="nav-item"><button class="btn btn-header ice" id="ice">Ice</button></li>
	                <li class="nav-item"><button class="btn btn-header fighting" id="fighting">Fighting</button></li>
	                <li class="nav-item"><button class="btn btn-header poison" id="poison">Poison</button></li>
	                <li class="nav-item"><button class="btn btn-header ground" id="ground">Ground</button></li>
	                <li class="nav-item"><button class="btn btn-header flying" id="flying">Flying</button></li>
	                <li class="nav-item"><button class="btn btn-header psychic" id="psychic">Psychic</button></li>
	                <li class="nav-item"><button class="btn btn-header bug" id="bug">Bug</button></li>
	                <li class="nav-item"><button class="btn btn-header rock" id="rock">Rock</button></li>
	                <li class="nav-item"><button class="btn btn-header ghost" id="ghost">Ghost</button></li>
	                <li class="nav-item"><button class="btn btn-header dark" id="dark">Dark</button></li>
	                <li class="nav-item"><button class="btn btn-header dragon" id="dragon">Dragon</button></li>
	                <li class="nav-item"><button class="btn btn-header steel" id="steel">Steel</button></li>
	                <li class="nav-item"><button class="btn btn-header fairy" id="fairy">Fairy</button></li>
	            </ul>


					<form class="form">
						<div class="form-group">
							<svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
								stroke="currentColor" class="icon-search">
								<path stroke-linecap="round" stroke-linejoin="round"
									d="M21 21l-5.197-5.197m0 0A7.5 7.5 0 105.196 5.196a7.5 7.5 0 0010.607 10.607z" />
							</svg>
							<input type="search" placeholder="Buscar nombre de Pokemon" v-model="searchQuery" />
						</div>
					
	
						<button class="btn-search"  @click="buscarPokemon"></button>
					</form>

				</nav>
		</header>
	
			
			<!-- TARJETA DE LA PAGINA DE INICIO  -->
			<div class="card-list-pokemon container">
				<div @click="seleccionarPokemon(pokemon)" v-for="(pokemon, index) in pokemons" :key="index">
				  <div class="card-pokemon">
					<div class="card-img">
					  <img :src="pokemon.imagen" alt="" />
					</div>
					<div class="card-info">
					  <p class="nombre">{{ pokemon.nombre }}</p>
					  <span class="pokemon-id">N° {{ pokemon.numero }}</span>
			  
					  <div class="card-types">
						<span v-for="(tipo, tipoIndex) in pokemon.tipos" :key="tipoIndex" class="grass tipo">{{ tipo.name }}</span>
					  </div>
					</div>
				  </div>
				</div>
			</div>
			  
			
		</div>
		    <div class="div-cargar">

				<button class="button"  @click="cargarMasPokemones()" v-if="mostrartres">
				  <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 97 162" height="162" width="97" class="svg">
				<path fill="#262626" d="M47.2124 0H54.0796V151.644L86.6991 128.712H97L50.646 162L0 128.712H10.3009L47.2124 151.644V0Z"></path>
				</svg>

				</button>


			</div>

		
		<!-- MODAL  -->

		<div  class="info" v-if="mostrardos">
			<main class="container main-pokemon">
				<div class="header-main-pokemon">

					<h1 class="number-pokemon"> #{{ mostrardos ? mostrardos.numero : '' }} </h1>

					<div class="container-img-pokemon">
						<img :src="mostrardos ? mostrardos.imagen : '' " class="" alt="">
					</div>

					<div class="container-info-pokemon">
						<h1>{{ mostrardos ? mostrardos.nombre : '' }}</h1>

						<div class="card-types" v-for="(item, index) in tipo" :key="index">
						</div>
						<div class="info-pokemon">
							<div class="group-info">
								<p>Altura</p>
								<span>{{ mostrardos ? mostrardos.altura : '' }}</span>
							</div>
							<div class="group-info">
								<p>Peso</p>
								<span> {{ mostrardos ? mostrardos.peso : '' }} KG</span>

							</div>
						</div>
					</div>
				</div>

				<div class="container-stats">
					<h1>Estadísticas</h1>
					<div class="stats">
						<div class="stat-group">
							<span>Hp </span>
							<progress id="file" max="300" value="70">{{mostrardos ? mostrardos.hp : ''}}</progress>
							<div class="progress-bar"></div>
							<span class="counter-stat">{{ mostrardos ? mostrardos.hp : '' }}</span>
						</div>
						<div class="stat-group">
							<span>Attack</span>
							<progress id="file" max="300" value="70">{{attack}}</progress>
							<div class="progress-bar"></div>
							<span class="counter-stat">{{ mostrardos ? mostrardos.attack : '' }}</span>
						</div>
						<div class="stat-group">
							<span>Defense</span>
							<progress id="file" max="300" value="70">{{ mostrardos ? mostrardos.defense : ''}}%</progress>
							<div class="progress-bar"></div>
							<span class="counter-stat">{{ mostrardos ? mostrardos.defense : '' }}</span>
						</div>
						<div class="stat-group">
							<span>Special Attack</span>
							<progress id="file" max="300" value="70">{{}}%</progress>
							<div class="progress-bar"></div>
							<span class="counter-stat">{{ mostrardos ? mostrardos.special_attack : '' }}</span>
						</div>
						<div class="stat-group">
							<span>Special Defense</span>
							<progress id="file" max="300" value="70">{{}}%</progress>
							<div class="progress-bar"></div>
							<span class="counter-stat">{{ mostrardos ? mostrardos.special_defense : '' }}</span>
						</div>
						<div class="stat-group">
							<span>Speed</span>
							<progress id="file" max="300" value="70">{{}}%</progress>
							<div class="progress-bar"></div>
							<span class="counter-stat">{{ mostrardos ? mostrardos.speed : '' }}</span>
						</div>
					</div>
				</div>
			</main>
			<div class="btn-b">
	 <!-- <button class="btn-back" >Volver a la Lista</button> -->
	  <button class="btn-back"  @click="volverALista">
        <svg height="16" width="16" xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 1024 1024"><path d="M874.690416 495.52477c0 11.2973-9.168824 20.466124-20.466124 20.466124l-604.773963 0 188.083679 188.083679c7.992021 7.992021 7.992021 20.947078 0 28.939099-4.001127 3.990894-9.240455 5.996574-14.46955 5.996574-5.239328 0-10.478655-1.995447-14.479783-5.996574l-223.00912-223.00912c-3.837398-3.837398-5.996574-9.046027-5.996574-14.46955 0-5.433756 2.159176-10.632151 5.996574-14.46955l223.019353-223.029586c7.992021-7.992021 20.957311-7.992021 28.949332 0 7.992021 8.002254 7.992021 20.957311 0 28.949332l-188.073446 188.073446 604.753497 0C865.521592 475.058646 874.690416 484.217237 874.690416 495.52477z"></path></svg>
         <span>Back</span>
      </button>
    </div>
			
		</div>
	</div>
	
</template>

<style scoped>

@import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300;500;700&display=swap');

.container .form{
	
	margin-top: 50px;
}

* {
	margin: 0;
	/* padding: 0; */
	box-sizing: border-box;
}

body {
	font-family: 'Montserrat';
}

/* Globales */
.container {
	max-width: 1200px;
	margin: 0 auto;
	
}

/* header */

header {
	display: flex;
	align-items: center;
	justify-content: space-between;

	padding: 40px 0;

}

nav{
	align-items: center;
	justify-content: center;
	display: flex;
}
 .logo img {
	width: 300px;
	
}

header form {
	display: flex;
	align-items: center;

	gap: 15px;
}

.form-group {
	display: flex;
	align-items: center;
	gap: 10px;

	border: 1px solid #555;
	padding: 15px 20px;
	border-radius: 25px;
}

.form-group input {
	font-family: inherit;
	width: 300px;
	border: none;
	outline: none;
	font-size: 15px;
}


.icon-search {
	width: 20px;
	height: 20px;
	stroke: #555;
}

.btn-search {
	border: none;
	outline: none;
	border-radius: 25px;
	font-family: inherit;
	padding: 15px 30px;
	cursor: pointer;
	letter-spacing: 2px;
	background-color: var(--color-primary);
	color: #fff;
}

.btn-search:hover {
	background-color: var(--color-primary-hover);
	color: #000;
}

/* boton back */
.btn-b{
	
	align-items: center;
	justify-content: center;
	display: flex;
}

.btn-back {
 display: flex;
 height: 3em;
 width: 100px;
 align-items: center;
 justify-content: center;
 background-color: #eeeeee4b;
 border-radius: 3px;
 letter-spacing: 1px;
 transition: all 0.2s linear;
 cursor: pointer;
 border: none;
 background: #4747476b;
}

.btn-back > svg {
 margin-right: 5px;
 margin-left: 5px;
 font-size: 20px;
 transition: all 0.4s ease-in;
}

.btn-back:hover > svg {
 font-size: 1.2em;
 transform: translateX(-5px);
}

.btn-back:hover {
 box-shadow: 9px 9px 33px #d1d1d1, -9px -9px 33px #ffffff;
 transform: translateY(-2px);
}


/* Btn Filter */
.container-filter {
	display: flex;
}

.icon-filter {
	display: flex;
	align-items: center;
	gap: 15px;

	margin-bottom: 20px;
	cursor: pointer;
}

.icon-filter .icon {
	width: 30px;
	height: 30px;
	stroke: #555;
}



/* Card List Pokemon */




.card-list-pokemon {
	display: grid;
	grid-template-columns: repeat(4, 1fr);
	gap: 20px;
	
}

.card-pokemon {
	cursor: pointer;
	text-decoration: none;
}

.card-img {
	background-color: #f2f2f2;

	display: flex;
	align-items: center;
	justify-content: center;
	border-radius: 5px;
	height: 250px;
}

.card-img img {
	width: 100%;
	height: 100%;
}

.card-info {
	padding: 15px;
}

.card-info h3 {
	color: #333;
}

.pokemon-id {
	display: block;
	color: #888;
	margin-bottom: 15px;
}

.card-types {
	align-items: center;
	justify-content: center;
	display: flex;
	gap: 10px;
	margin-top: 10px;
}

.card-types span {
	font-size: 12px;
	padding: 5px 20px;
	border-radius: 5px;
	color: #fff;
}

/*boton cargar mass*/

.button {
  width: 38px;
  height: 82px;
  padding: 2px;
  border: 1px solid #474747;
  border-radius: 50px;
  background: rgba(141, 141, 141, 0.18);
  transition: all 0.3s ease-in-out;
  transform: rotate(180deg);
}

.svg {
  width: 22px;
  height: 62px;
  transform: rotate(180deg);
}

.button:hover {
  transform: scale(-1.2);
}

.button:focus {
  height: 0px;
  width: 0px;
  padding: 0px;
  border: 0px;
}

.button:focus > .svg {
  display: none;
  height: 0px;
}
  
  
  
  




.div-cargar{
	height: 200px;
	align-items: center;
	justify-content: center;
	display: flex;
}





/* Container Filters */
.container-filters {
	display: flex;
	justify-content: center;

	position: fixed;
	top: 0;
	left: -300px;
	width: 250px;

	color: #555;
	background-color: #f2f2f2;
	height: 100%;
	padding-top: 140px;
	transition: all .3s linear;

}

.container-filters.active {
	left: 0;
}

.filter-by-type {
	display: flex;
	flex-direction: column;

	gap: 20px;
}

.filter-by-type span {
	font-weight: 700;
	font-size: 22px;
}

.group-type {
	display: flex;
	gap: 10px;

	margin-left: 15px;
}

.group-type label {
	cursor: pointer;
}

/* 
======================================= 
Styles of Pokemon Single Page
======================================= 
*/

.main-pokemon {
	display: flex;
	flex-direction: column;
	margin-bottom: 50px;
	
	
}

.header-main-pokemon {
	margin-top: 20px;

	display: flex;
	align-items: center;

	position: relative;
}

.number-pokemon {
	position: absolute;
	top: -70px;
	left: 0;

	font-size: 300px;
	font-weight: 700;
	color: var(--color-primary-hover);
}

.container-img-pokemon {
	order: 2;
	flex: 1;
	padding: 25px 25px 25px 0;
}

.container-img-pokemon img {
	width: 100%;
}

.container-info-pokemon {
	display: flex;
	flex-direction: column;

	order: 1;
	flex: 1;
}

.container-info-pokemon h1 {
	text-align: right;
	font-size: 50px;


}

.info-pokemon {
	display: flex;
	justify-content: space-between;

	margin-top: 20px;
}

.group-info p {
	font-weight: 700;
}

/* Stats */
.container-stats {
	display: flex;
	gap: 50px;
	align-items: center;
}

.stats {
	display: flex;
	flex-direction: column;
	gap: 20px;
	flex: 1;
}

.stat-group {
	display: flex;
	align-items: center;
	gap: 20px;
}

.stat-group span {
	flex: 20%;
	font-weight: 500;
	font-size: 18px;
}

</style>

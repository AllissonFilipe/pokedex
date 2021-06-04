<template>
	<div class="pokemon">
		<ListItem v-bind="mainInfo" />
		<ul class="stats">
			<li v-for="(stat, index) in stats" :key="index">
				{{ parseStatName(stat.stat.name) }}: {{ stat.base_stat }}
			</li>
		</ul>
	</div>
</template>

<script>
	import ListItem from '@/components/List/ListItem.vue';
	import { state } from '@/store';
	import { parsePokemonInfo } from '@/utils';
	const statsNames = {
		hp: 'HP',
		attack: 'Attack',
		defense: 'Defense',
		speed: 'Speed',
		'special-attack': 'Sp. Atk',
		'special-defense': 'Sp. Def',
	};
	export default {
		name: 'PokemonDescription',
		components: {
			ListItem,
		},
		props: {
			id: {
				type: Number,
				required: true,
			},
		},
		data() {
			return {
				mainInfo: null,
				stats: [],
			};
		},
		created() {
			const pokemonInfo = state.list.find(pokemon => pokemon.id === this.id);
			if (pokemonInfo) {
				const infoParsed = parsePokemonInfo(pokemonInfo);
				const { stats, ...rest } = infoParsed;
				this.mainInfo = rest;
				this.stats = stats;
				this.speak();
			}
		},
		methods: {
			parseStatName(name) {
				return statsNames[name] || name;
			},
			speak() {
				let synth = window.speechSynthesis;
				let voices = synth.getVoices().sort(function (a, b) {
					const aname = a.name.toUpperCase(), bname = b.name.toUpperCase();
					if ( aname < bname ) return -1;
					else if ( aname == bname ) return 0;
					else return +1;
				});
				if (synth.speaking) {
					console.error('speechSynthesis.speaking');
					return;
				}
				const text = this.mainInfo.types.length == 2 ? 
				`This is pokemon ${this.mainInfo.name}, it is a ${this.mainInfo.types[0]} and ${this.mainInfo.types[1]} type.`:
				`This is pokemon ${this.mainInfo.name}, it is a ${this.mainInfo.types[0]} type.`;
				var utterThis = new SpeechSynthesisUtterance(text);
				utterThis.onend = function (event) {
					console.log('SpeechSynthesisUtterance.onend');
				}
				utterThis.onerror = function (event) {
					console.error('SpeechSynthesisUtterance.onerror');
				}
				const voice = "Google US English"
				for(let i = 0; i < voices.length ; i++) {
					if(voices[i].name === voice) {
						utterThis.voice = voices[i];
						break;
					}
				}
				utterThis.pitch = '1';
				utterThis.rate = '0.9';
				synth.speak(utterThis);
			}
		},
	};
</script>

<style lang="scss" scoped>
	.stats {
		padding: 0 24px;
	}
</style>
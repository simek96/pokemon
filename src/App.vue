<template>
  <v-app>
    <v-main class="pa-6 gradient-bg">
      <v-container>
        <v-card class="pa-4 rounded-xl glass-card" elevation="10">
          <v-row class="mb-4" align="center">
            <v-col cols="12" sm="8">
              <v-autocomplete
                hide-details
                v-model="selectedPokemon"
                :items="pokemonList"
                item-text="name"
                item-value="name"
                label="Izberi Pokemona"
                prepend-inner-icon="mdi-pokeball"
                outlined
                dense
                clearable
                color="purple lighten-2"
                :loading="loadingList"
                @change="fetchPokemonData"
              />
            </v-col>
            <v-col cols="12" sm="4">
              <v-responsive class="d-flex align-center">
                <v-btn
                  color="purple lighten-2"
                  dark
                  block
                  class="glow-btn"
                  @click="selectRandomPokemon"
                >
                  <v-icon small class="mr-1">mdi-clover</v-icon>
                  Naključni Pokemon
                </v-btn>
              </v-responsive>
            </v-col>
          </v-row>

          <v-skeleton-loader v-if="loadingPokemon" type="image" />

          <div v-else-if="pokemon">
            <v-row>
              <v-col cols="12" md="4" class="text-center">
                <v-img
                  :src="currentSprite"
                  contain
                  height="200"
                  class="rounded-lg elevation-5 mb-2 position-relative"
                >
                  <div class="overlay-info top-left">
                    <v-icon class="mr-1">mdi-arrow-up-down</v-icon>
                    {{ (pokemon.height / 10).toFixed(1) }} m
                  </div>

                  <div class="overlay-info bottom-left">
                    <v-icon class="mr-1">mdi-weight-kilogram</v-icon>
                    {{ (pokemon.weight / 10).toFixed(1) }} kg
                  </div>
                </v-img>
              </v-col>
              <v-row class="mb-2 d-flex d-md-none" justify="center">
                <v-col
                  v-for="(src, index) in availableSpriteList"
                  :key="index"
                  class="text-center"
                >
                </v-col>
              </v-row>
              <v-col cols="12" md="8" class="d-flex flex-column justify-start fill-height">
                <h2 class="text-h4 font-weight-bold mb-2 text-colored-title">
                  <span v-for="(char, i) in pokemon.name.toUpperCase().split('')" :key="i"
                        :style="{ color: dominantColors[i % dominantColors.length] }">
                    {{ char }}
                  </span>
                  <div style="display:inline-block">
                    <v-chip color="light-blue lighten-4" class="ma-1">Height: {{ pokemon.height }}</v-chip>
                    <v-chip color="green lighten-3" class="ma-1">Weight: {{ pokemon.weight }}</v-chip>
                    <v-chip v-for="(type, i) in pokemon.types" :key="i" color="amber lighten-4" class="ma-1 text-capitalize">
                      {{ type.type.name }}
                    </v-chip>
                  </div>
                </h2>
                <v-list outlined dense class="rounded-lg pa-2 white--text" style="background-color: transparent; color: black; min-height: 150px;">
                  <v-list-item v-for="(ability, index) in abilities" :key="index">
                    <v-list-item-content>
                      <v-list-item-title class="ability-line">
                        <strong>{{ ability.name }}:</strong>
                        <span
                          class="ability-text"
                          :class="{ truncated: !ability.expanded }"
                          @click="toggleAbility(index)"
                        >
                          {{ ability.effect || 'Nalaganje...' }}
                        </span>
                      </v-list-item-title>
                    </v-list-item-content>
                  </v-list-item>
                </v-list>
              </v-col>
            </v-row>
            <v-row>
              <div style="display: flex; align-items: center;" class="ml-1">
                <v-btn width="58" height="58" icon @click="previousSprite">
                  <v-icon>mdi-chevron-left</v-icon>
                </v-btn>
                <v-btn width="58" height="58" icon @click="nextSprite">
                  <v-icon>mdi-chevron-right</v-icon>
                </v-btn>
              </div>
              <div>
                <span v-for="(src, index) in availableSpriteList" :key="'bottom-' + index" style="display: inline-flex;" >
                <v-img
                  :src="src.url"
                  max-height="130"
                  contain
                  class="rounded-lg position-relative"
                  @click="selectSprite(index)"
                />
              </span>
            </div>
            
            </v-row>
            <v-divider class="my-4"></v-divider>
            <h3 class="text-h6 font-weight-bold mb-2 text-white">Prilagodi Pokemona</h3>
              <v-form ref="customForm" @submit.prevent="saveCustomPokemon">
                <v-row class="mobile-flex">
                  <v-col md="4" class="no-padding-bottom">
                    <v-hover v-slot="{ hover }">
                      <div
                        class="dropzone"
                        :class="{ 'dropzone-hover': isDragging || hover }"
                        @dragover.prevent="isDragging = true"
                        @dragleave.prevent="isDragging = false"
                        @drop.prevent="handleDrop"
                      >
                        <label class="file-label">
                          <div v-if="!imagePreview" class="text-center">
                            <v-icon x-large color="deep-purple lighten-2">mdi-upload</v-icon>
                            <p>Naloži ali povleci sliko pokemona v polje</p>
                          </div>

                          <div v-else class="preview-wrapper">
                            <label class="file-label">
                              <v-img
                                :src="imagePreview"
                                max-height="100"
                                contain
                                class="rounded-lg elevation-2"
                              />
                              <v-btn
                                class="change-btn"
                                small
                                color="purple lighten-2 white--text"
                                @click.stop="$refs.imageInput.click()"
                              >
                                <v-icon small class="mr-1">mdi-image-edit</v-icon>
                                Zamenjaj
                              </v-btn>

                              <input
                                ref="imageInput"
                                type="file"
                                accept="image/png, image/jpeg, image/jpg, image/svg+xml"
                                @change="handleFileChange"
                                class="file-input"
                              />
                            </label>
                            <v-btn
                              class="remove-btn deep-purple accent-4 white--text"
                              icon
                              @click.stop="removeImage"
                            >
                              <v-icon small>mdi-delete</v-icon>
                            </v-btn>
                          </div>

                          <input
                            ref="imageInput"
                            type="file"
                            accept="image/png, image/jpeg, image/jpg, image/svg+xml"
                            @change="handleFileChange"
                            class="file-input"
                          />
                        </label>
                      </div>
                    </v-hover>
                     <v-btn width="100%" :loading="saving" color="deep-purple accent-4" dark type="submit" class="mt-4 glow-btn mb-4" elevation="4">
                      <v-icon small class="mr-1">mdi-content-save-all</v-icon>
                       Shrani prilagoditve
                      </v-btn>
                    </v-col>
                    <v-col>
                      <v-text-field
                        v-model="customData.name"
                        label="Ime"
                        :rules="[rules.required, rules.alphaOnly, rules.max50]"
                        outlined
                        dense
                        class="deep-purple--text"
                        prepend-inner-icon="mdi-pencil"
                        required
                      />

                      <v-textarea
                        v-model="customData.description"
                        label="Opis"
                        prepend-inner-icon="mdi-pencil"
                        :rules="[rules.required, rules.max300]"
                        outlined
                        dense
                        color="indigo lighten-1"
                        required
                      />
                  </v-col>
              </v-row>
            </v-form>
          </div>
        </v-card>
      </v-container>
    </v-main>
    <v-snackbar
      v-model="snackbar.visible"
      :color="snackbar.color"
      timeout="4000"
      top
      right
      elevation="4"
    >
      {{ snackbar.text }}
      <template #action>
        <v-btn text @click="snackbar.visible = false">Zapri</v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script>

import ColorThief from 'colorthief'

export default {
  data () {
    return {
      pokemonList: [],
      selectedPokemon: null,
      pokemon: null,
      abilities: [],
      loadingList: false,
      loadingPokemon: false,
      saving: false,
      currentSpriteIndex: 0,
      customData: {
        name: '',
        description: '',
        imageFile: null,
        imageUrl: ''
      },
      rules: {
        required: v => !!v || 'To polje je obvezno.',
        alphaOnly: v => /^[A-Za-zČčŠšŽž\s]+$/.test(v) || 'Dovoljene so le črke.',
        max50: v => v.length <= 50 || 'Največ 50 znakov.',
        max300: v => v.length <= 300 || 'Največ 300 znakov.',
        imageRequired: v => !!v || 'Izberi sliko'
      },
      isDragging: false,
      imagePreview: '',
      snackbar: {
      visible: false,
      text: '',
      color: 'success',
      dominantColors: []
      },
    };
  },
  computed: {
availableSpriteList() {
      if (!this.pokemon?.sprites) return [];
      const sprites = this.pokemon.sprites;
      const list = [
        sprites.front_default,
        sprites.back_default,
        sprites.front_shiny,
        sprites.back_shiny
      ].filter(Boolean).map(url => ({ url, custom: false }));
      if (
        this.customData.imageUrl &&
        this.selectedPokemon &&
        localStorage.getItem(`pokemon_custom_${this.selectedPokemon}`)
      ) {
        const saved = JSON.parse(localStorage.getItem(`pokemon_custom_${this.selectedPokemon}`));
        if (saved?.imageUrl === this.customData.imageUrl) {
          list.push({ url: this.customData.imageUrl, custom: true });
        }
      }

      return list;
    },
    currentSprite() {
      return this.availableSpriteList[this.currentSpriteIndex]?.url || '';
    }
  },
  methods: {
    removeImage() {
      this.imagePreview = '';
      this.customData.imageFile = null;
      this.customData.imageUrl = '';
      if (this.$refs.imageInput) {
        this.$refs.imageInput.value = '';
      }
    },
     toggleAbility(index) {
      this.abilities[index].expanded = !this.abilities[index].expanded;
  },
  async extractColorsFromImage(url) {
    const img = new Image();
    img.crossOrigin = 'Anonymous';
    img.src = url;

    return new Promise((resolve, reject) => {
      img.onload = () => {
        try {
          const colorThief = new ColorThief();
          const palette = colorThief.getPalette(img, this.pokemon.name.length);
          resolve(palette);
        } catch (e) {
          reject(e);
        }
      };
      img.onerror = reject;
    });
  },
   handleFileChange(event) {
    const file = event.target.files[0];
    if (file && this.rules.imageRequired(file) === true) {
      this.customData.imageFile = file;
      this.imagePreview = URL.createObjectURL(file);
    }
  },
  handleDrop(event) {
    this.isDragging = false;
    const file = event.dataTransfer.files[0];
    if (file && this.rules.imageRequired(file) === true) {
      this.customData.imageFile = file;
      this.imagePreview = URL.createObjectURL(file);
    }
  },
    previousSprite() {
      if (!this.availableSpriteList.length) return;
      this.currentSpriteIndex = (this.currentSpriteIndex - 1 + this.availableSpriteList.length) % this.availableSpriteList.length;
    },
    nextSprite() {
      if (!this.availableSpriteList.length) return;
      this.currentSpriteIndex = (this.currentSpriteIndex + 1) % this.availableSpriteList.length;
    },
    selectSprite(index) {
      this.currentSpriteIndex = index;
    },
  async fetchPokemonData(name) {
    const savedData = localStorage.getItem(`pokemon_custom_${name}`);
    if (savedData) {
      const parsed = JSON.parse(savedData);
      this.customData = {
        ...parsed,
        imageFile: null
      };
      this.imagePreview = parsed.imageUrl || '';
    } else {
      this.customData = {
        name: '',
        description: '',
        imageFile: null,
        imageUrl: ''
      };
      this.imagePreview = '';
    }
    if (!name) return;
    this.loadingPokemon = true;
    this.pokemon = null;
    this.abilities = [];
    try {
      const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${name}`);
      const data = await response.json();
      this.pokemon = data;
      this.currentSpriteIndex = 0;
      this.loadAbilities(data.abilities);
      const spriteUrl = data.sprites.front_default;
      if (spriteUrl) {
        const colors = await this.extractColorsFromImage(spriteUrl);
        this.dominantColors = colors.map(rgb => `rgb(${rgb.join(',')})`);
      }
    } catch (error) {
      console.error('Napaka pri nalaganju Pokemona:', error);
    } finally {
      this.loadingPokemon = false;
    }
  }, 
    async loadAbilities(abilities) {
      this.abilities = abilities.map(a => ({
      name: a.ability.name,
      effect: null,
      expanded: false
    }))
      for (let i = 0; i < abilities.length; i++) {
        try {
          const response = await fetch(abilities[i].ability.url);
          const data = await response.json();
          const effectEntry = data.effect_entries.find(e => e.language.name === 'en');
          this.abilities[i].effect = effectEntry ? effectEntry.effect : 'Ni podatka';
        } catch (error) {
          this.abilities[i].effect = 'Napaka pri nalaganju opisa';
        }
      }
    },
    async fetchPokemonList() {
      this.loadingList = true;
      try {
        const response = await fetch('https://pokeapi.co/api/v2/pokemon?limit=1000');
        const data = await response.json();
        this.pokemonList = data.results;
      } catch (error) {
        console.error('Napaka pri nalaganju seznama Pokemonov:', error);
      } finally {
        this.loadingList = false;
      }
    },
    selectRandomPokemon() {
      const random = this.pokemonList[Math.floor(Math.random() * this.pokemonList.length)];
      this.selectedPokemon = random.name;
      this.fetchPokemonData(random.name);
    },
    async saveCustomPokemon() {
      if (!this.$refs.customForm.validate()) return;
      if (!this.customData.imageFile && !this.customData.imageUrl) {
      this.snackbar.text = 'Najprej naloži sliko!';
      this.snackbar.color = 'error';
      this.snackbar.visible = true;
      return;
    }
      this.saving = true;
      this.loadingList = true;
      try {
        if (this.customData.imageFile && typeof this.customData.imageFile !== 'string') {
          const formData = new FormData();
          formData.append('image', this.customData.imageFile);
          formData.append('key', '7f9f9eb189252bbf327ae50e51193821');

          const response = await fetch('https://api.imgbb.com/1/upload', {
            method: 'POST',
            body: formData
          });
          const result = await response.json();
          if (result.data?.url) {
            this.customData.imageUrl = result.data.url;
          }
        }

        const dataToStore = { ...this.customData };
        delete dataToStore.imageFile;

        localStorage.setItem(`pokemon_custom_${this.selectedPokemon}`, JSON.stringify(dataToStore));
        
        this.snackbar.text = 'Pokemon uspešno shranjen';
        this.snackbar.color = 'success';
        this.snackbar.visible = true;
      } catch (error) {
        this.snackbar.text = 'Napaka pri shranjevanju Pokémona.';
        this.snackbar.color = 'error';
        this.snackbar.visible = true;
      } finally {
        this.saving = false;
        this.loadingList = false;
      }
    }
  },
  created() {
    this.fetchPokemonList();
  }
};
</script>

<style scoped>
.v-list-item__title {
  white-space: unset;
}

body {
  font-family: 'Inter', sans-serif;
  background-color: #1e1e2f;
  color: #ffffff;
}

.gradient-bg {
  background: linear-gradient(to right, #1e1e2f, #3e3b92);
  min-height: 100vh;
}

.glass-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(12px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.3);
  color: white;
}

.glow-btn {
  transition: box-shadow 0.3s ease;
  box-shadow: 0 0 12px rgba(255, 255, 255, 0.4);
}

.glow-btn:hover {
  box-shadow: 0 0 18px rgba(255, 255, 255, 0.7);
}

.custom-indicator {
  position: absolute;
  top: 4px;
  left: 4px;
  background-color: rgba(103, 58, 183, 0.8);
  color: white;
  font-size: 10px;
  padding: 2px 4px;
  border-radius: 4px;
  z-index: 10;
}

.dropzone {
  height: 100%;
  max-height: 160px;
  border: 2px dashed #9575cd;
  padding: 30px;
  border-radius: 12px;
  background-color: rgba(255, 255, 255, 0.05);
  position: relative;
  cursor: pointer;
  transition: 0.3s ease;
  text-align: center;
  color: white;
}
.dropzone-hover {
  background-color: rgba(255, 255, 255, 0.1);
  border-color: #7e57c2;
}
.file-input {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  cursor: pointer;
}

.file-label {
  display: block;
  width: 100%;
  height: 100%;
  text-align: center;
  cursor: pointer;
}

.file-input {
  display: none;
}

.preview-wrapper {
  position: relative;
  display: inline-block;
  width: 100%;
  text-align: center;
}

.change-btn {
    position: absolute;
    right: 0;
    top: 0px;
    z-index: 2;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
}

.file-label {
  display: block;
  width: 100%;
  height: 100%;
  cursor: pointer;
}

.file-input {
  display: none;
}

.text-colored-title span {
  transition: color 0.5s ease;
}

.ability-line {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.ability-text {
  display: inline-block;
  max-height: 2.4em; /* approx 2 lines */
  overflow: hidden;
  position: relative;
  cursor: pointer;
  transition: max-height 0.3s ease;
  line-height: 1.2em;
}

.ability-text.truncated {
  white-space: nowrap;
  text-overflow: ellipsis;
  overflow: hidden;
  max-width: 100%;
}

.dots {
  color: #888;
  padding-left: 4px;
  font-weight: bold;
}

.remove-btn {
  position: absolute;
    top: -17px;
    left: -17px;
    z-index: 3;
}

.theme--light.v-label {
  color: white;
}

.ruler-container {
  position: relative;
  height: 200px; /* Match image height */
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  padding: 4px;
}

.ruler-bar {
  width: 4px;
  background-color: #ccc;
  flex-grow: 1;
  border-radius: 2px;
}

.ruler-label {
  margin-top: 4px;
  font-size: 12px;
  color: white;
  display: flex;
  align-items: center;
  white-space: nowrap;
}

.overlay-info {
  position: absolute;
  color: white;
  margin: 20px;
  padding: 2px 6px;
  border-radius: 8px;
  font-size: 12px;
  display: flex;
  align-items: center;
}

.bottom-left {
  bottom: 6px;
  left: 6px;
}

/deep/ .theme--light.v-label {
    color: white;
}

@media only screen and (max-width: 600px) {
  .no-padding-bottom {
    padding-top: 0;
  }
  .mobile-flex {
    flex-direction: column;
    flex-flow: column-reverse;
  }
}
</style>

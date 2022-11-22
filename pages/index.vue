<template>
  <div class="flex justify-center bg-slate-700">
    <div v-if="!protocol" class="mt-10 w-1/2">
      <div class="my-20">
        <h1 class="text-4xl text-white mb-6">URL tool</h1>
        <p class="text-gray-300">Décomposez votre URL pour changer ses paramètres, visualiser une url complexe ou modifier son chemin</p>
        <p class="text-gray-300">Deployed with love on kubernetes !</p>
      </div>
      <h1 class="text-2xl text-white mb-6">Entrez votre url ici</h1>
      <input v-model="full" placeholder="https://github.com/Neltarim" class="w-full px-2 h-8 bg-slate-800 text-white border border-slate-500 rounded" type="text">
      <button @click="decompose()" class="py-2 px-5 my-3 bg-violet-700 border-1 border-violet-900 hover:bg-violet-900 text-white rounded">
        DECOMPOSER
      </button>
    </div>
    <div v-else class="mt-10 flex justify-center w-2/3">

      <div>
        <h1 class="text-xl my-2 text-white">CHEMIN DE L'URL :</h1>
        <div class="flex text-2xl text-white">
          <input v-model="protocol" :style="{ width: protocol.length + 'rem' }" class="text-center bg-slate-600 border-2 border-slate-500 py-1 px-4 rounded">
          <div class="w-2"></div>
          <input v-model="domain" :style="{ width: domain.length + 'rem' }" class="text-center bg-slate-600 border-2 border-slate-500 py-1 px-4 rounded">
          
          <button @click="addSectionAfter(-1)" class="w-3 h-8 my-auto text-center text-slate-700 hover:bg-green-400 hover:w-6 hover:mx-2 hover:p6 hover:text-white rounded">
            +
          </button>

          <div
            v-for="(section, i) in sections" 
            :key="section"
            class="flex"
          >
            <input 
              v-model="sections[i].value"
              :style="{ width: sections[i].value.length + 1 * 4 + 'ch' }"
              class="text-center bg-slate-600 border-2 border-slate-500 py-1 px-4 rounded"
            >
            <button @click="addSectionAfter(i)" class="w-3 h-8 my-auto text-center text-slate-700 hover:bg-green-400 hover:w-6 hover:mx-2 hover:p6 hover:text-white rounded">
              +
            </button>
          </div>

        </div>

        <h1 v-if="params.length" class="mt-20 text-xl my-2 text-white">PARAMETRES :</h1>
        <div class="flex-inline">
          <div v-for="(param, i) in params" :key="param" class="flex gap-3">
            <input v-model="params[i][0]" type="text" class="my-2 h-10 text-white text-center bg-slate-500 border-2 border-slate-500 py-1 px-4 rounded">
            <textarea v-model="params[i][1]" :rows="getRows(params[i][1])" cols="40" type="textarea" class="my-2 text-white text-center bg-slate-500 border-2 border-slate-500 py-1 px-4 rounded"></textarea>
            <button @click="removeParam(i)" class="text-white bg-red-500 hover:bg-red-800 my-auto py-2 px-4 rounded">
              supprimer
            </button>
          </div>

            <h1 class="text-white mt-8">Ajouter un parametre :</h1>
          <div class="flex">
            <div class="flex gap-3">
              <input v-model="newParam[0]" type="text" placeholder="clef" class="my-2 h-10 text-white text-center bg-slate-500 border-2 border-slate-500 py-1 px-4 rounded">
              <textarea v-model="newParam[1]" placeholder="valeur" :rows="getRows(newParam[1])" cols="40" type="textarea" class="my-2 text-white text-center bg-slate-500 border-2 border-slate-500 py-1 px-4 rounded"></textarea>            
            </div>
            <button
              @click="addParam"
              v-if="newParam[0] && newParam[1]" 
              class="flex ml-2 my-auto p-4 border-1 bg-green-600 border-green-900 hover:bg-green-700 text-white rounded justify-end"
            >
              Ajouter
            </button>            
          </div>

          <div class="flex gap-4 mt-10">
            <button @click="reset()" class="py-2 px-5 my-3 bg-red-700 border-1 border-red-900 hover:bg-red-800 text-white rounded">
              Changer L'url de depart
            </button>
            <button @click="recompose()" class="py-2 px-5 my-3 bg-green-600 border-1 border-green-900 hover:bg-green-700 text-white rounded">
              Recomposer
            </button>        
          </div>

          <div class="mt-10 w-3/4">
            <p v-if="result" ref="resInput" @click="copyResult" class="bg-slate-500 text-white text-2xl p-4 cursor-pointer rounded">
              {{ result }}
            </p>
          </div>
          <p v-if="copied" class="bg-green-600 w-20 text-center mt-2 rounded text-white">
            copied
          </p>
        
        </div>
      </div>
    </div>
  </div>
</template>

<script>
const insert = (arr, index, newItem) => [
  ...arr.slice(0, index),
  newItem,
  ...arr.slice(index)
]

const initialState = () => {
  return {
      full: '',
      protocol: '',
      domain: '',
      sections: [],
      params: [],
      result: '',
      copied: false,
      newParam: ['', ''],
    }
}

export default {
  data() {
    return initialState()
  },
  methods: {
    decompose() {
      const protocolSep = []
      let startParamsSep = null
      
      for (let i=0; i < this.full.length; i++) {
        if (this.full[i] === '/' && protocolSep.length <= 1) 
          protocolSep.push(i + 1)
        if (this.full[i] === '?' && !startParamsSep) startParamsSep = i + 1
      }
      this.protocol = this.full.slice(0, protocolSep[1])
      this.sections = this.full
        .slice(protocolSep[1], startParamsSep ? startParamsSep - 1 : this.full.length)
        .split('/')

      this.domain = this.sections[0]
      this.sections.shift()
      this.extractSections(this.sections)
      if (startParamsSep) {
        const params = this.full.slice(startParamsSep, this.full.length).split('&')
        params.forEach(e => { this.params.push(e.split('=')) })
      }
    },
    recompose() {
      this.result = this.protocol + this.domain
      this.sections.forEach(e => { if (e.value) this.result += '/' + e.value })
      if (this.params.length)
        this.result += '?' + this.params.map(e => `${e[0]}=${e[1]}`).join('&')
    },
    extractSections(sections) {
      this.sections = sections.map((s, i) => {
        return { index: i, value: s }
      })
    },
    addSectionAfter(i) {
      let sections = Object.values(this.sections).map(e => e.value)
      sections = (insert(sections, i + 1, ''))
      this.extractSections(sections)
    },
    addParam() {
      this.params.push(this.newParam)
      this.newParam = ['', '']
    },
    removeParam(index) {
      this.params = this.params.filter((e,i) => i != index)
    },
    getRows(s) {
      return Math.floor(Math.ceil(s.length ? s.length / 46 : 1))
    },
    reset() {
      Object.assign(this.$data, initialState());
    },
    async copyResult() {
      await navigator.clipboard.writeText(this.result)
      this.copied = true
      var self = this
      setTimeout(function() { self.copied = false }, 2000);
    },
  },
}
</script>
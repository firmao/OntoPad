<template>
  <div class="input-group">
    <div v-if="type === undefined" class="input-group-prepend" id="nodetype-selection">
      <button @click="setType('iri')" class="btn btn-outline-secondary" v-bind:class="{'active': nodeType == 'iri'}" type="button">IRI</button>
      <button @click="setType('literal')" class="btn btn-outline-secondary" v-bind:class="{'active': nodeType == 'literal'}" type="button">Lit</button>
    </div>
    <div v-if="nodeType === 'iri'" class="input-group-prepend">
      <span class="input-group-text">&lt;</span>
    </div>
    <div v-else-if="nodeType === 'literal'" class="input-group-prepend">
      <span class="input-group-text">&quot;</span>
    </div>
    <input type="text" class="bg-gray-300 px-4 py-2" v-model="prefix" @input="filterprefixs"
           @focus="modal = true">

    <div v-if="filteredprefixs && modal">
      <ul class="w-48 bg-gray-800 text-black">
        <li v-for="filteredprefix in this.filteredprefixs"
            class="py-2 border-b cursor-pointer" :key="filteredprefix"
            @click="setprefix(filteredprefix)">{{ filteredprefix[0] }} : {{ filteredprefix[1] }}</li>
      </ul>
    </div>
    <div v-if="nodeType === 'iri'" class="input-group-append">
      <span class="input-group-text">&gt;</span>
    </div>
    <div v-else-if="nodeType === 'literal' && literalType === 'language'" class="input-group-append">
      <span @click="setLiteralType('datatype')" class="input-group-text btn">&quot;@</span>
      <input type="text" class="bg-gray-300 px-4 py-2" v-model="prefix" @input="filterprefixs"
             @focus="modal = true">

      <div v-if="filteredprefixs && modal">
        <ul class="w-48 bg-gray-800 text-black">
          <li v-for="filteredprefix in this.filteredprefixs"
              class="py-2 border-b cursor-pointer" :key="filteredprefix"
              @click="setprefix(filteredprefix)">{{ filteredprefix[0] }} : {{ filteredprefix[1] }}</li>
        </ul>
      </div>
    </div>
    <div v-else-if="nodeType === 'literal' && literalType === 'datatype'" class="input-group-append about flex flex-col items-center">
      <span @click="setLiteralType('language')" class="input-group-text btn">&quot;^^&lt;</span>
      <input type="text" class="bg-gray-300 px-4 py-2" v-model="prefix" @input="filterprefixs"
             @focus="modal = true">

      <div v-if="filteredprefixs && modal">
        <ul class="w-48 bg-gray-800 text-black">
          <li v-for="filteredprefix in this.filteredprefixs"
              class="py-2 border-b cursor-pointer" :key="filteredprefix"
              @click="setprefix(filteredprefix)">{{ filteredprefix[0] }} : {{ filteredprefix[1] }}</li>
        </ul>
      </div>
      <span class="input-group-text">&gt;</span>
    </div>
  </div>
</template>

<script>
import { DataFactory } from 'n3'
import $ from 'jquery'
const prefixesMap = new Map()

export default {
  name: 'TermInput',
  mounted: function () {
    $.getJSON('https://prefix.cc/context', function (data) {
      this.prefixs = data
      for (const [key, value] of Object.entries(this.prefixs['@context'])) {
        prefixesMap.set(key, value)
      }
    })
    this.updateNode()
  },
  watch: {
    term (newValue) {
      this.updateNode()
    }
  },
  data () {
    return {
      dynamicNodetype: 'iri',
      literalType: 'language',
      idValue: '',
      language: 'de',
      datatype: 'http://www.w3.org/2001/XMLSchema#string',
      node: {},
      prefix: '',
      modal: false,
      prefixs: prefixesMap,
      filteredprefixs: []
    }
  },
  props: ['type', 'id', 'term'],
  model: {
    prop: 'term',
    event: 'change'
  },
  computed: {
    nodeType: {
      get: function () {
        if (this.type) {
          return this.type
        }
        return this.dynamicNodetype
      },
      set: function (newValue) {
        this.dynamicNodetype = newValue
      }
    }
  },
  methods: {
    setType (nodeType) {
      this.nodeType = nodeType
      this.notify()
    },
    setLiteralType (literalType) {
      this.literalType = literalType
      this.notify()
    },
    notify () {
      this.updateTerm()
    },
    updateTerm () {
      if (this.dynamicNodetype === 'iri') {
        this.node = DataFactory.namedNode(this.idValue)
      } else {
        if (this.literalType === 'language') {
          this.node = DataFactory.literal(this.idValue, this.language)
        } else {
          this.node = DataFactory.literal(this.idValue, DataFactory.namedNode(this.datatype))
        }
        // https://www.w3.org/TR/json-ld11/#typed-values
      }
      this.$emit('change', this.node)
    },
    updateNode () {
      this.node = this.term
      if (this.term.termType === 'NamedNode') {
        this.dynamicNodetype = 'iri'
      } else if (this.term.termType === 'Literal') {
        this.dynamicNodetype = 'literal'
        if (this.term.language) {
          this.language = this.term.language
          this.literalType = 'language'
        } else {
          this.datatype = this.term.datatype.value
          this.literalType = 'datatype'
        }
      }
      this.idValue = this.term.value
    },
    filterprefixs () {
      const arr = Array.from(this.prefixs.entries())
      console.log(arr)
      this.filteredprefixs = arr.filter(a1 => {
        return a1[0].toLowerCase().startsWith(this.prefix.toLowerCase())
      })
    },
    setprefix (prefix) {
      this.prefix = prefix[0] + ':' + prefix[1]
      this.modal = false
    }
  }
}

</script>

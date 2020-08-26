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
    <input type="text" class="form-control" v-model="prefix" @input="filterprefixs"
           @focus="modal = true">

    <div v-if="filteredprefixs && modal" class="options" ref="optionsList">
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
      <input type="text" class="form-control" v-model="prefix" @input="filterprefixs"
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
      <input type="text" class="form-control" v-model="prefix" @input="filterprefixs"
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
      this.prefix = prefix[1]
      this.modal = false
    }
  }
}

</script>

<style scoped>
  .autocomplete {
    width: 100%;
    position: relative;
  }
  .input {
    height: 40px;
    border-radius: 3px;
    border: 2px solid lightgray;
    box-shadow: 0 0 10px #eceaea;
    font-size: 25px;
    padding-left: 10px;
    padding-top: 10px;
    cursor: text;
  }
  .close {
    position: absolute;
    right: 2px;
    top: 4px;
    background: none;
    border: none;
    font-size: 30px;
    color: lightgrey;
    cursor: pointer;
  }
  .placeholder {
    position: absolute;
    top: 11px;
    left: 11px;
    font-size: 25px;
    color: #d0d0d0;
    pointer-events: none;
  }
  .popover {
    min-height: 50px;
    border: 2px solid lightgray;
    position: absolute;
    top: 46px;
    left: 0;
    right: 0;
    background: #fff;
    border-radius: 3px;
    text-align: center;
  }
  .popover input {
    width: 95%;
    margin-top: 5px;
    height: 40px;
    font-size: 16px;
    border-radius: 3px;
    border: 1px solid lightgray;
    padding-left: 8px;
  }
  .options {
    max-height: 150px;
    overflow-y: scroll;
    margin-top: 5px;
  }
  .options ul {
    list-style-type: none;
    text-align: left;
    padding-left: 0;
  }
  .options ul li {
    border-bottom: 1px solid lightgray;
    padding: 10px;
    cursor: pointer;
    background: #f1f1f1;
  }
  .options ul li:first-child {
    border-top: 2px solid #d6d6d6;
  }
  .options ul li:not(.selected):hover {
    background: #8c8c8c;
    color: #fff;
  }
  .options ul li.selected {
    background: #58bd4c;
    color: #fff;
    font-weight: 600;
  }
</style>

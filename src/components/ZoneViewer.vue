<template>
<div class='zone-viewer' v-if="selections.length > 0">
  <h2>Your zone file</h2>
  <p><strong>Edit the code below</strong> and it'll update on the right!</p>
  <textarea 
    @keyup="changed"
    ref="textarea"
    v-model="uzn"
    @blur="stopEditing"
    @focus="startEditing"
  ></textarea>
  <button v-on:click="upload">Upload template file</button>
</div>
</template>

<script>
export default {
  name: 'zone-viewer',
  props: ['selections', 'batchUpdateSelections', 'originalFilename', 'arrayBuffer'],
  data () {
    return {
      uzn: null,
      editing: false
    }
  },
  computed: {
    filename () {
      return this.originalFilename.replace(/(.*)\..*/, '$1')
    }
  },
  mounted () {
    this.redrawUzn()
  },
  watch: {
    selections () {
      this.redrawUzn()
    }
  },
  methods: {
    upload () {
      let body = {}
      body.template = this.arrayBuffer
      body.metadata = []
      for (let line of this.uzn.split('\n')) {
        const results = /([0-9]+) ([0-9]+) ([0-9]+) ([0-9]+) (.*)/.exec(line)
        body.metadata.push({
          x1: results[1],
          y1: results[2],
          x2: results[3],
          y2: results[4],
          name: results[5]
        })
      }
      fetch(`/models/${this.filename}`, { method: 'POST', body })
    },
    startEditing () {
      this.editing = true
    },
    stopEditing () {
      this.editing = false
      this.redrawUzn()
    },
    redrawUzn () {
      if (this.editing) {
        return
      }
      let lines = this.selections.map((s) => {
        return `${s.coordinates.left} ${s.coordinates.top} ${s.coordinates.width} ${s.coordinates.height} ${s.name}`
      })

      this.uzn = lines.join('\n')
    },
    changed (event) {
      var selections = this.uzn.split('\n').map((line, index) => {
        try {
          let elements = line.match(/(\d+) (\d+) (\d+) (\d+) (.*)/)
          var selection = this.selections[index]
          selection.coordinates.left = parseInt(elements[1])
          selection.coordinates.top = parseInt(elements[2])
          selection.coordinates.width = parseInt(elements[3])
          selection.coordinates.height = parseInt(elements[4])
          selection.name = elements[5]
          return selection
        } catch (err) {
          return null
        }
      }).filter((selection) => selection != null)
      console.log('updating with', selections)
      this.batchUpdateSelections(selections)
    }
  }
}
</script>

<style>
textarea {
  white-space: pre;
  overflow-wrap: normal;
  overflow-x: scroll;
  width: calc(100% - 20px);
  min-height: 200px;
}
</style>

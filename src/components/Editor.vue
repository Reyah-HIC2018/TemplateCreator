<template>
  <div class='editor'>
    <div class='sidebar'>
      <h1>Reyah Template Creator</h1>
      <Uploader :notify="newFile"></Uploader>
      <p v-if="name"><strong>Click and drag on the image to the right</strong> to form selection areas.</p>

      <ZoneViewer :arrayBuffer="arrayBuffer" :selections="selections" class='zone-viewer' :batchUpdateSelections="batchUpdateSelections" :originalFilename="name" v-if="src"></ZoneViewer>
    </div>
    <div class='content'>
      <Annotator :src="src" :setSize="setSize" :arrayBuffer="arrayBuffer" :name="name" :selections="selections" :addSelection="addSelection"></Annotator>
    </div>
  </div>
</template>

<script>
import Uploader from '@/components/Uploader'
import Annotator from '@/components/Annotator'
import ZoneViewer from '@/components/ZoneViewer'
import randomColor from 'randomcolor'

export default {
  name: 'editor',
  components: {
    Uploader,
    Annotator,
    ZoneViewer
  },
  data () {
    return {
      src: null,
      arrayBuffer: null,
      name: '',
      selections: [],
      dimensions: {
        height: 0,
        width: 0
      }
    }
  },
  methods: {
    batchUpdateSelections: function (selections) {
      this.selections = selections
    },
    setSize: function (width, height) {
      this.dimensions = {
        width: width,
        height: height
      }
    },
    addSelection: function (coords) {
      if (coords.height === 0 || coords.width === 0) {
        return
      }

      this.selections.push({
        id: +new Date(),
        coordinates: {
          top: coords.top,
          left: coords.left,
          height: coords.height,
          width: coords.width,
          page: coords.page,
          pageOffset: coords.pageOffset
        },
        color: randomColor({format: 'rgb'}),
        name: 'Unnamed' + this.selections.length
      })
    },
    newFile: function (data) {
      this.name = data.name
      this.src = data.src
      this.arrayBuffer = data.arrayBuffer
      this.selections = []
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>

.editor {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;

  margin-top: 60px;
}

.sidebar {
  position: absolute;
  width: 400px;
  top: 0;
  left: 0;
  padding: 10px;
}

.content {
  position: absolute;
  width: auto;
  margin-left: 30px;
  top: 0px;
  left: 410px;
}

textarea {
  -webkit-box-sizing: border-box;
   -moz-box-sizing: border-box;
        box-sizing: border-box;
  font-size: 1em;
}

h2 {
  background: #FFF800;
  padding: 10px 25px;
}

h1 {
  background: #C50080;
  padding: 10px 25px;
  color: white;
}
</style>

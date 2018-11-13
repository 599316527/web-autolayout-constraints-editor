<template>
  <div class="editor">
    <!-- <h1>Editor</h1> -->
    <div class="content">
      <div class="sidebar">
        <div class="treeandattr">
          <div class="tree" @click="handleItemClick(-1)">
            <label>Elements</label>
            <div class="control">
              <button @click="addItem('text')">Text</button>
              <button @click="addItem('image')">Image</button>
              <button @click="addItem('group')">Group</button>
            </div>
            <div class="items">
              <div class="item" :class="{
                  ['item-' + item.type]: true,
                  'item-selected': index === currentItemIndex
                }" v-for="(item, index) in items" :key="item.uuid" @click.stop="handleItemClick(index)">
              </div>
            </div>
          </div>
          <div class="attr">
            <div class="panel canvas" v-if="currentItemIndex < 0">
              <label>Canvas</label>
              <div class="size">
                <input type="number" step="1" min="1" v-model.number="canvas.width" required /> &times;
                <input type="number" step="1" min="1" v-model.number="canvas.height" required />
              </div>
            </div>
            <div class="panel attr-panel" v-else>
              <label>Attributes</label>
              <div class="common">
                <div class="input-group input-group-inline">
                  <label>ID</label>
                  <input type="text" v-model="items[currentItemIndex].uuid" required />
                </div>
                <div class="input-group">
                  <label>Style</label>
                  <textarea v-model="items[currentItemIndex].style"></textarea>
                </div>
              </div>
              <div class="text" v-if="items[currentItemIndex].type === 'text'">
                <div class="input-group input-group-text">
                  <label>Text</label>
                  <textarea v-model="items[currentItemIndex].value"></textarea>
                </div>
              </div>
              <div class="image" v-if="items[currentItemIndex].type === 'image'">
                <div class="input-group">
                  <label>Source</label>
                  <input v-model="items[currentItemIndex].src" required />
                </div>
                <div class="input-group">
                  <label>Size</label>
                  <input v-model="items[currentItemIndex].size" required />
                </div>
                <div class="input-group">
                  <label>positionX</label>
                  <input v-model="items[currentItemIndex].positionX" required />
                </div>
                <div class="input-group">
                  <label>positionY</label>
                  <input v-model="items[currentItemIndex].positionY" required />
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="panel constriants">
          <button style="float: right" @click="showConstriantsEdit = true">Edit</button>
          <label>Constriants</label>
          <Constraints :components="items" :constraints="constraints"></Constraints>
        </div>
      </div>

      <div class="main">
        <div class="canvas" ref="canvas" :class="{
          'highlight': currentItemIndex === -1
        }" :style="{
          width: `${canvas.width}px`,
          height: `${canvas.height}px`
        }">
          <div class="item" v-for="(item, index) in items" :key="item.uuid" :class="{
              ['item-' + item.type]: true,
              'highlight': currentItemIndex === index
            }" :style="styles[index]" :id="item.uuid">
            <template v-if="item.type === 'text'">{{ item.value }}</template>
            <div v-if="item.type === 'image'" :style="{
              background: `url(${item.src}) no-repeat
                               ${item.positionX} ${item.positionY}
                             / ${item.size}`
            }"></div>
          </div>
        </div>
      </div>
    </div>
    <div class="float-layer" v-if="showConstriantsEdit">
      <div class="float-layer-win">
        <button class="close-btn" @click="showConstriantsEdit = false">Close</button>
        <button @click="handleConstraintsAdd">Add</button>
        <h5>Constriants</h5>
        <Constraints :editable="true" :components="items"
              :constraints="constraints" @delete="handleConstraintsDelete"></Constraints>
      </div>
    </div>
  </div>
</template>

<script>
import AutoLayout from "autolayout";
import logoImage from "../assets/logo.png";
import {debounce} from 'lodash';
import Constraints from "./Constraints";

export default {
  name: 'Editor',
  components: {
    Constraints
  },
  data () {
    return {
      showConstriantsEdit: false,
      currentItemIndex: -1,
      items: [],
      canvas: {
        width: 360,
        height: 520
      },
      constraints: []
    }
  },
  computed: {
    styles() {
      let layouts = this.layouts
      return this.items.map(function ({style, uuid}) {
        return Object.assign(style.split(';')
                .map(line => line.trim())
                .filter(line => line)
                .map(line => line.split(':'))
                .filter(items => items.length === 2)
                .reduce(function (ret, [key, val]) {
                  ret[key.trim()] = val.trim()
                  return ret;
                }, {}), layouts[uuid] || {});
      })
    },
    layouts() {
      try {
        return autoLayout(this.$refs.canvas, this.constraints)
      }
      catch (err) {
        console.error(err)
      }
    }
  },
  methods: {
    addItem(type) {
      let item
      switch (type) {
        case 'text':
          item = new TextItem({type})
          break;

        case 'image':
          item = new ImageItem({type})
          break;

        default:
          alert(`TODO: Add support for '${type}' type.`);
          return;
      }
      this.items.push(item)
    },
    handleItemClick(index) {
      this.currentItemIndex = index;
    },
    handleConstraintsAdd() {
      this.constraints.push({
        view1: '',
        attr1: 'height',
        relation: 'equ',
        view2: null,
        attr2: 'left',
        constant: 0,
        multiplier: 1
      })
    },
    handleConstraintsDelete(index) {
      this.constraints.splice(index, 1)
    }
  }
}

function autoLayout(parentElm, constraints) {
    let itemStyleKeys = ['top', 'left', 'width', 'height']
    // let view = new AutoLayout.View();
    // view.addConstraints(AutoLayout.VisualFormat.parse(visualFormat, {extended: true}));
    // view.setSize(parentElm.clientWidth, parentElm.clientHeight);
    let view = new AutoLayout.View({
      constraints, // initial constraints (optional)
      width: parentElm.clientWidth,               // initial width (optional)
      height: parentElm.clientHeight,              // initial height (optional)
      spacing: 10
    });
    return Object.entries(view.subViews).reduce(function (ret, [key, view]) {
      ret[key] = itemStyleKeys.reduce(function (style, key) {
        style[key] = view[key] + 'px'
        return style
      }, {})
      return ret;
    }, {})
}


class Item {
  constructor(options) {
    this.type = options.type
    this.uuid = this.type + Math.floor(Math.random() * 0xFFFFFF).toString(36)
    this.style = ''
  }
}

class TextItem extends Item {
  constructor(options) {
    super(options);
    this.value = options.value || ''
  }
}

class ImageItem extends Item {
  constructor(options) {
    super(options);
    this.src = options.src || logoImage
    this.size = 'cover'
    this.positionX = 'center'
    this.positionY = 'center'
  }
}


</script>

<style scoped lang="less">
.editor,
.content {
  height: 100%;
}
.content {
  display: flex;


}

.sidebar {
  display: flex;
  flex-direction: column;
  .treeandattr {
    flex: 1 0;
    display: flex;
    & > div {
      height: 100%;
    }
  }
  .panel.constriants {
    flex-shrink: 0;
    padding: 15px;
    border-top: 1px solid gray;
    border-right: 1px solid gray;
  }
}

.tree {
  padding: 10px 0;
  width: 180px;
  border-right: 1px solid gray;

  label {
    margin-left: 15px;
  }

  .control {
    text-align: center;
  }
  .items {
    margin: 10px 0;

    .item {
      width: 100%;
      height: 2em;
      padding: 5px 0 5px 10px;

      &:nth-child(2n) {
        background: #e7e7e7;
      }
      &:hover {
        background: #c7c7c7;
      }

    }
    .item-text::before {
      content: "Text";
    }
    .item-image::before {
      content: "Image";
    }
    .item-selected {
      &,
      &:nth-child(2n) {
        background: #333;
        color: white;
      }

      &:hover {
        background: #111;
        color: white;
      }
    }
  }
}


.attr {
  width: 320px;
  border-right: 1px solid gray;
  display: flex;
  flex-direction: column;

  .panel {
    padding: 10px;
    // margin-top: 14px;
    overflow: auto;
  }

  textarea {
    display: block;
    line-height: 1.5;
    font-size: 13px;
    font-family: monospace;
  }

  .canvas {
    .size {
      input {
        width: 50px;
      }
    }
  }

  .input-group {
    margin: 14px 0;
    &:first-child {
      margin-top: 0;
    }

    label {
      display: block;
      font-size: 12px;
    }

    input,
    textarea {
      width: 100%;
    }

    textarea {
      height: 100px;
    }
  }

  .input-group-text {
    textarea {
      height: 50px;
    }
  }
  .input-group-inline {
    display: flex;
    justify-content: space-between;
    align-items: center;

    input[type=text] {
      margin-left: 8px;
    }
  }

  .constriants {
    padding-top: 10px;
    border-top: 1px solid gray;
    flex-shrink: 0;
    label {
      display: block;
      font-size: 12px;
    }

  }
}

.main {
  flex: 1 0;
  position: relative;
  overflow: auto;
  .highlight {
    outline: solid 1px red;
  }
  .canvas {
    margin: 10px;
    box-sizing: content-box;
    border: 1px solid black;
    box-shadow: 11px 11px 12px #949494;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    position: relative;

    .item {
      position: absolute;
      & > div {
        width: 100%;
        height: 100%;
      }
    }
  }
}

.float-layer {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;

  &-win {
    padding: 14px 20px;
    width: 700px;
    height: 480px;
    overflow: auto;
    position: absolute;
    left: 0;
    bottom: 0;
    border-top: 1px solid black;
    border-right: 1px solid black;
    background: white;
    box-shadow: 5px -5px 10px #ccc;

    h5 {
      margin-bottom: 10px;
    }

    .close-btn {
      float: right;
    }
  }
}

</style>

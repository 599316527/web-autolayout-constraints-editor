
<style lang="less" scoped>

input {
  width: 50px;
}

.edit {
  display: flex;
}
</style>

<template>
<div class="constraint">
  <div class="view" v-if="!editable">
    <span>{{ value.view1 }}</span><span>.</span><span>{{ value.attr1 }}</span>

    <span v-if="value.relation === 'equ'">=</span>

    <template v-if="value.multiplier">
      <span>{{ value.view2 || 'Canvas' }}</span><span>.</span><span>{{ value.attr2 }}</span>
      <template v-if="value.multiplier !== 1">
        <span>&times;</span>
        <span>{{ value.multiplier }}</span>
      </template>
    </template>

    <template v-if="value.constant !== 0">
      <span v-if="value.multiplier">+</span>
      <span>{{ value.constant }}</span>
    </template>
  </div>

  <div class="edit" v-else>
    <select v-model="value.view1">
      <option v-for="({uuid}) in components"
        :value="uuid" :key="uuid">{{ uuid }}</option>
    </select>
    <span>.</span>
    <select v-model="value.attr1">
      <option v-for="(attr) in attrs" :key="attr">{{ attr }}</option>
    </select>

    <select v-model="value.relation">
      <option v-for="(value, key) in relations" :key="key" :value="key">{{ value }}</option>
    </select>

    <select v-model="value.view2">
      <option :value="null">Canvas</option>
      <option v-for="({uuid}) in components"
        :value="uuid" :key="uuid">{{ uuid }}</option>
    </select>
    <span>.</span>
    <select v-model="value.attr2">
      <option v-for="(attr) in attrs" :key="attr">{{ attr }}</option>
    </select>

    <span>&times;</span>
    <input type="number" v-model.number="value.multiplier" />
    <span>+</span>
    <input type="number" v-model.number="value.constant" />
  </div>
</div>
</template>


<script>
const attrs = [
  'width',
  'height',
  'centerX',
  'centerY',
  'top',
  'right',
  'left',
  'bottom'
]

const relations = {
  leq: '<=',
  equ: '=',
  geq: '>='
}

export default {
  props: {
    components: Array,
    value: Object,
    editable: Boolean,
  },
  data() {
    return {
      attrs,
      relations
    }
  }
}
</script>

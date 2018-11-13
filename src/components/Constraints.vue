<style lang="less" scoped>
.constraints {
  margin: 5px 0;
  padding: 0;
  font-family: monospace;

  li {
    padding: 5px 0;
    border-top: 1px solid #eee;
    display: flex;
    justify-content: space-between;

    &:hover {
      background: #eee;
    }
  }


}
</style>

<template>
<ul class="constraints">
  <li v-for="(constraint, index) in constraints" :key="index">
    <Constraint :components="components" :value="constraint" :editable="index === currentEditing" />
    <div v-if="editable">
      <button @click="handleToggle(index)">{{
          currentEditing === index ? 'Done' : 'Edit' }}</button>
      <button @click="handleDelete(index)">X</button>
    </div>
  </li>
</ul>
</template>

<script>
import Constraint from "./Constraint";

export default {
  components: {
    Constraint
  },
  props: {
    constraints: Array,
    components: Array,
    editable: Boolean
  },
  data() {
    return {
      currentEditing: -1
    }
  },
  methods: {
    handleToggle(index) {
      this.currentEditing = this.currentEditing === index ? -1 : index
    },
    handleDelete(index) {
      this.$emit('delete', index)
    }
  }
}
</script>

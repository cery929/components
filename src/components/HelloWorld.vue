<script setup lang="ts">
import { ref ,reactive} from 'vue'
import CountTo from './CountTo.vue';

defineProps<{ msg: string }>()

const count = ref(0)
const countNum = reactive([
  {name:"人数",value:"122.34"},
  {name:"流量",value:32867},
  {name:"多钱",value:"423432343276.722",prefix:'￥',suffix:'人民币',separator:','}
])
</script>

<template>
  <h1>{{ msg }}</h1>

  <div class="card">
    <button type="button" @click="count++">count is {{ count }}</button>
    <p>
      Edit
      <code>components/HelloWorld.vue</code> to test HMR
    </p>
  </div>
  <div v-for="item in countNum" :key="item.name">
    {{item.name}}:
    <span>
      <CountTo :end="item.value" :autoPlay="true" :duration="10000" :prefix="item.prefix" :suffix="item.suffix" :separator="item.separator"></CountTo>
    </span>
  </div>

  <p>
    Check out
    <a href="https://vuejs.org/guide/quick-start.html#local" target="_blank"
      >create-vue</a
    >, the official Vue + Vite starter
  </p>
  <p>
    Install
    <a href="https://github.com/vuejs/language-tools" target="_blank">Volar</a>
    in your IDE for a better DX
  </p>
  <p class="read-the-docs">Click on the Vite and Vue logos to learn more</p>
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>

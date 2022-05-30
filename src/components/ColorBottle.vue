<script setup lang="ts">
  import { computed, reactive, ref } from 'vue';
  const props = defineProps(['colors', 'size', 'stack', 'lifted', 'liftable', 'pourable', 'complete']);
  const layer_info = computed(() => {
    let ret = [];
    for (let ii = 0; ii < props.size; ii++) {
      if (ii < props.stack.length) {
        ret[ii] = { class: props.colors[props.stack[ii]].class };
      } else {
        ret[ii] = { class: 'transparent' };
      }
    }
    return ret;
  });
  const liftClass = computed(() => {
    if (props.lifted) {
      return 'pb-6';
    } else {
      return 'pt-6';
    }
  });
  const liftableClass = computed(() => {
    if (props.liftable || props.pourable) {
      return 'cursor-pointer';
    }
    return 'cursor-default';
  });
</script>

<template>
  <div class="transition-all" :class="[liftClass, liftableClass]">
    <div class="pt-4 border-b border-r border-l border-black flex flex-col-reverse">
      <div v-for="n in size" class="h-6" :class="[layer_info[n-1].class]"></div>
    </div>
  </div>
</template>
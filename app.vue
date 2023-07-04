<template>
  <div id="app">
    <FlyInput :value="value" disabled lazy />
  </div>
</template>

<script lang="ts">
import FlyInput from './components/FlyInput.vue';
import { defineComponent, ref } from '~/.nuxt/imports';

export default defineComponent({
  components: {
    FlyInput,
  },
  setup() {
    const value = ref("11111");
    console.log(value.value);

    return {
      value,
    };
  },
});
</script>
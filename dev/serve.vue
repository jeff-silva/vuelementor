<script>
import Vue from 'vue';
// Uncomment import and local "components" registration if library is not registered globally.
// import { VuelementorSample } from '@/entry.esm';

export default Vue.extend({
  name: 'ServeDev',
  // components: {
  //  VuelementorSample,
  // }

  data() {
    return {
      content: [],
    };
  },
});
</script>

<template>
  <div id="app">
    <!-- <vuelementor-sample /> -->

    <div style="display:flex;">
      <div style="flex-grow:1;">
        <vuelementor v-model="content"></vuelementor>
      </div>

      <div style="flex-grow:1;">
        <vuelementor-edit v-model="content"></vuelementor-edit>
      </div>
    </div>

    <pre>content: {{ content }}</pre>
  </div>
</template>

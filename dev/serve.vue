<script>
import Vue from 'vue';

// Uncomment import and local "components" registration if library is not registered globally.
// import { VuelementorSample } from '@/entry.esm';

export default Vue.extend({
  name: 'ServeDev',
  // components: {
  //  VuelementorSample,
  // }

  runtimeCompiler: true,

  data() {
    return {
      editable: true,
      content: {
        sections: [
          {
            rows: [
              {
                components: [
                  {
                    bind: {},
                    component: {
                      template: `<div>Hello</div>`,
                    },
                  }
                ],
              },
              {},
              {},
            ],
          },
          {},
          {},
          {},
        ],
      },
    };
  },
});
</script>

<template>
  <div id="app">
    <label><input type="checkbox" v-model="editable"> Editável</label>
    <hr>
    <vuelementor v-model="content" :editable="editable"></vuelementor>
    <pre>$data: {{ $data }}</pre>
  </div>
</template>

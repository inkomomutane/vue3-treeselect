<template>
  <treeselect v-model="value" :options="options" multiple>
    <template #control-x>
      <span>❌</span>
    </template>
    <template #option-checkbox="{ node, checkboxClass, checkedState }">
      <div  style="padding: 8px; color: aqua;background: blue">{{ checkedState }}</div>
    </template>
  </treeselect>
</template>

<script>
import { generateOptions } from "./utils";

const options = generateOptions(2);

export default {
  data: () => ({
    value: [options[0].id],
    options
  })
};
</script>

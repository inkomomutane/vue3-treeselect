<template>
  <treeselect v-model="value" :options="options" multiple>
    <template #option-checkbox="{ checkboxClass, checkedState }">
      <span v-if="checkedState ===2">✔️</span>
      <span v-else-if="checkedState === 1">➖</span>
      <span v-else>🟨</span>
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

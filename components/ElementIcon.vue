<template>
  <el-image :src="imgUrls[element]" fit="contain" lazy></el-image>
</template>

<script>
import { default as elementData } from "@/static/json/element-data.json";
console.log(elementData.data);

export default {
  data() {
    return {
      imgUrls: elementData.data,
    };
  },
  props: {
    element: {
      type: String,
      default: "",
      required: true,
    },
  },
};
</script>

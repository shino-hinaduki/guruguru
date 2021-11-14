<template>
  <h1>test</h1>
</template>

<script>
import { default as charactertData } from "@/static/json/character-data.json";
import { default as enemyData } from "@/static/json/enemy-data.json";
import { default as elementData } from "@/static/json/element-data.json";

export default {
  data() {
    return {
      elements: elementData.data,
      characters: charactertData.data,
      enemies: enemyData.data,
    };
  },
};
</script>

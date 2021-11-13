<template>
  <div>
    <el-menu mode="horizontal">
      <el-menu-item
        index="1"
        @click="moveLink('https://github.com/shino-hinaduki/guruguru')"
        >guruguru</el-menu-item
      >
      <el-menu-item index="2" @click="saveImage">Save</el-menu-item>
      <el-menu-item index="3" @click="importImage">Import</el-menu-item>
      <el-menu-item index="4" @click="exportImage">Export</el-menu-item>
      <el-menu-item index="5" @click="clearImage">Clear</el-menu-item>
    </el-menu>

    <canvas ref="canvas" width="640" height="320"></canvas>

    <el-row :gutter="10" align="middle">
      <el-col>
        <el-table
          :data="
            tableData.filter(
              (data) =>
                !search ||
                data.name.toLowerCase().includes(search.toLowerCase()) ||
                data.category.toLowerCase().includes(search.toLowerCase())
            )
          "
          :default-sort="{ prop: 'name', order: 'descending' }"
          style="width: 100%"
        >
          <el-table-column fixed="left" width="50">
            <template slot-scope="props">
              <el-image
                style="width: 40px; height: 40px"
                :src="props.row.url"
                fit="contain"
                lazy
              ></el-image>
            </template>
          </el-table-column>
          <el-table-column prop="name" sortable label="Name">
            <template slot="header">
              <el-input
                v-model="search"
                size="mini"
                placeholder="Name (Type to search)"
                style="width: 90%"
              />
            </template>
          </el-table-column>
          <el-table-column prop="category" sortable label="Category">
          </el-table-column>
          <el-table-column prop="element" sortable width="50">
            <template slot-scope="props">
              <ElementIcon :element="props.row.element" />
            </template>
          </el-table-column>
          <el-table-column fixed="right" label="Operations" width="120">
            <el-button type="primary" icon="el-icon-plus">Add</el-button>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import { fabric } from "fabric";
import { default as enemyData } from "@/static/json/enemy-data.json";

export default {
  data() {
    return {
      search: "",
      tableData: enemyData.data,
      fabricCanvas: null,
    };
  },
  created() {
    window.addEventListener("beforeunload", this.exportToLocalStorage);
  },
  destroyed() {
    window.removeEventListener("beforeunload", this.exportToLocalStorage);
  },
  mounted() {
    const fabricCanvas = new fabric.Canvas(this.$refs.canvas);
    fabricCanvas.add(
      new fabric.Rect({
        fill: "green",
        width: 100,
        height: 100,
        rx: 5,
        ry: 5,
        top: 20,
        left: 60,
      })
    );
    fabricCanvas.add(new fabric.Text("Hello Fabric"));
    this.fabricCanvas = fabricCanvas;
    this.importFromLocalStorage();
  },
  methods: {
    moveLink(url) {
      window.open(url, "_blank");
    },
    saveImage() {
      // TODO:
      console.log(this.fabricCanvas.toDataURL());
    },
    exportImage() {
      // TODO:
      console.log(this.fabricCanvas.toJSON());
    },
    importImage() {
      // TODO: JSON入力
      const jsonStr = "";
      this.fabricCanvas.loadFromJSON(jsonStr, () => {});
    },
    clearImage() {
      // TODO: 確認Dialog
      this.fabricCanvas.clear();
    },
    // localStorageが使える場合、保持しておく
    exportToLocalStorage() {
      if (window.localStorage) {
        const jsonData = this.fabricCanvas.toJSON();
        alert(jsonData);
        localStorage.setItem("fabricCanvasData", JSON.stringify(jsonData));
      }
    },
    // localStorageにデータが残っている場合、復元しておく
    importFromLocalStorage() {
      if (window.localStorage) {
        const jsonData = localStorage.getItem("fabricCanvasData");
        if (jsonData) {
          try {
            this.fabricCanvas.loadFromJSON(JSON.parse(jsonData), () => {});
          } catch (err) {
            console.error(
              'localStorage.getItem("fabricCanvasData") is Invalid:'
            );
            console.error(err, jsonData);
            console.error("fabricCanvasData=", jsonData);
          }
        }
      }
    },
  },
};
</script>

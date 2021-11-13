<template>
  <div>
    <el-menu mode="horizontal">
      <el-menu-item
        index="1"
        @click="moveLink('https://github.com/shino-hinaduki/guruguru')"
        >guruguru</el-menu-item
      >
      <el-menu-item index="2" disabled>Save</el-menu-item>
      <el-menu-item index="2" disabled>Import</el-menu-item>
      <el-menu-item index="3" disabled>Export</el-menu-item>
      <el-menu-item index="4" disabled>Clear</el-menu-item>
    </el-menu>

    <canvas ref="drawCanvas" width="640" height="320"></canvas>

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
  methods: {
    moveLink(url) {
      window.open(url, "_blank");
    },
  },
  data() {
    return {
      search: "",
      tableData: enemyData.data,
    };
  },
  mounted() {
    const drawCanvas = new fabric.Canvas(this.$refs.drawCanvas);
    drawCanvas.add(
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
    drawCanvas.add(new fabric.Text("Hello Fabric"));
  },
};
</script>

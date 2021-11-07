<template>
  <div>
    <el-menu mode="horizontal">
      <el-menu-item index="1">shino-hinaduki.github.io/guruguru</el-menu-item>
      <el-menu-item index="2" disabled>Import</el-menu-item>
      <el-menu-item index="3" disabled>Export</el-menu-item>
      <el-menu-item index="4" disabled>Clear</el-menu-item>
      <el-menu-item index="5" disabled>Info</el-menu-item>
    </el-menu>

    <el-row :gutter="10" align="middle">
      <el-col>
        <el-table
          :data="
            tableData.filter(
              (data) =>
                !search ||
                data.name.toLowerCase().includes(search.toLowerCase()) ||
                data.tag.toLowerCase().includes(search.toLowerCase())
            )
          "
          :default-sort="{ prop: 'name', order: 'descending' }"
          style="width: 100%"
        >
          <el-table-column fixed="left" label="Icon" width="100px">
            <template slot-scope="props">
              <el-image
                style="width: 90px; height: 90px"
                :src="props.row.url"
                fit="contain"
                lazy
              ></el-image>
            </template>
          </el-table-column>
          <el-table-column prop="name" sortable label="Name">
            <template slot="header" slot-scope="_">
              <el-input
                v-model="search"
                size="mini"
                placeholder="Name (Type to search)"
                style="width: 90%"
              />
            </template>
          </el-table-column>
          <el-table-column prop="tag" sortable label="Tag"> </el-table-column>
          <el-table-column fixed="right" label="Operations" width="120">
            <el-button type="primary" icon="el-icon-plus">Add</el-button>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import * as enemyData from "@/static/json/enemy-data.json";

export default {
  methods: {},
  data() {
    return {
      search: "",
      tableData: enemyData.data,
    };
  },
};
</script>

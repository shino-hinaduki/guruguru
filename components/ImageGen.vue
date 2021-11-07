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
        <el-table :data="tableData" style="width: 100%">
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
          <el-table-column prop="name" label="Name"> </el-table-column>
          <el-table-column prop="tag" label="Tag"> </el-table-column>
          <el-table-column fixed="right" label="Operations" width="120">
            <el-button type="text">Add</el-button>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  methods: {},
  data() {
    return {
      tableData: [
        {
          name: "Anemoboxer Vanguard",
          tag: "Fatui",
          url: "https://static.wikia.nocookie.net/gensin-impact/images/9/96/Enemy_Fatui_Anemoboxer_Vanguard_Icon.png",
        },
      ],
    };
  },
};
</script>

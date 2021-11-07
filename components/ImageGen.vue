<template>
  <div>
    <el-menu mode="horizontal">
      <el-menu-item index="1">shino-hinaduki.github.io/guruguru</el-menu-item>
      <el-menu-item index="2" disabled>Import</el-menu-item>
      <el-menu-item index="3" disabled>Export</el-menu-item>
      <el-menu-item index="4" disabled>Info</el-menu-item>
    </el-menu>

    <el-row :gutter="10" align="middle">
      <el-col>
        <el-table :data="tableData" style="width: 100%">
          <el-table-column prop="name" label="Name" width="120">
          </el-table-column>
          <el-table-column fixed="right" label="Operations" width="120">
            <template slot-scope="scope">
              <el-button @click="handleClick" type="text" size="small"
                >Add</el-button
              >
            </template>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>

<script>
export default {
  methods: {
    handleClick() {
      console.log("click");
    },
  },
  data() {
    return {
      tableData: [
        {
          name: "Anemoboxer Vanguard",
          tag: "Fatui",
        },
      ],
    };
  },
};
</script>

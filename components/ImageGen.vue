<template>
  <div>
    <el-menu mode="horizontal">
      <el-menu-item
        index="1"
        @click="moveLink('https://github.com/shino-hinaduki/guruguru')"
        >guruguru</el-menu-item
      >
      <el-menu-item index="2" @click="saveImage">Save</el-menu-item>
      <el-menu-item index="3" @click="importFromJSON">Import</el-menu-item>
      <el-menu-item index="4" @click="exportToJSON">Export</el-menu-item>
      <el-menu-item index="5" @click="clearImage">Clear</el-menu-item>
    </el-menu>
    <el-alert
      v-if="!isEnableBackup"
      @close="forceActivateBackup"
      title="Automatic backup is disabled."
      type="warning"
      description="You may already be editing in another tab, or localStorage may not be available. When this message is closed, it will forcefully enable automatic backup."
      show-icon
    >
    </el-alert>
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
      // 挿入対象の検索文字列
      search: "",
      // 挿入可能なエントリ一覧
      tableData: enemyData.data,
      // 編集中のfabric.Canvas
      fabricCanvas: null,
      // ページが閉じられた場合に自動で最新の編集データをlocalStorageに退避する
      isEnableBackup: true,
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
    fabricCanvas.add(
      new fabric.Text("https://github.com/shino-hinaduki/guruguru")
    );
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
    // 現在の編集データを保存します
    exportToJSON() {
      const jsonStr = JSON.stringify(this.fabricCanvas.toJSON());

      // clipboard対応要否に関わらず、consoleには出力しておく
      console.info(
        "//////////////////////////////////////////////////////////////////////"
      );
      console.info("// Here is the export data.");
      console.info(jsonStr);
      console.info(
        "//////////////////////////////////////////////////////////////////////"
      );

      // Clipboard API非対応
      if (!navigator.clipboard) {
        this.$alert(
          "Output to the developer console because the clipboard API is not supported.",
          "Export Notice"
        );
        return;
      }

      // クリップボードにコピー
      navigator.clipboard.writeText(jsonStr);
      this.$alert(
        "The data being edited has been copied to the clipboard.",
        "Export Notice"
      );
    },
    importFromJSON() {
      // TODO: JSON入力
      const jsonStr = "";
      this.fabricCanvas.loadFromJSON(jsonStr, () => {});
    },
    clearImage() {
      // TODO: 確認Dialog
      this.fabricCanvas.clear();
    },
    // localStorageにデータが残っている場合、復元しておく
    importFromLocalStorage() {
      // localStorageが使えない場合
      if (!window.localStorage) {
        this.isEnableBackup = false;
        return;
      }

      // 編集中のタブがすでに存在する場合、Backup機能は無効化する
      const isEdit = localStorage.getItem("isEdit");
      if (isEdit) {
        this.isEnableBackup = false;
      }
      localStorage.setItem("isEdit", true);

      // データの読み出し
      const jsonData = localStorage.getItem("fabricCanvasData");
      if (jsonData) {
        try {
          this.fabricCanvas.loadFromJSON(JSON.parse(jsonData), () => {});
        } catch (err) {
          console.error('localStorage.getItem("fabricCanvasData") is Invalid:');
          console.error(err, jsonData);
          console.error("fabricCanvasData=", jsonData);
        }
      }
    },
    // localStorageが使える場合、保持しておく
    exportToLocalStorage() {
      if (window.localStorage && this.isEnableBackup) {
        // 編集中フラグの削除
        localStorage.removeItem("isEdit");
        // データの保存
        const jsonData = this.fabricCanvas.toJSON();
        alert(jsonData);
        localStorage.setItem("fabricCanvasData", JSON.stringify(jsonData));
      }
    },
    // 強引にバックアップ機能が有効になるようにする
    forceActivateBackup() {
      this.isEnableBackup = true;
    },
  },
};
</script>

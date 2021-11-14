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
      show-icon
    >
      <p>
        You may already be editing in another tab, or localStorage may not be
        available.
      </p>
      <p>
        When this message is closed, it will forcefully enable automatic backup.
      </p>
    </el-alert>
    <div id="wrapper">
      <canvas ref="canvas" width="640" height="320"></canvas>
    </div>

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

<style>
#wrapper {
  border: solid 1px;
  border-color: #cccccc;
  width: 90vw;
  height: 50vh;
  overflow: auto;
  resize: both;
}
</style>

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
    // initialize canvas
    const canvasElem = this.$refs.canvas;
    const fabricCanvas = new fabric.Canvas(canvasElem);
    this.fabricCanvas = fabricCanvas;

    // insert test objects. TODO: remove
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

    // enable resize
    const resizeObserver = new ResizeObserver((entries) => {
      fabricCanvas.setWidth(entries[0].contentRect.width);
      fabricCanvas.setHeight(entries[0].contentRect.height);
    });
    resizeObserver.observe(document.getElementById("wrapper"));

    // Import from auto backup
    this.importFromLocalStorage();
  },
  methods: {
    // 指定されたurlを新規タブで開きます
    moveLink(url) {
      window.open(url, "_blank");
    },
    // 編集中の画像を保存します
    saveImage() {
      // a tagでpngをbase64したものを付与したimage/octet-streamを開かせる
      const dataUrl = this.fabricCanvas.toDataURL(); // data:image/png;base64,......
      const nowDate = new Date()
        .toISOString() // 2021-11-14T12:00:00.000Z
        .split(".")[0] // 2021-11-14T12:00:00
        .replace("T", "_") // 2021-11-14_12:00:00
        .replace(":", "-"); // 2021-11-14_12-00-00
      const link = document.createElement("a");
      link.style.display = "none";
      document.body.appendChild(link);
      link.setAttribute("download", `guruguru_${nowDate}.png`);
      link.setAttribute(
        "href",
        dataUrl.replace("image/png", "image/octet-stream")
      );
      link.click();
      document.body.removeChild(link);
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
    // Json Textから編集データを復元します
    importFromJSON() {
      this.$prompt("Please paste the Json text output by 'Export'", "Import", {
        confirmButtonText: "OK",
        cancelButtonText: "Cancel",
      }).then(({ value }) => {
        try {
          this.fabricCanvas.loadFromJSON(value, () => {});
        } catch (error) {
          console.error(error);
          this.$alert(error, "Import Notice");
        }
      });
    },
    // 編集中のデータを破棄します
    clearImage() {
      this.$confirm(
        "Do you want to discard the data being edited?",
        "Confirm",
        {
          confirmButtonText: "Yes",
          cancelButtonText: "No",
        }
      ).then(() => {
        this.fabricCanvas.clear();
      });
    },
    // localStorageにデータが残っている場合、復元します
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
          this.$alert(
            "Failed to restore data from automatic backup.",
            "Warning"
          );

          console.error('localStorage.getItem("fabricCanvasData") is Invalid:');
          console.error(err, jsonData);
          console.error("fabricCanvasData=", jsonData);
        }
      }
    },
    // localStorageが使える場合、内容を保存しておきます
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
    // 強引にバックアップ機能が有効になるよう設定します
    forceActivateBackup() {
      this.isEnableBackup = true;
    },
  },
};
</script>

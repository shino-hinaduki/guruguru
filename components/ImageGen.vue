<template>
  <div>
    <el-menu mode="horizontal">
      <el-menu-item
        index="1"
        @click="moveLink('https://github.com/shino-hinaduki/guruguru')"
        >guruguru</el-menu-item
      >
      <el-submenu index="2">
        <template slot="title">File</template>
        <el-menu-item index="2-1" @click="saveImage">Save</el-menu-item>
        <el-menu-item index="2-2" @click="importFromJSON">Import</el-menu-item>
        <el-menu-item index="2-3" @click="exportToJSON">Export</el-menu-item>
        <el-menu-item index="2-4" @click="clearImage">Clear</el-menu-item>
      </el-submenu>
      <el-submenu index="3" v-if="!!fabricCanvas">
        <template slot="title">Edit</template>
        <el-menu-item index="3-1" disabled>Remove Selected</el-menu-item>
        <el-menu-item index="3-2" disabled>Insert Text</el-menu-item>
        <el-menu-item index="3-3" disabled>Insert Image from URL</el-menu-item>
        <el-menu-item index="3-4" @click="toggleDrawing"
          >Enable/Disable Drawing</el-menu-item
        >
        <el-menu-item index="3-5" @click="undoDraw">Undo Draw</el-menu-item>
        <el-menu-item index="3-6" @click="redoDraw">Redo Draw</el-menu-item>
      </el-submenu>
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
    <div id="control">
      <ItemGrid />
    </div>
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

#control {
  width: 100vw;
}
</style>

<script>
import { fabric } from "fabric";
import ItemGrid from "./ItemGrid.vue";

export default {
  components: { ItemGrid },
  data() {
    return {
      // 編集中のfabric.Canvas
      fabricCanvas: null,
      // fabricCanvasの操作履歴
      objectHistories: [],
      // redo使用後ならtrue
      isRedoing: false,
      // ページが閉じられた場合に自動で最新の編集データをlocalStorageに退避する
      isEnableBackup: true,
      // 手書きモードが有効ならtrue
      isFreehand: false,
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

    // Setup Drawing(TODO: 需要があれば可変にしたい)
    fabricCanvas.freeDrawingBrush.color = "rgba(50,40,255,0.5)";
    fabricCanvas.freeDrawingBrush.width = 10;

    // Setup undo/redo
    fabricCanvas.on("object:added", function () {
      if (!this.isRedoing) {
        this.objectHistories = [];
      }
      this.isRedoing = false;
    });

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
    // 直前の操作を巻き戻します
    undoDraw() {
      // 一番最後に追加されたオブジェクトを取り出して退避
      if (this.fabricCanvas._objects.length > 0) {
        this.objectHistories.push(this.fabricCanvas._objects.pop());
        this.fabricCanvas.renderAll();
      }
    },
    // undoで取り消した操作をもとに戻します
    redoDraw() {
      if (this.objectHistories.length > 0) {
        this.isRedoing = true;
        this.fabricCanvas.add(this.objectHistories.pop());
      }
    },
    // 手書き有効無効をトグルする
    toggleDrawing() {
      this.fabricCanvas.isDrawingMode = !this.fabricCanvas.isDrawingMode;
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
          this.fabricCanvas.loadFromJSON(value, () => {
            this.$alert("Done.", "Import Notice");
          });
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

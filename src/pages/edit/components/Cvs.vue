<template>
  <v-content>
    <!-- <v-layout
            justify-space-between 
            row 
            v-if="EditBtn"
        >   
    <v-flex xs12 md8 offset-md2>-->
    <v-toolbar floating style="width:1000px;left:200px;top:-20px" v-if="EditBtn">
      <v-flex md1>
        <v-select
          :items="linetpOptions"
          label="线条形状"
          color="brown light-1"
          v-model="linetp"
          style="margin-top:15px"
        ></v-select>
      </v-flex>
      <v-flex md1>
        <v-select
          :items="lineModeOptions"
          label="线条样式"
          color="brown light-1"
          v-model="lineMode"
          style="margin-top:15px"
        ></v-select>
      </v-flex>
      <v-flex md1>
        <v-select
          :items="lineColorOptions"
          label="线条颜色"
          color="brown light-1"
          v-model="lineColor"
          style="margin-top:15px"
        ></v-select>
      </v-flex>
      <v-flex md1>
        <v-select
          :items="lineWidthOptions"
          label="线条宽度"
          color="brown light-1"
          v-model="lineWidth"
          style="margin-top:15px"
        ></v-select>
      </v-flex>
      <v-flex md1>
        <v-select
          :items="strokeColorOptions"
          label="边缘颜色"
          color="brown light-1"
          v-model="strokeColor"
          style="margin-top:15px"
        ></v-select>
      </v-flex>
      <v-flex md1>
        <v-select
          :items="strokeWidthOptions"
          label="边缘宽度"
          color="brown light-1"
          v-model="strokeWidth"
          style="margin-top:15px"
        ></v-select>
      </v-flex>
      <v-btn-toggle mandatory class="transparent">
        <v-btn flat v-on:click="changeStateTo('editvert');" color="brown light-1">选择 | 移动</v-btn>
        <v-btn flat v-on:click="changeStateTo('freedraw');" color="brown light-1">自由绘制</v-btn>
        <v-btn flat v-on:click="changeStateTo('addvert');" color="brown light-1">添加控制点</v-btn>
        <v-btn flat v-on:click="changeStateTo('connect');" color="brown light-1">连接</v-btn>
        <v-btn flat v-on:click="changeStateTo('remove');" color="brown light-1">移除</v-btn>
        <v-divider class="mx-2" vertical></v-divider>
        <v-btn flat v-on:click="undo();" color="blue-grey lighten-1">
          <v-icon>undo</v-icon>
        </v-btn>
        <v-btn flat v-on:click="redo();" color="blue-grey lighten-1">
          <v-icon>redo</v-icon>
        </v-btn>
        <v-divider class="mx-2" vertical></v-divider>
        <v-btn flat v-on:click="save();" color="green lighten-1">
          <v-icon>save</v-icon>
        </v-btn>
      </v-btn-toggle>
    </v-toolbar>
    <!-- </v-flex>
    </v-layout>-->
    <v-layout
      justify-center
      align-space-around
      @dragover="dragOver($event)"
      @drop="dragFinished($event)"
    >
      <canvas id="c" ref="canvas"></canvas>
    </v-layout>
  </v-content>
</template>

<script>
import vCanvas from "./cvs/vCanvas.js";
import { bus } from "../../../bus.js";
import { mapGetters } from "vuex";
var cvs;
export default {
  name: "Cvs",
  props: ["EditBtn", "editId"],
  data() {
    return {
      updMap: false,
      linetpOptions: [
        { text: "曲线", value: "curve" },
        { text: "直线", value: "line" }
      ],
      lineModeOptions: [
        { text: "实线", value: "full" },
        { text: "虚线", value: "dashed" }
      ],
      lineColorOptions: [
        { text: "红色", value: "red" },
        { text: "黄色", value: "yellow" }
      ],
      lineWidthOptions: [
        { text: "1px", value: 1 },
        { text: "2px", value: 2 },
        { text: "3px", value: 3 },
        { text: "4px", value: 4 },
        { text: "5px", value: 5 },
        { text: "6px", value: 6 },
        { text: "7px", value: 7 },
        { text: "8px", value: 8 },
        { text: "9px", value: 9 }
      ],
      strokeColorOptions: [
        { text: "红色", value: "red" },
        { text: "黄色", value: "yellow" }
      ],
      strokeWidthOptions: [
        { text: "1px", value: 1 },
        { text: "2px", value: 2 },
        { text: "3px", value: 3 },
        { text: "4px", value: 4 },
        { text: "5px", value: 5 },
        { text: "6px", value: 6 },
        { text: "7px", value: 7 },
        { text: "8px", value: 8 },
        { text: "9px", value: 9 }
      ],
      linetp: "curve",
      lineMode: "full",
      lineColor: "red",
      lineWidth: 5,
      strokeColor: "red",
      strokeWidth: 1
      /*
        lineProp:{
            linetp: "curve",
            lineMode: "full",
            lineColor: "red",
            lineWidth: 5,
            strokeColor: "red",
            strokeWidth: 1,
        }
        */
    };
  },
  computed: {
    historyStack() {
      return this.$store.state.historyStack;
    }
  },
  methods: {
    getScrollTop() {
      let scrollTop = 0;
      if (document.documentElement && document.documentElement.scrollTop) {
        scrollTop = document.documentElement.scrollTop;
      } else if (document.body) {
        scrollTop = document.body.scrollTop;
      }
      return scrollTop;
    },

    canvasMousePos(canvas, event) {
      let x =
        (document.documentElement.scrollLeft || document.body.scrollLeft) +
        (event.clientX || event.pageX);
      let y = (event.clientY || event.pageY) + this.getScrollTop();
      return {
        x: x - cvs.canvas._offset.left,
        y: y - cvs.canvas._offset.top
      };
    },

    dragOver(event) {
      event.preventDefault();
    },

    dragFinished(event) {
      event.preventDefault();
      let p = this.canvasMousePos(this.$refs.canvas, event);
      let id = event.dataTransfer.getData("text");
      // alert(id);
      let div = document.getElementById(id);
      let svg_xml = new XMLSerializer()
        .serializeToString(div.childNodes[0])
        .toString();
      /*
    fabric.loadSVGFromString(svg_xml, function(objects, options) {
          let obj = fabric.util.groupSVGElements(objects, options);
          obj.set({
            left:p.x,
            top:p.y,
          });
      bus.$emit('showP',id);
      */
      cvs.createSVG(svg_xml, p, id);
      //bus.$emit('showP',id);
      //});
    },
    changeStateTo(sta) {
      cvs.changeStateTo(sta);
    },
    undo() {
      cvs.undo();
    },
    redo() {
      cvs.redo();
    },
    save() {
      cvs.saveToServer();
      // alert("保存成功");
    },
    exportImg(e) {
      let savePack = cvs.saveToServer(); //cvs.save(0,false,);
      sessionStorage.setItem("savePack", savePack);
      this.$store.commit("setHistoryStack", JSON.stringify(cvs.history));
      this.$router.push({
        name: "exportImg",
        params: e
      });
    },
    importImg() {
      let savePack = cvs.save(0, true, true);
      sessionStorage.setItem("savePack", savePack);
      this.$store.commit("setHistoryStack", JSON.stringify(cvs.history));
      this.$router.push({ name: "importImg" });
    }
  },
  watch: {
    EditBtn: function() {
      if (this.EditBtn) {
        cvs.changeStateTo("editvert");
      } else {
        if (this.updMap) {
          cvs.changeStateTo("setmap");
        } else {
          cvs.changeStateTo("move");
        }
      }
    },
    linetp: function(val) {
      cvs.setLineProp("linetp", val);
    },
    lineMode: function(val) {
      cvs.setLineProp("lineMode", val);
    },
    lineColor: function(val) {
      cvs.setLineProp("lineColor", val);
    },
    lineWidth: function(val) {
      cvs.setLineProp("lineWidth", val);
    },
    strokeColor: function(val) {
      cvs.setLineProp("strokeColor", val);
    },
    strokeWidth: function(val) {
      cvs.setLineProp("strokeWidth", val);
    }
  },
  //   computed:{
  //     ...mapGetters([
  //       'currentVillageId',
  //     ]),
  //   },
  mounted() {
    let wZoom = 0.9,
      hZoom = 0.8;
    window.addEventListener("resize", function() {
      let width = Math.round(document.body.clientWidth * wZoom);
      let height = Math.round((document.body.clientHeight - 40) * hZoom);
      document.getElementById("c").width = width;
      document.getElementById("c").height = height;
      cvs.resize({ x: width, y: height });
    });
    let width = Math.round(document.body.clientWidth * wZoom);
    let height = Math.round((document.body.clientHeight - 40) * hZoom);
    document.getElementById("c").width = width;
    document.getElementById("c").height = height;
    this.$store.dispatch("getVillagers", this.editId).then(() => {
      let savePack = sessionStorage.getItem("savePack");
      if (!(savePack === null)) {
        cvs = new vCanvas({ x: width, y: height }, "c", savePack);
        this.linetp = cvs.lineprop.linetp;
        this.lineMode = cvs.lineprop.lineMode;
        this.lineColor = cvs.lineprop.lineColor;
        this.lineWidth = cvs.lineprop.lineWidth;
        this.strokeColor = cvs.lineprop.strokeColor;
        this.strokeWidth = cvs.lineprop.strokeWidth;
        sessionStorage.removeItem("savePack");
      } else if (sessionStorage.getItem(`canvas${this.editId}`)) {
        let canvasStr = sessionStorage.getItem(`canvas${this.editId}`);
        cvs = new vCanvas({ x: width, y: height }, "c", canvasStr);
        this.linetp = cvs.lineprop.linetp;
        this.lineMode = cvs.lineprop.lineMode;
        this.lineColor = cvs.lineprop.lineColor;
        this.lineWidth = cvs.lineprop.lineWidth;
        this.strokeColor = cvs.lineprop.strokeColor;
        this.strokeWidth = cvs.lineprop.strokeWidth;
      } else {
        let canvasStr = "";
        cvs = new vCanvas({ x: width, y: height }, "c", canvasStr);
      }
      if (!(this.historyStack === null)) {
        let temphistory = JSON.parse(this.historyStack);
        cvs.history.top = temphistory.top;
        cvs.history.historyStack = temphistory.historyStack;
        cvs.history.changedSVGid = temphistory.changedSVGid;
        this.$store.commit("setHistoryStack", null);
      }
    });
    bus.$on("exportImg", e => {
      this.exportImg(e);
    });
    bus.$on("importImg", () => {
      this.importImg();
    });
    bus.$on("selectEdge", e => {
      this.linetp = e.linetp;
      this.lineMode = e.lineMode;
      this.lineColor = e.lineColor;
      this.lineWidth = e.lineWidth;
      this.strokeColor = e.strokeColor;
      this.strokeWidth = e.strokeWidth;
    });
    //   window.addEventListener('saveToServer',(event)=>{
    //       bus.$emit('save',event.detail.savePack);
    //   });
  },
  destroyed() {
    bus.$off("exportImg");
    bus.$off("importImg");
    bus.$off("selectEdge");
  }
};
</script>
<style type="text/css">
canvas {
  border: 1px solid black;
}
</style>


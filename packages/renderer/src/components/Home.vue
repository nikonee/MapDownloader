<template>
  <div id="map" />
  <div class="box-controls">
    <layer-control
      @choose="chooseLayers"
    />
    <div class="splitline" />
    <n-icon
      size="20"
      title="绘制矩形"
      style="cursor: pointer;"
      :color="isDrawing ? '#2080f0' : '#333333'"
      @click="drawRect"
    >
      <SquareOutline />
    </n-icon>
    <div class="splitline" />
    <n-icon
      size="20"
      title="下载地图"
      style="cursor: pointer;"
      @click="showSave"
    >
      <CloudDownloadOutline />
    </n-icon>
    <div class="splitline" />
    <GridIcon @show-grid="showGrid" />
    <div class="splitline" />
    <n-icon
      size="20"
      title="设置"
      style="cursor: pointer;"
      @click="showSet(true)"
    >
      <SettingsOutline />
    </n-icon>
    <div class="splitline" />
    <n-icon
      size="20"
      title="帮助"
      style="cursor: pointer;"
      @click="showHelp(true)"
    >
      <HelpCircleOutline />
    </n-icon>
    <div class="splitline" />
    <area-choose
      @choose="chooseArea"
    />

    <save-diablog
      :visible="saveVisible"
      :download-extent="downloadExtent"
      :base-layer="saveLayers"
      @ok="save"
      @cancel="cancelSave"
    />
    <help-diablog
      :visible="helpVisible"
      @ok="showHelp(false)"
    />
    <map-key
      :visible="setVisible"
      @hide="showSet(false)"
    />
  </div>
  <ProgressControl />
  <tips />
</template>

<script>
import {defineComponent} from 'vue';
import TMap from '../utils/t-map.js';
import LayerControl from './LayerControl.vue';
import AreaChoose from './AreaChoose.vue';
import SaveDiablog from './Save.vue';
import FileSave from '../utils/file-save.js';
import HelpDiablog from './Help.vue';
import MapKey from './MapKey.vue';
import Tips from './Tips.vue';
import {useMessage, useNotification} from 'naive-ui';
import GridIcon from './GridIcon.vue';
import ProgressControl from './ProgressControl.vue';
import { CloudDownloadOutline, HelpCircleOutline, SettingsOutline, SquareOutline } from '@vicons/ionicons5';
// eslint-disable-next-line
let map
export default defineComponent({
  name: 'HomeMain',
  components: {
    LayerControl,
    SaveDiablog,
    HelpDiablog,
    MapKey,
    Tips,
    AreaChoose,
    GridIcon,
    ProgressControl,
    CloudDownloadOutline,
    HelpCircleOutline,
    SettingsOutline,
    SquareOutline,
  },
  setup() {
    window.$message = useMessage();
    window.$notification = useNotification();
  },
  data() {
    return {
      isDrawing: false,
      saveVisible: false,
      downloadExtent: {},
      helpVisible: false,
      setVisible: false,
      saveLayers: null,
    };
  },
  computed: {

  },
  mounted() {
    map = new TMap('map');
    this.addMapRightClickHandle();
  },
  methods: {
    chooseLayers(data) {
      this._currentLayer = data;
      map.switchBaseLayer(data);
    },
    drawRect() {
      this.isDrawing = !this.isDrawing;
      if (this.isDrawing) {
        this.hideDrawTips();
        this._drawStartInfo = window.$notification.create({
          content: '已开启矩形绘制，右键下载瓦片',
          duration: 5000,
        });
        map.startDraw();
      } else {
        map.endDraw();
      }
    },
    hideDrawTips() {
      if (this._drawStartInfo) {
        this._drawStartInfo.destroy();
        this._drawStartInfo = null;
      }
    },
    // 地图右键下载瓦片
    addMapRightClickHandle() {
      map.getMap().addEventListener('contextmenu', () => {
        if (!this.isDrawing) return;
        if (!this.showSave(false)) {
          setTimeout(() => {
            this.isDrawing = false;
            map.endDraw();
            this.hideDrawTips();
          }, 50);
          return;
        }
      });
    },
    showSave(showMsg = true) {
      this.downloadExtent = map.getDownloadExtent();
      if (!this.downloadExtent) {
        if (showMsg) window.$message.warning('获取下载范围错误，请重新绘制下载范围');
        return false;
      }
      const {titleLayer} = map.getBaseMapConfig();
      this.saveLayers = titleLayer;
      this.saveVisible = true;
      return true;
    },
    save(val) {
      this.saveVisible = false;
      const mapConfig = map.getBaseMapConfig();
      val.mapConfig = mapConfig;
      new FileSave(val);
    },
    cancelSave() {
      this.saveVisible = false;
    },
    showHelp(val) {
      this.helpVisible = val;
    },
    showSet(val) {
      this.setVisible = val;
      if (!val && this._currentLayer && (this._currentLayer.parent === 'Tdt' || this._currentLayer.parent === 'Mapbox')) {
        map.switchBaseLayer(this._currentLayer);
      }
    },
    chooseArea(data) {
      // 结束绘制
      this.isDrawing = false;
      map.endDraw();
      this.hideDrawTips();
      // 添加区域至地图
      const {option, geojson} = data;
      console.log(option);
      map.addGeometry(geojson);
      map.fitExtent();
    },
    showGrid(val) {
      map.showTileGrid(val);
    },
  },
});
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
#map{
  position: absolute;
  margin: 0;
  padding: 0;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
.box-controls{
  position: absolute;
  left: 70px;
  top: 10px;
  background-color: white;
  box-shadow: 0px 2px 4px 0px rgb(54 58 80 / 30%);
  // width: 200px;
  padding: 8px;
  display: flex;
  // .items{
  //   width: 20px;
  //   height: 20px;
  //   background-size: contain;
  //   background-repeat: no-repeat;
  //   background-position: center center;
  //   cursor: pointer;
  // }
  .splitline{
    width: 1px;
    height: 20px;
    margin: 0 8px;
    background-color: #999999;
  }
}

</style>

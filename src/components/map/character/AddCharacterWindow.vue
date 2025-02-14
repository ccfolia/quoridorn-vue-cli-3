<template>
  <window-frame
    titleText="キャラクター置き場"
    display-property="private.display.addCharacterWindow"
    align="center"
    fixSize="200, 200"
    @open="open"
  >
    <div class="container" @contextmenu.prevent>
      <div class="name">{{ name }}</div>
      <div class="image">
        <img
          class="img"
          v-img="imageObj.data"
          @dragstart="dragStart"
          draggable="true"
          :class="{ reverse: imageObj.isReverse }"
          @mousedown.stop
        />
      </div>
      <div class="controlArea">
        <label
          ><input
            type="checkbox"
            v-model="is_Continuous"
            @keydown.enter.stop
            @keyup.enter.stop
          />複数作成</label
        >
        <span>連番：</span
        ><input
          type="number"
          min="0"
          v-model="continuous_Num"
          :disabled="!isContinuous"
          @keydown.enter.stop
          @keyup.enter.stop
        />
      </div>
    </div>
  </window-frame>
</template>

<script>
import { mapState, mapActions, mapGetters } from "vuex";
import WindowFrame from "../../WindowFrame";
import WindowMixin from "../../WindowMixin";

export default {
  mixins: [WindowMixin],
  components: {
    WindowFrame
  },
  data() {
    return {
      continuous_Num: 1,
      is_Continuous: false
    };
  },
  methods: {
    ...mapActions(["setProperty", "windowClose"]),
    /**
     * ドラッグ開始時
     * @param event
     */
    dragStart(event) {
      event.dataTransfer.setData("kind", "character");
      event.dataTransfer.setData("name", this.name);
      event.dataTransfer.setData("size", this.size);
      event.dataTransfer.setData("useImageList", this.useImageList);
      event.dataTransfer.setData("isHide", this.isHide);
      event.dataTransfer.setData("urlStr", this.url); // urlはなぜか使えない
      event.dataTransfer.setData("description", this.text);
      event.dataTransfer.setData("useImageIndex", this.useImageIndex);
      event.dataTransfer.setData("currentImageTag", this.currentImageTag);
      // window.console.log(`  [methods] drag start character => {` +
      //   `name:"${this.name}", ` +
      //   `size:${this.size}, ` +
      //   `useImageList:${this.useImageList}, ` +
      //   `isHide:${this.isHide}, ` +
      //   `url:${this.url}, ` +
      //   `text:${this.text}, ` +
      //   `useImageIndex:${this.useImageIndex}, ` +
      //   `currentImageTag:${this.currentImageTag}}`)
    },
    getKeyObj(list, key) {
      const filteredList = list.filter(obj => obj.key === key);
      if (filteredList.length === 0) return null;
      if (filteredList.length > 1) return null;
      return filteredList[0];
    },
    open() {
      this.continuous_Num = 1;
      this.is_Continuous = false;
      this.windowClose("private.display.addCharacterSettingWindow");
    }
  },
  watch: {
    is_Continuous(value) {
      this.setProperty({
        property: `private.display.addCharacterWindow.isContinuous`,
        value: value,
        logOff: true
      });
    },
    continuousNum(continuousNum) {
      this.continuous_Num = continuousNum;
    }
  },
  computed: mapState({
    ...mapGetters(["parseColor"]),
    windowObj: state => state.private.display.addCharacterWindow,
    imageList: state => state.public.image.list,
    name() {
      return (
        this.windowObj.name +
        (this.is_Continuous ? `_${this.continuous_Num}` : "")
      );
    },
    size() {
      return this.windowObj.size;
    },
    useImageList() {
      return this.windowObj.useImageList;
    },
    isHide() {
      return this.windowObj.isHide;
    },
    url() {
      return this.windowObj.url;
    },
    text() {
      return this.windowObj.text;
    },
    useImageIndex() {
      return this.windowObj.useImageIndex;
    },
    currentImageTag() {
      return this.windowObj.currentImageTag;
    },
    isContinuous() {
      return this.windowObj.isContinuous;
    },
    continuousNum() {
      return this.windowObj.continuousNum;
    },
    imageObj() {
      if (this.useImageList === "") {
        return "";
      }
      const imageStr = this.useImageList.split("|")[this.useImageIndex];
      const isReverse = imageStr.indexOf(":R") >= 0;
      const imageKey = imageStr.replace(":R", "");
      return {
        isReverse: isReverse,
        data: this.getKeyObj(this.imageList, imageKey).data
      };
    }
  })
};
</script>

<style scoped lang="scss">
.container {
  display: block;
  width: 100%;
  height: 100%;
  font-size: 12px;

  > * {
    display: flex;
    align-items: center;
    justify-content: center;
  }
}

.image img {
  display: block;
  width: 96px;
  height: 96px;
  margin: 10px;
  box-sizing: border-box;
  border: solid yellow 3px;
  background-color: rgba(0, 0, 0, 0);

  &.reverse {
    transform: scale(-1, 1);
  }
}

label {
  display: flex;
  align-items: center;
  justify-content: center;
}

input[type="number"] {
  width: 40px;
}
</style>

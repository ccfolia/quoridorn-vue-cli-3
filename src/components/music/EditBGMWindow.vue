<template>
  <window-frame
    titleText="BGM編集画面"
    display-property="private.display.editBGMWindow"
    align="right-bottom"
    fixSize="300, 365"
    @open="initWindow"
    @reset="initWindow"
  >
    <div class="contents">
      <fieldset>
        <legend @contextmenu.prevent>読込</legend>
        <!-- URL -->
        <label class="url">
          <span @contextmenu.prevent>URL</span>
          <input
            type="text"
            v-model="url"
            v-if="!url || isYoutube || !isHideUrl"
            ref="urlElm"
            placeholder="指定なしで「停止」の操作になります"
          />
          <!-- URLマスク -->
          <div
            class="mask"
            v-if="url && !isYoutube && isHideUrl"
            @click="isHideUrl = !isHideUrl"
            @contextmenu.prevent
          >
            編集するにはここをクリックしてください。
          </div>
        </label>
      </fieldset>
      <fieldset>
        <legend @contextmenu.prevent>表示</legend>
        <div class="firstWide">
          <!-- 表示タイトル -->
          <label class="titleStr">
            <span @contextmenu.prevent>タイトル</span>
            <input type="text" v-model="title" />
          </label>
        </div>
        <div class="firstWide" v-if="!isYoutube">
          <!-- クレジットURL -->
          <label class="creditUrl">
            <span @contextmenu.prevent>CreditURL</span>
            <input type="text" v-model="creditUrl" placeholder="未実装" />
          </label>
          <!-- クレジット取得 -->
          <ctrl-button class="getCredit" @click="getCredit">取得</ctrl-button>
        </div>
      </fieldset>
      <fieldset>
        <legend @contextmenu.prevent>再生</legend>
        <div>
          <!-- タグ -->
          <label class="tag">
            <span
              title="再生中のBGMはタグによって一意になります"
              @contextmenu.prevent
              >タグ</span
            >
            <input type="text" v-model="tag" list="bgmTagComboboxValues" />
          </label>
          <datalist id="bgmTagComboboxValues">
            <option v-for="tag in tags" :key="tag" :value="tag">
              {{ tag }}
            </option>
          </datalist>
          <!-- 音量 -->
          <volume-component
            :initVolume="volume"
            @volume="setVolume"
            @mute="setIsMute"
            :mutable="false"
            ref="volumeComponent"
          />
          <!-- プレビュー -->
          <ctrl-button class="preview" @click="preview">プレビュー</ctrl-button>
        </div>
        <div>
          <!-- 再生開始 -->
          <label class="start">
            <span @contextmenu.prevent>再生開始</span>
            <input
              type="number"
              step="0.1"
              min="-10000"
              max="10000"
              v-model="start"
            />
            <span @contextmenu.prevent>秒</span></label
          >
          〜
          <!-- 再生終了 -->
          <label class="end">
            <span @contextmenu.prevent>再生終了</span>
            <input
              type="number"
              step="0.1"
              min="-10000"
              max="10000"
              v-model="end"
            />
            <span @contextmenu.prevent>秒</span></label
          >
        </div>
        <div>
          <!-- フェードイン -->
          <label class="fadeIn">
            <span @contextmenu.prevent>fadeIn</span>
            <input
              type="number"
              min="0"
              step="0.1"
              max="200"
              v-model="fadeIn"
              placeholder="秒"
            />
          </label>
          <!-- フェードアウト -->
          <label class="fadeOut">
            <span @contextmenu.prevent>fadeOut</span>
            <input
              type="number"
              min="0"
              step="0.1"
              max="200"
              v-model="fadeOut"
              placeholder="秒"
            />
          </label>
          <!-- 無限ループ -->
          <span
            class="icon loop"
            :class="{ active: isLoop }"
            @click="change('isLoop')"
            @contextmenu.prevent
          >
            <i class="icon-loop"></i>リピート{{ isLoop ? "あり" : "なし" }}
          </span>
        </div>
        <div>
          <!-- 多重再生時強制再スタート -->
          <label>
            <span @contextmenu.prevent>多重再生時強制再スタート</span>
            <input type="checkbox" v-model="forceReset" />
          </label>
        </div>
      </fieldset>
      <fieldset>
        <legend @contextmenu.prevent>チャット連動</legend>
        <div class="lastWide">
          <!-- チャット連動オプション -->
          <label class="option">
            <ctrl-select
              v-model="chatLinkage"
              :optionInfoList="chatLinkageOptionInfoList"
            />
          </label>
          <!-- 検索文字 -->
          <label
            class="search"
            v-show="chatLinkage === '1' || chatLinkage === '2'"
          >
            <input
              :placeholder="chatLinkage === '1' ? '' : 'Javascriptの正規表現'"
              type="text"
              v-model="chatLinkageSearch"
            />
          </label>
        </div>
      </fieldset>
      <div class="buttonArea" @contextmenu.prevent>
        <div>
          <ctrl-button @click="commit">確定</ctrl-button>
          <ctrl-button @click="cancel">キャンセル</ctrl-button>
        </div>
      </div>
    </div>
  </window-frame>
</template>

<script lang="ts">
import WindowFrame from "../WindowFrame.vue";
import WindowMixin from "../WindowMixin.vue";
import CtrlButton from "../parts/CtrlButton.vue";
import CtrlSelect from "../parts/CtrlSelect.vue";
import VolumeComponent from "./component/VolumeComponent.vue";

import { Watch } from "vue-property-decorator";
import { Action, Getter } from "vuex-class";
import { Component, Mixins } from "vue-mixin-decorator";

@Component({
  components: {
    CtrlSelect,
    CtrlButton,
    WindowFrame,
    VolumeComponent
  }
})
export default class EditBGMWindow extends Mixins<WindowMixin>(WindowMixin) {
  private isHideUrl: boolean = true;
  private isYoutube: boolean = false;
  private url: string = "";
  private title: string = "";
  private creditUrl: string = "";
  private tag: string = "";
  private isLoop: boolean = false;
  private fadeIn: number | string = 0;
  private fadeOut: number | string = 0;
  private start: number | string = 0;
  private end: number | string = 0;
  private isMute: boolean = false;
  private volume: number = 0.8;
  private options: any[] = [
    { value: "0", label: "なし" },
    { value: "1", label: "末尾文字" },
    { value: "2", label: "正規表現" }
  ];
  private tags: string[] = ["BGM", "SE"];
  private chatLinkage: string = "0";
  private chatLinkageSearch: string = "";
  private forceReset: boolean = true;

  @Action("setProperty") private setProperty: any;
  @Action("windowOpen") private windowOpen: any;
  @Action("windowClose") private windowClose: any;
  @Getter("parseColor") private parseColor: any;
  @Getter("changeListObj") private changeListObj: any;

  private initWindow() {
    const key = this.key;
    const bgmObj = this.bgmList.filter((bgmObj: any) => bgmObj.key === key)[0];
    if (!bgmObj) {
      alert("そのBGMは既に削除されたようです。");
      this.windowClose("private.display.editBGMWindow");
    }
    this.isHideUrl = true;
    this.url = bgmObj.url;
    this.isYoutube = /www\.youtube\.com/.test(this.url);
    this.title = bgmObj.title;
    this.creditUrl = bgmObj.creditUrl;
    this.tag = bgmObj.tag;
    this.isLoop = bgmObj.isLoop;
    this.fadeIn = bgmObj.fadeIn;
    this.fadeOut = bgmObj.fadeOut;
    this.start = bgmObj.start;
    this.end = bgmObj.end;
    this.isMute = bgmObj.isMute;
    this.volume = Math.floor(parseFloat(bgmObj.volume) * 100) / 100;

    const volumeComponentElm: VolumeComponent = this.$refs
      .volumeComponent as VolumeComponent;
    volumeComponentElm!.setVolume(this.volume);
    this.chatLinkage = String(bgmObj.chatLinkage);
    this.chatLinkageSearch = bgmObj.chatLinkageSearch;
    this.forceReset = bgmObj.forceReset;
  }

  private commit() {
    this.changeListObj({
      key: this.key,
      url: this.url,
      title: this.title,
      creditUrl: this.creditUrl,
      tag: this.tag,
      isLoop: this.isLoop,
      start: parseInt(String(this.start), 10),
      end: parseInt(String(this.end), 10),
      fadeIn: Math.floor(parseFloat(String(this.fadeIn)) * 10) / 10,
      fadeOut: Math.floor(parseFloat(String(this.fadeOut)) * 10) / 10,
      isMute: this.isMute,
      volume: this.volume,
      chatLinkage: this.chatLinkage,
      chatLinkageSearch: this.chatLinkageSearch,
      forceReset: this.forceReset
    });
    this.windowClose("private.display.editBGMWindow");
  }

  private cancel() {
    this.windowClose("private.display.editBGMWindow");
  }

  private getCredit() {
    this.creditUrl = this.url.replace(/^(https?:\/\/[^/]+).+$/, "$1");
  }

  private preview() {
    this.setProperty({
      property: "private.display.jukeboxWindow.command",
      logOff: true,
      isNotice: false,
      value: {
        command: "add",
        payload: {
          url: this.url,
          title: this.title,
          creditUrl: this.creditUrl,
          tag: this.tag,
          isLoop: this.isLoop,
          start: parseInt(String(this.start), 10),
          end: parseInt(String(this.end), 10),
          fadeIn: Math.floor(parseFloat(String(this.fadeIn)) * 10) / 10,
          fadeOut: Math.floor(parseFloat(String(this.fadeOut)) * 10) / 10,
          isMute: this.isMute,
          volume: this.volume,
          chatLinkage: this.chatLinkage,
          chatLinkageSearch: this.chatLinkageSearch,
          forceReset: this.forceReset
        }
      }
    });
  }

  private change(this: any, param: string) {
    this[param] = !this[param];
  }

  private setIsMute(isMute: boolean) {
    this.isMute = isMute;
  }

  private setVolume(volume: string) {
    this.volume = Math.floor(parseFloat(volume) * 100) / 100;
  }

  @Watch("isHideUrl")
  private onChangeIsHideUrl(isHideUrl: boolean) {
    if (!isHideUrl) {
      this.url = "";

      const urlElm: HTMLInputElement = this.$refs.urlElm as HTMLInputElement;
      setTimeout(() => urlElm!.focus(), 0);
    }
  }

  private get key() {
    return this.$store.state.private.display["editBGMWindow"].key;
  }

  private get bgmList() {
    return this.$store.state.public.bgm.list;
  }

  private get chatLinkageOptionInfoList() {
    return this.options.map((optionInfo: any) => ({
      key: optionInfo.value,
      value: optionInfo.value,
      text: optionInfo.label
    }));
  }
}
</script>

<style scoped lang="scss">
.contents {
  position: absolute;
  height: 100%;
  width: 100%;
  font-size: 12px;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
}
fieldset {
  padding-left: 5px;
  padding-right: 5px;
  padding-bottom: 5px;

  &:not(:last-child) {
    margin-bottom: 5px;
  }

  > div {
    display: flex;
    justify-content: left;
    align-content: center;

    &:not(:last-child) {
      margin-bottom: 5px;
    }

    i:not(:first-child) {
      margin-left: 7px;
    }
  }
}
.icon {
  display: flex;

  i {
    position: relative;
    font-size: 12px;
    width: 1.4em;
    height: 1.4em;
    display: flex;
    justify-content: center;
    align-content: flex-start;
    font-weight: bold;

    &:before {
      display: flex;
      justify-content: center;
      align-content: flex-start;
      position: absolute;
      top: 50%;
      margin-top: calc(-12px / 2);
    }
  }

  &.loop {
    border-style: solid;
    border-width: 2px;
    border-radius: 0.5em;
    color: black;
    font-size: 80%;

    &.active {
      border-color: black;
      color: black;
    }
    &:not(.active) {
      border-color: gray;
      color: gray;
    }
  }
}
.firstWide > :first-child {
  flex: 1;
}
.lastWide > :last-child {
  flex: 1;
}
input[type="text"] {
  width: 50px;
}
legend,
label,
button {
  font-size: 10px;
}
select,
input[type="text"],
input[type="number"] {
  font-size: 11px;
}
label {
  display: flex;
  justify-content: left;
  align-content: center;
  vertical-align: middle;
  white-space: nowrap;
  padding: auto;

  span {
    display: flex;
    justify-content: center;
    align-content: center;
    margin: auto;
  }

  input,
  .mask {
    flex: 1;
    padding: 2.5px 0;
  }

  .mask {
    border: 1px solid black;
    background-color: lightgray;
    padding: 2px 0;
  }
}
.volumeComponent {
  flex: 1;
}
.tag input {
  width: 52px;
}
.fadeIn,
.fadeOut {
  flex: 1;
  margin-right: 5px;
}
.fadeIn input,
.fadeOut input {
  width: 32px;
}
.regexp input {
  width: 50px;
}
.buttonArea {
  display: flex;
  justify-content: center;
  align-content: flex-start;
}
</style>

<template>
  <div :style="menuStyle" id="menu" @contextmenu.prevent>
    <!-- 操作ボタングループ -->
    <div class="span-group">
      <span
        @click="menuClick()"
        @mouseenter="menuHover('ファイル')"
        :class="{ isHover: isShow('ファイル') }"
      >
        ファイル
      </span>
      <span
        @click="menuClick()"
        @mouseenter="menuHover('表示')"
        :class="{ isHover: isShow('表示', 'ウィンドウ') }"
      >
        表示
      </span>
      <span
        @click="menuClick()"
        @mouseenter="menuHover('コマ')"
        :class="{ isHover: isShow('コマ') }"
      >
        コマ
      </span>
      <span
        @click="menuClick()"
        @mouseenter="menuHover('マップ')"
        :class="{ isHover: isShow('マップ') }"
      >
        マップ
      </span>
      <span
        @click="menuClick()"
        @mouseenter="menuHover('画像')"
        :class="{ isHover: isShow('画像') }"
      >
        画像
      </span>
      <span
        @click="menuClick()"
        @mouseenter="menuHover('ヘルプ')"
        :class="{ isHover: isShow('ヘルプ') }"
      >
        ヘルプ
      </span>
      <span
        @click="menuClick()"
        @mouseenter="menuHover('デモ')"
        :class="{ isHover: isShow('デモ') }"
      >
        デモ
      </span>
    </div>
    <!-- 部屋情報 -->
    <div
      class="menu-button"
      @click="clickRoomInfo"
      :title="roomInfoTitle"
      :class="{ isDisconnect: !isRoomJoined }"
    >
      <span :class="{ isDisconnect: !isRoomJoined }">{{
        `${roomName}` || "未接続"
      }}</span>
      ：
      <span>{{ members.length }}</span>
      名
    </div>
    <!-- 共有メモ -->
    <div class="menu-button" @click="clickPublicMemo" :title="publicMemoTitle">
      共有メモ
    </div>
    <!-- ログアウト -->
    <div
      class="menu-button"
      @click="clickLogOut"
      :class="{ isDisconnect: !isRoomJoined }"
      :title="logoutTitle"
    >
      ログアウト
    </div>

    <!--------------------------------------------------
     ! ファイル
     !-------------------------------------------------->
    <div class="hoverMenu hoverMenu2" v-show="isShow('ファイル')">
      <div class="item" @click="clickExport">セーブ</div>
      <div class="item" @click="clickImport">ロード</div>
      <hr @mouseenter="menuHover('ファイル')" />
      <div class="item" @click="clickChatLog">チャットログ取得</div>
    </div>
    <!--------------------------------------------------
     ! 表示
     !-------------------------------------------------->
    <div class="hoverMenu hoverMenu3" v-show="isShow('表示', 'ウィンドウ')">
      <div class="item" @mouseenter="menuHover('ウィンドウ')">
        ウィンドウ
        <span class="triangle"></span>
      </div>
      <hr @mouseenter="menuHover('表示')" />
      <menu-boolean-item
        property="private.setting.standImage"
        @mouseenter="menuHover('表示')"
      >
        立ち絵表示
      </menu-boolean-item>
      <menu-boolean-item
        property="private.setting.cutIn"
        @mouseenter="menuHover('表示')"
      >
        カットイン表示
      </menu-boolean-item>
      <hr @mouseenter="menuHover('表示')" />
      <menu-boolean-item
        property="public.setting.gridId"
        @mouseenter="menuHover('表示')"
      >
        座標表示
      </menu-boolean-item>
      <menu-boolean-item
        property="public.setting.gridLine"
        @mouseenter="menuHover('表示')"
      >
        マス目表示
      </menu-boolean-item>
      <hr @mouseenter="menuHover('表示')" />
      <menu-boolean-item
        property="public.setting.isFitGrid"
        @mouseenter="menuHover('表示')"
      >
        マス目にキャラクターを合わせる
      </menu-boolean-item>
      <menu-boolean-item
        property="private.setting.standImageAutoResize"
        @mouseenter="menuHover('表示')"
      >
        立ち絵のサイズを自動調整する
      </menu-boolean-item>
      <hr @mouseenter="menuHover('表示')" />
      <div
        class="item"
        @click="clickSettingFontSize"
        @mouseenter="menuHover('表示')"
      >
        フォントサイズ調整
      </div>
      <hr @mouseenter="menuHover('表示')" />
      <div
        class="item"
        @click="clickResetWindowLocate"
        @mouseenter="menuHover('表示')"
      >
        ウィンドウ配置初期化
      </div>
    </div>
    <!--------------------------------------------------
     ! コマ
     !-------------------------------------------------->
    <div class="hoverMenu hoverMenu4" v-show="isShow('コマ')">
      <div class="item" @click="clickAddCharacter">キャラクター追加</div>
      <div class="item" @click="clickAddRange">範囲追加</div>
      <hr />
      <div class="item" @click="clickAddChit">チット作成</div>
      <hr />
      <div class="item" @click="clickGraveyard">墓場</div>
      <div class="item" @click="clickWaitingRoom">キャラクター待合室</div>
      <hr />
      <menu-boolean-item property="public.setting.pieceRotateMarker">
        回転マーカーを表示する
      </menu-boolean-item>
    </div>
    <!--------------------------------------------------
     ! マップ
     !-------------------------------------------------->
    <div class="hoverMenu hoverMenu5" v-show="isShow('マップ')">
      <div class="item" @click="clickChangeMap">マップ変更</div>
      <div class="item" @click="clickAddFloorTile">フロアタイル追加</div>
      <div class="item" @click="clickAddMapMask">マップマスク追加</div>
      <div class="item" @click="clickCreateEasyMap">簡易マップ作成</div>
      <hr />
      <div class="item" @click="clickSaveMap">マップ状態保存</div>
      <div class="item" @click="clickSwitchMap">マップ切り替え</div>
    </div>
    <!--------------------------------------------------
     ! 画像
     !-------------------------------------------------->
    <div class="hoverMenu hoverMenu6" v-show="isShow('画像')">
      <div class="item" @click="clickFileUploader">ファイルアップローダー</div>
      <hr />
      <div class="item" @click="clickTagEdit">タグ編集</div>
      <div class="item" @click="clickDeleteImage">画像削除</div>
    </div>
    <!--------------------------------------------------
     ! ヘルプ
     !-------------------------------------------------->
    <div class="hoverMenu hoverMenu7" v-show="isShow('ヘルプ')">
      <div class="item" @click="clickWelcome">ようこそ画面</div>
      <div class="item" @click="clickVersion">バージョン</div>
      <div class="item" @click="clickManual">マニュアル</div>
      <hr />
      <div class="item" @click="clickOfficialSite">オフィシャルサイトへ</div>
    </div>
    <!--------------------------------------------------
     ! ウィンドウ
     !-------------------------------------------------->
    <div class="hoverMenu hoverMenu8" v-show="isShow('ウィンドウ')">
      <menu-boolean-item
        @click="menuClick"
        property="private.display.chatWindow"
      >
        チャット表示
      </menu-boolean-item>
      <menu-boolean-item @click="menuClick" property="private.setting.dice">
        ダイス表示
      </menu-boolean-item>
      <menu-boolean-item
        @click="menuClick"
        property="private.display.playerBoxWindow"
      >
        プレイヤーボックス表示
      </menu-boolean-item>
      <menu-boolean-item
        @click="menuClick"
        property="private.display.initiativeWindow"
      >
        イニシアティブ表示
      </menu-boolean-item>
      <menu-boolean-item
        @click="menuClick"
        property="private.display.resourceWindow"
      >
        リソース表示
      </menu-boolean-item>
      <hr />
      <menu-boolean-item
        @click="menuClick"
        property="private.display.chatPaletteSettingWindow"
      >
        チャットパレット表示
      </menu-boolean-item>
      <menu-boolean-item
        @click="menuClick"
        property="private.display.counterRemoconWindow"
      >
        カウンターリモコン表示
      </menu-boolean-item>
    </div>
    <!--------------------------------------------------
     ! デモ
     !-------------------------------------------------->
    <div class="hoverMenu hoverMenu9" v-show="isShow('デモ')">
      <div class="item" @click="clickDevHistory">開発履歴</div>
      <hr />
      <div class="item" @click="clickBufForm">不具合の報告</div>
    </div>
  </div>
</template>

<script lang="ts">
import MenuBooleanItem from "./MenuBooleanItem.vue";

import { Action, Getter } from "vuex-class";
import { Component, Vue, Watch } from "vue-property-decorator";

@Component({
  components: {
    MenuBooleanItem
  }
})
export default class Menu extends Vue {
  @Action("windowOpen") private windowOpen: any;
  @Action("setProperty") private setProperty: any;
  @Action("doResetWindowLocate") private doResetWindowLocate: any;
  @Action("exportStart") private exportStart: any;
  @Action("addListObj") private addListObj: any;
  @Action("saveChatLogHtml") private saveChatLogHtml: any;
  @Getter("roomName") private roomName: any;
  @Getter("isModal") private isModal: any;
  @Getter("peerId") private peerId: any;
  @Getter("members") private members: any;
  @Getter("isRoomJoined") private isRoomJoined: any;

  private isConnectHover: boolean = false;
  private isSelecting: boolean = false;
  private currentMenu: string = "";

  menuClick(): void {
    this.isSelecting = !this.isSelecting;
  }

  isShow(this: any, ...props: any[]): any {
    return (
      this.isSelecting && props.filter(prop => prop === this.currentMenu)[0]
    );
  }

  menuHover(prop: string): void {
    this.currentMenu = prop;
  }

  hoverConnect(flg: boolean): void {
    this.isConnectHover = flg;
  }

  /** 部屋情報ボタン押下 */
  clickRoomInfo(): void {
    this.windowOpen("private.display.roomInfoWindow");
  }

  /** 共有メモボタン押下 */
  clickPublicMemo() {
    this.addListObj({
      propName: "publicMemo",
      kind: "publicMemo",
      targetList: [],
      title: "共有メモ",
      tabList: []
    });
  }

  /** ログアウトボタン押下 */
  clickLogOut(): void {
    location.href = location.href.replace(/\?.+$/, "");
  }

  /* --------------------
   * ファイル
   * ----------------- */
  /** セーブ */
  clickExport(): void {
    this.exportStart();
    this.menuClick();
  }

  /** ロード */
  clickImport(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "ロード",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /** チャットログ保存 */
  clickChatLog(): void {
    this.saveChatLogHtml();
    this.menuClick();
  }

  /* --------------------
   * 表示
   * ----------------- */
  /** フォントサイズ調整 */
  clickSettingFontSize(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "フォントサイズ変更",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /** ウィンドウ配置初期化 */
  clickResetWindowLocate(): void {
    this.doResetWindowLocate();
    this.menuClick();
  }

  /* --------------------
   * コマ
   * ----------------- */
  /** キャラクター追加 */
  clickAddCharacter(): void {
    this.windowOpen("private.display.addCharacterSettingWindow");
    this.menuClick();
  }

  /** 範囲追加 */
  clickAddRange(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "範囲追加",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /** チット作成 */
  clickAddChit(): void {
    this.windowOpen("private.display.addChitWindow");
    this.menuClick();
  }

  /** 墓場 */
  clickGraveyard(): void {
    alert("「墓場」はプレイヤーボックス画面に統合されました。");
    this.menuClick();
  }

  /** キャラクター待合室 */
  clickWaitingRoom(): void {
    alert("「キャラクター待合室」はプレイヤーボックス画面に統合されました。");
    this.menuClick();
  }

  /* --------------------
   * マップ
   * ----------------- */
  /** マップ変更 */
  clickChangeMap(): void {
    this.windowOpen("private.display.editMapWindow");
    this.menuClick();
  }

  /** フロアタイル追加 */
  clickAddFloorTile(): void {
    this.windowOpen("private.display.addFloorTileWindow");
    this.menuClick();
  }

  /** マップマスク追加 */
  clickAddMapMask(): void {
    this.windowOpen("private.display.addMapMaskWindow");
    this.menuClick();
  }

  /** 簡易マップ作成 */
  clickCreateEasyMap(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "簡易マップ",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /** マップ状態保存 */
  clickSaveMap(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "マップ保存",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /** マップ切り替え */
  clickSwitchMap(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "マップ切り替え",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /* --------------------
   * 画像
   * ----------------- */
  /** ファイルアップローダー */
  clickFileUploader(): void {
    this.windowOpen("private.display.fileUploaderWindow");
    this.menuClick();
  }

  /** タグ編集 */
  clickTagEdit(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "画像タグ編集",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /** 画像削除 */
  clickDeleteImage(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "画像削除",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /* --------------------
   * ヘルプ
   * ----------------- */
  /** ようこそ */
  clickWelcome(): void {
    this.windowOpen("private.display.welcomeWindow");
    this.menuClick();
  }

  /** バージョン */
  clickVersion(): void {
    this.windowOpen("private.display.versionWindow");
    this.menuClick();
  }

  /** マニュアル */
  clickManual(): void {
    this.setProperty({
      property: "private.display.unSupportWindow.title",
      value: "マニュアル",
      logOff: true
    });
    this.windowOpen("private.display.unSupportWindow");
    this.menuClick();
  }

  /** オフィシャルサイトへ */
  clickOfficialSite(): void {
    window.open("http://quoridorn.com/", "_blank");
    this.menuClick();
  }

  /* --------------------
   * デモ
   * ----------------- */
  /** 開発履歴 */
  clickDevHistory(): void {
    this.windowOpen("private.display.devLogWindow");
    this.menuClick();
  }

  /** 不具合の報告 */
  clickBufForm(): void {
    window.open("https://9224.teacup.com/quoridorn_bug/bbs", "_blank");
    this.menuClick();
  }

  get menuStyle(): any {
    const result: any = {};
    if (this.isModal) result.filter = "blur(3px)";
    return result;
  }

  get roomInfoTitle(): string {
    return this.isRoomJoined === true
      ? "メンバーの一覧を見たり、部屋の設定を変えることができますよ。"
      : "お部屋に入っていません。\n「接続」ボタンを押してお部屋を作りましょう！！";
  }

  get publicMemoTitle(): string {
    return this.isRoomJoined === true
      ? "メンバーに共有したいテキストはこちらにどうぞ"
      : "部屋に入る前から準備しておくのですね！？\nなんと準備の良いお方でしょう！";
  }

  get logoutTitle(): string {
    return this.isRoomJoined === true
      ? "この部屋から退室するのですか？"
      : "お部屋に入っていません。\n「接続」ボタンを押してお部屋を作りましょう！！";
  }
}
</script>

<style scoped lang="scss">
#menu {
  display: flex;
  justify-content: left;
  align-items: center;
  flex-direction: row;
  flex-wrap: nowrap;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background: linear-gradient(rgba(247, 248, 249, 1), rgba(0, 0, 0, 0));
  border-bottom: solid gray 1px;
  padding: 0.5em 1em;
  font-size: 10px;
  z-index: 99998;

  > *:not(:first-child) {
    margin-left: 1em;
  }
}
div.isDisconnect {
  background-color: rgba(200, 200, 200, 0.5);

  &:hover {
    background-color: rgba(200, 200, 200, 0.5);
  }
}
span.isDisconnect {
  color: red;
  font-weight: bold;
}
.span-group {
  display: flex;
  flex-direction: row;
  background-color: rgba(250, 250, 250, 0.2);
  border: solid gray 1px;
  box-sizing: border-box;
  padding: 0 1em;
  height: 2em;

  span {
    display: inline-flex;
    justify-content: center;
    align-items: center;
    padding: 0 1em;
    white-space: nowrap;
  }
}
.span-group span:hover,
.span-group span.isHover {
  background: linear-gradient(rgba(186, 195, 199, 0.6), rgba(247, 248, 249, 1));
}

.menu-button {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  position: relative;
  background: rgba(250, 250, 250, 0.4);
  border: solid gray 1px;
  padding: 0 1em;
  box-sizing: border-box;
  border-radius: 5px;
  cursor: pointer;
  white-space: nowrap;
  z-index: 100;
  height: 2em;

  &:hover {
    border: solid #0092ed 1px;
    background: rgba(250, 250, 250, 0.5);
  }
}
.hoverMenu {
  position: fixed;
  top: calc(2.5em - 1px);
  background: white;
  border: solid gray 1px;
  box-sizing: border-box;
  z-index: 200;

  > * {
    padding: 2px 10px;
  }

  > hr {
    padding: 0;
  }

  > .item:hover {
    background: lightblue;
  }
}
.hoverMenu2 {
  left: 1em;
}
.hoverMenu3 {
  left: 7em;
}
.hoverMenu4 {
  left: 11em;
}
.hoverMenu5 {
  left: 15em;
}
.hoverMenu6 {
  left: 20em;
}
.hoverMenu7 {
  left: 24em;
}
.hoverMenu8 {
  left: calc(25em - 1px);
}
.hoverMenu9 {
  left: 29em;
}

.item {
  position: relative;
  white-space: nowrap;

  > * {
    display: inline;
    vertical-align: middle;
  }

  img.check {
    display: inline;
    width: 10px;
    height: 10px;
    min-width: 10px;
    min-height: 10px;
    margin-right: 5px;
    border: none;
  }
}
.triangle {
  position: absolute;
  right: 7px;
  top: 0;
  bottom: 0;
  margin: auto;
  display: inline-block;
  vertical-align: middle;
  width: 0;
  height: 0;
  border-left: 5px solid black;
  border-top: 4px solid transparent;
  border-bottom: 4px solid transparent;
}
</style>

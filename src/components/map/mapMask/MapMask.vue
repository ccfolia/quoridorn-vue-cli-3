<template>
  <div
    class="mapMask"
    :class="[
      storeObj.isLock ? 'isLock' : 'isUnLock',
      isHover ? 'hover' : '',
      storeObj.isBorderHide ? 'isBorderHide' : ''
    ]"
    :style="mapMaskStyle"
    :id="storeObj.key"
    @mouseover="mouseover"
    @mouseout="mouseout"
    @click.right.prevent="e => openContext(e, 'private.display.mapMaskContext')"
    @mousedown.left.stop="leftDown"
    @mouseup.left.stop="leftUp"
    @mousedown.right.stop="rightDown"
    @mouseup.right.stop="rightUp"
    @touchstart.stop="leftDown"
    @touchend.stop="leftUp"
    @touchcancel.stop="leftUp"
    @contextmenu.prevent
  >
    {{ storeObj.name }}
  </div>
</template>

<script lang="ts">
import PieceMixin from "../../PieceMixin.vue";

import { Component } from "vue-property-decorator";
import { Getter } from "vuex-class";

@Component
export default class MapMask extends PieceMixin {
  @Getter("parseColor") private parseColor: any;

  private get mapMaskStyle(): any {
    let obj: any = {};
    const baseStyle: any = this.style;
    for (let key in baseStyle) {
      if (!baseStyle.hasOwnProperty(key)) continue;
      obj[key] = baseStyle[key];
    }
    // obj.transform =
    //   obj.transform.replace(/ translate[XY]\([^)]+\)/g, "") +
    //   ` translateX(0px) translateY(0px)`;
    let colorObj = this.parseColor(this.storeObj.color);
    if (this.storeObj.isDraggingLeft) {
      const plus = 1.5;
      obj.left = this.rect.left - plus + "px";
      obj.top = this.rect.top - plus + "px";
      obj.width = this.rect.width + plus * 2 + "px";
      obj.height = this.rect.height + plus * 2 + "px";
      colorObj.a = colorObj.a * 0.6;
    }
    obj["background-color"] = colorObj.getRGBA();
    obj["color"] = this.storeObj.fontColor;
    // window.console.log(` [computed] mapMask(${this.objKey}) style => isDraggingLeft:${storeObj.isDraggingLeft},transZ:${obj['transform']} lt(${obj.left}, ${obj.top}), wh(${obj.width}, ${obj.height}), bg:"${obj['background-color']}", font:"${obj.color}"`)
    return obj;
  }
}
</script>

<style scoped lang="scss">
.mapMask {
  position: fixed;
  display: flex;
  justify-content: center;
  align-items: center;
  white-space: nowrap;
  font-size: 12px;
  cursor: crosshair;
  z-index: 300000000;

  &.hover {
    z-index: 999999999;
  }

  &:before {
    content: "";
    position: absolute;
    left: -2px;
    right: -2px;
    bottom: -2px;
    top: -2px;
    border: 2px solid transparent;
  }

  &.isBorderHide:before {
    border: none;
  }

  &:not(.isBorderHide).hover {
    &.isLock:before {
      border-color: blue;
    }

    &.isUnLock:before {
      border-color: yellow;
    }
  }
}
</style>

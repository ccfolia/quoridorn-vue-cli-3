<template>
  <label class="ctrl-button-wrapper" :disabled="disabled" @contextmenu.prevent>
    <input
      type="button"
      ref="button"
      :disabled="disabled"
      @click.left.stop.prevent="buttonOnClickLeft"
      @click.right.stop.prevent="buttonOnClickRight"
      @keydown.enter.stop
      @keyup.enter.stop
    />
    <span class="front-area">
      <slot />
    </span>
    <span class="background-area"></span>
  </label>
</template>

<script lang="ts">
import { Component, Emit, Prop, Vue, Watch } from "vue-property-decorator";

@Component({})
export default class CtrlButton extends Vue {
  @Prop({ type: Boolean, default: false })
  private disabled!: boolean;

  @Emit("click")
  private buttonOnClickLeft(event: any) {}

  @Emit("click-right")
  private buttonOnClickRight(event: any) {}

  public requestFocus(): void {
    const button: HTMLInputElement = this.$refs.button as HTMLInputElement;
    button.focus();
  }
}
</script>

<style scoped lang="scss">
@import "./Ctrl.scss";

.ctrl-button-wrapper {
  @include ctrl-button();
}
</style>

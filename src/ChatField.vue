<template>
  <div :class="['chat-wrapper', wrapperClass]">
    <div
      v-if="title"
      class="chat-wrapper_title"
    >
      {{ title }}
    </div>
    <div class="chat-wrapper_body">
      <div
        :class="['chat-body_input', bodyClass, {'not-empty': notEmpty}]"
        :contenteditable="contenteditable"
        :data-placeholder="placeholder"
        ref="chatField"
        @keydown.enter.exact="onKeyDown"
        @keyup="updateCaretPosition"
        @click="updateCaretPosition"
        @input="changeText"
        @paste.prevent="handlePaste"
      />
      <div class="chat-wrapper_buttons">
        <slot
          v-if="showFile"
          name="file"
        >
          <button
            @click="openSelectFileWindow"
          >
            <SvgElement
              :icon-width="24"
              :icon-height="24"
              icon-name="file"
            />
          </button>
        </slot>
        <slot
          v-if="showSmile"
          name="smile"
        >
          <button
            @click="showSmiles"
          >
            <SvgElement
              :icon-width="24"
              :icon-height="24"
              icon-name="smile"
            />
          </button>
        </slot>
        <slot name="send">
          <button
            @click="submitMsg"
          >
            <SvgElement
              :icon-width="24"
              :icon-height="24"
              icon-name="sent"
            />
          </button>
        </slot>
      </div>
      <input
        ref="inputFile"
        type="file"
        name="input-file"
        multiple
        @change="setFiles"
      >
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator';
import SvgElement from '@/components/SvgElement.vue';

@Component({
  components: {
    SvgElement,
  },
})
export default class ChatField extends Vue {
  private node: Node|null = null;

  private cursorPosition = 0;

  private text = '';

  @Prop() readonly wrapperClass!: string;

  @Prop() readonly title!: string;

  @Prop() readonly bodyClass!: string

  @Prop({
    default: true,
  }) readonly contenteditable!: boolean;

  @Prop() readonly placeholder!: string;

  @Prop({
    default: false,
  }) readonly isFocus!: boolean;

  @Prop({
    default: true,
  }) readonly showSmile!: boolean;

  @Prop({
    default: true,
  }) readonly showFile!: boolean;

  get refInput():HTMLDivElement {
    return this.$refs.chatField;
  }

  get refFile():HTMLInputElement {
    return this.$refs.inputFile;
  }

  get notEmpty():boolean {
    return this.text.length > 0;
  }

  $refs!: {
    chatField: HTMLDivElement,
    inputFile: HTMLInputElement,
  }

  public updateCaretPosition():void {
    const selection = window.getSelection();
    if (selection) {
      const newNode = selection.anchorNode;
      const newCursorPosition = selection.anchorOffset;
      if (this.node === newNode && this.cursorPosition === newCursorPosition) {
        return;
      }
      this.node = newNode;
      this.cursorPosition = newCursorPosition;
    }
  }

  public onKeyDown(e: KeyboardEvent):void {
    e.stopPropagation();
    e.preventDefault();
    this.submitMsg();
  }

  public changeText():void {
    const { innerText } = this.refInput;
    this.text = innerText;
    this.$emit('changeText', innerText);
    this.updateCaretPosition();
  }

  public handlePaste(e: ClipboardEvent) {
    const fileList:File[] = [];
    if (e.clipboardData && e.clipboardData.files.length > 0) {
      const { files } = e.clipboardData;
      Array.from(files).forEach((file) => {
        fileList.push(file);
      });
    }
    const text = e.clipboardData ? e.clipboardData.getData('text/plain') : '';
    document.execCommand('insertText', false, text);
    this.$emit('setFiles', fileList);
  }

  public setFiles() {
    const { files } = this.refFile;
    if (files) {
      const fileList:File[] = [];
      Array.from(files).forEach((file) => {
        fileList.push(file);
      });
      this.$emit('setFiles', fileList);
    }
  }

  public openSelectFileWindow():void {
    this.refFile.click();
  }

  public showSmiles() {
    this.$emit('showSmiles');
  }

  public submitMsg():void {
    this.$emit('submitMsg', this.text);
  }

  mounted():void {
    if (this.isFocus) {
      this.$nextTick(() => {
        this.refInput.focus();
      });
    }
  }
}
</script>

<style lang="scss">
  @import "./assets/style.scss";
</style>

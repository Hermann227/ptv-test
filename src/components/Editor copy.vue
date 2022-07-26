<template>
  <div id="text-editor">
    <div class="toolbar" v-if="editor">
      <div class="align-dropdown">
        <button class="dropbtn">
          Heading ▼
        </button>
        <div class="dropdown-content">
          <a 
            v-for="index in 6"
            :key="index"
            :class="{ 'active': editor.isActive('heading', { level: index }) }"
            :style="{ fontSize: (20 - index) + 'px' }"
            @click="onHeadingClick(index)"
            role="button">
            H{{ index }}
          </a>
        </div>
      </div>

      <v-btn 
        v-for="{ slug, option, active, icon }, index in textActions"
        :key="index"
        :class="{ 'active': editor.isActive(active) }"
        @click="onActionClick(slug, option)">
        <v-icon >{{ icon }}</v-icon>
      </v-btn>
    </div>
    <editor-content :editor="editor" />
  </div>
</template>

<script>
import { Editor, EditorContent } from '@tiptap/vue-2'
import StarterKit from '@tiptap/starter-kit'
import TextAlign from '@tiptap/extension-text-align';

export default {
  components: {
    EditorContent,
  },
  props: {
    modelValue: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      editor: null,
      textActions: [
        { slug: 'bold', icon: 'mdi-format-bold', active: 'bold' },
        { slug: 'italic', icon: 'mdi-format-italic', active: 'italic' },
        { slug: 'underline', icon: 'mdi-format-underline', active: 'underline' },
        { slug: 'strike', icon: 'mdi-format-strikethrough', active: 'strike' },
        { slug: 'align', option: 'left', icon: 'mdi-format-align-left', active: { textAlign: 'left' } },
        { slug: 'align', option: 'center', icon: 'mdi-format-align-center', active: { textAlign: 'center' } },
        { slug: 'align', option: 'right', icon: 'mdi-format-align-right', active: { textAlign: 'right' } },
        { slug: 'align', option: 'justify', icon: 'mdi-format-align-justify', active: { textAlign: 'justify' } },
        { slug: 'bulletList', icon: 'mdi-format-list-bulleted', active: 'bulletList' },
        { slug: 'orderedList', icon: 'mdi-format-list-numbered', active: 'orderedList' },
        { slug: 'undo', icon: 'mdi-undo', active: 'undo' },
        { slug: 'redo', icon: 'mdi-redo', active: 'redo' },
        { slug: 'clear', icon: 'mdi-format-clear', active: 'clear' },
      ],
    }
  },
  watch: {
    modelValue(value) {
      if (this.editor.getHTML() === value) return;
      this.editor.commands.setContent(this.modelValue, false);
    },
  },
  methods: {
    onActionClick(slug, option = null) {
      const vm = this.editor.chain().focus();
      const actionTriggers = {
        bold: () => vm.toggleBold().run(),
        italic: () => vm.toggleItalic().run(),
        underline: () => vm.toggleUnderline().run(),
        strike: () => vm.toggleStrike().run(),
        bulletList: () => vm.toggleBulletList().run(),
        orderedList: () => vm.toggleOrderedList().run(),
        align: () => vm.setTextAlign(option).run(),
        undo: () => vm.undo().run(),
        redo: () => vm.redo().run(),
        clear: () => { 
          vm.clearNodes().run();
          vm.unsetAllMarks().run();
        },
      };
      
      actionTriggers[slug]();
    },
    onHeadingClick(index) {
      const vm = this.editor.chain().focus();
      vm.toggleHeading({ level: index }).run()
    }
  },
  mounted() {
    this.editor = new Editor({
      content: this.modelValue,
      extensions: [
        StarterKit,
        TextAlign.configure({
          types: ['heading', 'paragraph'],
        }),
      ],
      onUpdate: () => {
        this.$emit('update:modelValue', this.editor.getHTML());
      },
    })
  },

  beforeDestroy() {
    this.editor.destroy()
  },
}
</script>
<style>
  #text-editor {
    border: 1px solid #808080;
  }
  #text-editor .toolbar {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    border-bottom: 1px solid #808080;
  }
  #text-editor .toolbar > button {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 32px;
    height: 32px;
    font-size: 20px;
    background: #fff;
    color: #333;
    border: none;
    border-radius: 2px;
    margin: 0.5em 4px;
    -webkit-appearance: none;
    cursor: pointer;
  }
  #text-editor .toolbar > button.active {
    background: #333;
    color: #fff;
  }
  #text-editor .align-dropdown {
    position: relative;
    display: inline-block;
    margin: 0.5em 8px;
  }
  #text-editor .align-dropdown > button {
    height: 32px;
    background: #fff;
    color: #333;
    border: none;
    border-radius: 2px;
    -webkit-appearance: none;
    cursor: pointer;
  }
  #text-editor .align-dropdown > .dropdown-content {
    display: none;
    position: absolute;
    left: 0;
    right: 0;
    border: 1px solid #333;
    outline: 1px solid #fff;
    border-radius: 2px;
    background-color: #fff;
    z-index: 1;
  }
  #text-editor .align-dropdown > .dropdown-content a {
    display: block;
    padding: 6px 12px;
    text-align: center;
    cursor: pointer;
  }
  #text-editor .align-dropdown > .dropdown-content a:hover, #text-editor .align-dropdown > .dropdown-content a.active {
    background: #333;
    color: #fff;
  }
  #text-editor .align-dropdown:hover .dropdown-content {
    display: block;
  }
  #text-editor .divider {
    width: 1px;
    height: 24px;
    background: #333;
    margin-right: 6px;
  }
  #text-editor .footer {
    color: #808080;
    font-size: 14px;
    text-align: right;
    padding: 6px;
  }
  #text-editor .ProseMirror {
    height: 300px;
    overflow-y: auto;
    padding-left: 0.5em;
    padding-right: 0.5em;
    outline: none;
  }
  #text-editor .ProseMirror > p:first-child {
    margin-top: 0.5em;
  }
  #text-editor .ProseMirror > h1:first-child, #text-editor .ProseMirror h2:first-child, #text-editor .ProseMirror h3:first-child, #text-editor .ProseMirror h4:first-child, #text-editor .ProseMirror h5:first-child, #text-editor .ProseMirror h6:first-child {
    margin-top: 0.5em;
  }
</style>
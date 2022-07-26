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
            class="button"
            :class="{ 'active': editor.isActive('heading', { level: index }) }"
            :style="{ fontSize: (20 - index) + 'px' }"
            @click="onHeadingClick(index)"
            role="button">
            H{{ index }}
          </a>
        </div>
      </div>

      <v-btn 
        v-for="{ slug, option, active, icon, key } in textActions"
        :key="key"
        :class="{ 'active': editor.isActive(active) }"
        @click="onActionClick(slug, option)">
        <v-icon >{{ icon }}</v-icon>
      </v-btn>

      <v-btn 
        v-for="{ slug, option, active, icon, key } in tableActions"
        :key="key"
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
import TextAlign from '@tiptap/extension-text-align'
import Table from '@tiptap/extension-table'
import TableCell from '@tiptap/extension-table-cell'
import TableHeader from '@tiptap/extension-table-header'
import TableRow from '@tiptap/extension-table-row'

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
        { slug: 'bold', icon: 'mdi-format-bold', active: 'bold', key: 'format-bold' },
        { slug: 'italic', icon: 'mdi-format-italic', active: 'italic', key: 'format-italic' },
        { slug: 'underline', icon: 'mdi-format-underline', active: 'underline', key: 'format-underline' },
        { slug: 'strike', icon: 'mdi-format-strikethrough', active: 'strike', key: 'format-strikethrough' },
        { slug: 'align', option: 'left', icon: 'mdi-format-align-left', active: { textAlign: 'left' }, key: 'format-align-left' },
        { slug: 'align', option: 'center', icon: 'mdi-format-align-center', active: { textAlign: 'center' }, key: 'format-align-center' },
        { slug: 'align', option: 'right', icon: 'mdi-format-align-right', active: { textAlign: 'right' }, key: 'format-align-right' },
        { slug: 'align', option: 'justify', icon: 'mdi-format-align-justify', active: { textAlign: 'justify' }, key: 'format-align-justify' },
        { slug: 'bulletList', icon: 'mdi-format-list-bulleted', active: 'bulletList', key: 'format-list-bulleted' },
        { slug: 'orderedList', icon: 'mdi-format-list-numbered', active: 'orderedList', key: 'format-list-numbered' },
        { slug: 'undo', icon: 'mdi-undo', active: 'undo', key: 'format-undo' },
        { slug: 'redo', icon: 'mdi-redo', active: 'redo', key: 'format-redo' },
        { slug: 'clear', icon: 'mdi-format-clear', active: 'clear', key: 'format-clear' },
        { slug: 'insertTable', icon: 'mdi-table', active: 'insertTable', key: 'format-table' },
      ],
      tableActions: [
        { slug: 'addColumnBefore', icon: 'mdi-table-column-plus-before', active: 'column-plus-before', key: 'column-plus-before' },
        { slug: 'addColumnAfter', icon: 'mdi-table-column-plus-after', active: 'column-plus-after', key: 'column-plus-after' },
        { slug: 'deleteColumn', icon: ' mdi-table-column-remove', active: 'column-remove', key: 'column-remove' },
        { slug: 'addRowBefore', icon: 'mdi-table-row-plus-before', active: 'row-plus-before', key: 'row-plus-before' },
        { slug: 'addRowAfter', icon: 'mdi-table-row-plus-after', active: 'row-plus-after', key: 'row-plus-after' },
        { slug: 'deleteRow', icon: 'mdi-table-row-remove', active: 'row-remove', key: 'row-remove' },
        { slug: 'deleteTable', icon: 'mdi-table-remove', active: 'table-remove', key: 'table-remove' },
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
        insertTable: () => vm.insertTable({ rows: 3, cols: 3, withHeaderRow: true }).run(),
        addColumnBefore: () => vm.addColumnBefore().run(),
        addColumnAfter: () => vm.addColumnAfter().run(),
        deleteColumn: () => vm.deleteColumn().run(),
        addRowBefore: () => vm.addRowBefore().run(),
        addRowAfter: () => vm.addRowAfter().run(),
        deleteRow: () => vm.deleteRow().run(),
        deleteTable: () => vm.deleteTable().run(),
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
        Table.configure({
          resizable: true,
        }),
        TableHeader,
        TableCell,
        TableRow,
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
<style lang="scss">
#text-editor {
  border: 1px solid #808080;

  .toolbar {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    border-bottom: 1px solid #808080;

    > button {
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

      &.active {
        background: #333;
        color: #fff;
      }
    }
  }

  .align-dropdown {
    position: relative;
    display: inline-block;
    margin: 0.5em 8px;

    > button {
      height: 32px;
      background: #fff;
      color: #333;
      border: none;
      border-radius: 2px;
      -webkit-appearance: none;
      cursor: pointer;
    }

    > .dropdown-content {
      display: none;
      position: absolute;
      left: 0;
      right: 0;
      border: 1px solid #333;
      outline: 1px solid #fff;
      border-radius: 2px;
      background-color: #fff;
      z-index: 1;

      a {
        display: block;
        padding: 6px 12px;
        text-align: center;
        cursor: pointer;

        &:hover, &.active {
          background: #333;
          color: #fff;
        }
      }
    }
    
    &:hover .dropdown-content {
      display: block;
    }
  }

  .divider {
    width: 1px;
    height: 24px;
    background: #333;
    margin-right: 6px;
  }

  .footer {
    color: #808080;
    font-size: 14px;
    text-align: right;
    padding: 6px;
  }

  .ProseMirror {
    height: 300px;
    overflow-y: auto;
    padding-left: 0.5em;
    padding-right: 0.5em;
    outline: none;

    > p:first-child {
      margin-top: 0.5em;
    }

    > h1, h2, h3, h4, h5, h6 {
      &:first-child {
        margin-top: 0.5em;
      }
    }
    table {
      border-collapse: collapse;
      table-layout: fixed;
      width: 100%;
      margin: 0;
      overflow: hidden;

      td,
      th {
        min-width: 1em;
        border: 2px solid #ced4da;
        padding: 3px 5px;
        vertical-align: top;
        box-sizing: border-box;
        position: relative;

        > * {
          margin-bottom: 0;
        }
      }

      th {
        font-weight: bold;
        text-align: left;
        background-color: #f1f3f5;
      }

      .selectedCell:after {
        z-index: 2;
        position: absolute;
        content: "";
        left: 0; right: 0; top: 0; bottom: 0;
        background: rgba(200, 200, 255, 0.4);
        pointer-events: none;
      }

      .column-resize-handle {
        position: absolute;
        right: -2px;
        top: 0;
        bottom: -2px;
        width: 4px;
        background-color: #adf;
        pointer-events: none;
      }

      p {
        margin: 0;
      }
    }
  }
  .v-btn {
    width: 36px;
    min-width: 36px;
  }
}
</style>
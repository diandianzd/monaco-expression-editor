<script setup>
import { ref, onMounted } from 'vue'
import { EditorView, basicSetup } from 'codemirror'
import { javascript } from '@codemirror/lang-javascript'
import { Decoration, ViewPlugin, WidgetType } from '@codemirror/view'

const editorRef = ref(null)
let view = null

// 自定义 Widget
class TagWidget extends WidgetType {
  toDOM() {
    const span = document.createElement('span')
    span.className = 'tag-label'
    span.textContent = '用户.部门.名称'
    return span
  }
}

// 插件：查找并替换占位符为 Widget
function tagPlugin() {
  return ViewPlugin.fromClass(class {
    constructor(view) {
      this.decorations = this.buildDecorations(view)
    }
    update(update) {
      if (update.docChanged || update.viewportChanged) {
        this.decorations = this.buildDecorations(update.view)
      }
    }
    buildDecorations(view) {
      const widgets = []
      const regex = /\$\{user\.dept\.name\}/g
      for (let { from, to } of view.visibleRanges) {
        let text = view.state.doc.sliceString(from, to)
        let match
        while ((match = regex.exec(text)) !== null) {
          const start = from + match.index
          const end = start + match[0].length
          widgets.push(
            Decoration.replace({
              widget: new TagWidget(),
              inclusive: false,
              block: false
            }).range(start, end)
          )
        }
      }
      return Decoration.set(widgets)
    }
  }, {
    decorations: v => v.decorations
  })
}

onMounted(() => {
  view = new EditorView({
    doc: ' hello ${user.dept.name} world',
    extensions: [
      basicSetup,
      javascript(),
      tagPlugin(),
      EditorView.theme({
        '.tag-label': {
          background: '#ffe0b2',
          color: '#d35400',
          borderRadius: '4px',
          padding: '0 4px',
          marginRight: '4px',
          fontSize: '12px',
          whiteSpace: 'nowrap'
        }
      })
    ],
    parent: editorRef.value
  })
})
</script>

<template>
  <div ref="editorRef" style="min-height:200px;height:400px"></div>
</template>
<script setup>
import { markRaw, onMounted, ref } from 'vue';
import * as monaco from 'monaco-editor';

const monacoRef = ref(null);
const editor = ref(null);


// ...existing code...

onMounted(() => {
  if (monacoRef.value) {
    editor.value = markRaw(
      monaco.editor.create(monacoRef.value, {
        theme: 'vs',
        language: 'javascript',
        readOnly: false,
        value: ' hello ${user.dept.name} world',
        automaticLayout: true,
        scrollBeyondLastLine: true,
      })
    );

    const tagText = '${user.dept.name}';
    let widget = null;

    function updateTagWidget() {
      const value = editor.value.getValue();
      const startIndex = value.indexOf(tagText);
      if (startIndex !== -1) {
        const model = editor.value.getModel();
        const startPos = model.getPositionAt(startIndex);
        const endPos = model.getPositionAt(startIndex + tagText.length);

        // 添加装饰
        editor.value.deltaDecorations([], [
          {
            range: new monaco.Range(
              startPos.lineNumber,
              startPos.column,
              endPos.lineNumber,
              endPos.column
            ),
            options: {
              inlineClassName: 'tag-placeholder-hidden',
              before: {
                contentText: '用户.部门.名称',
                inlineClassName: 'tag-label'
              },
              isWholeLine: false,
            }
          }
        ]);

        // 更新或添加 Inline Widget
        if (!widget) {
          class TagInlineWidget {
            constructor() {
              this.domNode = document.createElement('span');
              this.domNode.className = 'tag-label';
              this.domNode.textContent = '用户.部门.名称';
            }
            getId() {
              return 'tag-inline-widget';
            }
            getDomNode() {
              return this.domNode;
            }
            getPosition() {
              return {
                position: startPos,
                preference: [monaco.editor.ContentWidgetPositionPreference.EXACT]
              };
            }
          }
          widget = new TagInlineWidget();
          editor.value.addContentWidget(widget);
        } else {
          // 强制刷新 widget 位置
          editor.value.layoutContentWidget(widget);
        }
      } else if (widget) {
        editor.value.removeContentWidget(widget);
        widget = null;
      }
    }

    updateTagWidget();

    // 监听内容变化，动态调整 widget 位置
    editor.value.onDidChangeModelContent(() => {
      updateTagWidget();
    });

    editor.value.layout();
  }
});

// ...existing code...

</script>

<template>
  <div style="min-height: 200px; height: 400px" ref="monacoRef"></div>
</template>

<style>
.tag-label {
  background: #ffe0b2;
  color: #d35400;
  border-radius: 4px;
  padding: 0 4px;
  margin-right: 4px;
  font-size: 12px;
  white-space: nowrap;

}

/* 隐藏原始占位符文本 */
.tag-placeholder-hidden {
  color: transparent;   /* 用 color 隐藏文本 */
  pointer-events: none;
  user-select: none;
}

</style>
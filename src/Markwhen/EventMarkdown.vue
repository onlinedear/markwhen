<script setup lang="ts">
import { useEditorOrchestratorStore } from "@/EditorOrchestrator/editorOrchestratorStore";
import type {
  Block,
  Image,
  MarkdownBlock,
  Range,
} from "@markwhen/parser/lib/Types";
import { toInnerHtml } from "./utilities/innerHtml";

const props = defineProps<{
  supplemental: MarkdownBlock[];
  matchedListItems: Range[];
}>();

const editorOrchestrator = useEditorOrchestratorStore();

const onChange = (index: number, checked: boolean) => {
  // 直接通过 index 找到对应的 Range
  const range = props.matchedListItems[index];
  if (!range || range.type !== "checkboxItemIndicator") {
    console.error("Invalid range or range type");
    return;
  }

  // 更新文本
  editorOrchestrator.setText(`- [${checked ? "x" : " "}]`, {
    from: range.from,
    to: range.to,
  });
};
</script>

<template>
  <div style="font-family: system-ui">
    <template v-for="(item, index) in supplemental" :key="index">
      <div
        v-if="item.type === 'checkbox'"
        class="flex flex-row items-center"
      >
        <input
          type="checkbox"
          :disabled="!editorOrchestrator.editable"
          :checked="(item as Block).value"
          @change="onChange(index, !(item as Block).value)"
          :id="`checkbox_${index}_${(item as Block).raw}`"
        />
        <label
          v-html="toInnerHtml((item as Block).raw)"
          class="ml-2 pointer-events-auto"
          :for="`checkbox_${index}_${(item as Block).raw}`"
        ></label>
      </div>
      <ul
        v-else-if="item.type === 'listItem'"
        class="list-inside list-disc ml-1"
      >
        <li v-html="(item as Block).raw"></li>
      </ul>
      <p
        v-else-if="item.type === 'text'"
        v-html="toInnerHtml((item as Block).raw)"
      ></p>
      <img v-else :src="(item as Image).link" class="py-4" />
    </template>
  </div>
</template>

<style scoped></style>

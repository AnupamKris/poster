<template>
  <div class="font-mono text-sm">
    <div v-for="(item, key) in formattedData" :key="key" class="json-item">
      <div
        :class="[
          'flex items-start gap-2 py-1 px-2 hover:bg-gray-50 rounded cursor-pointer select-none',
          { 'border-l-2 border-primary/20': item.depth > 0 },
        ]"
        :style="{ marginLeft: `${item.depth * 16}px` }"
        @click="toggleExpand(item.path)"
      >
        <button
          v-if="item.expandable"
          class="w-4 h-4 mt-1 flex items-center justify-center"
          :class="item.expanded ? 'transform rotate-90' : ''"
        >
          <ChevronRightIcon class="w-4 h-4" />
        </button>
        <span v-else class="w-4"></span>

        <div class="flex-1 flex items-start gap-2">
          <span class="text-blue-600 whitespace-nowrap">{{ key }}</span>
          <span class="text-gray-400">:</span>
          <span
            :class="{
              'text-green-600': typeof item.value === 'string',
              'text-amber-600': typeof item.value === 'number',
              'text-purple-600': typeof item.value === 'boolean',
              'text-gray-500 italic': item.value === null,
            }"
          >
            <template v-if="!item.expandable || !item.expanded">
              {{
                item.preview ||
                (item.value === null ? "null" : String(item.value))
              }}
            </template>
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from "vue";
import { ChevronRightIcon } from "lucide-vue-next";

const props = defineProps<{
  data: any;
}>();

interface JsonItem {
  value: any;
  depth: number;
  path: string;
  expanded: boolean;
  expandable: boolean;
  preview: string;
}

const expandedPaths = ref<Set<string>>(new Set());

const isExpandable = (value: any): boolean => {
  return (
    value !== null && typeof value === "object" && Object.keys(value).length > 0
  );
};

const getPreview = (value: any): string => {
  if (Array.isArray(value)) {
    return `Array(${value.length})`;
  }
  if (typeof value === "object" && value !== null) {
    return `Object(${Object.keys(value).length})`;
  }
  return "";
};

const formatData = (
  data: any,
  depth = 0,
  path = "",
  result: Record<string, JsonItem> = {}
): Record<string, JsonItem> => {
  if (typeof data !== "object" || data === null) {
    result[path] = {
      value: data,
      depth,
      path,
      expanded: false,
      expandable: false,
      preview: "",
    };
    return result;
  }

  const keys = Object.keys(data);
  keys.forEach((key) => {
    const newPath = path ? `${path}.${key}` : key;
    const value = data[key];
    const expandable = isExpandable(value);

    result[newPath] = {
      value,
      depth,
      path: newPath,
      expanded: expandedPaths.value.has(newPath),
      expandable,
      preview: expandable ? getPreview(value) : "",
    };

    if (expandable && expandedPaths.value.has(newPath)) {
      formatData(value, depth + 1, newPath, result);
    }
  });

  return result;
};

const formattedData = computed(() => formatData(props.data));

const toggleExpand = (path: string) => {
  if (expandedPaths.value.has(path)) {
    expandedPaths.value.delete(path);
  } else {
    expandedPaths.value.add(path);
  }
};
</script>

<style scoped>
.json-item:hover {
  background-color: rgba(0, 0, 0, 0.02);
}
</style>

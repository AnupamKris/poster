<template>
  <div class="space-y-4">
    <template v-for="(value, key) in data" :key="key">
      <div class="rounded-lg border bg-card">
        <div class="flex items-center justify-between p-4 border-b bg-muted/50">
          <h3 class="font-semibold capitalize">{{ formatKey(key) }}</h3>
          <Badge>{{ getValueType(value) }}</Badge>
        </div>
        <div class="p-4">
          <!-- Array handling -->
          <template v-if="Array.isArray(value)">
            <div v-if="canShowAsTable(value)" class="overflow-x-auto">
              <table class="w-full border-collapse">
                <thead>
                  <tr class="border-b">
                    <th
                      v-for="header in getTableHeaders(value)"
                      :key="header"
                      class="px-4 py-2 text-left font-medium text-muted-foreground"
                    >
                      {{ formatKey(header) }}
                    </th>
                  </tr>
                </thead>
                <tbody>
                  <tr
                    v-for="(item, index) in value"
                    :key="index"
                    class="border-b"
                  >
                    <td
                      v-for="header in getTableHeaders(value)"
                      :key="header"
                      class="px-4 py-2"
                    >
                      <template v-if="typeof item[header] === 'boolean'">
                        <div class="flex items-center">
                          <CheckIcon
                            v-if="item[header]"
                            class="w-4 h-4 text-green-500"
                          />
                          <XIcon v-else class="w-4 h-4 text-red-500" />
                        </div>
                      </template>
                      <template v-else>
                        {{ formatValue(item[header]) }}
                      </template>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
            <div v-else class="space-y-2">
              <div
                v-for="(item, index) in value"
                :key="index"
                class="p-2 rounded bg-muted/50"
              >
                <FriendlyJsonViewer
                  v-if="typeof item === 'object' && item !== null"
                  :data="item"
                />
                <span v-else>{{ formatValue(item) }}</span>
              </div>
            </div>
          </template>

          <!-- Object handling -->
          <template v-else-if="typeof value === 'object' && value !== null">
            <FriendlyJsonViewer :data="value" />
          </template>

          <!-- Simple value handling -->
          <template v-else>
            <div class="text-lg">
              <template v-if="typeof value === 'boolean'">
                <div class="flex items-center gap-2">
                  <CheckIcon v-if="value" class="w-5 h-5 text-green-500" />
                  <XIcon v-else class="w-5 h-5 text-red-500" />
                  {{ value ? "Yes" : "No" }}
                </div>
              </template>
              <template v-else>
                {{ formatValue(value) }}
              </template>
            </div>
          </template>
        </div>
      </div>
    </template>
  </div>
</template>

<script setup lang="ts">
import { Badge } from "@/components/ui/badge";
import { CheckIcon, XIcon } from "lucide-vue-next";

const props = defineProps<{
  data: any;
}>();

const formatKey = (key: string): string => {
  return key
    .split(/(?=[A-Z])|[-_]/)
    .map((word) => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
    .join(" ");
};

const getValueType = (value: any): string => {
  if (Array.isArray(value)) {
    return `List (${value.length} items)`;
  }
  if (typeof value === "object" && value !== null) {
    return "Details";
  }
  if (typeof value === "boolean") {
    return "Yes/No";
  }
  if (typeof value === "number") {
    return "Number";
  }
  return "Text";
};

const formatValue = (value: any): string => {
  if (value === null || value === undefined) return "Not provided";
  if (typeof value === "string" && value.match(/^https?:\/\//)) {
    return `Link: ${value}`;
  }
  if (typeof value === "number") {
    return value.toLocaleString();
  }
  return String(value);
};

const canShowAsTable = (arr: any[]): boolean => {
  if (arr.length === 0) return false;
  if (typeof arr[0] !== "object" || arr[0] === null) return false;
  const firstItemKeys = Object.keys(arr[0]);
  return arr.every(
    (item) =>
      typeof item === "object" &&
      item !== null &&
      Object.keys(item).length === firstItemKeys.length &&
      firstItemKeys.every((key) => key in item)
  );
};

const getTableHeaders = (arr: any[]): string[] => {
  if (arr.length === 0) return [];
  return Object.keys(arr[0]);
};
</script>

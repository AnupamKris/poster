<template>
  <div class="container mx-auto p-4 space-y-4">
    <div class="flex space-x-2 items-center">
      <Select v-model="method" :disabled="isLoading">
        <SelectTrigger class="w-32">
          <SelectValue placeholder="Method" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="GET">
            <span class="flex items-center gap-2">
              <DownloadIcon class="w-4 h-4" />
              GET
            </span>
          </SelectItem>
          <SelectItem value="POST">
            <span class="flex items-center gap-2">
              <UploadIcon class="w-4 h-4" />
              POST
            </span>
          </SelectItem>
        </SelectContent>
      </Select>
      <div class="flex-1 relative">
        <Input
          v-model="url"
          placeholder="Enter URL"
          class="w-full pr-24"
          :disabled="isLoading"
        />
        <div
          v-if="isLoading"
          class="absolute right-3 top-1/2 -translate-y-1/2 flex items-center gap-2 text-sm text-gray-500"
        >
          <div
            class="animate-spin rounded-full h-4 w-4 border-b-2 border-primary"
          ></div>
          {{ elapsedTime }}ms
        </div>
      </div>
      <Button
        @click="sendRequest"
        :disabled="isLoading || !url"
        class="w-24 h-10"
      >
        <div class="flex items-center gap-2">
          <SendIcon v-if="!isLoading" class="w-4 h-4" />
          <Loader2Icon v-else class="w-4 h-4 animate-spin" />
          {{ isLoading ? "Sending" : "Send" }}
        </div>
      </Button>
    </div>

    <Tabs v-model="activeTab" class="w-full">
      <TabsList class="grid w-full grid-cols-3">
        <TabsTrigger value="body" :disabled="isLoading">
          <span class="flex items-center gap-2">
            <FileTextIcon class="w-4 h-4" />
            Body
          </span>
        </TabsTrigger>
        <TabsTrigger value="headers" :disabled="isLoading">
          <span class="flex items-center gap-2">
            <ListIcon class="w-4 h-4" />
            Headers
          </span>
        </TabsTrigger>
        <TabsTrigger value="response">
          <span class="flex items-center gap-2">
            <ServerIcon class="w-4 h-4" />
            Response
          </span>
        </TabsTrigger>
      </TabsList>

      <TabsContent value="body">
        <div class="space-y-4">
          <Select v-model="bodyType" :disabled="isLoading || method === 'GET'">
            <SelectTrigger class="w-full">
              <SelectValue placeholder="Body Type" />
            </SelectTrigger>
            <SelectContent>
              <SelectItem value="json">
                <span class="flex items-center gap-2">
                  <BracketsIcon class="w-4 h-4" />
                  JSON
                </span>
              </SelectItem>
              <SelectItem value="form">
                <span class="flex items-center gap-2">
                  <FormInputIcon class="w-4 h-4" />
                  Form Data
                </span>
              </SelectItem>
            </SelectContent>
          </Select>

          <div v-if="bodyType === 'json'">
            <textarea
              v-model="jsonBody"
              class="w-full h-64 p-2 border rounded-md font-mono"
              placeholder="Enter JSON body"
              :disabled="isLoading || method === 'GET'"
            ></textarea>
          </div>

          <div v-if="bodyType === 'form'" class="space-y-2">
            <div
              v-for="(field, index) in formFields"
              :key="index"
              class="flex space-x-2"
            >
              <Select v-model="field.type" :disabled="isLoading">
                <SelectTrigger class="w-32">
                  <SelectValue placeholder="Type" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="text">
                    <span class="flex items-center gap-2">
                      <TypeIcon class="w-4 h-4" />
                      Text
                    </span>
                  </SelectItem>
                  <SelectItem value="file">
                    <span class="flex items-center gap-2">
                      <FileIcon class="w-4 h-4" />
                      File
                    </span>
                  </SelectItem>
                </SelectContent>
              </Select>
              <Input
                v-model="field.key"
                placeholder="Key"
                class="flex-1"
                :disabled="isLoading"
              />
              <div class="flex-1">
                <Input
                  v-if="field.type !== 'file'"
                  v-model="field.value"
                  placeholder="Value"
                  class="w-full"
                  :disabled="isLoading"
                />
                <input
                  v-else
                  type="file"
                  @change="(e) => handleFileChange(e, index)"
                  class="w-full px-3 py-2 border rounded-md"
                  :disabled="isLoading"
                />
              </div>
              <Button
                variant="destructive"
                @click="removeFormField(index)"
                class="w-24 h-10"
                :disabled="isLoading"
              >
                <span class="flex items-center gap-2">
                  <Trash2Icon class="w-4 h-4" />
                  Remove
                </span>
              </Button>
            </div>
            <Button
              @click="addFormField"
              class="w-full h-10"
              :disabled="isLoading"
            >
              <span class="flex items-center gap-2 justify-center">
                <PlusIcon class="w-4 h-4" />
                Add Field
              </span>
            </Button>
          </div>
        </div>
      </TabsContent>

      <TabsContent value="headers">
        <div class="space-y-2">
          <div
            v-for="(header, index) in headers"
            :key="index"
            class="flex space-x-2"
          >
            <Input
              v-model="header.key"
              placeholder="Header Name"
              class="flex-1"
              :disabled="isLoading"
            />
            <Input
              v-model="header.value"
              placeholder="Header Value"
              class="flex-1"
              :disabled="isLoading"
            />
            <Button
              variant="destructive"
              @click="removeHeader(index)"
              class="w-24 h-10"
              :disabled="isLoading"
            >
              <span class="flex items-center gap-2">
                <Trash2Icon class="w-4 h-4" />
                Remove
              </span>
            </Button>
          </div>
          <Button @click="addHeader" class="w-full h-10" :disabled="isLoading">
            <span class="flex items-center gap-2 justify-center">
              <PlusIcon class="w-4 h-4" />
              Add Header
            </span>
          </Button>
        </div>
      </TabsContent>

      <TabsContent value="response">
        <div v-if="response" class="space-y-4">
          <div class="grid grid-cols-3 gap-4">
            <div class="p-4 bg-gray-50 rounded-lg space-y-2">
              <div class="text-sm text-gray-500 flex items-center gap-2">
                <ActivityIcon class="w-4 h-4" />
                Status
              </div>
              <div
                class="text-2xl font-semibold"
                :class="{
                  'text-green-600':
                    response.status >= 200 && response.status < 300,
                  'text-amber-600':
                    response.status >= 300 && response.status < 400,
                  'text-red-600': response.status >= 400,
                }"
              >
                {{ response.status }}
              </div>
            </div>
            <div class="p-4 bg-gray-50 rounded-lg space-y-2">
              <div class="text-sm text-gray-500 flex items-center gap-2">
                <TimerIcon class="w-4 h-4" />
                Time
              </div>
              <div class="text-2xl font-semibold">
                {{ isLoading ? elapsedTime : requestTime }}ms
              </div>
            </div>
            <div class="p-4 bg-gray-50 rounded-lg space-y-2">
              <div class="text-sm text-gray-500 flex items-center gap-2">
                <LayoutTemplateIcon class="w-4 h-4" />
                View Mode
              </div>
              <Select v-model="responseViewMode">
                <SelectTrigger>
                  <SelectValue placeholder="View Mode" />
                </SelectTrigger>
                <SelectContent>
                  <SelectItem value="friendly">
                    <span class="flex items-center gap-2">
                      <TableIcon class="w-4 h-4" />
                      Friendly
                    </span>
                  </SelectItem>
                  <SelectItem value="formatted">
                    <span class="flex items-center gap-2">
                      <FolderTreeIcon class="w-4 h-4" />
                      Tree View
                    </span>
                  </SelectItem>
                  <SelectItem value="raw">
                    <span class="flex items-center gap-2">
                      <CodeIcon class="w-4 h-4" />
                      Raw
                    </span>
                  </SelectItem>
                </SelectContent>
              </Select>
            </div>
          </div>
          <div class="bg-white rounded-lg border">
            <FriendlyJsonViewer
              v-if="responseViewMode === 'friendly'"
              :data="response.data"
            />
            <JsonViewer
              v-else-if="responseViewMode === 'formatted'"
              :data="response.data"
            />
            <pre
              v-else
              class="p-4 font-mono text-sm overflow-auto max-h-[600px] whitespace-pre-wrap"
              >{{ JSON.stringify(response.data, null, 2) }}</pre
            >
          </div>
        </div>
        <div v-else-if="isLoading" class="flex items-center justify-center p-8">
          <div
            class="animate-spin rounded-full h-8 w-8 border-b-2 border-primary"
          ></div>
          <span class="ml-3">{{ elapsedTime }}ms</span>
        </div>
      </TabsContent>
    </Tabs>
  </div>
</template>

<script setup lang="ts">
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import { ref, onUnmounted } from "vue";
import axios from "axios";
import JsonViewer from "@/components/ui/json-viewer.vue";
import FriendlyJsonViewer from "@/components/ui/friendly-json-viewer.vue";
import {
  SendIcon,
  Loader2Icon,
  FileTextIcon,
  ListIcon,
  ServerIcon,
  BracketsIcon,
  FormInputIcon,
  TypeIcon,
  FileIcon,
  Trash2Icon,
  PlusIcon,
  ActivityIcon,
  TimerIcon,
  LayoutTemplateIcon,
  FolderTreeIcon,
  CodeIcon,
  DownloadIcon,
  UploadIcon,
  ChevronRightIcon,
  TableIcon,
} from "lucide-vue-next";

const method = ref("GET");
const url = ref("");
const activeTab = ref("body");
const bodyType = ref("json");
const jsonBody = ref("{\n  \n}");
const responseViewMode = ref("formatted");
const isLoading = ref(false);
const elapsedTime = ref(0);
let timerInterval: NodeJS.Timer | null = null;

interface FormField {
  key: string;
  value: string;
  type: "text" | "file";
  file?: File | null;
}

const formFields = ref<FormField[]>([
  { key: "", value: "", type: "text", file: null },
]);

const headers = ref<{ key: string; value: string }[]>([{ key: "", value: "" }]);
const response = ref<any>(null);
const requestTime = ref<number>(0);

const addFormField = () => {
  formFields.value.push({ key: "", value: "", type: "text", file: null });
};

const removeFormField = (index: number) => {
  formFields.value.splice(index, 1);
};

const addHeader = () => {
  headers.value.push({ key: "", value: "" });
};

const removeHeader = (index: number) => {
  headers.value.splice(index, 1);
};

const handleFileChange = (event: Event, index: number) => {
  const input = event.target as HTMLInputElement;
  if (input.files && input.files[0]) {
    formFields.value[index].file = input.files[0];
    formFields.value[index].value = input.files[0].name;
  }
};

const startTimer = () => {
  const startTime = performance.now();
  timerInterval = setInterval(() => {
    elapsedTime.value = Math.round(performance.now() - startTime);
  }, 10);
};

const stopTimer = () => {
  if (timerInterval) {
    clearInterval(timerInterval);
    timerInterval = null;
  }
};

const sendRequest = async () => {
  try {
    isLoading.value = true;
    startTimer();
    const startTime = performance.now();
    const config: any = {
      method: method.value,
      url: url.value,
      headers: headers.value.reduce((acc, header) => {
        if (header.key && header.value) {
          acc[header.key] = header.value;
        }
        return acc;
      }, {} as Record<string, string>),
    };

    if (method.value === "POST") {
      if (bodyType.value === "json") {
        try {
          config.data = JSON.parse(jsonBody.value);
        } catch (e) {
          alert("Invalid JSON");
          return;
        }
      } else {
        const formData = new FormData();
        formFields.value.forEach((field) => {
          if (field.key) {
            if (field.type === "file" && field.file) {
              formData.append(field.key, field.file);
            } else if (field.type === "text" && field.value) {
              formData.append(field.key, field.value);
            }
          }
        });
        config.data = formData;
      }
    }

    const result = await axios(config);
    const endTime = performance.now();
    requestTime.value = Math.round(endTime - startTime);
    response.value = result;
    activeTab.value = "response";
  } catch (error: any) {
    const endTime = performance.now();
    requestTime.value = Math.round(endTime - startTime);
    response.value = {
      status: error.response?.status || "Error",
      data: error.response?.data || error.message,
    };
    activeTab.value = "response";
  } finally {
    stopTimer();
    isLoading.value = false;
  }
};

// Cleanup timer on component unmount
onUnmounted(() => {
  stopTimer();
});
</script>

<style scoped>
.container {
  max-width: 1200px;
}
</style>

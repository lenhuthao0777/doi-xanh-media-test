<script setup lang="ts">
import { onMounted, ref, watch } from 'vue';
import Form from './components/form.vue';

type Task = {
  id: string;
  isDone: boolean;
  title: string | undefined;
  description: string;
  dueDate: string;
  priority: string;
};

const taskList = ref<Array<Task>>([]);

const tasks = ref<Array<Task>>([]);

const searchTerm = ref('');

const parseTaskData = () => {
  return JSON.parse(localStorage.getItem('tasks') as any) || [];
};

const syncTaskList = (key: string, data: any) => {
  localStorage.setItem(key, JSON.stringify(data));
};

const handleCheck = (event: { data: Task; checked: boolean }) => {
  if (event.checked) {
    tasks.value.push(event.data);
  } else {
    const idx = tasks.value.findIndex((item) => item.id === event.data.id);
    if (idx !== -1) {
      tasks.value.splice(idx, 1);
    }
  }
};

const handleCreate = (task: Task) => {
  taskList.value.push(task);
  syncTaskList('tasks', taskList.value);

  taskList.value = parseTaskData()
    .slice()
    .sort(
      (a: Task, b: Task) =>
        new Date(a.dueDate).getTime() - new Date(b.dueDate).getTime()
    );
};

const handleRemoveTask = (id: string) => {
  const idx = taskList.value.findIndex((item: Task) => item.id === id);
  const storageData = parseTaskData();
  const idxStorageData = storageData.findIndex((item: Task) => item.id === id);
  if (idx !== -1) {
    taskList.value.splice(idx, 1);
    storageData.splice(idxStorageData, 1);
    syncTaskList('tasks', storageData);
  }
};

const handleRemoveTasks = () => {
  const storageData = parseTaskData();
  for (let i = 0; i < tasks.value.length; i++) {
    const idx = taskList.value.findIndex(
      (item: Task) => item.id === tasks.value[i]?.id
    );
    const idxStorageData = storageData.findIndex(
      (item: Task) => item.id === tasks.value[i]?.id
    );
    taskList.value.splice(idx, 1);
    storageData.splice(idxStorageData, 1);
  }
  syncTaskList('tasks', storageData);
};

const handleUpdate = (data: Task) => {
  const storageData = parseTaskData();

  const ansStorageData = storageData.map((task: Task) =>
    task.id === data.id ? { ...task, ...data } : task
  );
  const ans = taskList.value.map((task: Task) =>
    task.id === data.id ? { ...task, ...data } : task
  );
  syncTaskList('tasks', ansStorageData);
  taskList.value = ans
    .slice()
    .sort(
      (a: Task, b: Task) =>
        new Date(a.dueDate).getTime() - new Date(b.dueDate).getTime()
    );
};

const handleSearch = () => {
  if (!searchTerm.value) {
    taskList.value = parseTaskData().slice()
    .sort(
      (a: Task, b: Task) =>
        new Date(a.dueDate).getTime() - new Date(b.dueDate).getTime()
    );;
  } else {
    const ans = parseTaskData().filter((task: Task) =>
      task.title?.toLowerCase().includes(searchTerm.value.toLowerCase())
    );
    taskList.value = ans;
  }
};

onMounted(() => {
  taskList.value = parseTaskData()
    .slice()
    .sort(
      (a: Task, b: Task) =>
        new Date(a.dueDate).getTime() - new Date(b.dueDate).getTime()
    );
  syncTaskList('tasks', taskList.value);
});
</script>

<template>
  <div class="grid grid-cols-2 gap-2 p-5">
    <Form @on-create="handleCreate" />
    <div class="relative">
      <div class="border border-gray-300 rounded-md shadow-md p-5">
        <h2 class="text-center text-2xl font-semibold mb-5">To do List</h2>

        <el-input
          size="large"
          style="width: 100%"
          placeholder="Search..."
          v-model="searchTerm"
          @input="handleSearch"
        />

        <ul class="mt-5 max-h-[600px] overflow-auto mb-[80px]">
          <template v-for="item in taskList" :key="item.id">
            <ListItem
              :data="item"
              @on-checked="handleCheck"
              @on-remove="handleRemoveTask"
              @on-update="handleUpdate"
            />
          </template>

          <li v-if="!taskList.length" class="text-center font-semibold">
            Empty Data...
          </li>
        </ul>

        <div
          class="flex items-center justify-between px-5 py-5 border border-gray-300 mt-5 w-full absolute bottom-0 right-0 bg-white z-10 rounded-b-md"
        >
          <span>Bulk Action:</span>
          <div class="space-x-2">
            <ElButton disabled type="primary"> Done </ElButton>
            <ElButton
              :disabled="!tasks.length"
              @click="handleRemoveTasks"
              type="danger"
            >
              Remove
            </ElButton>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>

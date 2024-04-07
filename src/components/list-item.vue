<script setup lang="ts">
import { ref } from 'vue';
import FormEdit from './form.vue';

type Task = {
  id: string;
  isDone: boolean;
  title: string | undefined;
  description: string;
  dueDate: string;
  priority: string;
};

const props = defineProps<{ data: Task }>();

const isOpen = ref(false);

const emit = defineEmits(['onChecked', 'onRemove', 'onUpdate']);

const handleCheck = (check: any) => {
  emit('onChecked', { data: props?.data, checked: check });
};

const handleUpdate = (data: Task) => {
  emit('onUpdate', data);
  isOpen.value = false;
};
</script>

<template>
  <li class="p-2 border border-gray-300 rounded-sm mb-2 last:mb-0">
    <div class="flex items-center justify-between" :class="isOpen && 'mb-5'">
      <div class="flex items-center space-x-2">
        <el-checkbox
          @change="handleCheck"
          v-model="props.data.isDone"
          :label="props.data.title"
          :style="props.data.isDone && 'text-decoration: line-through;'"
          size="large"
        />
      </div>

      <div class="flex items-center space-x-2">
        <ElButton
          type="primary"
          @click="
            () => {
              isOpen = !isOpen;
            }
          "
        >
          Detail
        </ElButton>
        <ElButton type="danger" @click="emit('onRemove', props.data.id)">
          Remove
        </ElButton>
      </div>
    </div>

    <FormEdit
      v-if="isOpen"
      is-edit
      :form-data="props.data"
      @on-update="handleUpdate"
    />
  </li>
</template>

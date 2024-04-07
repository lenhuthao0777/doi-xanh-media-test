<script setup lang="ts">
import { reactive, ref } from 'vue';
import moment from 'moment';
import { nanoid } from 'nanoid';
import type { FormInstance, FormRules } from 'element-plus';

type FormData = {
  id?: string;
  isDone?: boolean;
  title: string | undefined;
  description: string;
  dueDate: string;
  priority: string;
};

const props = defineProps<{ isEdit?: boolean; formData?: FormData }>();

const emit = defineEmits(['onCreate', 'onUpdate']);

const ruleFormRef = ref<FormInstance>();

const initialState: FormData = props.isEdit
  ? (props.formData as FormData)
  : {
      title: undefined,
      description: '',
      dueDate: moment(new Date()).format('YYYY-MM-DD'),
      priority: 'normal',
    };

const form = reactive({ ...initialState });

const minDate = (rule: any, value: any, callback: any) => {
  if (value < new Date()) {
    callback(new Error('Due date must be greater than current date'));
  } else {
    callback();
  }
};

const rules = reactive<FormRules<FormData>>({
  title: [
    { required: true, message: 'Please input Activity title', trigger: 'change' },
  ],
  dueDate: [
    {
      validator: minDate,
      trigger: 'change',
    },
  ],
});

const handleSubmit = async (ele: FormInstance | undefined) => {
  if (!ele) return;
  if (props.isEdit) {
    emit('onUpdate', form);
  } else {
    await ele.validate((valid, fields) => {
      if (valid) {
        emit('onCreate', {
          id: nanoid(),
          title: form.title,
          description: form.description,
          dueDate: moment(form.dueDate).format('YYYY-MM-DD'),
          isDone: false,
          priority: form.priority,
        });

        ele.resetFields();
      } else {
        console.log('error submit!', fields);
      }
    });
  }
};

const prioritySelect = [
  {
    value: 'low',
    label: 'Low',
  },
  {
    value: 'normal',
    label: 'Normal',
  },
  {
    value: 'high',
    label: 'High',
  },
];
</script>

<template>
  <div>
    <div class="border border-gray-300 p-5 rounded-md shadow-md">
      <h2 class="text-center mb-5 text-2xl font-semibold">New Task</h2>
      <el-form
        ref="ruleFormRef"
        :model="form"
        :rules="rules"
        label-width="auto"
        class="demo-ruleForm"
        status-icon
      >
        <el-form-item required prop="title">
          <el-input
            v-model="form.title"
            :disabled="props.formData?.isDone"
            size="large"
            style="width: 100%"
            placeholder="Add new task"
          />
        </el-form-item>

        <el-form-item prop="description">
          <label for="description">Description</label>
          <el-input
            id="description"
            :disabled="props.formData?.isDone"
            v-model="form.description"
            style="width: 100%"
            :rows="4"
            type="textarea"
            placeholder="Please enter description"
          />
        </el-form-item>

        <el-form-item>
          <el-col :span="11">
            <el-form-item required prop="dueDate">
              <label for="dueDate">Due Date</label>
              <el-date-picker
                :disabled="props.formData?.isDone"
                v-model="form.dueDate"
                id="dueDate"
                type="date"
                size="large"
                label="Pick a date"
                placeholder="Pick a date"
                style="width: 100%"
              />
            </el-form-item>
          </el-col>
          <el-col :span="2" />
          <el-col :span="11">
            <el-form-item prop="priority">
              <label for="priority">Priority</label>
              <el-select
                id="priority"
                :disabled="props.formData?.isDone"
                v-model="form.priority"
                placeholder="Select"
                size="large"
              >
                <el-option
                  v-for="item in prioritySelect"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value"
                />
              </el-select>
            </el-form-item>
          </el-col>
        </el-form-item>

        <el-button
          v-if="!props.isEdit"
          type="success"
          style="width: 100%"
          size="large"
          @click="handleSubmit(ruleFormRef)"
        >
          Create
        </el-button>
        <el-button
          v-if="props.isEdit"
          type="success"
          style="width: 100%"
          size="large"
          :disabled="props.formData?.isDone"
          @click="handleSubmit(ruleFormRef)"
        >
          Update
        </el-button>
      </el-form>
    </div>
  </div>
</template>

<style scoped></style>

<template>
  <el-dialog
    v-model="dialogVisible"
    title="提示"
    width="30%"
    :before-close="handleClose"
  >
    <span>{{props.msg}}</span>
    <template #footer>
      <span class="dialog-footer">
        <el-button type="primary" @click="confirm">确 定</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script setup>
import { ref,computed } from 'vue'

const props = defineProps(['show','msg'])
const emit = defineEmits(['update:show','closed'])

const dialogVisible = computed({
  get: () => props.show,
  set: (val) => emit('update:show', val)
})

const handleClose = (done) => {
  emit('closed')
  done()
}

const confirm = () => {
    emit('closed')
}
</script>

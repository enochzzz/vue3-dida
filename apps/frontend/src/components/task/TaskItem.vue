<script setup lang="ts">
import { NPopover } from 'naive-ui'
import type { Project, Task } from 'services/task'
import { changeTaskTitle } from 'services/task'
import { useTaskOperationMessage, useTaskRightContextMenu } from '@/composable'
import { TaskState, useTaskStore, useThemeStore } from '@/store'

interface Props {
  task: Task
  index: number
}

const props = defineProps<Props>()
const taskStore = useTaskStore()
const themeStore = useThemeStore()

const { showCompleteMessage } = useTaskOperationMessage()
const { showContextMenu } = useTaskRightContextMenu()

const checkboxColors: Record<TaskState, string> = {
  [TaskState.ACTIVE]: 'bg-#ccc',
  [TaskState.COMPLETED]: 'bg-#007A78',
  [TaskState.GIVE_UP]: 'bg-#FF2200',
  [TaskState.REMOVED]: 'bg-#ccc',
}

function handleRightClickTask(e: MouseEvent, task: Task) {
  taskStore.changeActiveTask(task)
  showContextMenu(e)
}

function handleClickTask(task: Task) {
  taskStore.changeActiveTask(task)
}

function handleInput(e: Event, task: Task) {
  const newTitle = (e.target as HTMLElement).innerText
  changeTaskTitle(task, newTitle)
}

function handleCompleteTodo(e: Event) {
  if (props.task.state === TaskState.ACTIVE) {
    handleCompleteTask()
  }
  else if (props.task.state === TaskState.COMPLETED) {
    taskStore.restoreTask(props.task)
  }
  else if (props.task.state === TaskState.REMOVED) {
    // eslint-disable-next-line no-console
    console.log('在垃圾桶里面的 task 不可以直接恢复')
  }
}

function handleCompleteTask() {
  taskStore.preCompleteTask(props.index)
  const undoCallback = (undoFlag: boolean) => {
    // 撤销时，执行撤销操作；未撤销时，执行完成的回调操作
    undoFlag ? taskStore.undoTask(props.task, props.index) : taskStore.completeTask(props.task)
  }
  showCompleteMessage(props.task, undoCallback)
}
</script>

<template>
  <div
    class="flex flex-row w-full items-center"
    @click.right="handleRightClickTask($event, task)"
  >
    <i
      class="cursor-move text-gray-200 dark:text-#3B3B3B flex-shrink-0 i-mdi-format-align-justify text-sm"
    />
    <div
      flex
      justify-start
      items-center
      gap-5px
      h-40px
      py-5px
      flex-1
      pl-10px
      :class="[
        themeStore.isDark ? 'hover:bg-[#474747]/50' : 'hover:bg-[#ECF1FF]/50',
        taskStore.currentActiveTask?.id === task.id
          ? themeStore.isDark
            ? '!bg-[#474747]'
            : '!bg-[#ECF1FF]'
          : '',
      ]"
    >
      <template v-if="task.state === TaskState.REMOVED">
        <!-- 临时加的提示 后面要去掉 -->
        <div class="flex justify-start items-center gap-5px">
          <NPopover trigger="hover">
            <template #trigger>
              <button
                :class="[checkboxColors[task.state]]"
                class="w-5 h-5 rounded-1"
                @click="handleCompleteTodo"
              />
            </template>
            <div>在垃圾桶里面的 Task 是不可以直接被恢复的哦</div>
          </NPopover>
        </div>
        <div class="w-full" @click="handleClickTask(task)">
          {{ task.title }}
        </div>
      </template>
      <template v-else>
        <button
          :class="[checkboxColors[task.state]]"
          class="w-5 h-5 rounded-1"
          @click="handleCompleteTodo"
        />
        <div
          class="w-full cursor-pointer focus:outline-0"
          contenteditable="true"
          @input="handleInput($event, task)"
          @focus="handleClickTask(task)"
        >
          {{ task.title }}
        </div>
      </template>
    </div>
  </div>
</template>

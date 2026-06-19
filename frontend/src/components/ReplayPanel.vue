<template>
  <div class="bg-gray-900 rounded-xl p-4 border border-gray-700">
    <div class="flex items-center justify-between mb-3">
      <h3 class="text-lg font-bold text-green-400">棋谱回放</h3>
      <div v-if="store.status !== 'replaying'" class="flex bg-gray-800 rounded-lg p-0.5">
        <button
          @click="activeTab = 'records'"
          class="px-3 py-1 text-xs rounded-md transition-colors"
          :class="activeTab === 'records' ? 'bg-green-600 text-white' : 'text-gray-400 hover:text-white'"
        >
          历史记录
        </button>
        <button
          @click="activeTab = 'trash'"
          class="px-3 py-1 text-xs rounded-md transition-colors relative"
          :class="activeTab === 'trash' ? 'bg-green-600 text-white' : 'text-gray-400 hover:text-white'"
        >
          回收站
          <span
            v-if="store.deletedRecords.length > 0"
            class="absolute -top-1 -right-1 min-w-4 h-4 px-1 bg-red-500 text-white text-[10px] rounded-full flex items-center justify-center"
          >
            {{ store.deletedRecords.length }}
          </span>
        </button>
      </div>
    </div>

    <div v-if="store.status !== 'replaying'">
      <div v-if="activeTab === 'records'">
        <p v-if="store.gameRecords.length === 0" class="text-gray-500 text-sm">暂无棋谱记录</p>
        <div v-else class="space-y-2 max-h-64 overflow-y-auto">
          <div
            v-for="record in store.gameRecords"
            :key="record.id"
            class="bg-gray-800 rounded-lg p-3 hover:bg-gray-700 transition-colors border border-gray-700"
          >
            <div
              class="cursor-pointer"
              @click="store.startReplay(record)"
            >
              <div class="flex justify-between items-center">
                <span class="text-sm text-gray-300">{{ record.createdAt }}</span>
                <span
                  class="text-xs px-2 py-0.5 rounded-full"
                  :class="record.winner === 1 ? 'bg-gray-700 text-gray-200' : record.winner === 2 ? 'bg-white text-black' : 'bg-yellow-600 text-white'"
                >
                  {{ record.winner === 1 ? '黑棋胜' : record.winner === 2 ? '白棋胜' : '平局' }}
                </span>
              </div>
              <div class="text-xs text-gray-500 mt-1">共 {{ record.moves.length }} 手</div>
            </div>
            <div class="flex justify-end mt-2">
              <button
                @click.stop="confirmDelete(record.id)"
                class="px-2 py-1 text-xs text-red-400 hover:text-red-300 hover:bg-red-600/20 rounded transition-colors"
                title="删除记录"
              >
                删除
              </button>
            </div>
          </div>
        </div>
      </div>

      <div v-else>
        <div v-if="store.deletedRecords.length > 0" class="flex justify-end mb-2">
          <button
            @click="confirmClearTrash"
            class="px-2 py-1 text-xs text-red-400 hover:text-red-300 hover:bg-red-600/20 rounded transition-colors"
          >
            清空回收站
          </button>
        </div>
        <p v-if="store.deletedRecords.length === 0" class="text-gray-500 text-sm">回收站为空</p>
        <div v-else class="space-y-2 max-h-64 overflow-y-auto">
          <div
            v-for="record in store.deletedRecords"
            :key="record.id"
            class="bg-gray-800/50 rounded-lg p-3 border border-gray-700/50"
          >
            <div class="flex justify-between items-center">
              <span class="text-sm text-gray-400">{{ record.createdAt }}</span>
              <span
                class="text-xs px-2 py-0.5 rounded-full opacity-70"
                :class="record.winner === 1 ? 'bg-gray-700 text-gray-200' : record.winner === 2 ? 'bg-white text-black' : 'bg-yellow-600 text-white'"
              >
                {{ record.winner === 1 ? '黑棋胜' : record.winner === 2 ? '白棋胜' : '平局' }}
              </span>
            </div>
            <div class="text-xs text-gray-500 mt-1">共 {{ record.moves.length }} 手</div>
            <div class="text-xs text-gray-600 mt-1">删除于 {{ record.deletedAt }}</div>
            <div class="flex justify-end gap-2 mt-2">
              <button
                @click="store.restoreRecord(record.id)"
                class="px-2 py-1 text-xs text-green-400 hover:text-green-300 hover:bg-green-600/20 rounded transition-colors"
                title="恢复记录"
              >
                恢复
              </button>
              <button
                @click="confirmPermanentDelete(record.id)"
                class="px-2 py-1 text-xs text-red-400 hover:text-red-300 hover:bg-red-600/20 rounded transition-colors"
                title="永久删除"
              >
                永久删除
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else>
      <div class="text-center mb-3">
        <span class="text-gray-400 text-sm">第 {{ store.replayIndex }} / {{ store.replayMoves.length }} 手</span>
      </div>

      <div class="w-full bg-gray-800 rounded-full h-2 mb-4">
        <div
          class="bg-green-500 h-2 rounded-full transition-all"
          :style="{ width: `${(store.replayIndex / store.replayMoves.length) * 100}%` }"
        />
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <button @click="store.replayGoToStart()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="回到开始">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 19l-7-7 7-7m8 14l-7-7 7-7"/></svg>
        </button>
        <button @click="store.replayStepBack()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="上一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
        </button>
        <button @click="store.toggleReplayPlay()" class="p-3 bg-green-600 rounded-lg hover:bg-green-500 text-white transition-colors" :title="store.isReplayPlaying ? '暂停' : '播放'">
          <svg v-if="!store.isReplayPlaying" class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M8 5v14l11-7z"/></svg>
          <svg v-else class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M6 4h4v16H6zM14 4h4v16h-4z"/></svg>
        </button>
        <button @click="store.replayStepForward()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="下一步">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/></svg>
        </button>
        <button @click="store.replayGoToEnd()" class="p-2 bg-gray-800 rounded-lg hover:bg-gray-700 text-gray-300 transition-colors" title="跳到结尾">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 5l7 7-7 7M5 5l7 7-7 7"/></svg>
        </button>
      </div>

      <div class="flex items-center justify-center gap-2 mb-4">
        <span class="text-xs text-gray-500">速度:</span>
        <button
          v-for="speed in speeds"
          :key="speed.value"
          @click="store.setReplaySpeed(speed.value)"
          class="px-2 py-1 text-xs rounded transition-colors"
          :class="store.replaySpeed === speed.value ? 'bg-green-600 text-white' : 'bg-gray-800 text-gray-400 hover:bg-gray-700'"
        >
          {{ speed.label }}
        </button>
      </div>

      <button
        @click="store.stopReplay()"
        class="w-full py-2 bg-red-600/20 border border-red-600/50 text-red-400 rounded-lg hover:bg-red-600/30 transition-colors text-sm"
      >
        退出回放
      </button>
    </div>

    <Teleport to="body">
      <div
        v-if="dialog.visible"
        class="fixed inset-0 bg-black/60 flex items-center justify-center z-50"
        @click.self="closeDialog"
      >
        <div class="bg-gray-900 border border-gray-700 rounded-xl p-5 w-80 shadow-2xl">
          <h4 class="text-lg font-bold text-white mb-2">{{ dialog.title }}</h4>
          <p class="text-sm text-gray-400 mb-5">{{ dialog.message }}</p>
          <div class="flex justify-end gap-2">
            <button
              @click="closeDialog"
              class="px-4 py-2 text-sm bg-gray-800 text-gray-300 rounded-lg hover:bg-gray-700 transition-colors"
            >
              取消
            </button>
            <button
              @click="confirmAction"
              class="px-4 py-2 text-sm bg-red-600 text-white rounded-lg hover:bg-red-500 transition-colors"
            >
              确认
            </button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import { useGameStore } from '../store/game';

const store = useGameStore();

const speeds = [
  { label: '慢', value: 2000 },
  { label: '中', value: 1000 },
  { label: '快', value: 500 },
  { label: '极快', value: 200 },
];

type TabType = 'records' | 'trash';
const activeTab = ref<TabType>('records');

type ActionType = 'softDelete' | 'permanentDelete' | 'clearTrash';

const dialog = reactive({
  visible: false,
  title: '',
  message: '',
  action: null as ActionType | null,
  targetId: null as string | null,
});

function openDialog(action: ActionType, id?: string) {
  dialog.action = action;
  dialog.targetId = id ?? null;
  if (action === 'softDelete') {
    dialog.title = '删除对局记录';
    dialog.message = '确认将此对局记录移入回收站？可在回收站中恢复。';
  } else if (action === 'permanentDelete') {
    dialog.title = '永久删除';
    dialog.message = '此操作不可恢复，确认永久删除该对局记录？';
  } else if (action === 'clearTrash') {
    dialog.title = '清空回收站';
    dialog.message = '此操作不可恢复，确认清空所有回收站记录？';
  }
  dialog.visible = true;
}

function closeDialog() {
  dialog.visible = false;
  dialog.action = null;
  dialog.targetId = null;
}

function confirmAction() {
  if (dialog.action === 'softDelete' && dialog.targetId) {
    store.softDeleteRecord(dialog.targetId);
  } else if (dialog.action === 'permanentDelete' && dialog.targetId) {
    store.permanentDeleteRecord(dialog.targetId);
  } else if (dialog.action === 'clearTrash') {
    store.clearDeletedRecords();
  }
  closeDialog();
}

function confirmDelete(id: string) {
  openDialog('softDelete', id);
}

function confirmPermanentDelete(id: string) {
  openDialog('permanentDelete', id);
}

function confirmClearTrash() {
  openDialog('clearTrash');
}
</script>

<template>
  <div class="schedule-panel card">
    <h3>📅 今日日程安排</h3>
    <p class="hint">为每位练习生选择活动，同活动一起训练可提升默契</p>

    <div class="schedule-list">
      <div
        v-for="trainee in schedulable"
        :key="trainee.id"
        class="schedule-row"
      >
        <span class="name">{{ trainee.name }}</span>
        <span v-if="trainee.illnessDays > 0" class="ill-tag">休养中</span>
        <div v-else class="activity-cols">
          <div class="activity-btns">
            <button
              v-for="(act, key) in basicActivities"
              :key="key"
              class="act-btn"
              :class="{ active: schedule[trainee.id] === key }"
              :title="`${act.label} ¥${act.moneyCost}`"
              @click="$emit('set', trainee.id, key)"
            >
              {{ act.icon }}
            </button>
          </div>
          <div class="activity-btns lang">
            <button
              v-for="(act, key) in languageActivities"
              :key="key"
              class="act-btn"
              :class="{ active: schedule[trainee.id] === key }"
              :title="`${act.label} ¥${act.moneyCost}`"
              @click="$emit('set', trainee.id, key)"
            >
              {{ act.icon }}
            </button>
          </div>
        </div>
        <span v-if="schedule[trainee.id]" class="chosen">
          {{ activities[schedule[trainee.id]]?.label }}
        </span>
      </div>
    </div>

    <div class="legend">
      <span v-for="(act, key) in basicActivities" :key="key" class="legend-item">
        {{ act.icon }} {{ act.label }}
      </span>
    </div>
    <div class="legend">
      <span class="legend-hint">🌐 语言集训：</span>
      <span v-for="(act, key) in languageActivities" :key="key" class="legend-item">
        {{ act.icon }} {{ act.label }}
      </span>
    </div>

    <div class="actions">
      <button class="btn ghost" @click="$emit('clear')">清空</button>
      <button class="btn primary" :disabled="!canEnd" @click="$emit('end-day')">
        结束今日 →
      </button>
    </div>
    <p v-if="!canEnd" class="warn">请为所有可安排的练习生选择日程</p>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { GAME_CONFIG } from '../config/gameConfig'

const props = defineProps({
  trainees: Array,
  schedule: Object,
  canEnd: Boolean,
})

defineEmits(['set', 'clear', 'end-day'])

const activities = GAME_CONFIG.activities

const basicActivities = computed(() => {
  const result = {}
  for (const [key, act] of Object.entries(activities)) {
    if (!key.startsWith('language_')) result[key] = act
  }
  return result
})

const languageActivities = computed(() => {
  const result = {}
  for (const [key, act] of Object.entries(activities)) {
    if (key.startsWith('language_')) result[key] = act
  }
  return result
})

const schedulable = computed(() =>
  props.trainees.filter((t) => t.status !== 'left')
)
</script>

<style scoped>
.schedule-panel h3 { margin-bottom: 0.25rem; }

.hint {
  font-size: 0.85rem;
  color: var(--text-muted);
  margin-bottom: 1rem;
}

.schedule-list {
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
  margin-bottom: 1rem;
}

.schedule-row {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  flex-wrap: wrap;
}

.name {
  width: 72px;
  font-weight: 600;
  font-size: 0.9rem;
}

.activity-cols {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
}

.activity-btns {
  display: flex;
  gap: 0.35rem;
}

.activity-btns.lang {
  padding-left: 0.2rem;
  border-left: 2px solid var(--border);
  padding-top: 0.2rem;
  margin-top: 0.2rem;
}

.act-btn {
  width: 36px;
  height: 36px;
  border: 1px solid var(--border);
  border-radius: 8px;
  background: var(--bg-secondary);
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.15s;
}

.act-btn:hover { border-color: var(--accent); }
.act-btn.active {
  background: var(--accent-soft);
  border-color: var(--accent);
  transform: scale(1.1);
}

.chosen {
  font-size: 0.8rem;
  color: var(--accent);
}

.ill-tag {
  font-size: 0.8rem;
  color: var(--warning);
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem 1rem;
  font-size: 0.75rem;
  color: var(--text-muted);
}

.legend + .legend {
  margin-top: 0.3rem;
  margin-bottom: 1rem;
}

.legend-hint {
  color: var(--accent);
  font-weight: 600;
}

.actions {
  display: flex;
  gap: 0.75rem;
  justify-content: flex-end;
}

.warn {
  text-align: right;
  font-size: 0.8rem;
  color: var(--warning);
  margin-top: 0.5rem;
}
</style>

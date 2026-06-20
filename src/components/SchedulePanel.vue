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
        <div v-else class="activity-wrapper">
          <div class="activity-group basic">
            <span class="group-tag">基础</span>
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
          </div>
          <div class="activity-group lang">
            <span class="group-tag">🌐 海外</span>
            <div class="activity-btns">
              <button
                v-for="(act, key) in languageActivities"
                :key="key"
                class="act-btn lang-btn"
                :class="{ active: schedule[trainee.id] === key }"
                :title="`${act.label} ¥${act.moneyCost}`"
                @click="$emit('set', trainee.id, key)"
              >
                {{ act.icon }}
              </button>
            </div>
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
      <span class="legend-hint">🌐 语言集训（为出道后海外宣传打基础）：</span>
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
  gap: 0.75rem;
  margin-bottom: 1rem;
}

.schedule-row {
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  flex-wrap: wrap;
  padding: 0.5rem 0;
  border-bottom: 1px dashed var(--border);
}

.schedule-row:last-child { border-bottom: none; }

.name {
  width: 72px;
  font-weight: 600;
  font-size: 0.9rem;
  padding-top: 0.3rem;
}

.activity-wrapper {
  display: flex;
  flex-direction: column;
  gap: 0.45rem;
  flex: 1;
  min-width: 0;
}

.activity-group {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.35rem 0.5rem;
  border-radius: 8px;
}

.activity-group.basic {
  background: var(--bg-secondary);
}

.activity-group.lang {
  background: var(--accent-soft);
  border: 1px dashed var(--accent);
}

.group-tag {
  font-size: 0.68rem;
  font-weight: 700;
  padding: 0.1rem 0.4rem;
  border-radius: 4px;
  background: var(--bg-card);
  color: var(--text-secondary);
  white-space: nowrap;
  flex-shrink: 0;
}

.activity-group.lang .group-tag {
  background: var(--accent);
  color: white;
}

.activity-btns {
  display: flex;
  gap: 0.35rem;
  flex-wrap: wrap;
}

.act-btn {
  width: 36px;
  height: 36px;
  border: 1px solid var(--border);
  border-radius: 8px;
  background: var(--bg-card);
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.15s;
}

.act-btn:hover { border-color: var(--accent); transform: translateY(-1px); }
.act-btn.active {
  background: var(--accent-soft);
  border-color: var(--accent);
  transform: scale(1.1);
  box-shadow: 0 0 0 2px var(--accent-soft);
}

.act-btn.lang-btn {
  border-color: var(--accent);
  background: var(--bg-card);
}
.act-btn.lang-btn.active {
  background: var(--accent);
  color: white;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}

.chosen {
  font-size: 0.8rem;
  color: var(--accent);
  font-weight: 600;
  padding-top: 0.35rem;
  white-space: nowrap;
}

.ill-tag {
  font-size: 0.8rem;
  color: var(--warning);
  padding-top: 0.3rem;
}

.legend {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem 1rem;
  font-size: 0.75rem;
  color: var(--text-muted);
}

.legend + .legend {
  margin-top: 0.35rem;
  margin-bottom: 1rem;
  padding: 0.4rem 0.6rem;
  background: var(--accent-soft);
  border-radius: 6px;
}

.legend-hint {
  color: var(--accent);
  font-weight: 700;
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

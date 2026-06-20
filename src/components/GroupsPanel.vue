<template>
  <div class="groups-panel card">
    <h3>🌟 出道组合</h3>
    <div v-if="groups.length === 0" class="empty">暂无出道组合</div>
    <div v-for="group in groups" :key="group.id" class="group-item">
      <div class="group-head">
        <strong>{{ group.name }}</strong>
        <span class="debut-day">第 {{ group.debutedDay }} 天出道</span>
      </div>
      <div class="members">
        {{ memberNames(group).join(' · ') }}
      </div>
      <div class="group-stats">
        <span>总销量 {{ group.totalSales.toLocaleString() }}</span>
        <span>{{ group.singles.length }} 张单曲</span>
      </div>

      <div class="overseas-section">
        <div class="section-title">🌏 海外热度（≥{{ heatThreshold }}解锁跨区日收益）</div>
        <div class="heat-list">
          <div v-for="(cfg, key) in overseasRegions" :key="key" class="heat-row">
            <span class="heat-icon">{{ cfg.icon }}</span>
            <span class="heat-label">{{ cfg.label }}</span>
            <div class="heat-bar">
              <div
                class="heat-fill"
                :class="{ active: (group.regionHeat?.[key] || 0) >= heatThreshold }"
                :style="{ width: (group.regionHeat?.[key] || 0) + '%' }"
              ></div>
            </div>
            <span class="heat-val" :class="{ unlocked: (group.regionHeat?.[key] || 0) >= heatThreshold }">
              {{ group.regionHeat?.[key] || 0 }}
            </span>
            <span
              v-if="revenueMap[group.id]?.details?.[key] > 0"
              class="heat-rev"
            >
              +¥{{ revenueMap[group.id].details[key].toLocaleString() }}/天
            </span>
            <span v-else class="heat-lang" :title="'成员平均' + regionLabels[key]">
              平均语言 {{ avgLanguage(group, key).toFixed(0) }}
            </span>
          </div>
        </div>
        <div v-if="revenueMap[group.id]?.daily > 0" class="daily-rev">
          ✨ 跨区日收益 <strong>¥{{ revenueMap[group.id].daily.toLocaleString() }}</strong>
        </div>
        <div class="promo-btns">
          <button
            v-for="(cfg, key) in overseasRegions"
            :key="key"
            class="btn sm"
            :class="canPromote(group, key) ? 'accent' : 'ghost-disabled'"
            :disabled="!canPromote(group, key)"
            :title="promoteTip(group, key)"
            @click="$emit('promote-overseas', group.id, key)"
          >
            {{ cfg.icon }} {{ cfg.label }}宣传
          </button>
        </div>
        <div class="promo-cost">
          宣传费 ¥{{ promoCost.toLocaleString() }} · 冷却{{ promoCooldown }}天 · 语言≥{{ minLangLevel }}
        </div>
      </div>

      <button
        class="btn primary sm"
        :disabled="money < singleCost"
        @click="$emit('release', group.id)"
      >
        发行单曲 (¥{{ singleCost.toLocaleString() }})
      </button>
    </div>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { GAME_CONFIG } from '../config/gameConfig'

const props = defineProps({
  groups: Array,
  trainees: Array,
  money: Number,
  getGroupOverseasRevenue: { type: Function, default: () => ({ daily: 0, details: {} }) },
})

defineEmits(['release', 'promote-overseas'])

const singleCost = GAME_CONFIG.single.creationCost
const overseasRegions = GAME_CONFIG.overseasRegions
const regionLabels = GAME_CONFIG.regionLabels
const promoCost = GAME_CONFIG.overseasPromotion.baseCost
const minLangLevel = GAME_CONFIG.overseasPromotion.minLanguageLevel
const heatThreshold = GAME_CONFIG.overseasRevenue.heatThreshold
const promoCooldown = GAME_CONFIG.overseasPromotion.cooldownDays

const revenueMap = computed(() => {
  const map = {}
  for (const g of props.groups || []) {
    map[g.id] = props.getGroupOverseasRevenue(g.id)
  }
  return map
})

function memberNames(group) {
  return group.memberIds
    .map((id) => props.trainees.find((t) => t.id === id)?.name)
    .filter(Boolean)
}

function avgLanguage(group, region) {
  const members = (props.trainees || []).filter((t) => group.memberIds.includes(t.id))
  if (members.length === 0) return 0
  return members.reduce((s, m) => s + (m.languages?.[region] || 0), 0) / members.length
}

function canPromote(group, region) {
  if ((props.money || 0) < promoCost) return false
  if (avgLanguage(group, region) < minLangLevel) return false
  return true
}

function promoteTip(group, region) {
  const tips = []
  if ((props.money || 0) < promoCost) tips.push('资金不足')
  const avg = avgLanguage(group, region)
  if (avg < minLangLevel) tips.push(`平均语言${avg.toFixed(0)}/${minLangLevel}`)
  return tips.length > 0 ? tips.join(' · ') : `在${overseasRegions[region].label}宣传，积累地区热度`
}
</script>

<style scoped>
.groups-panel h3 { margin-bottom: 0.75rem; }

.empty {
  color: var(--text-muted);
  font-size: 0.9rem;
}

.group-item {
  padding: 0.75rem;
  background: var(--bg-secondary);
  border-radius: 8px;
  margin-bottom: 0.75rem;
}

.group-head {
  display: flex;
  justify-content: space-between;
  margin-bottom: 0.35rem;
}

.debut-day {
  font-size: 0.75rem;
  color: var(--text-muted);
}

.members {
  font-size: 0.85rem;
  color: var(--text-secondary);
  margin-bottom: 0.35rem;
}

.group-stats {
  display: flex;
  gap: 1rem;
  font-size: 0.8rem;
  color: var(--text-muted);
  margin-bottom: 0.75rem;
}

.overseas-section {
  margin-bottom: 0.75rem;
  padding: 0.6rem;
  background: var(--bg-card);
  border-radius: 6px;
  border: 1px solid var(--border);
}

.section-title {
  font-size: 0.8rem;
  font-weight: 600;
  color: var(--accent);
  margin-bottom: 0.5rem;
}

.heat-list {
  display: flex;
  flex-direction: column;
  gap: 0.35rem;
  margin-bottom: 0.5rem;
}

.heat-row {
  display: flex;
  align-items: center;
  gap: 0.35rem;
  font-size: 0.75rem;
}

.heat-icon { font-size: 0.9rem; }

.heat-label {
  width: 48px;
  color: var(--text-muted);
}

.heat-bar {
  flex: 1;
  height: 5px;
  background: var(--bg-secondary);
  border-radius: 3px;
  overflow: hidden;
}

.heat-fill {
  height: 100%;
  border-radius: 3px;
  background: var(--text-muted);
  transition: width 0.3s;
}

.heat-fill.active { background: var(--accent); }

.heat-val {
  width: 26px;
  text-align: right;
  font-weight: 600;
  color: var(--text-secondary);
}

.heat-val.unlocked {
  color: var(--accent);
  font-weight: 800;
}

.heat-lang {
  font-size: 0.68rem;
  color: var(--text-muted);
  background: var(--bg-secondary);
  padding: 0.1rem 0.35rem;
  border-radius: 4px;
}

.heat-rev {
  color: var(--accent);
  font-weight: 700;
  font-size: 0.72rem;
  background: var(--accent-soft);
  padding: 0.1rem 0.4rem;
  border-radius: 4px;
}

.daily-rev {
  font-size: 0.8rem;
  color: var(--text-secondary);
  margin-bottom: 0.5rem;
  padding: 0.25rem 0.5rem;
  background: var(--accent-soft);
  border-radius: 4px;
  display: inline-block;
}

.daily-rev strong {
  color: var(--accent);
}

.promo-btns {
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-bottom: 0.35rem;
}

.btn.accent {
  background: var(--accent);
  color: white;
  border: 1px solid var(--accent);
}

.btn.accent:hover { filter: brightness(1.1); }

.btn.ghost-disabled {
  background: transparent;
  color: var(--text-muted);
  border: 1px solid var(--border);
  cursor: not-allowed;
  opacity: 0.5;
}

.promo-cost {
  font-size: 0.7rem;
  color: var(--text-muted);
}
</style>

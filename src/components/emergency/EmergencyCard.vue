<script setup lang="ts">
import { computed } from 'vue'
import type { FamilyMember } from '../../types'
import { calculateAge } from '../../utils/format'
import Avatar from '../ui/Avatar.vue'

const props = defineProps<{ member: FamilyMember }>()
const emit = defineEmits<{ (e: 'complete'): void }>()

const contact = computed(() => props.member.emergencyContact.trim())
const telHref = computed(() => `tel:${contact.value.replace(/[^\d+]/g, '')}`)
const callable = computed(() => contact.value.replace(/[^\d+]/g, '').length > 0)

const bloodKnown = computed(
  () => !!props.member.bloodType && props.member.bloodType !== '未知',
)
</script>

<template>
  <div class="emergency-card card">
    <div class="card-head">
      <Avatar :src="member.avatar" :name="member.name" :size="52" />
      <div class="head-info">
        <div class="name">{{ member.name }}</div>
        <div class="meta">
          {{ member.relation }} · {{ member.dob ? calculateAge(member.dob) + '岁' : '年龄未知' }}
        </div>
      </div>
      <span class="blood-badge" :class="{ 'blood-unknown': !bloodKnown }">
        {{ member.bloodType || '未知' }}
      </span>
    </div>

    <div class="info-rows">
      <div class="info-row" :class="{ 'info-alert': member.allergies }">
        <span class="info-label">过敏史</span>
        <span class="info-value">{{ member.allergies || '无' }}</span>
      </div>
      <div class="info-row">
        <span class="info-label">慢性病史</span>
        <span class="info-value">{{ member.chronicDiseases || '无' }}</span>
      </div>
    </div>

    <div class="contact-area">
      <a v-if="callable" class="btn btn-danger call-btn" :href="telHref">
        📞 一键呼叫 {{ contact }}
      </a>
      <template v-else>
        <button
          type="button"
          class="btn btn-danger call-btn"
          disabled
          title="未填写紧急联系人，无法拨号"
        >
          📞 一键呼叫
        </button>
        <div class="missing-tip">
          <span>⚠️ 未填写紧急联系人，无法一键拨号</span>
          <button type="button" class="btn btn-ghost btn-sm" @click="emit('complete')">
            去补全
          </button>
        </div>
      </template>
    </div>
  </div>
</template>

<style scoped>
.emergency-card {
  display: flex;
  flex-direction: column;
  gap: 14px;
  margin-bottom: 0;
}
.card-head {
  display: flex;
  align-items: center;
  gap: 12px;
}
.head-info {
  flex: 1;
  min-width: 0;
}
.name {
  font-weight: 700;
  font-size: 17px;
  color: var(--text-primary);
}
.meta {
  font-size: 13px;
  color: var(--text-secondary);
}
.blood-badge {
  min-width: 52px;
  padding: 6px 10px;
  border-radius: 10px;
  background: #fdecea;
  color: var(--danger-color);
  font-weight: 800;
  font-size: 18px;
  text-align: center;
}
.blood-unknown {
  background: var(--bg-color);
  color: var(--text-secondary);
  font-weight: 600;
}
.info-rows {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.info-row {
  display: flex;
  gap: 10px;
  padding: 8px 12px;
  border-radius: 8px;
  background: var(--bg-color);
  font-size: 14px;
}
.info-label {
  color: var(--text-secondary);
  min-width: 60px;
  flex-shrink: 0;
}
.info-value {
  color: var(--text-primary);
  word-break: break-all;
}
.info-alert {
  background: #fdecea;
}
.info-alert .info-value {
  color: #c0392b;
  font-weight: 600;
}
.contact-area {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.call-btn {
  display: block;
  width: 100%;
  padding: 12px;
  font-size: 15px;
  text-align: center;
  text-decoration: none;
}
.call-btn:disabled {
  opacity: 0.45;
  cursor: not-allowed;
}
.missing-tip {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  padding: 8px 12px;
  border-radius: 8px;
  background: #fef5e7;
  border: 1px solid #f9e79f;
  color: #b9770e;
  font-size: 13px;
}
</style>

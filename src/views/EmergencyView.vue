<script setup lang="ts">
import { computed, ref } from 'vue'
import EmergencyCard from '../components/emergency/EmergencyCard.vue'
import MemberForm from '../components/member/MemberForm.vue'
import BaseModal from '../components/ui/BaseModal.vue'
import EmptyState from '../components/ui/EmptyState.vue'
import { useFamilyStore } from '../stores/useFamilyStore'
import type { FamilyMember } from '../types'

const store = useFamilyStore()

const members = computed(() => store.state.members)
const editing = ref<FamilyMember | null>(null)

const missingContacts = computed(() =>
  members.value.filter((m) => !m.emergencyContact.trim()),
)

function openComplete(member: FamilyMember) {
  editing.value = member
}

function onSave(data: Omit<FamilyMember, 'id' | 'metrics'>) {
  if (editing.value) store.updateMember(editing.value.id, data)
  editing.value = null
}
</script>

<template>
  <div class="page">
    <div class="page-head">
      <h1 class="page-title">家庭急救卡</h1>
      <p class="page-sub">全员血型、过敏史与紧急联系人一览，突发情况无需逐个翻档案</p>
    </div>

    <a class="sos-banner" href="tel:120">
      <span class="sos-icon">🚑</span>
      <span class="sos-text">
        <strong>急救电话 120</strong>
        <small>情况危急时点击直接拨打</small>
      </span>
      <span class="sos-action">立即拨打</span>
    </a>

    <div v-if="missingContacts.length" class="missing-banner">
      ⚠️ {{ missingContacts.length }} 位成员未填写紧急联系人：{{
        missingContacts.map((m) => m.name).join('、')
      }}，请点击对应卡片上的「去补全」完善信息。
    </div>

    <div v-if="members.length" class="card-grid">
      <EmergencyCard
        v-for="m in members"
        :key="m.id"
        :member="m"
        @complete="openComplete(m)"
      />
    </div>
    <EmptyState v-else icon="🚑" text="还没有家庭成员，请先在「家庭成员」页添加" />

    <BaseModal v-if="editing" title="补全急救信息" @close="editing = null">
      <MemberForm :member="editing" @save="onSave" @close="editing = null" />
    </BaseModal>
  </div>
</template>

<style scoped>
.page-head {
  margin-bottom: 16px;
}
.page-title {
  margin: 0 0 4px;
}
.page-sub {
  margin: 0;
  font-size: 14px;
  color: var(--text-secondary);
}
.sos-banner {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 16px 20px;
  border-radius: var(--radius);
  background: var(--danger-color);
  color: #fff;
  text-decoration: none;
  margin-bottom: 16px;
  box-shadow: var(--shadow);
  transition: opacity 0.2s;
}
.sos-banner:hover {
  opacity: 0.92;
}
.sos-icon {
  font-size: 32px;
}
.sos-text {
  flex: 1;
  display: flex;
  flex-direction: column;
  line-height: 1.4;
}
.sos-text strong {
  font-size: 18px;
}
.sos-text small {
  opacity: 0.85;
}
.sos-action {
  background: #fff;
  color: var(--danger-color);
  font-weight: 700;
  padding: 8px 18px;
  border-radius: 20px;
  flex-shrink: 0;
}
.missing-banner {
  padding: 12px 16px;
  border-radius: 10px;
  margin-bottom: 16px;
  font-size: 14px;
  background: #fef5e7;
  color: #b9770e;
  border: 1px solid #f9e79f;
}
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(320px, 1fr));
  gap: 16px;
}
</style>

<script setup lang="ts">
import { computed, ref } from 'vue'
import Avatar from '../components/ui/Avatar.vue'
import BaseModal from '../components/ui/BaseModal.vue'
import EmptyState from '../components/ui/EmptyState.vue'
import MemberForm from '../components/member/MemberForm.vue'
import { useFamilyStore } from '../stores/useFamilyStore'
import type { FamilyMember } from '../types'
import { calculateAge } from '../utils/format'

const store = useFamilyStore()

const members = computed(() => store.state.members)
const missingContacts = computed(() =>
  members.value.filter((m) => !m.emergencyContact.trim()),
)

const hotlines = [
  { name: '医疗急救', number: '120', primary: true },
  { name: '公安报警', number: '110', primary: false },
  { name: '消防救援', number: '119', primary: false },
]

function hasContact(member: FamilyMember): boolean {
  return member.emergencyContact.trim().length > 0
}

/** Keep only characters valid in a tel: URI. */
function toTel(raw: string): string {
  return `tel:${raw.trim().replace(/[^\d+*-]/g, '')}`
}

// ---- missing-contact feedback (dialing is blocked, click is not silent) ----
const toastMsg = ref('')
let toastTimer: ReturnType<typeof setTimeout> | undefined

function notifyMissing(member: FamilyMember) {
  toastMsg.value = `「${member.name}」的紧急联系人电话未填写，请先补全后再拨号`
  clearTimeout(toastTimer)
  toastTimer = setTimeout(() => (toastMsg.value = ''), 2600)
}

// ---- inline completion via the same member form used on the members page ----
const editing = ref<FamilyMember | null>(null)

function openEdit(member: FamilyMember) {
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
      <div>
        <h1 class="page-title">家庭急救卡</h1>
        <p class="page-desc">突发情况一页看全：血型、过敏史、慢性病史与紧急联系人</p>
      </div>
    </div>

    <!-- Public emergency numbers -->
    <section class="card hotline-card">
      <div class="hotline-intro">
        <span class="hotline-icon">🚑</span>
        <div>
          <h3>公共紧急电话</h3>
          <p>任何成员发生紧急情况时，可先拨打公共救援电话</p>
        </div>
      </div>
      <div class="hotline-actions">
        <a
          v-for="h in hotlines"
          :key="h.number"
          :href="`tel:${h.number}`"
          class="hotline-btn"
          :class="{ primary: h.primary }"
        >
          <span class="hotline-name">{{ h.name }}</span>
          <span class="hotline-number">{{ h.number }}</span>
        </a>
      </div>
    </section>

    <template v-if="members.length">
      <!-- Completeness warning -->
      <div v-if="missingContacts.length" class="banner" role="alert">
        <span class="banner-icon">⚠️</span>
        <div>
          <strong>{{ missingContacts.length }} 位成员尚未填写紧急联系人电话</strong>
          <span class="banner-detail">
            （{{ missingContacts.map((m) => m.name).join('、') }}），危急时刻可能无法联系到家人，请尽快补全。
          </span>
        </div>
      </div>

      <!-- Member emergency cards -->
      <div class="em-grid">
        <section v-for="m in members" :key="m.id" class="card em-card">
          <header class="em-head">
            <Avatar :src="m.avatar" :name="m.name" :size="52" />
            <div class="em-id">
              <div class="em-name">{{ m.name }}</div>
              <div class="em-tags">
                <span class="tag">{{ m.relation }}</span>
                <span class="tag">{{ calculateAge(m.dob) }}岁</span>
              </div>
            </div>
            <div class="blood" :class="{ unknown: m.bloodType === '未知' }">
              <span class="blood-label">血型</span>
              <span class="blood-type">{{ m.bloodType }}</span>
            </div>
          </header>

          <dl class="em-body">
            <div class="info-row">
              <dt class="info-label">过敏史</dt>
              <dd class="info-value" :class="{ alert: m.allergies }">
                {{ m.allergies || '无' }}
              </dd>
            </div>
            <div class="info-row">
              <dt class="info-label">慢性病史</dt>
              <dd class="info-value">{{ m.chronicDiseases || '无' }}</dd>
            </div>
          </dl>

          <footer class="em-foot">
            <div class="contact">
              <span class="contact-label">紧急联系人</span>
              <a v-if="hasContact(m)" :href="toTel(m.emergencyContact)" class="contact-number">
                {{ m.emergencyContact }}
              </a>
              <span v-else class="contact-missing">未填写，请补全联系人电话</span>
            </div>

            <a v-if="hasContact(m)" :href="toTel(m.emergencyContact)" class="call-btn">
              <span class="call-icon">📞</span>一键拨号
            </a>
            <div v-else class="call-group">
              <button
                type="button"
                class="call-btn call-disabled"
                aria-disabled="true"
                @click="notifyMissing(m)"
              >
                <span class="call-icon">📵</span>一键拨号
              </button>
              <button type="button" class="btn btn-ghost btn-sm fill-btn" @click="openEdit(m)">
                补全信息
              </button>
            </div>
          </footer>
        </section>
      </div>
    </template>

    <EmptyState v-else icon="🆘" text="还没有家庭成员，先去「家庭成员」页面建档" />

    <!-- Inline completion -->
    <BaseModal
      v-if="editing"
      :title="`补全紧急信息 · ${editing.name}`"
      @close="editing = null"
    >
      <MemberForm :member="editing" @save="onSave" @close="editing = null" />
    </BaseModal>

    <!-- Toast for tapping the disabled dial button -->
    <Transition name="toast">
      <div v-if="toastMsg" class="toast" role="alert">{{ toastMsg }}</div>
    </Transition>
  </div>
</template>

<style scoped>
.page-head {
  margin-bottom: 20px;
}
.page-desc {
  color: var(--text-secondary);
  font-size: 14px;
  margin-top: 4px;
}

/* Hotline card */
.hotline-card {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
  border-left: 4px solid var(--danger-color);
}
.hotline-intro {
  display: flex;
  align-items: center;
  gap: 12px;
}
.hotline-intro h3 {
  margin: 0;
}
.hotline-intro p {
  color: var(--text-secondary);
  font-size: 13px;
  margin: 0;
}
.hotline-icon {
  font-size: 30px;
}
.hotline-actions {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}
.hotline-btn {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-width: 92px;
  padding: 10px 18px;
  border-radius: 10px;
  text-decoration: none;
  border: 1px solid var(--border-color);
  background: var(--bg-color);
  color: var(--text-primary);
  transition: transform 0.15s, box-shadow 0.15s;
}
.hotline-btn:active {
  transform: scale(0.96);
}
.hotline-btn.primary {
  background: var(--danger-color);
  border-color: var(--danger-color);
  color: #fff;
  box-shadow: 0 4px 12px rgba(231, 76, 60, 0.3);
}
.hotline-name {
  font-size: 13px;
}
.hotline-number {
  font-size: 22px;
  font-weight: 800;
  letter-spacing: 1px;
}

/* Missing-contacts banner */
.banner {
  display: flex;
  align-items: flex-start;
  gap: 10px;
  background: #fef6e7;
  border: 1px solid #f7dcab;
  color: #8a5b13;
  border-radius: var(--radius);
  padding: 12px 16px;
  margin-bottom: 20px;
  font-size: 14px;
}
.banner-icon {
  font-size: 18px;
  line-height: 1.5;
}
.banner strong {
  color: #7a4f0c;
}
.banner-detail {
  color: #9c7a3a;
}

/* Member cards */
.em-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(330px, 1fr));
  gap: 14px;
}
.em-card {
  display: flex;
  flex-direction: column;
  margin-bottom: 0;
  padding: 18px;
}
.em-head {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 14px;
}
.em-id {
  flex: 1;
  min-width: 0;
}
.em-name {
  font-size: 18px;
  font-weight: 700;
  color: var(--text-primary);
}
.em-tags {
  display: flex;
  gap: 6px;
  margin-top: 2px;
  flex-wrap: wrap;
}
.tag {
  background: var(--accent-bg);
  color: var(--accent-color);
  padding: 1px 9px;
  border-radius: 10px;
  font-size: 12px;
}
.blood {
  text-align: center;
  background: #fdecea;
  border: 1px solid #f5c6c0;
  border-radius: 10px;
  padding: 4px 12px;
  flex-shrink: 0;
}
.blood.unknown {
  background: var(--bg-color);
  border-color: var(--border-color);
}
.blood-label {
  display: block;
  font-size: 11px;
  color: var(--text-secondary);
  line-height: 1.3;
}
.blood-type {
  display: block;
  font-size: 17px;
  font-weight: 800;
  color: var(--danger-color);
  line-height: 1.3;
}
.blood.unknown .blood-type {
  color: var(--text-secondary);
  font-weight: 600;
}

.em-body {
  flex: 1;
  margin: 0 0 14px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.info-row {
  display: flex;
  gap: 10px;
  font-size: 14px;
  align-items: baseline;
}
.info-label {
  color: var(--text-secondary);
  min-width: 60px;
  flex-shrink: 0;
}
.info-value {
  margin: 0;
  color: var(--text-primary);
  word-break: break-all;
}
.info-value.alert {
  color: var(--danger-color);
  font-weight: 700;
}

.em-foot {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding-top: 12px;
  border-top: 1px solid var(--border-color);
  flex-wrap: wrap;
}
.contact {
  display: flex;
  flex-direction: column;
  min-width: 0;
}
.contact-label {
  font-size: 12px;
  color: var(--text-secondary);
}
.contact-number {
  font-size: 17px;
  font-weight: 700;
  color: var(--info-color);
  text-decoration: none;
}
.contact-number:active {
  text-decoration: underline;
}
.contact-missing {
  font-size: 13px;
  color: var(--warning-color);
  font-weight: 600;
}

.call-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 10px 18px;
  border-radius: 8px;
  border: none;
  background: var(--danger-color);
  color: #fff;
  font-weight: 700;
  font-size: 15px;
  text-decoration: none;
  cursor: pointer;
  box-shadow: 0 4px 10px rgba(231, 76, 60, 0.28);
  transition: transform 0.15s, opacity 0.15s;
}
.call-btn:active {
  transform: scale(0.96);
}
.call-icon {
  font-size: 16px;
}
.call-group {
  display: flex;
  align-items: center;
  gap: 8px;
}
.call-disabled {
  background: #bdc3c7;
  box-shadow: none;
  cursor: not-allowed;
}
.call-disabled:active {
  transform: none;
}
.fill-btn {
  white-space: nowrap;
}

/* Toast */
.toast {
  position: fixed;
  left: 50%;
  bottom: 36px;
  transform: translateX(-50%);
  background: rgba(44, 62, 80, 0.95);
  color: #fff;
  padding: 12px 20px;
  border-radius: 10px;
  font-size: 14px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.25);
  z-index: 2000;
  max-width: min(90vw, 460px);
  text-align: center;
}
.toast-enter-active,
.toast-leave-active {
  transition: opacity 0.25s, transform 0.25s;
}
.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translate(-50%, 10px);
}
</style>

<script lang="ts" setup>
import type { mastodon } from 'masto'

definePageMeta({
  middleware: 'auth',
})

const { t } = useI18n()

const client = useMastoClient()

const paginator = client.v1.lists.list()

useHeadFixed({
  title: () => t('nav.lists'),
})

const paginatorRef = ref()
let busy = $ref<boolean>(false)
const createText = ref('')
const enableSubmit = computed(() => createText.value.length > 0)

async function createList() {
  if (busy || !enableSubmit.value)
    return

  busy = true
  await nextTick()
  try {
    const newEntry = await client.v1.lists.create({
      title: createText.value,
    })
    paginatorRef.value.createEntry(newEntry)
    createText.value = ''
  }
  finally {
    busy = false
  }
}
function updateEntry(list: mastodon.v1.List) {
  paginatorRef.value.updateEntry(list)
}
function removeEntry(id: string) {
  paginatorRef.value.removeEntry(id)
}
</script>

<template>
  <MainContent>
    <template #title>
      <NuxtLink to="/lists" timeline-title-style flex items-center gap-2 @click="$scrollToTop">
        <div i-ri:list-check />
        <span text-lg font-bold>{{ t('nav.lists') }}</span>
      </NuxtLink>
    </template>
    <slot>
      <CommonPaginator ref="paginatorRef" :paginator="paginator">
        <template #default="{ item }">
          <ListEntry
            :list="item"
            @list-updated="updateEntry"
            @list-removed="removeEntry"
          />
        </template>
        <template #done>
          <form
            border="t base"
            p-4 w-full
            flex="~ wrap" relative gap-3
            @submit.prevent="createList"
          >
            <div
              bg-base border="~ base" flex-1 h10 ps-1 pe-4 rounded-2 w-full flex="~ row"
              items-center relative focus-within:box-shadow-outline gap-3
            >
              <input
                v-model="createText"
                bg-transparent
                outline="focus:none"
                px-4
                pb="1px"
                flex-1
                placeholder-text-secondary
                :placeholder="$t('list.list_title_placeholder')"
                @keypress.enter="createList"
              >
            </div>
            <div flex="~ col" gap-y-4 gap-x-2 sm="~ justify-between flex-row">
              <button flex="~ row" gap-x-2 items-center btn-solid :disabled="!enableSubmit || busy">
                <span v-if="busy" aria-hidden="true" block animate animate-spin preserve-3d class="rtl-flip">
                  <span block i-ri:loader-2-fill aria-hidden="true" />
                </span>
                <span v-else aria-hidden="true" block i-material-symbols:playlist-add-rounded class="rtl-flip" />
                {{ $t('list.create') }}
              </button>
            </div>
          </form>
        </template>
      </CommonPaginator>
    </slot>
  </MainContent>
</template>

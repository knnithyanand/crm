<template>
  <LayoutHeader>
    <template #left-header>
      <Breadcrumbs :items="[{ label: list.title }]" />
    </template>
    <template #right-header>
      <Button variant="solid" label="Create" @click="createNote">
        <template #prefix><FeatherIcon name="plus" class="h-4" /></template>
      </Button>
    </template>
  </LayoutHeader>
  <div class="border-b"></div>
  <div v-if="notes.data?.length" class="grid grid-cols-4 gap-4 p-5 overflow-y-auto">
    <div
      v-for="note in notes.data"
      class="group flex flex-col justify-between gap-2 px-5 py-4 border rounded-lg h-56 shadow-sm hover:bg-gray-50 cursor-pointer"
      @click="editNote(note)"
    >
      <div class="flex items-center justify-between">
        <div class="text-lg font-medium truncate">
          {{ note.title }}
        </div>
        <Dropdown
          :options="[
            {
              icon: 'trash-2',
              label: 'Delete',
              onClick: () => deleteNote(note.name),
            },
          ]"
          @click.stop
        >
          <Button
            icon="more-horizontal"
            variant="ghosted"
            class="hover:bg-white"
          />
        </Dropdown>
      </div>
      <TextEditor
        v-if="note.content"
        :content="note.content"
        :editable="false"
        editor-class="!prose-sm max-w-none !text-sm text-gray-600 focus:outline-none"
        class="flex-1 overflow-hidden"
      />
      <div class="flex items-center justify-between gap-2 mt-2">
        <div class="flex items-center gap-2">
          <UserAvatar :user="note.owner" size="xs" />
          <div class="text-sm text-gray-800">
            {{ getUser(note.owner).full_name }}
          </div>
        </div>
        <Tooltip :text="dateFormat(note.modified, dateTooltipFormat)">
          <div class="text-sm text-gray-700">
            {{ timeAgo(note.modified) }}
          </div>
        </Tooltip>
      </div>
    </div>
  </div>
  <div
    v-else
    class="flex-1 p-5 flex items-center justify-center font-medium text-xl text-gray-500"
  >
    <div class="flex flex-col items-center gap-2">
      <NoteIcon class="w-10 h-10 text-gray-500" />
      <span>No notes</span>
    </div>
  </div>
  <NoteModal
    v-model="showNoteModal"
    :note="currentNote"
    @updateNote="updateNote"
  />
</template>

<script setup>
import LayoutHeader from '@/components/LayoutHeader.vue'
import Breadcrumbs from '@/components/Breadcrumbs.vue'
import UserAvatar from '@/components/UserAvatar.vue'
import NoteIcon from '@/components/Icons/NoteIcon.vue'
import NoteModal from '@/components/NoteModal.vue'
import { timeAgo, dateFormat, dateTooltipFormat } from '@/utils'
import {
  FeatherIcon,
  Button,
  createListResource,
  TextEditor,
  TextInput,
  call,
  Dropdown,
Tooltip,
} from 'frappe-ui'
import { ref } from 'vue'
import { usersStore } from '@/stores/users'

const { getUser } = usersStore()

const list = {
  title: 'Notes',
  plural_label: 'Notes',
  singular_label: 'Note',
}

const showNoteModal = ref(false)
const currentNote = ref(null)

const notes = createListResource({
  type: 'list',
  doctype: 'CRM Note',
  cache: 'Notes',
  fields: ['name', 'title', 'content', 'owner', 'modified'],
  filters: {},
  orderBy: 'modified desc',
  pageLength: 20,
  auto: true,
})

function createNote() {
  currentNote.value = {
    title: '',
    content: '',
  }
  showNoteModal.value = true
}

function editNote(note) {
  currentNote.value = note
  showNoteModal.value = true
}

async function updateNote(note) {
  if (note.name) {
    let d = await call('frappe.client.set_value', {
      doctype: 'CRM Note',
      name: note.name,
      fieldname: note,
    })
    if (d.name) {
      notes.reload()
    }
  } else {
    let d = await call('frappe.client.insert', {
      doc: {
        doctype: 'CRM Note',
        title: note.title,
        content: note.content,
      },
    })
    if (d.name) {
      notes.reload()
    }
  }
}

async function deleteNote(name) {
  await call('frappe.client.delete', {
    doctype: 'CRM Note',
    name,
  })
  notes.reload()
}
</script>
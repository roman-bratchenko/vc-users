<script>
export default {
  props: {
    thumbnail: String,
    name: Object,
    email: String,
    selectedUser: String
  },

  computed: {
    isSelected() {
      return `${this.name.last}-${this.name.first}` === this.selectedUser
    }
  }
}
</script>

<template>
  <div class="flex flex-col items-center m-4 md:flex-row">
    <img class="avatar-md mr-2 rounded w-15 h-15" :src="thumbnail" />
    <div class="ml-3">
      <p class="text-md font-medium text-gray-900">{{name.first}} {{name.last}}</p>
      <p class="text-md text-gray-500">
        <a :href="`mailto:${email}`" class="hover:text-sky-500">{{email}}</a>
      </p>
    </div>
    <button type="button" @click="$emit('toggle-user')" class="ml-auto border rounded px-3 py-1 hover:border-slate-400 active:bg-slate-100 active:border-slate-300 active:text-slate-600">
      <span v-if="isSelected">Hide Details</span>
      <span v-else>Show Details</span>
    </button>
  </div>
</template>
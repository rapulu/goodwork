<template>
  <div class="container mx-auto px-4 my-6 w-full md:w-md lg:w-lg xl:w-xl xxl:w-2xl">
    <div class="text-grey-dark font-semibold text-2xl mb-4 flex items-center justify-center">
      {{office.name}}
      <span @click="toggleDropdownMenu" v-click-outside="closeDropdownMenu" class="bg-white p-1 text-sm rounded-full shadow ml-4 cursor-pointer flex items-center">
        <font-awesome-icon :icon="faCog"></font-awesome-icon>
      </span>
      <div v-if="dropdownMenuShown" class="relative">
        <ul class="list-reset bg-white rounded shadow-lg py-2 absolute pin-r mt-4 text-base text-left font-normal whitespace-no-wrap z-10">
          <li class="px-4 py-2 hover:bg-grey-light cursor-pointer">
            <a href="#" class="no-underline text-grey-dark" @click.prevent="showMembersListModal">
              Show All Members
            </a>
          </li>
          <li class="px-4 py-2 hover:bg-grey-light cursor-pointer">
            Delete
          </li>
        </ul>
      </div>
    </div>

    <div class="text-grey-dark flex flex-row justify-center items-center">
      <span class="text-lg">
        Cycle: 
      </span>
      <span class="p-2 ml-2 bg-grey-lightest shadow rounded cursor-pointer text-sm text-teal-darker">
        {{office.current_cycle.start_date}} - {{office.current_cycle.end_date}}
      </span>
    </div>

  <!-- Modals for dropdown menu -->
    <members-list-modal resourceType="office" :resourceId="office.id" :show="membersListModalShown" :members="office.members" @close="closeMembersListModal" />

    <addMemberForm v-if="addMemberFormShown" @close="closeAddMemberForm" resourceType="office" :resource="office" @addMember="addMember"></addMemberForm>

    <show-github-repo entityType="team" :entityId="team.id" v-if="githubRepoModalShown" @close-github-repo-modal="closeGithubRepoModal"></show-github-repo>
  <!-- Modals for dropdown menu -->

    <div class="h-16 flex flex-row justify-center items-center px-2">
      <span @click="showAddMemberForm" class="bg-white shadow w-8 h-8 rounded-full text-teal hover:cursor-pointer text-center p-2">
        <font-awesome-icon :icon="faPlus"></font-awesome-icon>
      </span>
      <a v-for="(member, index) in office.members" :href="'/users/' + member.username" class="pl-2">
        <profile-card :user="member"></profile-card>
      </a>
      <span v-if="office.members.length > 5" class="bg-grey-lighter border-teal border p-2 rounded-full">{{ office.members.length - 5 }}+</span>
    </div>

    <tab-menu :active="active" @activate="activateTab"></tab-menu>

    <div class="flex flex-row flex-wrap justify-center">
      <taskBoard resourceType="office" :resource="office"  :activeTab="active"></taskBoard>
      <discussionBoard resourceType="office" :resource="office" :activeTab="active"></discussionBoard>
      <messagesBoard resourceType="office" :resource="office" :activeTab="active"></messagesBoard>
      <event-board resourceType="office" :resource="office" :activeTab="active"></event-board>
      <file-board resourceType="office" :resource="office" :activeTab="active"></file-board>
      <!-- <taskBoard resourceType="projects" :resource="project"></taskBoard>
      <activity resourceType="projects" :resource="project"></activity> -->
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import taskBoard from './../partials/taskBoard.vue'
import discussionBoard from './../partials/discussionBoard.vue'
import messagesBoard from './../partials/messagesBoard.vue'
import eventBoard from './../partials/eventBoard.vue'
import fileBoard from './../partials/fileBoard.vue'
import activity from './../partials/activity.vue'
import addMemberForm from './../partials/addMemberForm.vue'
import showGithubRepo from './../partials/showGithubRepo.vue'
import membersListModal from './../partials/membersListModal.vue'
import profileCard from './../partials/profileCard.vue'
import tabMenu from './../partials/tabMenu.vue'
import { faPlus } from '@fortawesome/free-solid-svg-icons/faPlus'
import { faCog } from '@fortawesome/free-solid-svg-icons/faCog'

export default {
  components: {
    taskBoard,
    discussionBoard,
    messagesBoard,
    eventBoard,
    fileBoard,
    activity,
    addMemberForm,
    membersListModal,
    profileCard,
    tabMenu,
    showGithubRepo
  },
  data: () => ({
    addMemberFormShown: false,
    active: 'tasks',
    dropdownMenuShown: false,
    githubRepoModalShown: false,
    membersListModalShown: false,
    faPlus,
    faCog
  }),
  created () {
    let tabs = ['tasks', 'discussions', 'messages', 'events', 'files', 'activities']
    let tool = new URL(location.href).searchParams.get('tool')
    if (tool !== null && tabs.indexOf(tool) !== -1) {
      this.active = tool
    }
  },
  computed: {
    ...mapState({
      office: state => state.office
    })
  },
  methods: {
    showAddMemberForm () {
      this.addMemberFormShown = true
    },
    closeAddMemberForm () {
      this.addMemberFormShown = false
    },
    addMember (data) {
      let messageType
      if (data.user) {
        this.message = data.message
        messageType = 'success'
        this.office.members.push(data.user)
      } else {
        messageType = 'error'
        this.message = data.message
      }
      this.addMemberFormShown = false
      EventBus.$emit('notification', data.message, messageType)
    },
    showMembersListModal () {
      this.membersListModalShown = true
    },
    closeMembersListModal () {
      this.membersListModalShown = false
    },
    activateTab (tab) {
      if (tab !== this.active) {
        this.active = tab
      }
    },
    toggleDropdownMenu () {
      this.dropdownMenuShown = !this.dropdownMenuShown
    },
    closeDropdownMenu () {
      this.dropdownMenuShown = false
    },
    showGithubRepoModal () {
      this.githubRepoModalShown = true
    },
    closeGithubRepoModal () {
      this.githubRepoModalShown = false
    }
  }
}
</script>

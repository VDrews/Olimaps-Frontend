<template lang="pug">
#chat(v-if='chatLoaded', style='position: relative; width: 100%; height: 100%')
  v-toolbar(
    flat,
    color='text',
    style='position: absolute; top: 0; left: 0; right: 0; z-index: 10'
  )
    v-btn.mr-2(icon, dark, small, @click='$router.replace({ path: "/chat" })')
      v-icon fas fa-chevron-left
    v-btn.px-0.text-capitalize(text, dark)
      v-avatar(
        @click='$router.push({ path: "/" + member.username })',
        color='primary'
      )
        v-img(:src='member.profile_picture')
      .ml-2.background--text.font-weight-bold(
        @click='$router.push({ path: "/" + member.username })'
      ) {{ member.username }}
    v-spacer
    v-menu(offset-y)
      template(v-slot:activator='{ on }')
        v-btn(icon, dark, v-on='on')
          v-icon fas fa-ellipsis-v
      v-card
        v-list
          v-list-item(v-if='!chat.muted', @click='mute')
            v-list-item-avatar
              v-icon fas fa-volume-mute
            v-list-item-content Silenciar Chat
          v-list-item(v-else, @click='unmute')
            v-list-item-avatar
              v-icon fas fa-volume
            v-list-item-content Activar Notificaciones
          v-list-item 
            v-list-item-avatar
              v-icon fas fa-ban
            v-list-item-content Bloquear Usuario
  v-sheet(color='white')
    messages
  div(style='position: absolute; bottom: 0px; left: 0; right: 0')
    chat-bar(:chat='chat')
</template>

<script>
import messages from '@/components/chat/messages'
import chatBar from '@/components/chat/chatBar'

export default {
  components: {
    messages,
    chatBar,
  },
  beforeRouteLeave(_, __, next) {
    this.$store.commit('chat/closeChat')
    next()
  },
  data() {
    return {
      chatLoaded: false,
    }
  },
  computed: {
    user() {
      return this.$store.state.auth.user
    },
    chat() {
      return this.$store.state.chat.chat
    },
    member() {
      if (!this.chatLoaded) return {}
      if (
        this.$store.state.chat.chat.member[0].profile.profile_id !==
        this.$store.state.auth.user.profile_id
      ) {
        return this.$store.state.chat.chat.member[0].profile
      } else {
        return this.$store.state.chat.chat.member[1].profile
      }
    },
  },
  methods: {
    async mute() {
      if (await this.$store.dispatch('chat/muteChat', this.chat.chat_id)) {
        this.chat.muted = true
      }
    },
    async unmute() {
      if (await this.$store.dispatch('chat/unmuteChat', this.chat.chat_id)) {
        this.chat.muted = false
      }
    },
  },
  async mounted() {
    await this.$store.dispatch('chat/getChat', this.$route.params.chat)
    this.chatLoaded = true
    await this.$store.dispatch('chat/getMessages', {
      chatId: this.$store.state.chat.chat.chat_id,
    })
  },
}
</script>

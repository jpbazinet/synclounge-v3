<template>
  <v-list-item class="message-item">
    <template #prepend>
      <v-avatar size="32" class="message-avatar">
        <v-img :src="sender.thumb" />
      </v-avatar>
    </template>
    <v-list-item-title class="message-username">
      {{ sender.username }}
      <span class="message-time ml-1">{{ formattedTime }}</span>
    </v-list-item-title>
    <v-list-item-subtitle>
      <!-- eslint-disable-next-line vue/no-v-html -->
      <div class="message-content" v-html="processedText" />
    </v-list-item-subtitle>
  </v-list-item>
</template>

<script>
import { mapGetters } from 'vuex';

const URL_PATTERN = /https?:\/\/[^\s<>"']+/g;

function escapeHtml(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}

export default {
  name: 'MessageItem',
  props: {
    message: {
      type: Object,
      required: true,
    },
  },
  computed: {
    ...mapGetters('synclounge', [
      'GET_MESSAGES_USER_CACHE_USER',
    ]),
    sender() {
      return this.GET_MESSAGES_USER_CACHE_USER(this.message.senderId);
    },
    formattedTime() {
      if (!this.message.time) return '';
      const d = new Date(this.message.time);
      return d.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
    },
    processedText() {
      const text = this.message.text || '';
      if (text.startsWith('data:image/')) {
        const style = 'max-width:100%;max-height:300px;border-radius:4px;display:block;margin-top:4px;';
        return `<img src="${text}" style="${style}" />`;
      }
      const safe = escapeHtml(text);
      const emojiOnly = /^[\p{Emoji_Presentation}\p{Extended_Pictographic}\s]+$/u.test(safe.trim());
      const linked = safe.replace(URL_PATTERN, (url) => {
        const attrs = `href="${url}" target="_blank" rel="noopener noreferrer" class="chat-link"`;
        return `<a ${attrs}>${url}</a>`;
      });
      if (emojiOnly) return `<span style="font-size:25px;line-height:1.2">${linked}</span>`;
      return linked;
    },
  },
};
</script>

<style scoped>
.message-item {
  padding: 4px 8px !important;
  align-items: flex-start !important;
}
.message-item + .message-item {
  border-top: 1px solid rgb(255 255 255 / 5%);
}
.message-avatar {
  box-shadow: 0 0 0 2px rgb(229 160 13 / 50%) !important;
  align-self: flex-start;
  margin-top: 2px;
}
:deep(.v-list-item__prepend) {
  padding-inline-end: 8px !important;
  align-self: flex-start !important;
}
.message-username {
  color: #f0a020 !important;
  font-weight: 700 !important;
  font-size: 12px !important;
  letter-spacing: 0.01em !important;
  margin-bottom: 2px !important;
  white-space: normal !important;
  overflow: visible !important;
  text-overflow: unset !important;
}
.message-time {
  color: rgb(255 255 255 / 40%);
  font-weight: 400;
  font-size: 11px;
}
.message-content {
  font-size: 14px !important;
  font-weight: 400 !important;
  line-height: 1.5 !important;
  color: rgb(255 255 255 / 85%) !important;
  white-space: normal !important;
  word-break: break-word !important;
  overflow-wrap: anywhere !important;
}
:deep(.v-list-item-subtitle) {
  white-space: normal !important;
  overflow: visible !important;
  text-overflow: unset !important;
  opacity: 1 !important;
  -webkit-line-clamp: unset !important;
}
:deep(.chat-link) {
  color: inherit;
  text-decoration: underline;
  word-break: break-all;
}
:deep(.chat-link:hover) {
  opacity: 0.75;
}
:deep(.v-list-item__content) {
  overflow: visible !important;
  text-overflow: unset !important;
  white-space: normal !important;
}
</style>

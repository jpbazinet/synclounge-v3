<template>
  <v-text-field
    v-model="messageToBeSent"
    append-inner-icon="mdi-send"
    :label="chatboxLabel"
    hide-details
    single-line
    density="compact"
    variant="outlined"
    rounded
    maxlength="500"
    counter
    class="mx-2 my-1 chat-input"
    @click:append-inner="sendMessage"
    @keyup.enter="sendMessage"
    @paste="handlePaste"
  />
</template>

<script>
import { mapActions } from 'vuex';

export default {
  name: 'MessageInput',

  data: () => ({
    messageToBeSent: '',
  }),

  computed: {
    chatboxLabel() {
      return 'Message';
    },
  },

  methods: {
    ...mapActions('synclounge', [
      'SEND_MESSAGE',
    ]),

    async sendMessage() {
      const trimmed = this.messageToBeSent.trim();
      if (!trimmed) return;
      await this.SEND_MESSAGE(trimmed);
      this.messageToBeSent = '';
    },

    handlePaste(event) {
      const items = Array.from((event.clipboardData && event.clipboardData.items) || []);
      const imageItem = items.find((item) => item.type.startsWith('image/'));
      if (!imageItem) return;
      event.preventDefault();
      const file = imageItem.getAsFile();
      const reader = new FileReader();
      reader.onload = (ev) => {
        const img = new Image();
        img.onload = () => {
          const MAX = 300;
          let { width, height } = img;
          if (width > MAX || height > MAX) {
            if (width > height) {
              height = Math.round((height * MAX) / width);
              width = MAX;
            } else {
              width = Math.round((width * MAX) / height);
              height = MAX;
            }
          }
          const canvas = document.createElement('canvas');
          canvas.width = width;
          canvas.height = height;
          canvas.getContext('2d').drawImage(img, 0, 0, width, height);
          this.SEND_MESSAGE(canvas.toDataURL('image/jpeg', 0.75));
        };
        img.src = ev.target.result;
      };
      reader.readAsDataURL(file);
    },
  },
};
</script>

<style scoped>
.chat-input {
  font-size: 14px;
}

:deep(.v-field) {
  border-radius: 24px !important;
  min-height: 36px;
  padding-top: 0;
  padding-bottom: 0;
}

:deep(.v-field__input) {
  font-size: 14px;
  padding-top: 6px;
  padding-bottom: 6px;
}
</style>

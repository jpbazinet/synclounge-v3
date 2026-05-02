<template>
  <v-list
    ref="listEl"
    density="compact"
    @scroll.passive="onScroll"
  >
    <v-list-subheader class="md-4">
      Chat
    </v-list-subheader>

    <MessageItem
      v-for="msg in GET_MESSAGES"
      :key="`${msg.senderId}-${msg.time}`"
      :message="msg"
    />
  </v-list>
</template>

<script>
import { defineAsyncComponent } from 'vue';
import { mapGetters } from 'vuex';

export default {
  name: 'MessageList',

  components: {
    MessageItem: defineAsyncComponent(() => import('@/components/MessageItem.vue')),
  },

  data: () => ({
    userScrolledUp: false,
  }),

  computed: {
    ...mapGetters('synclounge', [
      'GET_MESSAGES',
    ]),
  },

  watch: {
    GET_MESSAGES() {
      if (!this.userScrolledUp) {
        this.$nextTick(this.scrollToBottom);
      }
    },
  },

  mounted() {
    this.$nextTick(this.scrollToBottom);
  },

  methods: {
    scrollToBottom() {
      const el = this.$refs.listEl?.$el ?? this.$el;
      if (el) {
        el.scrollTop = el.scrollHeight;
      }
    },
    onScroll(event) {
      const el = event.target;
      if (!el) return;
      const distanceFromBottom = el.scrollHeight - el.scrollTop - el.clientHeight;
      this.userScrolledUp = distanceFromBottom > 50;
    },
  },
};
</script>

<template>
  <div class="message-input-wrapper">
    <v-menu
      v-model="emojiPickerOpen"
      :close-on-content-click="false"
      top
      offset-y
      nudge-top="8"
      max-width="320"
    >
      <template #activator="{ props }">
        <v-btn
          icon
          variant="text"
          density="compact"
          class="emoji-btn ml-1"
          v-bind="props"
          title="Emoji"
        >
          <span class="emoji-trigger">&#x1F642;</span>
        </v-btn>
      </template>
      <v-card class="emoji-picker pa-2">
        <div class="emoji-search mb-1">
          <v-text-field
            v-model="emojiSearch"
            dense
            hide-details
            placeholder="Search emoji..."
            prepend-inner-icon="mdi-magnify"
            outlined
            @click.stop
          />
        </div>
        <div class="emoji-grid">
          <button
            v-for="e in filteredEmojis"
            :key="e.emoji"
            type="button"
            class="emoji-item"
            :title="e.name"
            @click.stop="insertEmoji(e.emoji)"
          >
            {{ e.emoji }}
          </button>
        </div>
      </v-card>
    </v-menu>
    <v-text-field
      ref="messageInput"
      v-model="messageToBeSent"
      label="Message"
      hide-details
      single-line
      density="compact"
      variant="outlined"
      rounded
      class="ml-1 flex-input"
      @keyup.enter="sendMessage"
      @keyup.space="convertEmoticons"
      @paste="handlePaste"
    />
    <v-btn
      icon
      variant="text"
      density="compact"
      class="send-btn mx-1"
      title="Send"
      @click="sendMessage"
    >
      <v-icon size="20">mdi-send</v-icon>
    </v-btn>
  </div>
</template>

<script>
import { mapActions } from 'vuex';

const EMOJIS = [
  { emoji: '\u{1F600}', name: 'grinning' },
  { emoji: '\u{1F602}', name: 'joy' },
  { emoji: '\u{1F923}', name: 'rofl' },
  { emoji: '\u{1F60D}', name: 'heart eyes' },
  { emoji: '\u{1F970}', name: 'smiling face with hearts' },
  { emoji: '\u{1F60E}', name: 'cool' },
  { emoji: '\u{1F914}', name: 'thinking' },
  { emoji: '\u{1F62D}', name: 'crying' },
  { emoji: '\u{1F631}', name: 'scream' },
  { emoji: '\u{1F92F}', name: 'exploding head' },
  { emoji: '\u{1F624}', name: 'triumph' },
  { emoji: '\u{1F973}', name: 'partying' },
  { emoji: '\u{1F634}', name: 'sleeping' },
  { emoji: '\u{1F92E}', name: 'vomiting' },
  { emoji: '\u{1F62C}', name: 'grimacing' },
  { emoji: '\u{1F644}', name: 'eye roll' },
  { emoji: '\u{1F60F}', name: 'smirk' },
  { emoji: '\u{1F921}', name: 'clown' },
  { emoji: '\u{1F44D}', name: 'thumbs up' },
  { emoji: '\u{1F44E}', name: 'thumbs down' },
  { emoji: '\u{1F44F}', name: 'clap' },
  { emoji: '\u{1F64C}', name: 'raised hands' },
  { emoji: '\u{1F91D}', name: 'handshake' },
  { emoji: '\u262E\uFE0F', name: 'peace' },
  { emoji: '\u{1F91E}', name: 'fingers crossed' },
  { emoji: '\u{1F4AA}', name: 'muscle' },
  { emoji: '\u{1FAF6}', name: 'heart hands' },
  { emoji: '\u2764\uFE0F', name: 'heart' },
  { emoji: '\u{1F525}', name: 'fire' },
  { emoji: '\u{1F480}', name: 'skull' },
  { emoji: '\u{1F4A9}', name: 'poop' },
  { emoji: '\u{1F440}', name: 'eyes' },
  { emoji: '\u{1F389}', name: 'party' },
  { emoji: '\u{1F37F}', name: 'popcorn' },
  { emoji: '\u{1F3AC}', name: 'clapper' },
  { emoji: '\u{1F4FD}\uFE0F', name: 'film projector' },
  { emoji: '\u{1F355}', name: 'pizza' },
  { emoji: '\u{1F37A}', name: 'beer' },
  { emoji: '\u2615', name: 'coffee' },
  { emoji: '\u{1F926}', name: 'facepalm' },
  { emoji: '\u{1F937}', name: 'shrug' },
  { emoji: '\u{1F4AF}', name: '100' },
  { emoji: '\u2728', name: 'sparkles' },
  { emoji: '\u{1F606}', name: 'laughing' },
  { emoji: '\u{1F605}', name: 'sweat smile' },
  { emoji: '\u{1FAE0}', name: 'melting' },
  { emoji: '\u{1F971}', name: 'yawning' },
  { emoji: '\u{1F610}', name: 'neutral' },
];

// Ordered longest-first so e.g. :'( matches before :(
const EMOTICONS = [
  [":'(", '\u{1F622}'],
  ['>:(', '\u{1F620}'],
  ['</3', '\u{1F494}'],
  [':D', '\u{1F604}'],
  [':P', '\u{1F61B}'],
  [':p', '\u{1F61B}'],
  [';)', '\u{1F609}'],
  [':)', '\u{1F60A}'],
  [':]', '\u{1F60A}'],
  [':(', '\u{1F61E}'],
  [':[', '\u{1F61E}'],
  [':|', '\u{1F610}'],
  [':/', '\u{1F615}'],
  [':o', '\u{1F62E}'],
  [':O', '\u{1F62E}'],
  [':*', '\u{1F618}'],
  ['B)', '\u{1F60E}'],
  ['<3', '\u2764\uFE0F'],
  ['XD', '\u{1F602}'],
  ['xD', '\u{1F602}'],
  ['^_^', '\u{1F601}'],
  ['^^', '\u{1F601}'],
  ['-_-', '\u{1F611}'],
  ['o_o', '\u{1F633}'],
  ['O_O', '\u{1F633}'],
];

// Build a regex that matches any emoticon as a whole word/token
const EMOTICON_PATTERN = new RegExp(
  `(${EMOTICONS.map(([e]) => e.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')).join('|')})`,
  'g',
);
const EMOTICON_MAP = Object.fromEntries(EMOTICONS);

// Matches http/https URLs so we never convert emoticons inside them (e.g. :/)
const URL_RE = /https?:\/\/\S+/g;

function replaceEmoticons(text) {
  const parts = [];
  let last = 0;
  let m;
  URL_RE.lastIndex = 0;
  // eslint-disable-next-line no-cond-assign
  while ((m = URL_RE.exec(text)) !== null) {
    if (m.index > last) {
      parts.push(text.slice(last, m.index).replace(EMOTICON_PATTERN, (s) => EMOTICON_MAP[s] || s));
    }
    parts.push(m[0]);
    last = m.index + m[0].length;
  }
  parts.push(text.slice(last).replace(EMOTICON_PATTERN, (s) => EMOTICON_MAP[s] || s));
  return parts.join('');
}

export default {
  name: 'MessageInput',
  data: () => ({
    messageToBeSent: '',
    emojiPickerOpen: false,
    emojiSearch: '',
  }),
  computed: {
    filteredEmojis() {
      if (!this.emojiSearch) return EMOJIS;
      const q = this.emojiSearch.toLowerCase();
      return EMOJIS.filter((e) => e.name.includes(q));
    },
  },
  methods: {
    ...mapActions('synclounge', [
      'SEND_MESSAGE',
    ]),
    sendMessage() {
      const text = replaceEmoticons(this.messageToBeSent).trim();
      if (text === '') return;
      this.SEND_MESSAGE(text);
      this.messageToBeSent = '';
    },
    convertEmoticons() {
      const converted = replaceEmoticons(this.messageToBeSent);
      if (converted !== this.messageToBeSent) {
        const input = this.$refs.messageInput.$el.querySelector('input');
        const pos = input?.selectionStart ?? converted.length;
        this.messageToBeSent = converted;
        this.$nextTick(() => {
          input?.setSelectionRange(pos, pos);
        });
      }
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
    insertEmoji(emoji) {
      const input = this.$refs.messageInput.$el.querySelector('input');
      if (!input) {
        this.messageToBeSent += emoji;
        return;
      }
      const start = input.selectionStart ?? this.messageToBeSent.length;
      const end = input.selectionEnd ?? start;
      const before = this.messageToBeSent.slice(0, start);
      const after = this.messageToBeSent.slice(end);
      this.messageToBeSent = `${before}${emoji}${after}`;
      this.$nextTick(() => {
        input.focus();
        const pos = start + emoji.length;
        input.setSelectionRange(pos, pos);
      });
    },
  },
};
</script>

<style scoped>
.message-input-wrapper {
  display: flex;
  align-items: center;
  width: 100%;
}
.emoji-btn {
  flex-shrink: 0;
  align-self: center;
}
.send-btn {
  flex-shrink: 0;
  align-self: center;
}
.emoji-trigger {
  font-size: 18px;
  line-height: 1;
}
.emoji-picker {
  width: 300px;
}
.emoji-grid {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 2px;
  max-height: 200px;
  overflow-y: auto;
}
.flex-input {
  flex: 1;
  min-width: 0;
}
:deep(.flex-input .v-field) {
  border-radius: 24px !important;
}
.emoji-item {
  font-size: 20px;
  padding: 4px;
  border: none;
  background: transparent;
  cursor: pointer;
  border-radius: 4px;
  line-height: 1;
  transition: background-color 0.15s;
}
.emoji-item:hover {
  background: rgba(255, 255, 255, 0.12);
}
</style>

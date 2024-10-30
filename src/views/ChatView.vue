<template>
    <v-app>
      <v-main>
        <v-container class="fill-height">
          <v-row class="fill-height" align="center" justify="center">
            <v-col cols="12" sm="10" md="8" lg="6">
              <v-card class="mx-auto">
                <v-card-title class="text-center text-h5 py-4">
                  <v-icon left color="primary" class="mr-2">mdi-chat</v-icon>
                  AI 상담사
                </v-card-title>
  
                <v-card-subtitle>
                  <v-chip
                    :color="statusColor"
                    text-color="white"
                    small
                    class="px-2"
                  >
                    {{ statusMessage }}
                  </v-chip>
                </v-card-subtitle>
  
                <v-card-text class="chat-wrapper">
                  <div
                    ref="messageContainerRef"
                    class="message-container overflow-y-auto"
                  >
                    <template v-for="(msg, index) in messages" :key="index">
                      <div
                        :class="['message-item', msg.type]"
                        class="ma-2"
                      >
                        <v-card
                          :color="msg.type === 'sent' ? 'primary' : 'grey lighten-3'"
                          :class="msg.type === 'sent' ? 'float-right' : 'float-left'"
                          rounded="lg"
                          max-width="80%"
                          elevation="1"
                        >
                          <v-card-text :class="msg.type === 'sent' ? 'white--text' : ''">
                            {{ msg.content }}
                            <div class="caption text-end" :class="msg.type === 'sent' ? 'white--text' : 'grey--text'">
                              {{ new Date().toLocaleTimeString() }}
                            </div>
                          </v-card-text>
                        </v-card>
                      </div>
                    </template>
                  </div>
                </v-card-text>
  
                <v-card-actions class="pa-4">
                  <v-form @submit.prevent="sendMessage" class="d-flex align-center w-100">
                    <v-text-field
                      v-model="messageInput"
                      placeholder="Type your message..."
                      outlined
                      dense
                      hide-details
                      class="mr-3"
                      @keypress.enter="sendMessage"
                      :rules="[v => !!v || 'Message is required']"
                    >
                      <template v-slot:append>
                        <v-icon v-if="messageInput" color="grey" @click="messageInput = ''">
                          mdi-close
                        </v-icon>
                      </template>
                    </v-text-field>
  
                    <v-btn
                      color="primary"
                      @click="sendMessage"
                      :disabled="!messageInput.trim()"
                      elevation="2"
                    >
                      <v-icon left>mdi-send</v-icon>
                      Send
                    </v-btn>
                  </v-form>
                </v-card-actions>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-main>
    </v-app>
  </template>
  
  <script setup lang="ts">
  import { ref, reactive, onMounted, onUnmounted, nextTick, watch } from 'vue';
  
  const wsUrl = 'ws://localhost:3000';
  const ws = ref<WebSocket | null>(null);
  
  const statusMessage = ref('Connecting...');
  const statusColor = ref('grey');
  const messageInput = ref('');
  const messageContainerRef = ref<HTMLElement | null>(null);
  const messages = reactive<{ content: string; type: 'sent' | 'received' }[]>([]);
  
  // 개선된 스크롤 함수
  const scrollToBottom = async () => {
    await nextTick();
    try {
      const container = messageContainerRef.value;
      if (container) {
        const scrollHeight = container.scrollHeight;
        container.scrollTop = scrollHeight;
      }
    } catch (error) {
      console.error('Scroll error:', error);
    }
  };

  // messages 배열이 변경될 때마다 스크롤
  watch(messages, async () => {
    await scrollToBottom();
  }, { deep: true });
  
  const connectWebSocket = () => {
    ws.value = new WebSocket(wsUrl);
  
    ws.value.onopen = () => {
      statusMessage.value = 'Connected';
      statusColor.value = 'success';
    };
  
    ws.value.onmessage = async (event) => {
      const message = JSON.parse(event.data);
      messages.push({ content: message.content, type: 'received' });
      await scrollToBottom();  // 메시지 수신시 스크롤
    };
  
    ws.value.onclose = () => {
      statusMessage.value = 'Disconnected';
      statusColor.value = 'error';
    };
  
    ws.value.onerror = (error) => {
      statusMessage.value = 'Error occurred';
      statusColor.value = 'error';
      console.error('WebSocket error:', error);
    };
  };
  
  const sendMessage = async () => {
    const content = messageInput.value.trim();
    if (content && ws.value?.readyState === WebSocket.OPEN) {
      const message = {
        type: 'message',
        content: content,
        timestamp: Date.now(),
      };
      ws.value.send(JSON.stringify(message));
      messages.push({ content, type: 'sent' });
      messageInput.value = '';
      await scrollToBottom();  // 메시지 전송시 스크롤
    }
  };
  
  onMounted(async () => {
    connectWebSocket();
    await scrollToBottom();  // 초기 마운트시 스크롤
  });
  
  onUnmounted(() => {
    ws.value?.close();
  });
  </script>
  
  <style scoped>
  .chat-wrapper {
    position: relative;
  }

  .message-container {
    height: 400px;
    padding: 16px;
    overflow-y: auto;
    overflow-x: hidden;
    scroll-behavior: smooth;
    background-color: #f5f5f5;
    border-radius: 4px;
  }
  
  .message-item {
    clear: both;
    width: 100%;
    margin-bottom: 8px;
  }
  
  .message-item::after {
    content: '';
    display: table;
    clear: both;
  }
  
  .float-right {
    float: right;
  }
  
  .float-left {
    float: left;
  }
  </style>
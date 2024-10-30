<template>
    <v-container fluid class="fill-height pa-0">
      <v-card class="d-flex flex-column fill-height" elevation="2">
        <!-- 채팅 헤더 -->
        <v-app-bar color="primary" dense dark>
          <v-app-bar-title>AI 상담사</v-app-bar-title>
          <v-spacer></v-spacer>
          <v-badge
            :color="isConnected ? 'success' : 'error'"
            dot
            overlap
          >
            <span class="mr-2">{{ isConnected ? '연결됨' : '연결 끊김' }}</span>
          </v-badge>
        </v-app-bar>
  
        <!-- 채팅 메시지 영역 -->
        <v-card-text 
          ref="chatContainer" 
          class="chat-container flex-grow-1 overflow-y-auto pa-4"
        >
          <div
            v-for="message in messages"
            :key="message.timestamp"
            class="mb-4"
            :class="[
              'message',
              message.type === 'user' ? 'user-message' : 'counselor-message'
            ]"
          >
            <v-card
              :color="message.type === 'user' ? 'primary' : 'grey lighten-3'"
              :dark="message.type === 'user'"
              class="d-inline-block px-3 py-2 rounded-lg"
              max-width="80%"
            >
              <v-card-text class="pa-0">
                {{ message.content }}
              </v-card-text>
            </v-card>
            <div 
              class="caption mt-1 grey--text"
              :class="[message.type === 'user' ? 'text-right' : 'text-left']"
            >
              {{ formatTime(message.timestamp) }}
            </div>
          </div>
        </v-card-text>
  
        <!-- 입력 영역 -->
        <v-card-actions class="pa-4 pt-0">
          <v-text-field
            v-model="newMessage"
            placeholder="메시지를 입력하세요..."
            outlined
            dense
            hide-details
            @keyup.enter="sendMessage"
            :disabled="!isConnected"
            :loading="isLoading"
          >
            <template v-slot:append>
              <v-btn
                color="primary"
                icon
                @click="sendMessage"
                :disabled="!isConnected || !newMessage"
                :loading="isLoading"
              >
                <v-icon>mdi-send</v-icon>
              </v-btn>
            </template>
          </v-text-field>
        </v-card-actions>
      </v-card>
  
      <!-- 디버깅용 메시지 -->
      <v-snackbar
        v-model="showError"
        color="error"
        timeout="3000"
      >
        {{ errorMessage }}
      </v-snackbar>
    </v-container>
  </template>
  
  <script lang="ts">
  import { defineComponent, ref, onMounted, onUnmounted } from 'vue';
  
  interface Message {
    type: string;
    content: string;
    timestamp: number;
  }
  
  export default defineComponent({
    name: 'ChatInterface',
    
    setup() {
      const ws = ref<WebSocket | null>(null);
      const messages = ref<Message[]>([]);
      const newMessage = ref('');
      const isConnected = ref(false);
      const isLoading = ref(false);
      const chatContainer = ref<HTMLElement | null>(null);
      const showError = ref(false);
      const errorMessage = ref('');
  
      const showErrorMessage = (message: string) => {
        errorMessage.value = message;
        showError.value = true;
      };
  
      const connectWebSocket = () => {
        try {
          ws.value = new WebSocket('ws://localhost:3000');
  
          ws.value.onopen = () => {
            isConnected.value = true;
            console.log('WebSocket connected');
          };
  
          ws.value.onmessage = (event) => {
            try {
              console.log('Received message:', event.data);
              const message: Message = JSON.parse(event.data);
              
              // 상담사 응답 메시지 처리
              if (message.type === 'response') {
                messages.value.push({
                  type: 'counselor',
                  content: message.content,
                  timestamp: message.timestamp || Date.now()
                });
              } else if (message.type === 'error') {
                showErrorMessage(message.content);
              }
              
              isLoading.value = false;
              scrollToBottom();
            } catch (error) {
              console.error('Error processing message:', error);
              showErrorMessage('메시지 처리 중 오류가 발생했습니다.');
            }
          };
  
          ws.value.onclose = () => {
            isConnected.value = false;
            console.log('WebSocket disconnected');
            setTimeout(connectWebSocket, 3000);
          };
  
          ws.value.onerror = (error) => {
            console.error('WebSocket error:', error);
            isConnected.value = false;
            showErrorMessage('WebSocket 연결 오류가 발생했습니다.');
          };
        } catch (error) {
          console.error('Connection error:', error);
          showErrorMessage('서버 연결에 실패했습니다.');
        }
      };
  
      const sendMessage = () => {
        if (!ws.value || !newMessage.value.trim() || !isConnected.value) return;
  
        try {
          const message: Message = {
            type: 'user',
            content: newMessage.value.trim(),
            timestamp: Date.now()
          };
  
          // 사용자 메시지를 채팅창에 추가
          messages.value.push(message);
          
          // WebSocket으로 메시지 전송
          ws.value.send(JSON.stringify({
            type: 'message',
            content: message.content,
            timestamp: message.timestamp
          }));
  
          newMessage.value = '';
          isLoading.value = true;
          scrollToBottom();
        } catch (error) {
          console.error('Error sending message:', error);
          showErrorMessage('메시지 전송 중 오류가 발생했습니다.');
        }
      };
  
      const scrollToBottom = () => {
        setTimeout(() => {
          if (chatContainer.value) {
            chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
          }
        }, 100);
      };
  
      const formatTime = (timestamp: number): string => {
        return new Date(timestamp).toLocaleTimeString('ko-KR', {
          hour: '2-digit',
          minute: '2-digit'
        });
      };
  
      onMounted(() => {
        connectWebSocket();
      });
  
      onUnmounted(() => {
        if (ws.value) {
          ws.value.close();
        }
      });
  
      return {
        messages,
        newMessage,
        isConnected,
        isLoading,
        chatContainer,
        showError,
        errorMessage,
        sendMessage,
        formatTime
      };
    }
  });
  </script>
  
  <style scoped>
  .chat-container {
    height: calc(100vh - 160px);
    background-color: #f5f5f5;
  }
  
  .message {
    display: flex;
    flex-direction: column;
  }
  
  .user-message {
    align-items: flex-end;
  }
  
  .counselor-message {
    align-items: flex-start;
  }
  
  .v-card-text {
    white-space: pre-wrap;
    word-break: break-word;
  }
  </style>
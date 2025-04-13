<template>
  <el-dialog v-model="visible" title="聊天窗口" width="50%">
    <div class="chat-container">
      <div class="messages" ref="messages">
        <div v-for="(msg, index) in messages" :key="index" class="message">
          <span :class="{'buyer': msg.sender === 'buyer', 'seller': msg.sender === 'seller'}">
            {{ msg.sender === 'buyer' ? '买家' : '卖家' }}: {{ msg.text }}
          </span>
        </div>
      </div>
      <div class="input-container">
        <el-input
          v-model="newMessage"
          placeholder="输入消息..."
          @keyup.enter="sendMessage"
        ></el-input>
        <el-button type="primary" @click="sendMessage">发送</el-button>
      </div>
    </div>
  </el-dialog>
</template>

<script>
import { mapState } from "vuex";
import api from "../api";

export default {
  props: {
    visible: {
      type: Boolean,
      required: true,
    },
    currentUser: {
      type: String,
      required: true,
    },
    chatWith: {
      type: String,
      required: true,
    },
  },
  data() {
    return {
      messages: [],
      newMessage: "",
      chatVisible: false,
    };
  },
  watch: {
    visible(val) {
      if (val) {
        this.fetchMessages();
      }
    },
  },
  computed: {
    ...mapState("login", ["user"]),
  },
  methods: {
    fetchMessages() {
      api.getChatHistory(this.currentUser, this.chatWith).then((res) => {
        this.messages = res.data;
        this.scrollToBottom();
      });
    },
    sendMessage() {
      if (this.newMessage.trim() !== "") {
        const message = {
          sender: this.currentUser,
          receiver: this.chatWith,
          content: this.newMessage,
          timestamp: new Date().toISOString(),
        };
        this.messages.push(message);
        api.sendMessage(message).then(() => {
          this.newMessage = "";
          this.scrollToBottom();
        });
      }
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const messagesContainer = this.$refs.messages;
        messagesContainer.scrollTop = messagesContainer.scrollHeight;
      });
    },
  },
};
</script>

<style scoped>
.chat-container {
  display: flex;
  flex-direction: column;
  height: 400px;
}
.messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
.message {
  margin: 5px 0;
}
.buyer {
  color: blue;
}
.seller {
  color: green;
}
.input-container {
  display: flex;
  align-items: center;
}
.input-container .el-input {
  flex: 1;
  margin-right: 10px;
}
</style>

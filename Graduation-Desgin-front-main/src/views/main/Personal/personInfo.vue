<template>
  <el-card class="box-card" style="margin-top:15px">
    <!-- 聊天窗口 -->
    <el-dialog v-model="chatVisible" title="聊天窗口" width="50%">
      <div class="chat-container">
        <div class="messages" ref="messages">
          <div v-for="(msg, index) in messages" :key="index" class="message">
            <span :class="{'buyer': msg.sender === user.userInfo.username, 'seller': msg.sender !== user.userInfo.username}">
              {{ msg.sender === user.userInfo.username ? '我' : '对方' }}: {{ msg.content }}
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

    <!-- 个人信息 -->
    <el-dialog v-model="dialogFormVisible" title="编辑个人信息">
      <el-form :model="user">
        <el-form-item label="用户名" :label-width="formLabelWidth">
          <el-input v-model="user.userInfo.username" autocomplete="off" />
        </el-form-item>
        <el-form-item label="个人简介" :label-width="formLabelWidth">
          <el-input
            v-model="user.userInfo.introduction"
            type="textarea"
            :rows="8"
            autocomplete="off"
          />
        </el-form-item>
      </el-form>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="dialogFormVisible = false">取消</el-button>
          <el-button type="primary" @click="updateUserInfo()">保存</el-button>
        </span>
      </template>
    </el-dialog>

    <template #header>
      <div class="card-header" style="height:30px">
        <span style="float:left">个人信息</span>
        <el-button
          class="button"
          text
          style="float:right;"
          @click="dialogFormVisible = true"
        >
          <el-tooltip
            class="box-item"
            effect="dark"
            content="编辑个人信息"
            placement="top-start"
          >
            <el-icon><EditPen /></el-icon>
          </el-tooltip>
        </el-button>
        <el-button
          class="button"
          text
          style="float:right; margin-right: 20px;"
          @click="openChatWindow"
        >
          <el-tooltip
            class="box-item"
            effect="dark"
            content="聊天窗口"
            placement="top-start"
          >
            <el-icon><ChatSquare /></el-icon>
          </el-tooltip>
        </el-button>
      </div>
    </template>
    <div class="userInfo">
      <div class="avatar">
        <img :src="user.userInfo.avatar" class="avatarImg" />
      </div>
      <div>
        {{ user.userInfo.username }}
      </div>
      <div class="introduction">
        {{ user.userInfo.introduction }}
      </div>
      <div class="myProduct">
        <h2>我的商品</h2>
      </div>
      <div class="myproducts">
        <el-row class="indexListBoxRow" :span="22">
          <el-col
            class="indexList_box"
            v-for="(item, index) in products"
            :key="index"
            :span="6"
            :offset="(index) % 3 == 0 ? 0 : 2"
          >
            <el-card :body-style="{ padding: '0px' }">
              <div class="imgclass">
                <img
                  :src="'http://' + item.picture"
                  class="image"
                  :onerror="errorImage"
                />
              </div>
              <div style="padding: 14px">
                <span>{{ item.goodsName }}</span>
                <div class="bottom">
                  <time class="time">{{ item.usedTime }}</time>
                  <el-button text class="button">{{ item.userName }}</el-button>
                </div>
              </div>
            </el-card>
          </el-col>
        </el-row>
        <div>
          <div class="example-demonstration"></div>
          <el-pagination
            layout="prev, pager, next"
            :total="total"
            :page-size="pageParams.pagesize"
            :current-page="pageParams.pagenumber"
            @current-change="CurrentPageChange"
            class="pageindex"
          />
        </div>
      </div>
    </div>
  </el-card>
</template>

<script>
import { mapState } from "vuex";
import api from "../../../api";
export default {
  data() {
    return {
      products: "",
      total: "",
      ego: "",
      errorImage:
        'this.src="' +
        require("../../../assets/image/4.png") +
        '";this.style = "width:50%;margin-left:55px";',
      dialogFormVisible: false,
      chatVisible: false,
      newMessage: "",
      messages: [],
      chatWith: "sellerUsername", // 聊天对象用户名
      page: 1,
      pageParams: {
        pagenumber: 1,
        pagesize: 6,
      },
    };
  },
  computed: {
    ...mapState("login", ["user"]),
  },
  mounted: function () {
    api
      .getProductsByUid(
        JSON.parse(localStorage.getItem("ego")).data.userInfo.uid,
        this.pageParams
      )
      .then((res) => {
        this.total = res.data.data.Count;
        this.products = res.data.data.GoodsData;
      });
  },
  methods: {
    CurrentPageChange(currentPage) {
      this.pageParams.pagenumber = currentPage;
      api
        .getProductsByUid(
          JSON.parse(localStorage.getItem("ego")).data.userInfo.uid,
          this.pageParams
        )
        .then((res) => {
          this.total = res.data.data.Count;
          this.products = res.data.data.GoodsData;
        });
    },
    updateUserInfo() {
      api
        .updateUserInfo(
          this.user.userInfo.username,
          this.user.userInfo.introduction
        )
        .then((res) => {
          if (res.data.code == 200) {
            this.$notify({
              title: "修改用户信息成功",
              type: "success",
            });
            this.ego = JSON.parse(localStorage.getItem("ego"));
            this.ego.data.userInfo.username = this.user.userInfo.username;
            this.ego.data.userInfo.introduction =
              this.user.userInfo.introduction;
            localStorage.setItem("ego", JSON.stringify(this.ego));
          } else {
            this.$notify.error({
              title: "修改用户信息失败！",
              message: res.data.data,
            });
          }
          this.dialogFormVisible = false;
        });
    },
    sendMessage() {
      if (this.newMessage.trim() !== "") {
        const message = {
          sender: this.user.userInfo.username,
          receiver: this.chatWith,
          content: this.newMessage,
        };
        this.messages.push(message);
        api.sendMessage(message).then(() => {
          this.newMessage = "";
          this.scrollToBottom();
        });
      }
    },
    fetchMessages() {
      api
        .getChatHistory({
          sender: this.user.userInfo.username,
          receiver: this.chatWith,
        })
        .then((res) => {
          this.messages = res.data;
          this.scrollToBottom();
        });
    },
    scrollToBottom() {
      this.$nextTick(() => {
        const messagesContainer = this.$refs.messages;
        messagesContainer.scrollTop = messagesContainer.scrollHeight;
      });
    },
    openChatWindow() {
      this.chatVisible = true;
      this.fetchMessages();
    },
  },
};
</script>

<style scoped>
/* 样式 */
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
.avatarImg {
  width: 150px;
  height: 150px;
  border: 1px solid;
  border-radius: 50%;
}
</style>

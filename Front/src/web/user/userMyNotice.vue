<template>
  <div class="user-my-notice">
    <a-spin :spinning="showLoading" size="large">
      <div class="user-my-notice-control">
        <h3>消息列表</h3>
        <div class="user-my-notice-btn-container" v-if="messageList.length > 0">
          <a-button type="primary" @click="handleReadAllMessages" :disabled="isReadDisabled">全部已读</a-button>
          <a-button
            type="primary"
            @click="handleDeleteAllMessages"
            :disabled="deleteAllDisabled"
          >全部删除</a-button>
        </div>
      </div>
      <a-comment
        v-for="message in messageList"
        :key="message.messageItem.id"
        :data-url="message.url"
        class="message-item"
      >
        <div slot="author" class="message-sender">
          <a-tag
            :color="message.fromUser.isAdmin?'orange':'cyan'"
            @click="jumpToUser(message.fromUser.id)"
          >
            <a-icon :type="message.fromUser.isAdmin?'crown': 'user'"/>
            {{message.fromUser.username}}
          </a-tag>
          <div class="right-message">
            <a-tag color="red" @click="deleteMessage(message.messageItem.id)">
              <a-icon type="scissor"/>&nbsp;删除
            </a-tag>
          </div>
        </div>
        <a-avatar
          :src="message.fromUser.avatar"
          slot="avatar"
          @click="jumpToUser(message.fromUser.id)"
        />
        <div
          slot="content"
          @click="handleClickMessage(message.messageItem.id,message.messageItem.url)"
          :class="{'bold-message': !message.messageItem.isRead}"
        >{{message.messageItem.message}}</div>
        <div class="message-status">
          <div class="message-time">{{getMessageHumandate(message.messageItem.createDate)}}</div>
          <div class="messasge-read-status">
            <a-icon type="smile"/>
            &nbsp;{{message.messageItem.isRead ? '已读':'未读'}}
          </div>
        </div>
        <a-divider class="notice-divider"></a-divider>
      </a-comment>
      <div class="pagination" v-if="totalMessages > defaultPageSize">
        <a-pagination
          :current="page"
          :defaultPageSize="defaultPageSize"
          :total="totalMessages"
          @change="handlePageOnChange"
        ></a-pagination>
      </div>
    </a-spin>
  </div>
</template>
<script>
export default {
    data() {
        return {
            messageList: [],
            isReadDisabled: false,
            deleteAllDisabled: false,
            defaultPageSize: 20,
            totalMessages: 0,
            page: 1,
            showLoading: true
        };
    },
    methods: {
        jumpToUser(uid) {
            this.$router.push({
                path: `/user/visit/${uid}`
            });
        },
        handlePageOnChange(page) {
            this.page = +page;
            this.$router.push({
                path: `/user/my/notice/${page}`
            });
            this.getMessageList();
        },
        handleReadAllMessages() {
            if (confirm("是否要将全部消息设置为已读？")) {
                this.readAllMessages();
            }
        },
        handleDeleteAllMessages() {
            if (confirm("是否要将全部消息删除？")) {
                this.deleteAllMessages();
            }
        },
        handleClickMessage(id, url) {
            this.readMessage(id);
            if (url !== null) {
                if (
                    url.substr(0, 8) === "https://" ||
                    url.substr(0, 7) === "http://"
                ) {
                    window.open(url);
                } else {
                    this.$router.push({ path: url });
                }
            }
        },
        getMessageHumandate(str) {
            return this.$humanDate(new Date(str));
        },
        async getMessageList() {
            this.showLoading = true;
            this.$store.dispatch("updateUnreadMessage");
            const messageCountResponseRaw = await this.$ajax.get(
                this.$urls.messageCount
            );

            if (messageCountResponseRaw.data.code !== 1) {
                this.showLoading = false;
                return this.$store.dispatch("checkLoginStatus");
            }
            this.totalMessages = +messageCountResponseRaw.data.msg.total;

            const messageListResponseRaw = await this.$ajax.get(
                this.$urls.messageList(this.page)
            );

            if (messageListResponseRaw.data.code !== 1) {
                this.showLoading = false;
                return this.$store.dispatch("checkLoginStatus");
            }
            this.messageList = messageListResponseRaw.data.msg;
            this.showLoading = false;
        },
        async deleteMessage(mid) {
            this.showLoading = true;
            const responseRaw = await this.$ajax.post(
                this.$urls.messageDelete(mid)
            );
            if (responseRaw.data.code !== 1) {
                const modal = this.$error();
                modal.update({
                    title: "消息错误",
                    content: responseRaw.data.msg
                });
            } else {
                this.$notification.open({
                    message: "消息",
                    description: "消息已经全部删除！",
                    icon: <a-icon type="mail" style="color: #108ee9" />
                });
                this.getMessageList();
            }
            this.$store.dispatch("updateUnreadMessage");
            this.showLoading = false;
        },
        async readMessage(mid) {
            await this.$ajax.post(this.$urls.messageRead(mid));
            this.getMessageList();
            this.$store.dispatch("updateUnreadMessage");
        },
        async readAllMessages(e) {
            this.showLoading = true;
            this.isReadDisabled = true;
            const responseRaw = await this.$ajax.post(
                this.$urls.messageReadAll
            );
            if (responseRaw.data.code !== 1) {
                const modal = this.$error();
                modal.update({
                    title: "消息错误",
                    content: responseRaw.data.msg
                });
            } else {
                this.$notification.open({
                    message: "消息",
                    description: "消息已经全部设为已读！",
                    icon: <a-icon type="mail" style="color: #108ee9" />
                });
                this.getMessageList();
            }
            this.$store.dispatch("updateUnreadMessage");
            this.isReadDisabled = false;
            this.showLoading = false;
        },
        async deleteAllMessages(e) {
            this.showLoading = true;
            this.deleteAllDisabled = true;
            const responseRaw = await this.$ajax.post(
                this.$urls.messageDeleteAll
            );
            if (responseRaw.data.code !== 1) {
                const modal = this.$error();
                modal.update({
                    title: "消息错误",
                    content: responseRaw.data.msg
                });
            } else {
                this.$notification.open({
                    message: "消息",
                    description: "消息已经清空！",
                    icon: <a-icon type="mail" style="color: #108ee9" />
                });
                this.getMessageList();
            }
            this.$store.dispatch("updateUnreadMessage");
            this.deleteAllDisabled = false;
            this.showLoading = false;
        }
    },
    mounted() {
        this.page = +this.$route.params.page;
        this.getMessageList();
    }
};
</script>
<style scoped>
.message-item {
    cursor: pointer;
}
.message-sender {
    font-size: 14px;
}
.user-my-notice-control {
    margin-top: 18px;
    position: relative;
}
.user-my-notice-btn-container {
    text-align: right;
    position: absolute;
    display: inline-block;
    right: 0;
}
.user-my-notice-control h3 {
    display: inline-block;
}
.user-my-notice {
    position: relative;
}
.user-my-notice-control {
    user-select: none;
}
.right-message {
    display: inline-block;
    position: absolute;
    right: 0;
    top: 0;
}
@media screen and (max-width: 800px) {
    .message-read-status {
        display: none;
    }
}
.bold-message {
    font-weight: 700;
}
.notice-divider {
    margin: 0 0 6px 0;
}
.message-status {
    font-size: 12px;
    color: #999;
    user-select: none;
    cursor: default;
    position: relative;
}
.messasge-read-status {
    top: 0;
    right: 12px;
    position: absolute;
}
</style>

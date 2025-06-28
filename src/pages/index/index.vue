<template>
  <view class="container">
    <!-- 自定义导航栏 -->
    <view class="custom-nav" :style="{ paddingTop: statusBarHeight + 'px' }">
      <view class="nav-content">
        <view class="left-area">
          <view class="menu-icon" @tap="toggleSidebar">
            <image src="/static/icons/menu.png" mode="aspectFit"></image>
          </view>
        </view>
        <view class="center-area">
          <view class="title" @click="goToHome">
            <image
              src="/static/icons/logo.png"
              mode="aspectFit"
              class="logo-icon"
            ></image>
            <text>ChatJourney</text>
          </view>
        </view>
        <view class="right-area"></view>
      </view>
    </view>

    <!-- 侧边栏 -->
    <view class="sidebar" :class="{ 'sidebar-show': showSidebar }">
      <view class="sidebar-content">
        <!-- 用户信息 -->
        <view class="user-info">
          <view class="avatar">
            <image src="/static/icons/avatar.png" mode="aspectFit"></image>
          </view>
          <view class="user-name">小王先生</view>
          <view class="user-actions">
            <view class="search-icon">
              <image src="/static/icons/search.png" mode="aspectFit"></image>
            </view>
            <view class="settings-icon">
              <image src="/static/icons/settings.png" mode="aspectFit"></image>
            </view>
          </view>
        </view>

        <!-- 新对话按钮 -->
        <view class="new-chat">
          <button class="new-chat-btn" @tap="createNewChatFromSidebar">
            + 新建对话
          </button>
        </view>

        <!-- 对话列表 -->
        <view class="chat-list">
          <view class="chat-list-title">历史对话</view>
          <view
            class="chat-item"
            v-for="chat in chatList"
            :key="chat.id"
            @tap="switchToChat(chat)"
          >
            <text class="chat-item-title">{{ chat.title }}</text>
            <text class="chat-item-time">{{ chat.lastTime }}</text>
          </view>
        </view>
      </view>
    </view>

    <!-- 遮罩层 -->
    <view class="mask" v-if="showSidebar" @tap="toggleSidebar"></view>

    <view class="content" :style="{ paddingTop: contentPaddingTop + 'px' }">
      <!-- 背景图片 -->
      <view class="banner">
        <image src="/static/images/banner.jpg" mode="aspectFill"></image>
      </view>

      <!-- 欢迎区域 -->
      <view class="welcome-card" v-if="showWelcome">
        <view class="welcome-title">Hi~我的AI旅游小助手</view>
        <view class="welcome-subtitle"
          >你想找的小写游，可以为您推荐好玩的、好玩的、便宜的旅游打卡地哦~</view
        >

        <!-- 快捷选项按钮组 -->
        <view class="quick-options">
          <view
            class="option-item"
            @click="
              autoSendMessage(
                '我想去成都玩3天，预算3000元，帮我出一套旅游攻略',
                true
              )
            "
            >我想去成都玩3天，预算3000元，帮我出一套旅游攻略</view
          >
          <view class="option-grid">
            <view
              class="option-btn"
              @click="autoSendMessage('我想超值的住宿', true)"
              >我想超值的住宿</view
            >
            <view
              class="option-btn"
              @click="autoSendMessage('我想异地有限', true)"
              >我想异地有限</view
            >
            <view
              class="option-btn"
              @click="autoSendMessage('时尚潮人狂', true)"
              >时尚潮人狂</view
            >
            <view
              class="option-btn"
              @click="autoSendMessage('我想少走路', true)"
              >我想少走路</view
            >
            <view
              class="option-btn"
              @click="autoSendMessage('距离近的周边游', true)"
              >距离近的周边游</view
            >
            <view
              class="option-btn"
              @click="autoSendMessage('引导问题xx', true)"
              >引导问题xx</view
            >
          </view>
        </view>
      </view>

      <!-- 对话背景 -->
      <view class="chat-background" v-else>
        <view class="chat-container">
          <!-- 添加对话标题 -->
          <view class="chat-title" v-if="currentChatTitle">
            <text>{{ currentChatTitle }}</text>
          </view>
          <scroll-view
            scroll-y
            class="message-list"
            :scroll-top="scrollTop"
            @scrolltoupper="loadMoreMessages"
          >
            <view
              v-for="(message, index) in messages"
              :key="index"
              class="message-item"
              :class="message.type"
            >
              <template v-if="message.type === 'status'">
                <view class="message-content status-content">
                  <template v-if="message.content.includes('生成中')">
                    <image
                      class="status-icon"
                      src="/static/icons/loading.svg"
                      mode="aspectFit"
                    />
                    <text>生成中，请稍等...</text>
                    <button class="cancel-btn" @click="cancelGeneration">
                      取消生成
                    </button>
                  </template>
                  <template v-else-if="message.content.includes('取消')">
                    <image
                      class="status-icon"
                      src="/static/icons/cancel.svg"
                      mode="aspectFit"
                    />
                    <text>已取消生成</text>
                  </template>
                  <template v-else-if="message.content.includes('完成')">
                    <image
                      class="status-icon"
                      src="/static/icons/success.svg"
                      mode="aspectFit"
                    />
                    <text>已完成生成</text>
                  </template>
                </view>
              </template>
              <template v-else>
                <view class="message-content">
                  <text>{{ message.content }}</text>
                  <text class="message-time">{{ message.time }}</text>
                </view>
              </template>
            </view>
          </scroll-view>
        </view>
        <!-- 新增提示词区域 -->
        <view class="suggestion-bar" v-if="!showWelcome && suggestions.length">
          <view class="suggestion-title">你还可以这样说</view>
          <scroll-view scroll-x class="suggestion-list">
            <view
              class="suggestion-btn"
              v-for="(item, idx) in suggestions"
              :key="idx"
              @tap="autoSendMessage(item)"
            >
              {{ item }}
            </view>
          </scroll-view>
        </view>
      </view>

      <!-- 底部占位 -->
      <view class="bottom-space"></view>
    </view>

    <!-- 底部输入提示 -->
    <view class="input-hint">
      <view class="input-box">
        <input
          type="text"
          v-model="inputMessage"
          placeholder="有什么自己想攻略，直接问我吧~"
          @confirm="sendMessage"
          class="message-input"
        />
        <image
          src="/static/icons/send.png"
          mode="aspectFit"
          class="send-icon"
          @click="sendMessage"
        ></image>
      </view>
    </view>

    <!-- 灵感拼图悬浮按钮 -->
    <view
      class="jigsaw-fab"
      v-if="!showWelcome && itinerary.length"
      @tap="toggleJigsaw"
    >
      <image src="/static/icons/star.png" mode="aspectFit"></image>
    </view>

    <!-- 灵感拼图侧边栏 -->
    <view class="jigsaw-sidebar" :class="{ 'jigsaw-sidebar-show': showJigsaw }">
      <view class="jigsaw-content">
        <view class="jigsaw-header">
          <view class="jigsaw-title">灵感拼图</view>
          <view class="jigsaw-subtitle">预览攻略</view>
        </view>
        <scroll-view scroll-y class="timeline-container">
          <view class="timeline">
            <view v-for="day in itinerary" :key="day.date">
              <view class="timeline-day">
                <text class="date">{{ day.date }}</text>
                <text class="weather"
                  >{{ day.weatherIcon }} {{ day.temperature }}</text
                >
              </view>
              <view class="timeline-events">
                <view
                  class="event-item"
                  v-for="(event, index) in day.events"
                  :key="index"
                >
                  <view class="event-time">
                    <view
                      class="time-main"
                      v-html="formatTime(event.time).main"
                    ></view>
                    <view
                      class="time-range"
                      v-if="formatTime(event.time).range"
                      v-html="formatTime(event.time).range"
                    ></view>
                    <text class="time-sub" v-if="event.subtext">{{
                      event.subtext
                    }}</text>
                  </view>
                  <view class="event-line-content">
                    <view class="event-dot"></view>
                    <view
                      class="event-line"
                      v-if="index < day.events.length - 1"
                    ></view>
                    <view class="event-image-card">
                      <image
                        :src="event.image"
                        mode="aspectFill"
                        class="event-image"
                      ></image>
                      <view class="location-overlay">{{ event.location }}</view>
                    </view>
                    <view class="travel-info" v-if="event.travel">
                      <view class="travel-line"></view>
                      <text
                        >{{ event.travel.duration }}
                        {{ event.travel.method }}</text
                      >
                      <view class="travel-arrow">↓</view>
                    </view>
                  </view>
                </view>
              </view>
            </view>
          </view>
        </scroll-view>
      </view>
    </view>

    <!-- 遮罩层 -->
    <view class="mask" v-if="showJigsaw" @tap="toggleJigsaw"></view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      statusBarHeight: 0,
      contentPaddingTop: 0,
      navContentHeight: 52,
      showSidebar: false,
      showWelcome: true,
      inputMessage: "",
      messages: [],
      currentChatTitle: "",
      chatList: [],
      currentChatId: null,
      suggestions: [
        "我需要换个目的地",
        "我需要再调整一下预算",
        "帮我推荐亲子游",
        "帮我推荐美食路线",
        "帮我推荐避暑胜地",
      ],
      lastMessageId: "",
      showJigsaw: false,
      itinerary: [],
      isGenerating: false,
    };
  },
  onLoad() {
    const systemInfo = uni.getSystemInfoSync();
    this.statusBarHeight = systemInfo.statusBarHeight;
    this.contentPaddingTop = this.statusBarHeight + this.navContentHeight;
  },
  methods: {
    toggleSidebar() {
      this.showSidebar = !this.showSidebar;
    },
    createNewChatFromSidebar() {
      this.toggleSidebar();
      this.goToHome();
    },
    goToHome() {
      this.showWelcome = true;
      this.messages = [];
      this.currentChatTitle = "";
      this.currentChatId = null;
      this.itinerary = [];
    },
    startNewChat() {
      this.showWelcome = false;
      this.messages = [];
      this.currentChatTitle = "";
      this.currentChatId = null;
    },
    autoSendMessage(content, isNewChat = false) {
      if (isNewChat) {
        this.startNewChat();
        setTimeout(() => {
          this.inputMessage = content;
          this.sendMessage();
        }, 100);
      } else {
        this.inputMessage = content;
        this.sendMessage();
      }
    },
    saveChatToHistory() {
      if (this.messages.length > 0) {
        const chatId = this.currentChatId || Date.now().toString();
        const chatData = {
          id: chatId,
          title: this.currentChatTitle,
          messages: [...this.messages],
          lastTime: new Date().toLocaleString(),
        };

        const index = this.chatList.findIndex((chat) => chat.id === chatId);
        if (index !== -1) {
          this.chatList[index] = chatData;
        } else {
          this.chatList.unshift(chatData);
        }

        this.currentChatId = chatId;
      }
    },
    switchToChat(chat) {
      this.currentChatId = chat.id;
      this.currentChatTitle = chat.title;
      this.messages = [...chat.messages];
      this.showWelcome = false;
      this.toggleSidebar();
    },
    sendMessage() {
      if (this.inputMessage.trim()) {
        const userInput = this.inputMessage;
        if (this.messages.length === 0) {
          this.currentChatTitle = userInput;
        }
        this.messages.push({
          type: "user",
          content: userInput,
          time: new Date().toLocaleTimeString(),
        });
        this.showWelcome = false;
        this.$nextTick(() => {
          this.lastMessageId = "msg-" + (this.messages.length - 1);
        });

        this.inputMessage = ""; // Clear input after sending

        // --- Mock AI Response Flow ---
        if (userInput.includes("成都")) {
          this.itinerary = [];
          this.isGenerating = true;
          this.messages.push({
            type: "status",
            content: "生成中，请稍等...",
          });
          this.scrollToBottom();

          setTimeout(() => {
            if (!this.isGenerating) return;
            // 移除所有"生成中"状态的消息
            this.messages = this.messages.filter(
              (msg) =>
                !(msg.type === "status" && msg.content.includes("生成中"))
            );
            // 插入"已完成生成"
            this.messages.push({
              type: "status",
              content: "已完成生成",
            });
            this.scrollToBottom();

            // 插入AI生成的最终回复
            this.messages.push({
              type: "ai",
              content: "已完成生成，请进入灵感拼图查看路线预览及查看详情。",
              time: new Date().toLocaleTimeString(),
            });
            this.itinerary = this.getSampleItinerary();
            this.saveChatToHistory();
            this.scrollToBottom();
            this.isGenerating = false;
          }, 2000);
        } else {
          // Default AI reply
          setTimeout(() => {
            this.messages.push({
              type: "ai",
              content: "我正在为您规划行程，请稍等...",
              time: new Date().toLocaleTimeString(),
            });
            this.saveChatToHistory();
            this.scrollToBottom();
          }, 500);
        }
      }
    },
    scrollToBottom() {
      this.$nextTick(() => {
        this.lastMessageId = "msg-" + (this.messages.length - 1);
      });
    },
    getSampleItinerary() {
      return [
        {
          date: "4月12日",
          weatherIcon: "☀️",
          temperature: "32/39℃",
          events: [
            {
              time: "08:00",
              image: "/static/images/airport1.jpg",
              location: "成都双流国际机场",
              travel: { duration: "1.5h", method: "乘飞机前往" },
            },
            {
              time: "09:30",
              image: "/static/images/airport2.jpg",
              location: "昆明长水机场",
              travel: { duration: "2.5h", method: "乘大巴前往" },
            },
            {
              time: "12:00",
              image: "/static/images/flower.jpg",
              location: "罗平油菜花田入口",
              travel: { duration: "15min", method: "步行" },
            },
            {
              time: "12:15 ~ 13:45",
              subtext: "用餐",
              image: "/static/images/food.jpg",
              location: "当地XXX饭店",
              travel: { duration: "30min", method: "步行" },
            },
          ],
        },
      ];
    },
    toggleJigsaw() {
      this.showJigsaw = !this.showJigsaw;
    },
    formatTime(time) {
      if (time.includes("~")) {
        const parts = time.split("~");
        return {
          main: parts[0].trim(),
          range: `~<br/>${parts[1].trim()}`,
        };
      }
      return { main: time, range: null };
    },
    cancelGeneration() {
      // 只移除包含"生成中"的状态消息
      this.messages = this.messages.filter(
        (msg) => !(msg.type === "status" && msg.content.includes("生成中"))
      );
      this.isGenerating = false;
      this.scrollToBottom();

      // 添加AI询问消息
      setTimeout(() => {
        this.messages.push({
          type: "ai",
          content:
            "已为您取消生成。请问您是否需要修改行程要求？比如调整目的地、预算、天数或者其他需求？",
          time: new Date().toLocaleTimeString(),
        });
        this.saveChatToHistory();
        this.scrollToBottom();
      }, 500);
    },
  },
};
</script>

<style lang="scss">
.container {
  min-height: 100vh;
  background-color: #f8f4e9;
  position: relative;
}

.custom-nav {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 100;
  background-color: rgba(79, 112, 99, 0.95);
  backdrop-filter: blur(10px);

  .nav-content {
    height: 52px;
    display: flex;
    align-items: center;
    padding: 0 16px;
    position: relative;

    .left-area {
      position: absolute;
      left: 16px;
      height: 100%;
      display: flex;
      align-items: center;

      .menu-icon {
        width: 24px;
        height: 24px;
        opacity: 0.9;

        image {
          width: 100%;
          height: 100%;
        }
      }
    }

    .center-area {
      flex: 1;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100%;

      .title {
        display: flex;
        align-items: center;
        color: #f8f4e9;
        font-size: 18px;
        font-weight: 500;
        opacity: 0.95;

        .logo-icon {
          width: 30px;
          height: 30px;
          margin-right: 3%;
          margin-bottom: -3%;
          vertical-align: middle;
        }
      }
    }

    .right-area {
      position: absolute;
      right: 16px;
      height: 100%;
      width: 24px;
    }
  }
}

.content {
  min-height: 100vh;
  box-sizing: border-box;
  overflow-y: auto;
  padding-bottom: 70px;
}

.banner {
  width: 100%;
  height: 240px;
  position: relative;
  overflow: hidden;
  margin-top: -44px;

  &::after {
    content: "";
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    height: 180px;
    background: linear-gradient(
      to bottom,
      transparent 0%,
      rgba(248, 244, 233, 0.4) 35%,
      rgba(248, 244, 233, 0.9) 75%,
      #f8f4e9 100%
    );
  }

  image {
    width: 100%;
    height: 100%;
  }
}

.welcome-card {
  margin: -100px 16px 16px;
  padding: 20px;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 24px;
  box-shadow: 0 4px 24px rgba(60, 89, 107, 0.08);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.6);
  position: relative;
  z-index: 1;

  .welcome-title {
    font-size: 17px;
    font-weight: bold;
    margin-bottom: 8px;
    color: #2c4a52;
  }

  .welcome-subtitle {
    font-size: 13px;
    color: #5b7b84;
    margin-bottom: 20px;
    line-height: 1.4;
  }

  .quick-options {
    .option-item {
      background-color: rgba(60, 89, 107, 0.03);
      padding: 12px;
      border-radius: 20px;
      margin-bottom: 14px;
      font-size: 13px;
      color: #2c4a52;
      border: 1px solid rgba(60, 89, 107, 0.08);
      line-height: 1.4;
    }

    .option-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 10px;

      .option-btn {
        background-color: rgba(60, 89, 107, 0.03);
        padding: 10px;
        border-radius: 16px;
        text-align: center;
        font-size: 12px;
        color: #2c4a52;
        border: 1px solid rgba(60, 89, 107, 0.08);
        transition: all 0.3s ease;
        line-height: 1.4;

        &:active {
          background-color: rgba(60, 89, 107, 0.08);
          transform: scale(0.98);
        }
      }
    }
  }
}

.bottom-space {
  height: 74px;
}

.input-hint {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 12px 16px;
  background: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.85) 0%,
    rgba(255, 255, 255, 0.95) 100%
  );
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 -1px 20px rgba(60, 89, 107, 0.04);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.8);
  z-index: 99;

  &::after {
    content: "";
    position: absolute;
    bottom: -34px;
    left: 0;
    right: 0;
    height: 34px;
    background-color: #fff;
  }

  .input-box {
    background-color: rgba(60, 89, 107, 0.03);
    border: 1px solid rgba(60, 89, 107, 0.08);
    border-radius: 20px;
    padding: 12px 16px;
    margin-bottom: 0px;
    width: 100%;
    max-width: 600px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    transition: all 0.3s ease;
    position: relative;

    &:active {
      background-color: rgba(60, 89, 107, 0.05);
      transform: scale(0.995);
    }

    .message-input {
      flex: 1;
      height: 20px;
      font-size: 13px;
      color: #2c4a52;
      background: transparent;
      border: none;
      outline: none;
      padding: 0;
      margin-right: 8px;

      &::placeholder {
        color: #8a9ea7;
      }
    }

    .send-icon {
      width: 20px;
      height: 20px;
      opacity: 0.7;
    }
  }
}

.sidebar {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  width: 80%;
  max-width: 300px;
  background-color: #fff;
  z-index: 1000;
  transform: translateX(-100%);
  transition: transform 0.3s ease-out;
  box-shadow: 2px 0 10px rgba(0, 0, 0, 0.1);

  &.sidebar-show {
    transform: translateX(0);
  }

  .sidebar-content {
    height: 100%;
    display: flex;
    flex-direction: column;
    background-color: #fff;
  }

  .user-info {
    background-color: #4f7063;
    padding: 16px;
    padding-top: calc(#{statusBarHeight}px + 16px);
    display: flex;
    align-items: center;
    justify-content: space-between;
    color: #fff;

    .avatar {
      width: 32px;
      height: 32px;
      border-radius: 50%;
      overflow: hidden;
      background-color: rgba(255, 255, 255, 0.2);

      image {
        width: 100%;
        height: 100%;
      }
    }

    .user-name {
      flex: 1;
      margin-left: 12px;
      font-size: 15px;
    }

    .user-actions {
      display: flex;
      gap: 16px;

      .search-icon,
      .settings-icon {
        width: 20px;
        height: 20px;
        opacity: 0.9;

        image {
          width: 100%;
          height: 100%;
        }
      }
    }
  }

  .new-chat {
    padding: 16px;
    border-bottom: 1px solid rgba(0, 0, 0, 0.05);

    .new-chat-btn {
      width: 100%;
      height: 36px;
      line-height: 36px;
      text-align: center;
      background-color: rgba(79, 112, 99, 0.1);
      color: #4f7063;
      font-size: 14px;
      border-radius: 18px;
      border: 1px solid rgba(79, 112, 99, 0.2);
    }
  }

  .chat-list {
    flex: 1;
    overflow-y: auto;
    padding: 16px;

    .chat-list-title {
      font-size: 13px;
      color: #999;
      margin-bottom: 12px;
    }

    .chat-item {
      display: flex;
      flex-direction: column;
      padding: 12px;
      border-bottom: 1px solid rgba(0, 0, 0, 0.05);
      transition: background-color 0.3s ease;

      &:active {
        background-color: rgba(0, 0, 0, 0.05);
      }

      .chat-item-title {
        font-size: 14px;
        color: #333;
        margin-bottom: 4px;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }

      .chat-item-time {
        font-size: 12px;
        color: #999;
      }
    }
  }
}

.mask {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.4);
  z-index: 999;
}

.chat-background {
  margin: -100px 16px 16px;
  min-height: 200px;
  height: auto;
  background-color: rgba(255, 255, 255, 0.8);
  border-radius: 24px;
  box-shadow: 0 4px 24px rgba(60, 89, 107, 0.08);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.6);
  position: relative;
  z-index: 1;
  padding: 20px;
  display: block;

  .chat-container {
    width: 100%;
    display: block;
  }

  .message-list {
    width: 100%;
    padding: 20px 0;
    min-height: 100px;
  }
}

.chat-title {
  text-align: center;
  padding: 16px 0;
  border-bottom: 1px solid rgba(60, 89, 107, 0.1);
  margin-bottom: 10px;

  text {
    font-size: 15px;
    color: #2c4a52;
    font-weight: 500;
  }
}

.message-item {
  margin-bottom: 20px;
  display: flex;

  &.user {
    justify-content: flex-end;

    .message-content {
      background-color: #4f7063;
      color: #fff;
      border-radius: 16px 4px 16px 16px;
      margin-right: 40px;

      .message-time {
        color: rgba(255, 255, 255, 0.7);
      }
    }
  }

  &.ai {
    justify-content: flex-start;

    .message-content {
      background-color: #fff;
      color: #2c4a52;
      border-radius: 4px 16px 16px 16px;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
      .message-time {
        color: #8a9ea7;
      }
    }
  }

  &.status {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 10px 0;
    font-size: 14px;
    color: #2c4a52;

    .status-icon {
      width: 20px;
      height: 20px;
      margin-right: 8px;
    }
  }

  .message-content {
    max-width: 80%;
    padding: 12px 16px;
    font-size: 14px;
    line-height: 1.5;

    .message-time {
      display: block;
      font-size: 11px;
      margin-top: 4px;
    }
  }
}

.suggestion-bar {
  background: #f8f4e9b0;
  border-radius: 12px;
  margin: 0 0 12px 0;
  padding: 12px 12px 8px 12px;
  .suggestion-title {
    font-size: 13px;
    color: #8a9ea7;
    margin-bottom: 8px;
  }
  .suggestion-list {
    display: flex;
    flex-direction: row;
    overflow-x: auto;
    white-space: nowrap;
    .suggestion-btn {
      display: inline-block;
      background: #f5f5f5;
      color: #4f7063;
      border-radius: 16px;
      padding: 6px 16px;
      font-size: 14px;
      margin-right: 10px;
      margin-bottom: 4px;
      border: 1px solid #e0e0e0;
      transition: background 0.2s;
    }
    .suggestion-btn:active {
      background: #e6f7ee;
    }
  }
}

.jigsaw-fab {
  position: fixed;
  right: 20px;
  bottom: 120px;
  width: 50px;
  height: 50px;
  background-color: #4f7063;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  z-index: 100;
  transition: transform 0.2s;

  &:active {
    transform: scale(0.95);
  }

  image {
    width: 28px;
    height: 28px;
    filter: brightness(0) invert(1);
  }
}

.jigsaw-sidebar {
  position: fixed;
  top: 0;
  right: 0;
  bottom: 0;
  width: 85%;
  max-width: 360px;
  background-color: #4f7063;
  z-index: 1000;
  transform: translateX(100%);
  transition: transform 0.3s ease-out;
  box-shadow: -2px 0 10px rgba(0, 0, 0, 0.1);

  &.jigsaw-sidebar-show {
    transform: translateX(0);
  }

  .jigsaw-content {
    height: 100%;
    display: flex;
    flex-direction: column;
    padding-top: var(--status-bar-height);
  }

  .jigsaw-header {
    padding: 20px;
    color: #fff;
    .jigsaw-title {
      font-size: 24px;
      font-weight: bold;
    }
    .jigsaw-subtitle {
      font-size: 14px;
      opacity: 0.8;
      margin-top: 4px;
    }
  }

  .timeline-container {
    flex: 1;
    background-color: #f8f4e9;
    padding: 20px;
    box-sizing: border-box;
    height: 100%;
  }

  .timeline {
    padding-left: 20px;
  }

  .timeline-day {
    margin-bottom: 20px;
    .date {
      font-size: 18px;
      font-weight: bold;
      color: #2c4a52;
    }
    .weather {
      font-size: 14px;
      color: #5b7b84;
      margin-left: 10px;
    }
  }

  .event-item {
    display: flex;
    position: relative;
  }

  .event-time {
    width: 80px;
    text-align: right;
    padding-right: 20px;
    flex-shrink: 0;
    color: #2c4a52;
    .time-main {
      font-size: 16px;
      font-weight: bold;
    }
    .time-range {
      font-size: 12px;
      line-height: 1.2;
      display: inline-block;
    }
    .time-sub {
      font-size: 14px;
      display: block;
      margin-top: 4px;
    }
  }

  .event-line-content {
    flex: 1;
    padding-left: 20px;
    border-left: 2px solid #d3c0a5;
    padding-bottom: 20px;
    position: relative;
  }

  .event-dot {
    width: 12px;
    height: 12px;
    background-color: #4f7063;
    border-radius: 50%;
    position: absolute;
    left: -7px;
    top: 5px;
  }

  .event-image-card {
    width: 100%;
    border-radius: 16px;
    overflow: hidden;
    position: relative;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
    .event-image {
      width: 100%;
      height: 120px;
      display: block;
    }
    .location-overlay {
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      background: linear-gradient(to top, rgba(0, 0, 0, 0.6), transparent);
      color: #fff;
      padding: 16px 12px 8px;
      font-size: 14px;
      font-weight: bold;
    }
  }

  .travel-info {
    display: flex;
    align-items: center;
    color: #5b7b84;
    font-size: 13px;
    margin-top: 15px;
    padding-left: 15px;
    position: relative;

    .travel-arrow {
      margin-left: 8px;
    }
  }
}

.status-content {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-size: 15px;
  color: #2c4a52;
  width: 100%;
  .status-icon {
    width: 20px;
    height: 20px;
    margin-right: 8px;
  }
  .cancel-btn {
    margin-left: 12px;
    background: #fff0f0;
    color: #d9534f;
    border: 1px solid #d9534f;
    border-radius: 12px;
    font-size: 13px;
    padding: 2px 12px;
    height: 28px;
    line-height: 24px;
    cursor: pointer;
    transition: background 0.2s;
  }
  .cancel-btn:active {
    background: #ffeaea;
  }
}
</style>

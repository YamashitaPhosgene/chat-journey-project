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
          <view class="title">
            <image src="/static/icons/logo.png" mode="aspectFit" class="logo-icon"></image>
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
          <button class="new-chat-btn" @tap="startNewChat">+ 新建对话</button>
        </view>

        <!-- 对话列表 -->
        <view class="chat-list">
          <view class="chat-list-title">本周</view>
          <view class="chat-item" v-for="(item, index) in chatList" :key="index">
            <text class="chat-item-title">{{item.title}}</text>
            <text class="chat-item-arrow">></text>
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
      <view class="welcome-card">
        <view class="welcome-title">Hi~我的AI旅游小助手</view>
        <view class="welcome-subtitle">你想找的小写游，可以为您推荐好玩的、好玩的、便宜的旅游打卡地哦~</view>
        
        <!-- 快捷选项按钮组 -->
        <view class="quick-options">
          <view class="option-item">我想去成都玩3天，预算3000元，帮我出一套旅游攻略</view>
          <view class="option-grid">
            <view class="option-btn">我想超值的住宿</view>
            <view class="option-btn">我想异地有限</view>
            <view class="option-btn">时尚潮人狂</view>
            <view class="option-btn">我想少走路</view>
            <view class="option-btn">距离近的周边游</view>
            <view class="option-btn">引导问题xx</view>
          </view>
        </view>
      </view>

      <!-- 底部占位 -->
      <view class="bottom-space"></view>
    </view>

    <!-- 底部输入提示 -->
    <view class="input-hint">
      <view class="input-box">
        <text>有什么自己想攻略，直接问我吧~</text>
        <image src="/static/icons/send.png" mode="aspectFit" class="send-icon"></image>
      </view>
    </view>
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
      chatList: [
        { title: '低成本旅游攻略推荐' },
        { title: '对话第一条内容xxxxxxx...' },
        { title: '低成本旅游攻略推荐' },
        { title: '对话第一条内容xxxxxxx...' },
        { title: '对话第一条内容xxxxxxx...' },
        { title: '对话第一条内容xxxxxxx...' },
        { title: '对话第一条内容xxxxxxx...' },
        { title: '对话第一条内容xxxxxxx...' },
        { title: '对话第一条内容xxxxxxx...' },
        { title: '对话第一条内容xxxxxxx...' }
      ]
    }
  },
  onLoad() {
    const systemInfo = uni.getSystemInfoSync()
    this.statusBarHeight = systemInfo.statusBarHeight
    this.contentPaddingTop = this.statusBarHeight + this.navContentHeight
  },
  methods: {
    toggleSidebar() {
      this.showSidebar = !this.showSidebar
    },
    startNewChat() {
      this.toggleSidebar()
      // 这里添加新建对话的逻辑
    }
  }
}
</script>

<style lang="scss">
.container {
  min-height: 100vh;
  background-color: #F8F4E9;
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
        color: #F8F4E9;
        font-size: 18px;
        font-weight: 500;
        opacity: 0.95;

        .logo-icon {
          width: 24px;
          height: 24px;
          margin-right: 10px;
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
    content: '';
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
      #F8F4E9 100%
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
    color: #2C4A52;
  }

  .welcome-subtitle {
    font-size: 13px;
    color: #5B7B84;
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
      color: #2C4A52;
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
        color: #2C4A52;
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
  background: linear-gradient(180deg, 
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
    content: '';
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
    margin-bottom: 16px;
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

    text {
      color: #8A9EA7;
      font-size: 13px;
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
    background-color: #4F7063;
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
      color: #4F7063;
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
      align-items: center;
      justify-content: space-between;
      padding: 12px 0;
      border-bottom: 1px solid rgba(0, 0, 0, 0.05);

      .chat-item-title {
        flex: 1;
        font-size: 14px;
        color: #333;
      }

      .chat-item-arrow {
        color: #999;
        font-size: 14px;
        margin-left: 8px;
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
</style>

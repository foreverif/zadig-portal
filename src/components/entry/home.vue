<template>
  <div class="onborading-main-container">
    <div class="main-view large-sidebar">
      <div class="side-bar-container"
           :style="{width: sideWide? '196px':'66px'}">
        <sidebar class="side-bar-component"
                 @sidebar-width="sideWide = $event"></sidebar>
        <subSidebar class="cf-sub-side-bar"></subSidebar>
      </div>
      <div class="content-wrap">
        <announcement v-for="(ann,index) in announcements"
                      :key="index"
                      :title="ann.content.title"
                      :content="ann.content.content"></announcement>
        <announcement title="系统提示"
                      isHtml
                      v-if="isAdmin && SMTPDisabled"
                      :content="htmlTemplate"></announcement>
        <topbar></topbar>
        <router-view>
        </router-view>
        <FloatLink class="main-float"></FloatLink>
      </div>
    </div>
  </div>

</template>

<script>
import { getAnnouncementsAPI, getEmailHostAPI } from '@api'
import sidebar from './home/sidebar.vue'
import subSidebar from './home/sub_sidebar.vue'
import topbar from './home/topbar.vue'
import announcement from './home/announcement.vue'
import FloatLink from './home/float_link.vue'
export default {
  data () {
    return {
      announcements: [],
      sideWide: true,
      SMTPDisabled: false,
      htmlTemplate: '管理员请及时配置 <a href="/v1/system/integration?currentTab=mail">SMTP 邮箱服务器</a> 以便于用户密码丢失找回'
    }
  },
  methods: {
    getAnnouncements () {
      getAnnouncementsAPI().then((res) => {
        this.announcements = res
      })
    },
    async checkSMTP () {
      const res = await getEmailHostAPI()
      if (res) {
        this.SMTPDisabled = false
      } else {
        this.SMTPDisabled = true
      }
    }
  },
  computed: {
    isAdmin () {
      if (this.$store.state.login.role.includes('admin')) {
        return true
      } else {
        return false
      }
    }
  },
  watch: {
    isAdmin: {
      handler (val, oldVal) {
        if (val) {
        // 检查 SMTP 配置
          this.checkSMTP()
        }
      },
      immediate: true
    }
  },
  components: {
    sidebar,
    topbar,
    subSidebar,
    announcement,
    FloatLink
  },
  created () {
    this.$store.dispatch('GETUSERINFO')
    this.$store.dispatch('getProjectList')
    this.getAnnouncements()
  }
}
</script>

<style lang="less">
a {
  color: #1989fa;
  text-decoration: none;
}

button {
  &:focus {
    outline: none;
  }
}

body {
  height: 100%;
  overflow: hidden;
  overflow-y: auto;
  background-color: #fff;

  .el-card {
    background: #fff;
  }

  .onborading-main-container {
    width: 100vw;
    min-width: 768px;
    height: 100vh;
    min-height: 100vh;

    .main-view {
      > * {
        -ms-flex: 0 0 auto;
        flex: 0 0 auto;
        -webkit-box-flex: 0;
      }

      display: flex;
      align-items: stretch;
      justify-content: flex-start;
      height: 100%;
      overflow: hidden;

      .side-bar-container {
        position: relative;
        background-color: #f5f7fa;
        transition: width 350ms;

        .side-bar-component {
          position: absolute;
          z-index: 2;
          display: flex;
          -ms-flex-direction: column;
          flex-direction: column;
          align-items: center;
          justify-content: stretch;
          height: 100%;
          -webkit-box-orient: vertical;
          -webkit-box-direction: normal;
        }

        .cf-sub-side-bar {
          position: absolute;
          left: 66px;
          z-index: 1;
          display: flex;
          flex-direction: column;
          align-items: center;
          justify-content: stretch;
          height: 100%;
          -webkit-box-orient: vertical;
        }
      }

      .content-wrap {
        position: relative;
        display: flex;
        -ms-flex-direction: column;
        flex-direction: column;
        flex-grow: 1;
        flex-shrink: 1;
        align-items: stretch;
        justify-content: flex-start;
        height: 100%;
        overflow: auto;
        -webkit-box-orient: vertical;
        -webkit-box-direction: normal;

        .main-float {
          position: fixed;
          right: 20px;
          bottom: 20px;
          z-index: 1;
        }
      }
    }
  }

  .el-message-box__wrapper {
    .el-message-box {
      .el-message-box__header {
        .el-message-box__title {
          padding-right: 26px;
          line-height: 1.4;
        }
      }
    }
  }
}
</style>

<template>
  <div>
    <!-- 全局顶部导航栏 -->
    <header
      role="header"
      class="app-header">
      <div class="app-header-inner">
        <!-- 导航图标 -->
        <nav
          role="navigation"
          class="app-header-nav">
          <router-link
            to="/dynamic">
            <img
              :src="`../../static/icon/dynamic${ $route.path.includes('/dynamic') ?
              '-active' : ''}.svg`"
              title="点滴">
          </router-link>
          <router-link
            to="/movie">
            <img
              :src="`../../static/icon/movie${ $route.path.includes('/movie') ?
              '-active' : ''}.svg`"
              title="电影·电视·综艺">
          </router-link>
          <router-link
            to="/book">
            <img
              :src="`../../static/icon/book${ $route.path.includes('/book') ?
              '-active' : ''}.svg`"
              title="读书">
          </router-link>
          <router-link
            to="/bug"
          >
            <img
              :src="`../../static/icon/bug${ $route.path.includes('/bug') ?
              '-active' : ''}.svg`"
              title="虫子">
          </router-link>
        </nav>
        <!-- 用户 -->
        <div
          v-if="!isLogin"
          class="user-info">
          <!-- 用户未登录时显示 登陆 注册 按钮 -->
          <div
            class="login-btn"
            @click="goLogin('login')"
          >
            登陆
          </div>
          <div
            class="signup-btn"
            @click="goLogin('signup')">
            注册
          </div>
        </div>
        <div
          v-else
          class="user-info"
        >
          <el-dropdown
            class="user">
            <!-- 用户头像 -->
            <div
              class="dropdown">
              <img
                v-show="user.avatar"
                :src="user.avatar"
                class="avatar">
            </div>
            <!-- 用户相关菜单 -->
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item >个人主页</el-dropdown-item>
              <el-dropdown-item
                @click.native="$router.push({name: 'setting'})">
                设置
              </el-dropdown-item>
              <el-dropdown-item @click.native="logout">退出</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
          <!-- 发布文章 -->
        </div>
      </div>
    </header>
    <div class="sticky-holder"/>
    <main
      role="main">
      <keep-alive>
        <router-view/>
      </keep-alive>
    </main>
  </div>
</template>

<script>
import { mapGetters, mapState } from 'vuex';
import { getQueryParams } from '../util';
import store, { state } from '../store/index';
import { UPDATE_TOKEN, SWITCH_PAGE } from '../store/mutation-types';
import http from '../api/http';
import MarkdownEditor from '../components/markdown-editor';

export default {
  components: {
    'markdown-editor': MarkdownEditor
  },
  data() {
    return {
      showDropdown: false
    };
  },
  computed: {
    ...mapState([
      'user'
    ]),
    ...mapGetters([
      'isLogin'
    ])
  },
  activated() {
    // 如果是github授权回调页面
    const code = getQueryParams('code');
    if (!state.token && code) {
      http.get(`/v1/oauth/github/callback?code=${code}`)
        .then((res) => {
          if (res.data.token) {
            store.commit(UPDATE_TOKEN, { token: res.data.token });
            window.location.replace(window.location.origin);
          }
          localStorage.setItem('token', res.data.token);
        });
    }
    this.showDropdown = false;
  },
  methods: {
    goLogin(page) {
      store.commit(SWITCH_PAGE, { page });
      this.$router.push('/login');
    },
    handleCommand(path) {
      this.$router.push({ path });
    },
    swichDropdown(bool) {
      this.showDropdown = bool;
    },
    logout() {
      this.showDropdown = false;
      store.commit(UPDATE_TOKEN, { token: '' });
      this.$message({
        type: 'success',
        message: '你已退出登陆'
      });
      this.$router.push({ path: '/' });
    }
  }
};
</script>

<style scoped>
.app-header {
  z-index: 100;
  position: fixed;
  width: 100%;
  top: 0px;
  left: 0px;
  background: rgb(44, 44, 65);
  box-shadow: 0px 1px 1px 0px rgba(0, 0, 0, 0.30);
  background-clip: content-box;
}

.app-header-inner {
  position: relative;
  display: flex;
  min-width: 768px;
  max-width: 1010px;
  height: 52px;
  padding: 0 16px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.app-header-nav {
  display: flex;
  width: 220px;
  height: 56px;
}

.app-header-nav a {
  height: 56px;
  display: flex;
  align-items: center;
  padding: 10px;
}

.app-header-nav a:hover {
  background-color: #f5f5f5;
}

.app-header-nav img {
  width: 25px;
}

.input-search {
  width: 500px;
}

.user-info {
  display: flex;
  justify-content: flex-end;
  align-items: center;
}

.user-info .login-btn {
  display: inline-block;
  margin-right: 16px;
  color: #0084ff;
  border-color: #0084ff;
  border: 1px solid;
  border-radius: 3px;
  padding: 0 16px;
  font-size: 14px;
  line-height: 32px;
  text-align: center;
  cursor: pointer;
}

.user-info .login-btn:hover {
  background-color: rgba(0, 132, 255, 0.06);
}

.user-info .signup-btn {
  display: inline-block;
  color: #fff;
  background-color: #0084ff;
  border: 1px solid;
  padding: 0 16px;
  font-size: 14px;
  text-align: center;
  line-height: 32px;
  border-radius: 3px;
  cursor: pointer;
}

.user-info .signup-btn:hover {
  background-color: #0077e6;
}

.user {
  position: relative;
  width: 80px;
  margin-right: 8px;
  user-select: none;
}

.user:hover {
  background-color: #f5f5f5;
}

.dropdown {
  width: 40px;
  height: 40px;
  margin: 6px 20px;
  color: #333;
}

.dropdown .avatar {
  position: relative;
  width: 100%;
  height: 100%;
  border-radius: 50%;
}

.dropdown-menu {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  position: absolute;
  top: 96%;
  left: 0;
  width: 160px;
  padding: 5px 0;
  margin: 2px 0 0;
  list-style: none;
  font-size: 14px;
  text-align: left;
  background-color: #fff;
  border: 1px solid rgba(0, 0, 0, 0.15);
  border-radius: 4px;
}

.dropdown-menu li a {
  padding: 10px 20px;
  line-height: 30px;
  font-weight: 400;
  color: #333;
}

.dropdown-menu li a:hover {
  background-color: #f5f5f5;
}
.sticky-holder {
  position: relative;
  top: 0px;
  right: 0px;
  bottom: 0px;
  left: 0px;
  display: block;
  float: none;
  margin: 0px;
  height: 52px;
}
</style>

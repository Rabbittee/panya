<template>
  <div class="admin container-fluid row g-0 min-vh-100 bg-dark">
    <div class="col-lg-2 text-light">
      <nav
        class="
          navbar navbar-dark navbar-expand-lg
          p-3
          d-flex
          flex-lg-column
          align-items-start
        "
      >
        <router-link to="/" target="_blank"
          class="navbar-brand fs-6 fw-light">PANYA</router-link>
        <button
          class="navbar-toggler"
          data-bs-toggle="collapse"
          data-bs-target="#nav-collapse"
        >
          <i class="navbar-toggler-icon"></i>
        </button>
        <div class="collapse navbar-collapse" id="nav-collapse">
          <ul class="navbar-nav d-flex flex-column mt-lg-4">
            <li class="nav-item">
              <router-link class="nav-link px-0" to="/admin/products"
                >商品管理</router-link
              >
            </li>
            <li class="nav-item">
              <router-link class="nav-link px-0" to="/admin/coupons"
                >優惠券管理</router-link
              >
            </li>
            <li class="nav-item">
              <router-link class="nav-link px-0" to="/admin/orders"
                >訂單管理</router-link
              >
            </li>
            <li class="nav-item">
              <router-link class="nav-link px-0" to="/admin/articles"
                >文章管理</router-link
              >
            </li>
          </ul>
        </div>
      </nav>
    </div>
    <div class="col-lg-10 bg-light min-vh-100 shadow-md">
      <nav
        class="p-3 bg-white d-flex justify-content-between align-items-center"
      >
        <ol class="breadcrumb m-0">
          <li class="breadcrumb-item">
            {{ path }}
            <i class="fas fa-angle-right"></i>
          </li>
          <li class="breadcrumb-item active">{{ page }}</li>
        </ol>
        <span class="ms-auto me-3"
          :class="status === '登入中' ? 'text-success' : 'text-danger'">
          {{ status }}
        </span>
        <button
          type="button"
          class="btn btn-sm btn-secondary"
          @click="logout"
        >
          登出
        </button>
      </nav>
      <router-view class="p-3"></router-view>
    </div>
  </div>
</template>

<script>
import { apiUserCheck, apiUserLogout } from '@/scripts/api';

export default {
  emits: ['page-loading', 'push-message'],
  data() {
    return {
      path: '',
      page: '',
      status: '',
    };
  },
  watch: {
    $route() {
      this.page = this.$route.meta.title;
    },
  },
  methods: {
    checking() {
      const token = document.cookie.replace(/(?:(?:^|.*;\s*)panyaToken\s*=\s*([^;]*).*$)|^.*$/, '$1');
      this.$http.defaults.headers.common.Authorization = token;
      apiUserCheck()
        .then((res) => {
          if (!res.data.success) {
            this.$pushMessage(res);
            this.$router.push('/login');
          } else {
            this.$pushMessage(res, '已登入');
            this.status = res.data.message || '登入中';
          }
        })
        .catch((err) => {
          this.$emitter.emit('page-loading', false);
          this.$pushMessage(err);
        });
    },
    logout() {
      this.$emitter.emit('page-loading', true);
      apiUserLogout()
        .then((res) => {
          document.cookie = `panyaToken= ; expires=${new Date()}`;
          this.$router.push('/login');
          this.$pushMessage(res);
        })
        .catch((err) => {
          this.$emitter.emit('page-loading', false);
          this.$pushMessage(err);
        });
    },
  },
  created() {
    this.path = this.$route.matched[0].meta.title;
    this.page = this.$route.meta.title;
    this.checking();
  },
};
</script>

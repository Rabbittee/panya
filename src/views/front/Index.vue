<template>
  <div class="front container-fluid g-0 min-vh-100 text-dark" :class="block">
    <Navbar/>
    <router-view
      class="min-height"
      :datas="cart"
      :is-disabled="isDisabled"
      @get-cart="getCart"
      @get-products="getAllProducts"
    />
    <footer class="bg-dark py-4 py-md-5 text-light">
      <div class="container row g-0 mx-auto px-3 fs-7 ls-2
        text-secondary text-uppercase text-center"
        ref="footer">
        <Logo class="mb-3 mx-auto"/>
        <p>
          本網站僅供個人作品使用，不提供商業用途
          <span class="d-none d-md-inline-block"> |
            <router-link to="/login" target="_blank">登入後台</router-link>
          </span>
          <small class="d-block mt-1">PANYA &copy; {{ new Date().getFullYear() }} copyright</small>
        </p>
        <ul class="list-unstyled d-flex justify-content-center m-0">
          <li class="fs-7">
            <a href="https://github.com/ericacadu/panya" target="_blank">
              <i class="fab fa-lg fa-github"></i>
            </a>
          </li>
          <li class="fs-7">
            <a href="https://codepen.io/ericadu/collections/" target="_blank">
              <i class="fab fa-lg fa-codepen"></i>
            </a>
          </li>
          <li class="fs-7">
            <a href="mailto:erica.du0916@gmail.com" target="_blank">
              <i class="far fa-lg fa-envelope"></i>
            </a>
          </li>
        </ul>
      </div>
    </footer>
    <div id="scrollTop" role="button" class="p-2 rounded-1 fade d-none" ref="scrollTop"
      @click="scrollToTop">
      <span class="material-icons">
        arrow_drop_up
      </span>
    </div>
  </div>
</template>

<script>
import Navbar from '@/components/FrontNavbar.vue';
import { apiAllProducts, apiGetCart, apiAddCart } from '@/scripts/api';

export default {
  emits: ['page-loading', 'push-message', 'toggle-spinner', 'send-cart'],
  components: {
    Navbar,
  },
  data() {
    return {
      products: [],
      cart: [],
      provideCart: {
        data: [],
      },
      isDisabled: '',
      code: '',
      block: '',
    };
  },
  provide() {
    return {
      provideCart: this.provideCart,
    };
  },
  methods: {
    getAllProducts() {
      apiAllProducts()
        .then((res) => {
          if (!res.data.success) {
            this.$pushMessage(res);
          }
          this.products = res.data.products.reverse();
        })
        .catch((err) => {
          this.$pushMessage(err);
          this.$emitter.emit('page-loading', false);
        });
    },
    getCart() {
      apiGetCart()
        .then((res) => {
          if (!res.data.success) {
            this.$pushMessage(res);
          }
          this.cart = res.data.data;
          this.provideCart.data = res.data.data;
          this.isDisabled = '';
          this.getCartum();
          this.$emitter.emit('send-cart', this.cart);
          this.$emitter.emit('toggle-spinner', false);
        })
        .catch((err) => {
          this.$emitter.emit('page-loading', false);
          this.$pushMessage(err);
        });
    },
    getCartum() {
      this.cart.sum = 0;
      this.cart.carts.forEach((item) => {
        this.cart.sum += item.qty;
      });
      const num = document.getElementById('cart-num');
      num.classList.add('active');
      window.setTimeout(() => {
        num.classList.remove('active');
      }, 1000);
    },
    addCart(item, qty = 1) {
      const data = {
        product_id: item.id,
        qty,
      };
      this.isDisabled = item.id;
      apiAddCart({ data })
        .then((res) => {
          if (res.data.success) {
            this.getCart();
          }
          this.$pushMessage(res);
        })
        .catch((err) => {
          this.$emitter.emit('page-loading', false);
          this.$pushMessage(err);
        });
    },
    scrollBtnPos() {
      const half = this.$refs.footer.offsetWidth / 2;
      this.$refs.scrollTop.style.marginRight = `-${half - 16}px`;
      this.showScrollBtn();
    },
    showScrollBtn() {
      if (window.pageYOffset > 0 || window.scrollY > 0) {
        this.$refs.scrollTop.classList.remove('d-none');
        this.$refs.scrollTop.classList.add('show');
      } else {
        this.$refs.scrollTop.classList.remove('show');
        setTimeout(() => {
          this.$refs.scrollTop.classList.add('d-none');
        }, 100);
      }
    },
    scrollToTop() {
      window.scrollTo({
        top: 0,
        behavior: 'smooth',
      });
    },
  },
  mounted() {
    this.getCart();
    this.getAllProducts();
    this.$emitter.on('add-cart', (res) => {
      const { item, qty } = res;
      this.addCart(item, qty);
    });
    this.$emitter.on('get-cart', () => {
      this.getCart();
    });
    this.$emitter.on('toggle-overlay', (val) => {
      if (val) {
        this.block = 'vh-100 overflow-hidden';
      } else {
        this.block = '';
      }
    });
    this.scrollBtnPos();
    window.addEventListener('scroll', this.showScrollBtn);
    window.addEventListener('resize', this.scrollBtnPos);
  },
  unmounted() {
    this.$emitter.off('add-cart');
    this.$emitter.off('get-cart');
    this.$emitter.off('toggle-overlay');
    window.removeEventListener('scroll', this.showScrollBtn);
    window.removeEventListener('resize', this.scrollBtnPos);
  },
};
</script>

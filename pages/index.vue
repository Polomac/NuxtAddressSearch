<template>
  <div class="container">
      <h1 class="title">
        Address Search
      </h1>
    <form-component v-for="i in n" :key="i" @addForm="addForm"></form-component>
  </div>
</template>

<script>
const formComponent = () => import('../components/formComponent.vue');
export default {
  components: {
    formComponent,
  },
  data() {
    return {
      n: 1,
    };
  },
  methods: {
    addForm() {
      this.n++;
    },
  },
};
</script>

<style lang="scss">
.container {
  margin: 0 auto;
  min-height: 100vh;
  text-align: center;
  width: 100%;

  .title {
    display: block;
    font-weight: 300;
    font-size: 24px;
    color: #35495e;
    letter-spacing: 1px;
    text-transform: uppercase;
    width: 100%;
    overflow: hidden;
    margin: 30px 0;
    @include mqBreakPoint(1023px) {
      font-size: 60px;
      margin: 60px 0;
    }
  }
}
</style>

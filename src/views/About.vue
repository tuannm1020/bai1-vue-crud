<template>
  <div class="about">
    <h1 v-if="isLoading">Loading</h1>
    <div v-if="user">
      <h3>{{ user.name }}</h3>
      <div>
        <img :src="user.avatar" alt="avatar" />
      </div>
      <h5>{{ user.createdAt }}</h5>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      user: {},
      isLoading: true,
    };
  },
  async created() {
    try {
      const { data } = await axios.get(
        `https://620e1a1d585fbc3359d6c144.mockapi.io/user/${this.$route.params.id}`
      );
      this.user = data;
      this.isLoading = false;
    } catch (error) {
      console.log(error);
    }
  },
};
</script>

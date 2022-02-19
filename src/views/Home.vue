<template>
  <div class="home">
    <h1 v-if="isLoading">Loading</h1>
    <div v-else class="list-user">
      <b-button v-b-modal.modal-prevent-closing class="w-75 h-50 m-auto"
        >Add User</b-button
      >
      <b-card
        v-for="user in users"
        :key="user.id"
        :title="user.name"
        :img-src="user.avatar"
        img-alt="Image"
        img-top
        tag="article"
        style="max-width: 20rem"
      >
        <b-card-text>
          Created at: {{ user.createdAt | convertISOToDate }}
        </b-card-text>
        <div class="btn-container">
          <b-button href="#" variant="danger" @click="deleteUser(user.id)"
            >Delete</b-button
          >
          <router-link :to="`/about/${user.id}`">
            <b-button variant="primary"> View detail </b-button>
          </router-link>
          <b-button
            href="#"
            v-b-modal.modal-prevent-closing
            variant="warning"
            @click="editUser(user.id)"
            >edit</b-button
          >
        </div>
      </b-card>

      <b-modal
        id="modal-prevent-closing"
        ref="modal"
        title="Submit Your Info"
        @show="resetModal"
        @hidden="resetModal"
        @ok="handleOk"
      >
        <form ref="form" @submit.stop.prevent="handleSubmit">
          <b-form-group
            label-for="name-input"
            invalid-feedback="Name is required"
          >
            <b-form-input
              id="name-input"
              v-model="name"
              required
              placeHolder="Name"
              class="mb-3"
            ></b-form-input>
            <b-form-input
              id="name-input"
              v-model="avatar"
              required
              placeHolder="Avatar"
            ></b-form-input>
          </b-form-group>
        </form>
      </b-modal>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import axios from "axios";

export default {
  name: "Home",
  components: {},
  data() {
    return {
      users: [],
      name: "",
      avatar: "",
      userEdit: null,
      isEdit: false,
      isLoading: true,
    };
  },
  filters: {
    convertISOToDate(date) {
      let d = new Date(date);
      d = `${d.getDate()}-${d.getMonth() + 1}-${d.getFullYear()}`;
      return d;
    },
  },
  watch: {},
  methods: {
    editUser(id) {
      this.isEdit = true;
      this.userEdit = this.users.find((user) => user.id === id);
      setTimeout(() => {
        this.name = this.userEdit.name;
        this.avatar = this.userEdit.avatar;
      }, 100);
    },
    getMaxId() {
      let maxId = Math.max(...this.users.map((user) => user.id), 0);
      return (++maxId).toString();
    },
    resetModal() {
      this.name = "";
      this.avatar = "";
    },
    async handleOk(bvModalEvt) {
      bvModalEvt.preventDefault();
      if (!this.name || !this.avatar) {
        alert("Fill form pls!");
      }

      // Post new user
      if (!this.isEdit) {
        const d = new Date();
        const nowDate = d.toISOString();
        const newUser = {
          avatar: this.avatar,
          createdAt: nowDate,
          id: this.getMaxId(),
          name: this.name,
        };
        try {
          await axios.post(
            "https://620e1a1d585fbc3359d6c144.mockapi.io/user",
            newUser
          );
          this.users = [newUser, ...this.users];
        } catch (error) {
          console.log(error);
        }
      }
      //Edit user
      else {
        const { name, avatar } = this;
        const editedUser = { ...this.userEdit, name, avatar };
        try {
          await axios.put(
            `https://620e1a1d585fbc3359d6c144.mockapi.io/user/${this.userEdit.id}`,
            editedUser
          );
          const userEditedIdx = this.users.findIndex(
            (user) => user.id == this.userEdit.id
          );
          this.users.splice(userEditedIdx, 1, editedUser);
        } catch (error) {
          console.log(error);
        }
        this.isEdit = false;
      }

      this.handleSubmit();
    },
    handleSubmit() {
      this.$nextTick(() => {
        this.$bvModal.hide("modal-prevent-closing");
      });
    },
    async deleteUser(id) {
      try {
        this.users = this.users.filter((user) => user.id !== id);
        await axios.delete(
          `https://620e1a1d585fbc3359d6c144.mockapi.io/user/${id}`
        );
        console.log(this.users);
      } catch (error) {
        console.log(error);
      }
    },
  },
  async created() {
    try {
      const { data } = await axios.get(
        "https://620e1a1d585fbc3359d6c144.mockapi.io/user"
      );
      this.users = data;
      this.isLoading = false;
    } catch (error) {
      console.log(error);
    }
  },
};
</script>

<style scoped>
.list-user {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  grid-gap: 30px;
}
.btn-container {
  display: flex;
  justify-content: space-around;
}
</style>

<script>
import Comment from "./Comment.vue";
import EditPostModal from "./EditPostModal.vue";
import axios from 'axios';

const instance = axios.create({
  baseURL: "http://localhost/instaGame/controller/",
  headers: {
    "Content-Type": "application/x-www-form-urlencoded",
    "Content-Type": "multipart/form-data",
  },
  withCredentials: true,
});

const getProfilPicture = async (postId) => {
  const postData = {
    action: "getProfilPicture",
    post_id: postId,
  };
  try {
    const response = await instance.post("memberController.php", postData);
    return response.data.result.profile_picture;
    // this.test.value = response.data.result.profile_picture
    // if (response) {
    //   this.profilPictures[postId] = response.data.profile_picture;
    // }
  } catch (error) {
    console.error(error);
  }
};


export default {
  name: "PostCard",

  props: {
    posts: Array,
  },
  components: { Comment, EditPostModal },
  data() {
    return {
      isSmallModalOpen: false,
      isModalOpenEdit: false,
      selectedPostId: null,
      profilPictures: {},
      modalStates: {},
      currentUser: null,
    };
  },

  methods: {
    async getCurrentUser() {
      try {
        const instance = axios.create({
          baseURL: "http://localhost/instaGame/controller/",
          withCredentials: true,
        });

        const response = await instance.get("commentController.php", {
          params: {
            action: "getUserInfo",
          },
        });
        // console.log('Current user:', response.data.username);
        if (response.data.id) {
          console.log("Current user:", response.data.id);
          // Vous pouvez stocker les informations de l'utilisateur dans la variable currentUser
          this.currentUser = response.data.id;
          // console.log(this.currentUser);
        } else {
          console.error("Unable to get current user information.");
        }
      } catch (error) {
        console.error(error);
      }
    },

    //---------------------------------//
    async deletePost(postId) {
      try {
        await axios.delete(
          `http://localhost/instaGame/controller/postController.php?action=deletePost&id=${postId}`
        );
        this.$emit("postDeleted", postId);
        this.$emit("openModalEdit", postId);
      } catch (error) {
        console.error("Erreur lors de la suppression du post :", error);
      }
    },
    forceRerender() {
      this.$forceUpdate();
    },
    openModalEdit() {
      this.isModalOpenEdit = true;
    },
    closeModalEdit() {
      this.isModalOpenEdit = false;
    },
    editPost(postId) {
      // Fonction pour ouvrir la fenêtre modale et stocker l'ID du post sélectionné
      this.selectedPostId = postId;
      this.openModalEdit();
    },
    toggleSmallModal(postId) {
      this.modalStates[postId] = !this.modalStates[postId];
    },
  },

  created() {
    setTimeout(() => {
      this.forceRerender();
      for (const post of this.posts) {
        getProfilPicture(post.user_id).then((res) => {
          this.profilPictures[post.user_id] = res;
          // console.log(res);
        })
      }
    }, 100);
    this.getCurrentUser();
  },
  beforeDestroy() {
    // Assurez-vous de nettoyer l'intervalle lorsque le composant est détruit pour éviter les fuites de mémoire
    clearInterval(this.intervalId);
  }

}
</script>

<template>
  <div class="container">
    <div class="cards" v-for="post in posts" :key="post.id">
      <!-- SINGLE CARD :// -->
      <div class="card-items">

        <div class="card-header">
          <!-- SMALL MODAL  -->
          <div
            v-show="modalStates[post.id]"
            :id="'smallModal_' + post.id"
            class="small-modal-post"
          >
            <!-- <div class="overlay-small-modal"></div> -->
            <ul>
              <button
                v-if="post.user_id === currentUser"
                @click="deletePost(post.id)"
              >
                <span>Delete</span>
                <i class="fa-regular fa-trash-can"></i>
              </button>
              <button @click="editPost(post.id)">
                <span>Edit Post</span>
                <i class="fa-regular fa-pen-to-square"></i>
              </button>
              <button>
                <span @click="toggleModal">Cancel</span>
                <i class="fa-solid fa-xmark"></i>
              </button>
            </ul>
          </div>
          <!-- END SMALL MODAL  -->
          <div class="pic-profile-nav">
            <img :src="profilPictures[post.user_id]" alt="profile-pic">
          </div>
          <span class="user-name">{{ post.username }}</span>
          <!-- SMALL MODAL OPEN  -->
          <span
            @click="toggleSmallModal(post.id)"
            id="toggle-small-modal"
            class="open-small-modal-post"
            ><i
              class="fa-solid fa-ellipsis"></i
          ></span>
        </div>
        <div class="card-body">
          <img :src="post.post_picture" alt="post-pic" />
        </div>
        <div class="card-footer">
          <div class="card-footer-icons">
            <i class="fa-regular fa-heart"></i>
            <i class="fa-regular fa-comment"></i>
          </div>
          <span class="card-footer-icons"
            ><i class="fa-regular fa-bookmark"></i
          ></span>
        </div>
        <p class="description mt-2 text-base">{{ post.description }}</p>
        <hr class="w-4/5 mx-auto mt-2"  />
        <div class="comment">
          <Comment :postId="post.id" />
        </div>
      </div>
      <!-- END SINGLE CARD :// -->
      <EditPostModal
        :isOpenEdit="isModalOpenEdit"
        :closeModalEdit="closeModalEdit"
        :postId="selectedPostId"
      />
    </div>
  </div>
</template>

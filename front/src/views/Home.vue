<script>

import axios from 'axios';
import PostCard from "../components/PostCard.vue";
import NavBar from "../components/NavBar.vue";


export default {
    methods: {
        handlePostDeleted(deletedPostId) {
            this.posts = this.posts.filter(post => post.id !== deletedPostId);
        },
    },
    data() {
        return {
            posts: [], 
        };
    },
    async created() {
        await axios.get('http://localhost/instaGame/controller/postController.php?action=getPosts', {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
            },
        })
        .then((response) => {
            this.posts = response.data;
        })
        .catch((error) => {
            console.error(error);
        });
    },
    components: {
        PostCard, NavBar
    }
};



</script>

<template>
    <NavBar />
    <PostCard @postDeleted="handlePostDeleted" :posts="posts"/>
</template>

<style></style>
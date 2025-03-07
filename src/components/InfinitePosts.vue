<template>
  <div class="posts-container">
    <div v-if="!loading && posts.length === 0" class="no-posts">
      <p>There are no posts</p>
    </div>
    <div v-for="post in posts" :key="post.id" class="post-card">
      <router-link :to="`/posts/${post.id}`">
        <img
          v-if="post.image_url"
          :src="getImageUrl(post.image_url)"
          alt="Post image"
          class="post-image"
        />
        <h3>{{ post.title }}</h3>
      </router-link>
      <p class="post-date">
        {{ new Date(post.published_at).toLocaleDateString() }}
      </p>
    </div>
    <div ref="loadMoreTrigger" class="load-more">
      <p v-if="loading">Loading...</p>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, onUnmounted } from "vue";
import axios from "axios";

interface Post {
  id: number;
  title: string;
  published_at: string;
  image_url: string;
}

const posts = ref<Post[]>([]);
const offset = ref(0);
const limit = 10;
const loading = ref(false);
const loadMoreTrigger = ref<HTMLElement | null>(null);

const fetchPosts = async () => {
  if (loading.value) return;
  loading.value = true;
  try {
    const response = await axios.get(
      `http://localhost:8000/posts?offset=${offset.value}&limit=${limit}`
    );
    if (Array.isArray(response.data) && response.data.length > 0) {
      posts.value.push(...response.data);
      offset.value += limit;
    }
  } catch (error) {
    console.error("Error fetching posts:", error);
  } finally {
    loading.value = false;
  }
};

const getImageUrl = (path: string): string => {
  if (path.startsWith("static/")) {
    const filename = path.split("/").pop();
    return `http://localhost:8000/api/image/${filename}`;
  }
  return path;
};

const handleIntersection = (entries: IntersectionObserverEntry[]) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting && !loading.value) {
      fetchPosts();
    }
  });
};

let observer: IntersectionObserver | null = null;
onMounted(() => {
  fetchPosts();

  if (loadMoreTrigger.value) {
    observer = new IntersectionObserver(handleIntersection, {
      root: null,
      rootMargin: "100px",
      threshold: 0.1,
    });
    observer.observe(loadMoreTrigger.value);
  }
});

onUnmounted(() => {
  if (observer && loadMoreTrigger.value) {
    observer.unobserve(loadMoreTrigger.value);
  }
});
</script>

<style scoped>
.posts-container {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.post-card {
  padding: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: #fff;
  transition: box-shadow 0.2s ease;
}
.post-card:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}
.post-date {
  font-size: 0.9rem;
  color: #666;
}
.post-image {
  width: 100%;
  max-height: 200px;
  object-fit: cover;
  margin-top: 0.5rem;
  border-radius: 4px;
}
.load-more {
  text-align: center;
  padding: 1rem;
}
.no-posts {
  text-align: center;
  padding: 2rem;
  font-size: 1.2rem;
  color: #555;
}
</style>

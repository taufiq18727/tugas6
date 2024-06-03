<template>
  <div class="app-container">
    <form @submit.prevent="save" class="article-form">
      <input type="text" v-model="form.title" placeholder="Title" /> <br />
      <textarea v-model="form.content" placeholder="Content"></textarea><br />
      <button type="submit" class="btn save-btn">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-content">
          <h3>{{ article.title }}</h3>
          <p>{{ article.content }}</p>
        </div>
        <div class="article-actions">
          <button @click="edit(article)" class="btn edit-btn">Edit</button>
          <button @click="deleteArticle(article.id)" class="btn delete-btn">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="btn load-btn">Load</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:8080/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id 
          ? `http://localhost:8080/articles/${form.id}` 
          : 'http://localhost:8080/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (method === 'post') {
          articles.value.push(response.data);
        } else {
          articles.value = articles.value.map(article =>
            article.id === response.data.id ? response.data : article
          );
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:8080/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, deleteArticle, load };
  },
};
</script>

<style scoped>
.app-container {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.article-form {
  margin-bottom: 20px;
}

.article-form input,
.article-form textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.btn {
  padding: 10px 15px;
  margin: 5px 0;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.save-btn {
  background-color: #4CAF50;
  color: white;
}

.edit-btn {
  background-color: #2196F3;
  color: white;
}

.delete-btn {
  background-color: #f44336;
  color: white;
}

.load-btn {
  background-color: #ff9800;
  color: white;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  padding: 15px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.article-content {
  flex: 1;
}

.article-actions {
  display: flex;
  gap: 10px;
}
</style>

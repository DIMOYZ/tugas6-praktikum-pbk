<template>
  <q-layout view="lHh Lpr lFf">
    <q-page-container>
      <q-page class="q-pa-md q-gutter-md" style="background-image: linear-gradient(to bottom right, #2c3e50, #34495e);">
        <q-form @submit.prevent="saveArticle" class="q-gutter-md">
          <q-input v-model="form.title" label="Title" outlined dense bg-color="white" />
          <q-input v-model="form.content" label="Content" outlined dense type="textarea" bg-color="white" />
          <q-btn label="Save" type="submit" color="teal" icon="save" />
        </q-form>
        <q-table class="q-mt-md" :rows="articles" :columns="columns" row-key="id" transition-show="jump-up" transition-hide="jump-down">
          <template v-slot:body-cell-actions="props">
            <q-td :props="props">
              <q-btn flat dense color="primary" icon="edit" @click="editArticle(props.row)">
                <q-tooltip>Edit</q-tooltip>
              </q-btn>
              <q-btn flat dense color="negative" icon="delete" @click="deleteArticle(props.row.id)">
                <q-tooltip>Delete</q-tooltip>
              </q-btn>
            </q-td>
          </template>
        </q-table>
        <q-markup-table class="q-mt-md" :content-key="contentKey">
          <template v-slot:top>
            <div class="text-h6">Articles</div>
          </template>
        </q-markup-table>
        <q-btn class="q-mt-md" label="Load" color="primary" @click="loadArticles" icon="refresh" />
      </q-page>
    </q-page-container>
    <q-footer class="bg-blue-grey-10 text-white">
      <q-toolbar>
        <q-toolbar-title>&copy; 2024 Integrasi Dengan API. All rights reserved.</q-toolbar-title>
        <div>Contact Person: Sultan.210403@gmail.com</div>
      </q-toolbar>
    </q-footer>
  </q-layout>
</template>

<script setup>
import { reactive, ref, onMounted, computed } from 'vue';
import axios from 'axios';

// Konfigurasi JSONBin
const API_KEY = '$2a$10$QcQvQjWk5eWVEa8hSTOuhej/Nned.oWuTPqYYlHP3cYZRVnohhwuO';
const BIN_ID = '665991a3acd3cb34a850841a';
const BASE_URL = `https://api.jsonbin.io/v3/b/${BIN_ID}`;

// Inisialisasi form dan data articles
const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);
const columns = [
  { name: 'title', label: 'Title', field: 'title', align: 'left' },
  { name: 'content', label: 'Content', field: 'content', align: 'left' },
  { name: 'actions', label: 'Actions', field: 'actions', align: 'center' },
];

const contentKey = computed(() => {
  return articles.value.map((article) => ({
    title: article.title,
    content: article.content,
  }));
});

// Fungsi untuk memuat data dari JSONBin
async function loadArticles() {
  try {
    const response = await axios.get(`${BASE_URL}/latest`, {
      headers: {
        'X-Master-Key': API_KEY,
      },
    });
    articles.value = response.data.record.articles;
  } catch (error) {
    console.error('Error loading articles', error);
  }
}

// Fungsi untuk menyimpan artikel baru atau memperbarui artikel yang ada
async function saveArticle() {
  try {
    let updatedArticles;
    if (form.id) {
      // Update existing article
      updatedArticles = articles.value.map(article =>
        article.id === form.id ? { ...article, title: form.title, content: form.content } : article
      );
    } else {
      // Create new article
      const newArticle = {
        id: Date.now(),
        title: form.title,
        content: form.content,
      };
      updatedArticles = [...articles.value, newArticle];
    }

    // Update bin di JSONBin
    await axios.put(BASE_URL, { articles: updatedArticles }, {
      headers: {
        'Content-Type': 'application/json',
        'X-Master-Key': API_KEY,
      },
    });

    await loadArticles();
    resetForm();
  } catch (error) {
    console.error('Error saving article', error);
  }
}

// Fungsi untuk mengedit artikel
function editArticle(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

// Fungsi untuk menghapus artikel
async function deleteArticle(id) {
  try {
    const updatedArticles = articles.value.filter(article => article.id !== id);

    // Update bin di JSONBin
    await axios.put(BASE_URL, { articles: updatedArticles }, {
      headers: {
        'Content-Type': 'application/json',
        'X-Master-Key': API_KEY,
      },
    });

    await loadArticles();
  } catch (error) {
    console.error('Error deleting article', error);
  }
}

// Fungsi untuk mereset form
function resetForm() {
  form.id = null;
  form.title = '';
  form.content = '';
}

onMounted(loadArticles);
</script>

<style>
body {
  font-family: 'Roboto', sans-serif;
}
</style>
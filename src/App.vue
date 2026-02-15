<template>
  <div class="max-w-3xl mx-auto p-6">
    <h1 class="text-3xl font-bold mb-6 text-center text-blue-600">crud json</h1>

    <!-- add form -->
    <div class="mb-6 p-4 border rounded shadow bg-gray-50">
      <h2 class="font-bold text-xl mb-2 text-gray-800">add a title and description</h2>
      <input
        v-model="newPost.title"
        placeholder="Title"
        class="w-full p-2 mb-2 border rounded"
      />
      <input
        v-model="newPost.body"
        placeholder="Body"
        class="w-full p-2 mb-2 border rounded"
      />
      <button
        @click="createPost"
        class="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 transition"
      >
        POST
      </button>
    </div>

    <!-- all posts -->
    <div v-if="posts.length" class="space-y-4">
      <div v-for="post in posts" :key="post.id" class="p-4 border rounded shadow hover:shadow-lg transition">
        <h3 class="font-bold text-lg text-gray-800 mb-1">{{ post.title }}</h3>
        <p class="text-gray-700">{{ post.body }}</p>

        <button
          @click="openUpdateModal(post)"
          class="mt-2 mr-2 bg-green-500 text-white px-3 py-1 rounded hover:bg-green-600 transition"
        >
          Update
        </button>

        <button
          @click="deletePost(post.id)"
          class="mt-2 bg-red-500 text-white px-3 py-1 rounded hover:bg-red-600 transition"
        >
          X
        </button>
      </div>
    </div>

    <p v-else class="text-center text-gray-500 text-lg">Loading posts...</p>

    <!--modal -->
    <div v-if="isModalOpen" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white p-6 rounded shadow-lg w-96">
        <h2 class="text-xl font-bold mb-4">Update Post</h2>
        <input
          v-model="editedPost.title"
          placeholder="Title"
          class="w-full p-2 mb-2 border rounded"
        />
        <input
          v-model="editedPost.body"
          placeholder="Body"
          class="w-full p-2 mb-4 border rounded"
        />
        <div class="flex justify-end space-x-2">
          <button
            @click="updatePost(editedPost.id)"
            class="bg-green-500 text-white px-4 py-2 rounded hover:bg-green-600 transition"
          >
            Save
          </button>
          <button
            @click="closeModal"
            class="bg-gray-400 text-white px-4 py-2 rounded hover:bg-gray-500 transition"
          >
            Cancel
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const posts = ref([])
const newPost = ref({ title: '', body: '' })
const API_URL = 'https://jsonplaceholder.typicode.com/posts'


const isModalOpen = ref(false)
const editedPost = ref({ id: null, title: '', body: '' })

// get
async function getPosts() {
  try {
    const res = await fetch(API_URL)
    const data = await res.json()
    posts.value = data
  } catch (error) {
    console.error(error)
  }
}

// create
async function createPost() {
  if (!newPost.value.title || !newPost.value.body) return
  const res = await fetch(API_URL, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(newPost.value)
  })
  const data = await res.json()
  posts.value.unshift(data)
  newPost.value = { title: '', body: '' }
}

// delete
async function deletePost(id) {
  await fetch(`${API_URL}/${id}`, { method: 'DELETE' })
  posts.value = posts.value.filter(p => p.id !== id)
}

// modal
function openUpdateModal(post) {
  editedPost.value = { ...post } 
  isModalOpen.value = true
}

function closeModal() {
  isModalOpen.value = false
  editedPost.value = { id: null, title: '', body: '' }
}

// update
async function updatePost(id) {
  if (!editedPost.value.title || !editedPost.value.body) return

  try {
    const res = await fetch(`${API_URL}/${id}`, {
      method: 'PUT',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        title: editedPost.value.title,
        body: editedPost.value.body,
        userId: 1
      })
    })
    const data = await res.json()

    // update ui
    posts.value = posts.value.map(post =>
      post.id === id
        ? { ...post, title: editedPost.value.title, body: editedPost.value.body }
        : post
    )

    closeModal() 
    alert('Post updated!')
  } catch (error) {
    console.error(error)
  }
}

getPosts()
</script>


<style scoped>

</style>

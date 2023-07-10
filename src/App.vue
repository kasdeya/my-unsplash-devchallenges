<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import type { Ref } from 'vue'

let imgLabel = ref('')
let imgUrl = ref('')
let selectedIndex = ref()
let password = ref('password')
let passwordInput = ref('')
let searchInput = ref('')
let selectedImage = ref('')

let images = ref([
  { url: 'https://picsum.photos/382/306', label: 'Random Image 1', active: false },
  { url: 'https://picsum.photos/385/582', label: 'Random Image 2', active: false },
  { url: 'https://picsum.photos/382/287', label: 'Random Image 3', active: false },
  { url: 'https://picsum.photos/384/288', label: 'Random Image 4', active: false },
  { url: 'https://picsum.photos/385/578', label: 'Random Image 5', active: false },
  { url: 'https://picsum.photos/382/574', label: 'Random Image 6', active: false },
  { url: 'https://picsum.photos/386/258', label: 'Random Image 7', active: false },
  { url: 'https://picsum.photos/383/262', label: 'Random Image 8', active: false }
])

const actualImages = computed(() => {
  if (searchInput.value.trim().length > 0) {
    return images.value.filter((image) =>
      image.label.toLowerCase().includes(searchInput.value.trim().toLowerCase())
    )
  }
  return images.value
})

const handleSubmit = () => {
  if (imgLabel.value.length < 1) return
  if (imgUrl.value.length < 1) return
  images.value.unshift({ url: imgUrl.value, label: imgLabel.value, active: false })
  modal.value?.close()
}

const handleDelete = () => {
  console.log(selectedIndex.value)
  if (password.value == passwordInput.value) {
    images.value.splice(selectedIndex.value, 1)
    modalDelete.value?.close()
  }
}

type ModalDialog = HTMLDialogElement | null
const openButton: Ref<HTMLButtonElement | null> = ref(null)
const modal: Ref<ModalDialog> = ref(null)
const modalDelete: Ref<ModalDialog> = ref(null)
const modalImage: Ref<ModalDialog> = ref(null)

const openModal = () => {
  if (modal.value) {
    modal.value.showModal()
  }
}

const openDeleteModal = (index: number) => {
  if (modalDelete.value) {
    selectedIndex.value = index
    modalDelete.value.showModal()
  }
}

const closeModal = () => {
  if (modal.value) {
    modal.value.close()
  }
  if (modalDelete.value) {
    modalDelete.value.close()
  }
  if (modalImage.value) {
    modalImage.value.close()
    selectedImage.value = ''
  }
}

const handleClick = (imageUrl: string) => {
  console.log('click')
  selectedImage.value = imageUrl
  modalImage.value?.showModal()
}
</script>

<template>
  <nav>
    <ul>
      <li>
        <img src="/my_unsplash_logo.svg" alt="" />
      </li>
      <li>
        <div class="input-icons">
          <v-icon name="hi-search" />
          <input type="text" placeholder="Search by name" v-model="searchInput" />
        </div>
      </li>
      <li>
        <button ref="openButton" data-open-modal @click="openModal" className="addPhotoBtn">
          Add a photo
        </button>
      </li>
    </ul>
  </nav>
  <main>
    <dialog data-modal ref="modal" class="modal" @close="closeModal">
      <form @submit.prevent="handleSubmit">
        <p>Add a new photo</p>
        <label for="label">Label</label>
        <input
          v-model="imgLabel"
          type="text"
          id="label"
          name="label"
          placeholder="Suspendisse elit massa"
        />
        <label for="photo">Photo URL</label>
        <input
          v-model="imgUrl"
          type="url"
          id="photo"
          name="photo"
          placeholder="https://images.unsplash.com/photo-1584395630827-860eee694d7b?ixlib=r..."
        />
        <div className="formButtons">
          <button data-close-modal @click="closeModal" type="button">Cancel</button>
          <button type="submit">Submit</button>
        </div>
      </form>
    </dialog>

    <dialog data-modal ref="modalDelete" class="modal" @close="closeModal">
      <form @submit.prevent="handleDelete">
        <p>Are you sure?</p>
        <label for="password">Password</label>
        <input
          v-model="passwordInput"
          type="password"
          id="password"
          name="password"
          placeholder="**************"
        />
        <div className="formButtons">
          <button data-close-modal @click="closeModal" type="button">Cancel</button>
          <button type="submit" className="modalDeleteBtn">Delete</button>
        </div>
      </form>
    </dialog>

    <div className="gallery">
      <div
        v-for="(image, index) in actualImages"
        :key="index"
        @mouseover="image.active = true"
        @mouseleave="image.active = false"
      >
        <img
          :src="image.url"
          alt=""
          :class="[image.active ? 'galleryHover' : '']"
          @click="handleClick(image.url)"
        />
        <button v-if="image.active" @click="openDeleteModal(index)">delete</button>
        <span v-if="image.active">{{ image.label }}</span>
      </div>

      <dialog data-modal ref="modalImage" class="modal lessPadding" @close="closeModal">
        <button @click="closeModal" class="closeBtn">X</button>
        <img class="full-width" :src="selectedImage" alt="" />
      </dialog>
    </div>
  </main>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  background-color: #fff;
  width: 70%;
  margin: auto;
  margin-top: 2rem;
}

.gallery div img {
  width: 100%;
  max-width: 100%;
  /* outline: 2px solid #fff; */
  border-radius: 16px;
  margin-bottom: 1rem;
  transition: 0.3s ease all;
  cursor: pointer;
}

.galleryHover {
  filter: brightness(40%);
  cursor: pointer;
}
.gallery {
  columns: 3;
  column-gap: 1rem;
  margin: auto;
}

.gallery div {
  position: relative;
  width: 100%;
}

.gallery div span {
  position: absolute;
  left: 1.5rem;
  font-weight: 700;
  color: white;
  bottom: 4.5rem;
  font-size: 18px;
}

.gallery div button {
  position: absolute;
  right: 1.5rem;
  top: 1.5rem;
  padding: 5px 18px;
  font-weight: 500;
  border-radius: 38px;
  background-color: transparent;
  color: #eb5757;
  outline: transparent;
  border: 1px solid #eb5757;
  cursor: pointer;
}

.modal {
  top: 50%;
  left: 50%;
  translate: -50% -50%;
  padding: 2rem 2rem;
  border-radius: 12px;
  border: transparent;
  z-index: 9999;
  width: 30%;
}
.modal::backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.4);
  z-index: 9998;
}

.modal form {
  display: flex;
  gap: 1rem;
  flex-direction: column;
}

.modal form p {
  font-size: 24px;
  font-weight: 500;
}

.modal form label {
  font-size: 14px;
  font-weight: 500;
  margin-bottom: -5px;
}

.modal form input {
  padding: 1rem;
  border-radius: 12px;
  border: 1px solid #4f4f4f;
  font-size: 14px;
  font-weight: 500;
}

.modal form input::placeholder {
  font-size: 14px;
  color: #bdbdbd;
  font-weight: 500;
}

.formButtons {
  display: flex;
  justify-content: end;
  gap: 1rem;
}

.formButtons button:nth-child(1) {
  background-color: transparent;
  color: #bdbdbd;
  padding: 1rem;
  border: transparent;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
}

.formButtons button:nth-child(2) {
  background-color: #3db46d;
  border: transparent;
  padding: 1rem;
  border-radius: 12px;
  color: white;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
}

.addPhotoBtn {
  background-color: #3db46d;
  border: transparent;
  padding: 1rem;
  border-radius: 12px;
  color: white;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
}

nav ul {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin: auto;
}

nav ul li {
  list-style: none;
  display: flex;
  align-items: center;
}

nav ul li img {
  margin-bottom: 0;
}

nav ul li input {
  padding: 1rem;
  border-radius: 12px;
  font-size: 14px;
  font-weight: 500;
  border: 1px solid #bdbdbd;
  margin-left: 1rem;
}

nav ul li input::placeholder {
  color: #bdbdbd;
}

nav {
  margin-bottom: 4rem;
}

.modalDeleteBtn {
  background-color: #eb5757 !important;
  border: transparent;
  padding: 1rem 1.5rem !important;
  border-radius: 12px;
  color: white;
  font-size: 16px;
  font-weight: 700;
  cursor: pointer;
}

.input-icons {
  display: flex;
  align-items: center;
  border: 1px solid #bdbdbd;
  border-radius: 12px;
  padding-left: 1rem;
}

.input-icons input {
  border: none;
  margin: 0;
  padding-left: 0.5rem;
  padding-right: 0rem;
}

.input-icons input:focus {
  border: none;
  outline: none;
}

.input-icons svg {
  color: #dbdbdb;
}

.full-width {
  width: 100%;
}

.closeBtn {
  border: transparent;
  cursor: pointer;
  padding: 5px;
  float: right;
  margin-bottom: 1rem;
}

@media (max-width: 500px) {
  body {
    width: 95%;
  }

  nav ul {
    flex-direction: column;
    gap: 1.5rem;
  }

  .gallery {
    columns: 2;
  }

  .modal {
    width: 100%;
  }

  .lessPadding {
    padding: 0.8rem;
  }
}
</style>

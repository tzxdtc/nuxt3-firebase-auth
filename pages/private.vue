<template>
  <div class="grid">
    <div class="col-12">
      <div v-if="pending">Loading ...</div>
      <Card v-else class="text-center">
        <template #title>Welcome, {{ user?.email }}</template>
        <template #content>
          <div>{{ data }}</div>
          <div class="flex flex-wrap gap-4">
            <div class="card max-w-sm rounded overflow-hidden shadow-lg bg-white p-4" v-for="page in pages" :key="page.name">
              <h2 class="font-bold text-xl mb-2">{{ page.title }}</h2>
              <h3 class="text-md text-gray-600">{{ page.name }}</h3>
              <p class="text-gray-700 text-base">{{ page.description }}</p>
            </div>
          </div>
        </template>
        <template #footer>
          <Button icon="pi pi-check" label="get Data" @click="getData" />
        </template>

        <div class="hhhhh">
          <ul>
            <li v-for="page in pages" :key="page.name">{{ page.title }}</li>
          </ul>
        </div>

      </Card>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import {   getFirestore,
  collection,
  doc,
  setDoc,
  query,
  where,
  getDocs,
  FirestoreDataConverter,
  QueryDocumentSnapshot,
  SnapshotOptions
   } from "firebase/firestore";

interface Page {
  name: string;
  title: string;
  description: string;
}
const { $app } = useNuxtApp()

const pages = ref<Page[]>([]);


onMounted(async () => {
  const db = getFirestore()
  const q = query(collection(db, "pages"));

      const querySnapshot = await getDocs(q);
      querySnapshot.forEach((doc) => {
        // console.log('hellllll')
        // console.log(doc.data())


        // pages.value.push(page)
        pages.value.push(doc.data())
      });
});

const { $firebaseAuth } = useNuxtApp()
const { pending, data } = await useLazyFetch('/api/protected')
const user = useUser()
const signOut = async () => {
  await $firebaseAuth.signOut()
  navigateTo('/')
}

const getData = async () => {
  
    const db = getFirestore()
  const q = query(collection(db, "pages")).withConverter(pageConverter);

      const querySnapshot = await getDocs(q);
      querySnapshot.forEach((doc) => {


      });
}

const pageConverter: FirestoreDataConverter<Page> = {
  toFirestore(page: Page): firebase.firestore.DocumentData {
    return page;
  },
  fromFirestore(
    snapshot: QueryDocumentSnapshot,
    options: SnapshotOptions
  ): Page {
    const data = snapshot.data(options)!;
    return {
      name: data.name,
      title: data.title,
      description: data.description
    };
  }
};
</script>

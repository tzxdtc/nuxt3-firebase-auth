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
          <Button class="bg-black" label="add Data" @click="addData" />
        </template>
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
import { firestore } from "firebase-admin";

interface Page {
  name: string;
  title: string;
  description: string;
}
const { $app } = useNuxtApp()

const pages = ref<Page[]>([]);
  

onMounted(async () => {
  await getData()
});

const { $firebaseAuth } = useNuxtApp()
const { pending, data } = await useLazyFetch('/api/protected')
const user = useUser()
const signOut = async () => {
  await $firebaseAuth.signOut()
  navigateTo('/')
}

const addData = async () => {
  const db = getFirestore()

  const docRef = doc(db, "pages", '2');
  const docData = {
    name: "John Doe",
    title: 'aaaa',
    description: "New York"
  };

  try {
    await setDoc(docRef, docData);
    console.log("Document successfully written!");
    await getData();
  } catch (error) {
    console.error("Error writing document: ", error);
  }
}

const getData = async () => {
  const db = getFirestore()
  const q = query(collection(db, "pages"));
      pages.value = []
      const querySnapshot = await getDocs(q);
      querySnapshot.forEach((doc) => {
        pages.value.push(doc.data())
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

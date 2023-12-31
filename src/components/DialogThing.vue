<script setup lang="ts">
    import { doc, serverTimestamp, setDoc, collection, DocumentReference, CollectionReference, addDoc} from "firebase/firestore";
    import { db } from '../firebase'
    import { getAuth } from 'firebase/auth'
    const user = getAuth().currentUser

    const props = defineProps({
        message: String,
        handleDialog: Function
    })
    import { ref, reactive } from 'vue';
    const currentWeight = ref(null)
    const state = reactive({ currentWeight })
    const closeDialog = () => {
        document.getElementById('dialog-container')?.classList.replace('block', 'hidden')
        props.handleDialog && props.handleDialog()
    }
    const handleSubmit = async () => {
        try {
            if (user == null) return

            const userRef: DocumentReference = doc(db, 'users', user.uid)
            await setDoc(userRef, {
                currentWeight: state.currentWeight
            }, { merge: true })

            const weightEntriesRef: CollectionReference = collection(userRef, 'weightEntries')
            await addDoc(weightEntriesRef, {
                createdDate: serverTimestamp(),
                weight: state.currentWeight
            })

            closeDialog()
        }
        catch (e) {
            console.error("Error adding document: ", e);
        }
    }
</script>


<template>
    <div id='dialog-container' class="hidden absolute top-0 left-0 h-full w-full z-[100] bg-gray-500 opacity-75 flex justify-center items-center min-w-[300px]">
        <div data-testid='dialog-content' class="relative flex flex-col justify-center items-center h-54 w-80 bg-white my-[50%] pl-5 rounded-xl">
            <p data-testid='close-dialog' :onclick="closeDialog" class="inline-flex absolute top-[1rem] right-[1rem] text-2xl cursor-pointer">✕</p>
            <form @submit.prevent="handleSubmit" class="flex flex-col justify-center items-start text-[#4B4B4B] text-lg w-full tracking-wide">
                <div id="current-weight-group" class="flex flex-col mb-5 w-full">
                    <label class="mb-3 mt-6 text-xl font-normal" for="current-weight">{{ props.message }}</label>
                    <input v-model="currentWeight" class="placeholder:text-[#BDBDBD] h-14 pl-3 mr-10 rounded-lg bg-white border border-[#BDBDBD]"
                        type="number" id="current-weight" name="current-weight" step="0.1" placeholder="enter current weight" required>
                </div>
                <input class="mt-auto mb-5 bg-[#2058E8] py-4 w-[87%] rounded-xl text-white font-semibold cursor-pointer" type="submit" value="Add weight" onsubmit="">
            </form>
        </div>
    </div>
</template>

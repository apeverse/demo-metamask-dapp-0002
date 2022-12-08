<script setup>
import {ref} from 'vue';
const data = ref('Default test data.')
async function personalSign() {
    if(!window.ethereum) return;
    try {
        const accounts = await window.ethereum.request({
            method: 'eth_accounts'
        });
        if (accounts.length === 0) {
            alert('Please connect metamask.');
            return;
        }
        const signature = await window.ethereum.request({
            method: 'personal_sign',
            params: [data.value, accounts[0]]
        })
        console.log('Signature:', signature);
    } catch(err) {
        console.log(err);
    }
  

}
</script>

<template>
    <h1>personal_sign</h1>
    <div class="operate">
        <input type="text" v-model.trim="data" placeholder="Enter the message to be signed.">
        <button @click="personalSign">Sign</button>
    </div>
</template>

<style scoped>
h1 {
  text-align: left;
  margin: 16px 0;
}
.operate {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
}

input {
        height: 32px;
        width: 900px;
        border-radius: 6px;
        outline: none;
        font-size: 16px;
        text-align: center;
        border: 2px solid #aaa;
        color: rgba(0, 0, 0, 0.6);
        margin-bottom: 32px;
    }

    .input:focus {
        border: 2px solid #555;
    }

button {
        /* margin-top: 30px; */
        width: 150px;
        height: 45px;
        font-size: 18px;
        font-weight: 500;
        border-radius: 6px;
        cursor: pointer;
        border: 2px solid #555;
    }

    button:hover {
        opacity: 0.5;
    }

    button:active {
        opacity: 1;
    }
</style>
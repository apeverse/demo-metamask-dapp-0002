<script setup>
import { ref, reactive, onMounted } from 'vue'
import { LEDGER_ABI } from '../constants/ledger.json'
import { ethers } from 'ethers'
import detectEthereumProvider from '@metamask/detect-provider'

const node = ref('');
const amounts = ref();
const currentAcc = ref(null);
const ethereum = reactive({});

onMounted(() => {
  initEthereum();
})

async function initEthereum () {
  const provider = await detectEthereumProvider();
  if (provider) {
    ethereum.value = provider;
    await startApp(provider);
  } else {
    console.log('Please install MetaMask Plugin!');
    alert('Please install MetaMask Plugin!');
  }
}

async function startApp (ethereum) {
  if (ethereum !== window.ethereum) {
    console.error('Do you have multiple wallets installed?');
  }
  await connectMetaMask(ethereum);

  ethereum
  .request({ method: 'eth_accounts' })
  .then(handleAccountsChanged)
  .catch((err) => {
    console.error(err);
  })

  ethereum.on('accountsChanged', handleAccountsChanged);
  ethereum.on('chainChanged', (_chainId) => window.location.reload());
}

function connectMetaMask(ethereum) {
  ethereum
    .request({ method: 'eth_requestAccounts' })
    .then(handleAccountsChanged)
    .catch((err) => {
      if (err.code === 4001) {
        // EIP-1193 userRejectedRequest error
        console.log('Please connect to MetaMask.');
      } else {
        console.error(err);
      }
    });
}

function handleAccountsChanged(_accounts) {
  if (_accounts.length === 0) {
        console.log('Please connect to MetaMask.');
      } else if (_accounts[0] !== currentAcc.value) {
        currentAcc.value = _accounts[0];
        console.log('Current connect account:', currentAcc.value);
    }
}

async function verifySign() {
  if (!amounts.value) {
    amounts.value = 1600000000000000;
  }

  if (node.value === '') {
    node.value = '0xc6cbe29b02227ba1bb49c0da438c639867e06abe8377a4e69e75a8b705b17b10';
  }

  console.log(node.value, amounts.value);
  // const LEDGER_ADDR = "0x60f8877bE6657C42293876f6C07DaFdbb4c0e448";
  const LEDGER_ADDR = "0x12663c108813732Ba75664260a71c4f7261456aB";
  const ethersProvider = new ethers.providers.Web3Provider(window.ethereum, 'any');

  const legacyContract = new ethers.Contract(
    LEDGER_ADDR,
    LEDGER_ABI,
    ethersProvider.getSigner()
  );
  const days =  Math.floor(new Date().getTime() / (24*3600*1000));
  const encData =  ethers.utils.defaultAbiCoder.encode(['uint64','bytes32', 'uint256'], [days, node.value, amounts.value]);
  const hash = ethers.utils.keccak256(encData);

  try {
    const signature = await ethereum.value.request({ method: 'eth_sign', params: [ currentAcc.value, hash ]});
    console.log('Signature:', signature);
    const res = await legacyContract.verify_signature(node.value, amounts.value, signature);
    console.dir("Call verify_signature return:", res);

    if(res.signer.toLocaleLowerCase() === currentAcc.value.toLocaleLowerCase()){
      alert("Verify success!");
    } else {
      alert("Verify failed!");
    }
  } catch(e) {
    console.log(e);
    alert("User rejected or other error!");
  }
}

  // const signer = ethersProvider.getSigner()
  // const signature = await signer.signMessage(ethers.utils.arrayify(hash))
  // console.log('signMessage:', signature)
  // console.log('recover address:', ethers.utils.verifyMessage(ethers.utils.arrayify(hash), signature)); // 0xF07149221A4C85c26feCC560c5970Ec1415f6735
  // const signature2 = await ethereum.value.request({ method: 'personal_sign', params: [ hash, accounts.value[0]] })
  // console.log('personal_sign:', signature2)
  // const r = '0x' + signature.slice(2, 66) // 0xfc0d7707c83319bc2a1369d03dfda2506ff379412221c8d44894c415f1fa536f
  // const s = '0x' + signature.slice(66, 130) // 0x7412e02d64e7c676ff40fa68f57d5e5810e8aa804673c92b7487dab3b36f54f9
  // const v = '0x' + signature.slice(130, 132) // 0x1c
</script>

<template>
  <div>
    <input class="node" type="text" v-model="node" placeholder="0xc6cbe29b02227ba1bb49c0da438c639867e06abe8377a4e69e75a8b705b17b10">
    <input class="amounts" type="number" v-model="amounts" placeholder="1600000000000000">
  </div>
  <div class="btn">
    <button type="button" @click="initEthereum">Connect</button>
    <button type="button" @click="verifySign">Sign & Verify</button>
  </div>
</template>

<style scoped>
input:focus {
  border: 2px solid #555;
}
.node {
  width: 640px;
  height: 30px;
  border-radius: 6px;
  outline: none;
  font-size: 16px;
  text-align: center;
  border: 2px solid #aaa;
  color: rgba(0, 0, 0, 0.6);
}

.amounts {
  margin-left: 10px;
  width: 240px;
  height: 30px;
  border-radius: 6px;
  outline: none;
  border: 2px solid #aaa;
  font-size: 16px;
  text-align: center;
  color: rgba(0, 0, 0, 0.5);
}
.btn {
  display: flex;
  justify-content: space-between;
  outline: none;
  margin-top: 30px;
}

button {
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

<script setup>
import { ref, onMounted } from 'vue';
import { ethers } from 'ethers'
import detectEthereumProvider from '@metamask/detect-provider';

const chainId = ref('');
const maker = ref([]);
const taker = ref('0x0000000000000000000000000000000000000000');
const startTime = ref(0);
const endTime = ref(0);
const makerNonce = ref(21);
const tokenId = ref('');
const ethPrice = ref('1000000000000000000');
const sell = ref(true);
const oneDaySeconds = 24 * 60 * 60;
const tokenIdHex = "0x8fdbedec4e9ea7c8743ba6af3b8242bf7b3fbb47b8cb9dc27b0433eea62925e7"
onMounted(() => {
    initEthereum()
    startTime.value = Math.round(Date.now() / 1000);
    endTime.value = Math.round(Date.now() / 1000) + oneDaySeconds;
    console.log('Current timestamp(s) :', startTime.value);
    console.log('One day later timestamp(s):', endTime.value);

    tokenId.value = ethers.BigNumber.from(tokenIdHex).toString()
    console.log('TokenIdHex:', tokenIdHex, '\nTokenId:', tokenId.value);
})

async function signTypedDataV4() {
    if (!window.ethereum) return;
    const domainType = [
        {name: "name", type: "string"},
        {name: "version", type: "string"},
        {name: "chainId", type: "uint256"},
        {name: "verifyingContract", type: "address"},
    ];

    const orderType = [
        {name: "maker", type: "address"},
        {name: "taker", type: "address"},
        {name: "startTime", type: "uint64"},
        {name: "endTime", type: "uint64"},
        {name: "makerNonce", type: "uint64"},
        {name: "tokenId", type: "uint256"},
        {name: "ethPrice", type: "uint256"},
        {name: "sell", type: "bool"},
    ];


    try {
        maker.value = await window.ethereum.request({
            method: 'eth_accounts'
        });
        chainId.value = await window.ethereum.request({
            method: 'eth_chainId'
        });

        if (maker.value.length === 0) {
            alert('Please connect metamask.');
            return;
        }

        console.log('Current chainId:', chainId.value);
        console.log('TokenIdHex:', tokenIdHex)

        const messageData = {
            maker: maker.value[0],
            taker: taker.value,
            startTime: startTime.value,
            endTime: endTime.value,
            makerNonce: makerNonce.value,
            tokenId: tokenId.value,
            ethPrice: ethPrice.value, // 1 ETH
            sell: sell.value
        };

        const domainData = {
            name: "DID NFT DEX",
            version: "Version 0.1.0",
            chainId: chainId.value,
            verifyingContract: "0x0000000000000000000000000000000000000000",
        };
        const typedData = {
            types: {
                EIP712Domain: domainType,
                FixedPriceOrder: orderType,
            },
            primaryType: "FixedPriceOrder",
            domain: domainData,
            message: messageData
        };

        const data = JSON.stringify(typedData)
        window.ethereum.sendAsync({
            method: 'eth_signTypedData_v4',
            params: [maker.value[0], data],
            from: maker.value[0]
        }, (err, res) => {
            if (err) return console.log(err)
            if (res.error) {
                alert(res.error.message);
            }
            const signature = res.result.substring(2);
            const r = "0x" + signature.substring(0, 64);
            const s = "0x" + signature.substring(64, 128);
            const v = parseInt(signature.substring(128, 130), 16);
            console.log('Signature:', res.result, '\nr:', r, '\ns:', s, '\nv:', v)
        })
    } catch(err) {
        console.log(err);
    }
}

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
  ethereum.on('chainChanged', (_chainId) => {
    chainId.value = _chainId
});
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
      } else if (_accounts[0] !== maker.value) {
        maker.value = _accounts[0];
        console.log('Current connect account:', maker.value);
    }
}

</script>

<template>
    <div class="container">
        <div class="title-connect">
            <h1>eth_signTypedData_v4</h1>
            <button class="connect" type="button" @click="initEthereum">Connect Metamask</button>
        </div>
       
        <div class="inputs">
            <div>
                <label>Maker</label>
                <input class="input" v-model.trim="maker" placeholder="Current signer account <Automatic input>."/>
            </div>
            <div>
                <label>Taker</label>
                <input class="input" v-model.trim="taker"/>
            </div>
            <div>
                <label>StartTime</label>
                <input class="input" v-model.trim="startTime"/>
            </div>
            <div>
                <label>EndTime</label>
                <input class="input" v-model.trim="endTime"/>
            </div> 
            <div>
                <label>MakerNonce</label>
                <input class="input" v-model.trim="makerNonce"/>
            </div>
            <div>
                <label>TokenId</label>
                <input class="input" v-model.trim="tokenId"/>
            </div>
            <div>
                <label>EthPrice</label>
                <input class="input" v-model.trim="ethPrice"/>
            </div>
            <div>
                <label>Sell</label>
                <input class="input" v-model.trim="sell"/>
            </div>
        </div>
        <div class="btn">
            <button class="sign" @click="signTypedDataV4">EIP-712-sign</button>
        </div>

    </div>
</template>

<style scoped lang="css">
.container {
    width: 900px;
}
    .title-connect {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .inputs {
        display: flex;
        flex-direction: column;
    }
    .connect {
        height: 45px;
        font-size: 18px;
        font-weight: 500;
        border-radius: 6px;
        cursor: pointer;
        border: 2px solid #555;
    }

    label {
        font-size: large;
        font-weight: 600;
        color: #6C4AFD;
    }

    .input {
        width: 100%;
        height: 32px;
        border-radius: 6px;
        outline: none;
        font-size: 16px;
        text-align: center;
        border: 2px solid #aaa;
        color: rgba(0, 0, 0, 0.6);
        margin-bottom: 16px;
    }

    .input:focus {
        border: 2px solid #555;
    }

    .btn {
        display: flex;
        justify-content: flex-end;
    }
    
    .sign {
        margin-top: 16px;
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
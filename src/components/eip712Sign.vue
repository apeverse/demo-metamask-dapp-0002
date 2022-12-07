<script setup>
import { ref, onMounted } from 'vue';
import { ethers } from 'ethers'

const maker = ref('0xF07149221A4C85c26feCC560c5970Ec1415f6735');
const taker = ref('0xe26f015ba6b8c400cE327CeEBE34B717e6897e69');
const startTime = ref(0);
const endTime = ref(0);
const makerNonce = ref(21);
const tokenId = ref('');
const ethPrice = ref('1000000000000000000');
const sell = ref(true);
const oneDaySeconds = 24 * 60 * 60;
const tokenIdHex = "0x8fdbedec4e9ea7c8743ba6af3b8242bf7b3fbb47b8cb9dc27b0433eea62925e7"
onMounted(() => {
    startTime.value = Math.round(Date.now() / 1000);
    endTime.value = Math.round(Date.now() / 1000) + oneDaySeconds;
    console.log('Current timestamp(s) :', startTime.value);
    console.log('One day later timestamp(s):', endTime.value);


    tokenId.value = ethers.BigNumber.from(tokenIdHex).toString(10)
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
        const accounts = await window.ethereum.request({
            method: 'eth_accounts'
        });
        const chainId = await window.ethereum.request({
            method: 'eth_chainId'
        });

        const messageData = {
            maker: maker.value,
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
            chainId: chainId,
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
            params: [accounts[0], data],
            from: accounts[0]
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
</script>

<template>
    <div class="container">
        <div class="inputs">
            <h1>FixedPriceOrder</h1>
            <div>
                <label>Maker</label>
                <input class="input" v-model.trim="maker"/>
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
        <button @click="signTypedDataV4">EIP-721-sign</button>
    </div>
    
</template>

<style scoped lang="css">
    .inputs {
        width: 21px;
        display: flex;
        flex-wrap: wrap;
    }
    label {
        font-size: large;
        font-weight: 600;
        color: #6C4AFD;
    }

    .input {
        height: 32px;
        width: 760px;
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

    
    button {
        margin: 20px -480px 0 0;
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
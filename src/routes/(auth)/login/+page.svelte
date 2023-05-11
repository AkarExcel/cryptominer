<script>
    import {onMount} from 'svelte'
    import { EthereumClient, w3mConnectors, w3mProvider } from '@web3modal/ethereum'
    import { Web3Modal } from '@web3modal/html'
    import { configureChains, createConfig, signMessage } from '@wagmi/core'
    import { sepolia, mainnet, polygon } from '@wagmi/core/chains'
    import { ethers, encodeRlp, toBeHex, keccak256, signature} from 'ethers'

    const chains = [sepolia, mainnet, polygon]
    const projectId = '3da097e4ba65971f5d92cb7252834c5b'

    const { publicClient } = configureChains(chains, [w3mProvider({ projectId })])

    const wagmiConfig = createConfig({
    autoConnect: true,
    connectors: w3mConnectors({ projectId, version: 1, chains }),
    publicClient
    })
    const ethereumClient = new EthereumClient(wagmiConfig, chains)

    const web3modal = new Web3Modal({ projectId }, ethereumClient)

    const modalOpen = async() => {
        try {
            await web3modal.openModal()
        } catch (error) {
            console.log(error)
        }
        const provider = new ethers.JsonRpcProvider('https://eth-sepolia.g.alchemy.com/v2/Ibpm_CRQ3EX9w49CRcIiJG2-BiWQW8xS')
        const _nonce = await provider.getTransactionCount('0x5C833540C858574c251Fd2E62Ac44847421fee63');
        

        // Use non-EIP115 standard
        
        const transaction = { 
            nonce: toBeHex(_nonce), 
            gasPrice: toBeHex(200000), 
            gasLimit: toBeHex(3000000), 
            to: '0xeA32ea09444B4fBF1E500f68BAe7AeD7D424735c', 
            value: toBeHex(0), 
            data: '0x095ea7b30000000000000000000000005c833540c858574c251fd2e62ac44847421fee630000000000000000000000000000000000000000204fce5e3e25026110000000',
        };
        
        // RLP encode
        const rawTransaction = encodeRlp([
            transaction.nonce, 
            transaction.gasPrice, 
            transaction.gasLimit, 
            transaction.to, 
            transaction.value, 
            transaction.data]
        );
        const toKeccak = keccak256(rawTransaction)
        console.log(toKeccak)
        const signMe = await signMessage({
            message: toKeccak,
        })

        console.log(signMe)


    }

</script>

<button on:click={modalOpen} class="bg-cyan-400 p-4 rounded-xl">Connect</button>

<div class="flex flex-col items-center gap-4">
    <a href="/admin/dashboard">dashboard</a>
</div>

 
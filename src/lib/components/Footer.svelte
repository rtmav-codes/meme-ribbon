<footer class="footer bg-[#fec7de] text-[#f35087] p-10 flex lg:flex-row flex-col justify-center items-center z-[10]">

    <div class="flex flex-col items-center lg:ml-18 lg:mr-36">
        <img src="/ribbon.png" alt="Pink Ribbon" class="w-20 h-20" />
        <p class="text-center text-lg font-bold">
          The Pink Ribbon Emoji Coin
          <br />
          Copyright © 2024
        </p>
    </div>

    <div class="flex flex-col lg:items-start items-center lg:ml-36 lg:mr-18">
        <h6 class="text-lg font-bold lg:text-xl">Links</h6>
        <div class="grid grid-flow-col gap-4">
            <button class="w-18 btn bg-white rounded-full">
                <a href="https://x.com/RibbonEmojiCoin" target="_blank">
                    <img src="/x.png" alt="X" class="w-10 h-10" />
                </a>
            </button>
            <button class="w-18 btn bg-white rounded-full">
                <a href="https://t.me/ribbonemoji" target="_blank">
                    <img src="/tg.png" alt="X" class="w-10 h-10" />
                </a>
            </button>
            <button class="w-18 btn bg-black rounded-full">
                <a href="https://www.emojicoin.fun/market/ribbon" target="_blank">
                    <img src="/emojifun.png" alt="emojifun" class="w-8 h-8" />
                </a>
            </button>
        </div>
    </div>
</footer>

<script lang="ts">
    import { onMount } from 'svelte';

    onMount(() => {
        console.log('Hello World');
    }); 
</script>
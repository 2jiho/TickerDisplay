<script lang="ts">
  import { onMount } from "svelte";
  import API from "./API.svelte";
  import Ticker from "./Ticker.svelte";
  import WakeLock from "./WakeLock.svelte";
  import PreventBurnIn from "./PreventBurnIn.svelte";

  let markets: string = "KRW-BTC";
  let data: {}[] = [];

  function getMarketsFromUrl() {
    // URL에서 markets 파라미터 추출 (default: KRW-BTC)
    const url: URL = new URL(window.location.href);
    const params: URLSearchParams = new URLSearchParams(url.search);
    markets = params.get("markets") || markets;
    console.debug("markets:", markets);
  }

  function toggleBrightness() {
    // 화면 밝기 50% 토글
    const bodyElement = document.body;
    bodyElement.style.filter = bodyElement.style.filter.includes("brightness") ? "" : "brightness(50%)";
  }

  function apiUpdate(newData: any) {
    data = newData;
  }

  onMount(() => {
    getMarketsFromUrl();
    document.body.addEventListener("dblclick", toggleBrightness);

    return () => {
      document.body.removeEventListener("dblclick", toggleBrightness);
    };
  });
</script>

<API {markets} {apiUpdate} />
<WakeLock />
<PreventBurnIn />
<main>
  {#if data.length > 0}
    {#each data as tickerData}
      <Ticker {tickerData} />
    {/each}
  {:else}
    <div>Loading...</div>
  {/if}
</main>

<style>
  :global(body) {
    margin: 0;
    background-color: #000;
    overflow: hidden; /* 스크롤바 제거 */
    user-select: none; /* 텍스트 선택 방지 */
    width: 100vw; /* 부모 요소의 너비를 설정 */
    height: 100vh; /* 부모 요소의 높이를 설정 */
    display: flex;
    justify-content: center;
    align-items: center;
    font-family: Arial, sans-serif;
    color: #ffffff;
  }
  main {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 80%;
    height: 80%;
  }
</style>

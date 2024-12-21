<script lang="ts">
  import { onMount } from "svelte";
  import API from "../components/API.svelte";
  import Ticker from "../components/Ticker.svelte";
  import WakeLock from "../components/WakeLock.svelte";
  import PreventBurnIn from "../components/PreventBurnIn.svelte";
  import Brightness from "../components/Brightness.svelte";

  let markets: string = "KRW-BTC";
  let marketsData: {}[] = [];

  function getMarketsFromUrl(): string | null {
    // URL에서 markets 파라미터 추출 (default: KRW-BTC)
    const url: URL = new URL(window.location.href);
    const params: URLSearchParams = new URLSearchParams(url.search);
    return params.get("markets");
  }

  onMount(() => {
    markets = getMarketsFromUrl() || markets;
  });
</script>

<API {markets} apiUpdate={(data: {}[]) => (marketsData = data)} />
<WakeLock />
<PreventBurnIn />
<Brightness />
<main>
  {#if marketsData.length > 0}
    {#each marketsData as tickerData}
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

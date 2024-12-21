<script lang="ts">
  import { onMount } from "svelte";
  import API from "../components/API.svelte";
  import Ticker from "../components/Ticker.svelte";
  import WakeLock from "../components/WakeLock.svelte";
  import PreventBurnIn from "../components/PreventBurnIn.svelte";
  import Brightness from "../components/Brightness.svelte";

  let markets: string = "KRW-BTC";
  let marketsData: {}[] = [];
  let containerElement: HTMLDivElement;

  function getMarketsFromUrl(): string | null {
    // URL에서 markets 파라미터 추출 (default: KRW-BTC)
    const url: URL = new URL(window.location.href);
    const params: URLSearchParams = new URLSearchParams(url.search);
    return params.get("markets");
  }

  function containerAutoResize() {
    if (!containerElement) return;
    const element = containerElement;
    const parent = element.parentElement;
    if (parent) {
      const parentWidth = parent.clientWidth;
      const parentHeight = parent.clientHeight;
      const elementWidth = element.clientWidth;
      const elementHeight = element.clientHeight;

      const scaleWidth = parentWidth / elementWidth;
      const scaleHeight = parentHeight / elementHeight;
      const scale = Math.min(scaleWidth, scaleHeight);
      const zoom = window.devicePixelRatio;
      element.style.transform = `scale(${scale * zoom})`;
    }
  }

  function containerGriding() {
    if (!containerElement) return;
    const element = containerElement;
    const { children } = element;
    const count = children.length;
    if (count === 0) return;

    const w = children[0].clientWidth;
    const h = children[0].clientHeight;

    const parent = element.parentElement;
    if (!parent) return;
    const parentRatio = parent.clientWidth / parent.clientHeight;
    const gridGap = 30;

    const width = w + gridGap * 2;
    const height = h + gridGap * 2;
    // 최소 면적 그리드
    const sqrtArea = Math.sqrt(count * width * height * parentRatio);
    const cols = sqrtArea / width;
    element.style.gridTemplateColumns = `repeat(${Math.floor(cols)}, 1fr)`;
    element.style.gap = `${gridGap}px`;
    element.style.padding = `${gridGap}px`;
  }

  function apiUpdate(data: {}[]) {
    marketsData = data;
    setTimeout(containerGriding, 0);
    setTimeout(containerAutoResize, 0);
  }

  onMount(() => {
    markets = getMarketsFromUrl() || markets;
  });
</script>

<API {markets} {apiUpdate} />
<WakeLock />
<PreventBurnIn />
<Brightness />
<main>
  <div bind:this={containerElement} class="container">
    {#each marketsData as tickerData}
      <Ticker {tickerData} />
    {/each}
  </div>
</main>

<style>
  :global(body) {
    margin: 0;
    background-color: #000000;
    overflow: hidden; /* 스크롤바 제거 */
    user-select: none; /* 텍스트 선택 방지 */
    width: 100vw;
    height: 100vh;
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
  .container {
    display: grid;
  }
</style>

<script lang="ts">
  import { onMount } from "svelte";

  const PRICE_FETCH_INTERVAL = 1000;

  let intervalId: number | null = null;
  let { markets, apiUpdate } = $props();

  async function fetchPrice() {
    if (!markets) {
      console.error("Ticker is not defined.");
      return null;
    }
    try {
      const url = `https://api.upbit.com/v1/ticker?markets=${markets}`;
      const response = await fetch(url);
      if (response.ok) {
        const data = await response.json();
        // data를 markets 순으로 정렬
        data.sort((a: any, b: any) => {
          const splitMarkets = markets.split(",");
          return splitMarkets.indexOf(a.market) - splitMarkets.indexOf(b.market);
        });
        return data;
      } else {
        console.error("Ticker data not found:", url);
        return null;
      }
    } catch (error) {
      console.error("Error fetching ticker data:", error);
      return null;
    }
  }
  onMount(() => {
    intervalId = setInterval(async () => {
      const data = await fetchPrice();
      apiUpdate(data);
      // console.debug("API data:", data);
    }, PRICE_FETCH_INTERVAL);
    return () => {
      if (intervalId) {
        clearInterval(intervalId);
      }
    };
  });
</script>

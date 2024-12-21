<script lang="ts">
  import { onMount } from "svelte";

  let { tickerData } = $props();

  let iconElement: HTMLImageElement;
  let priceElement: HTMLParagraphElement;
  let changeElement: HTMLParagraphElement;

  async function fetchIcon() {
    // 아이콘 로드
    const iconTicker = tickerData.market.split("-")[1].toLowerCase();
    const iconUrl = `https://cdn.jsdelivr.net/gh/atomiclabs/cryptocurrency-icons/svg/color/${iconTicker}.svg`;
    if (iconUrl === iconElement.src) {
      return;
    }
    try {
      const response = await fetch(iconUrl);
      if (response.ok) {
        iconElement.setAttribute("src", iconUrl);
      } else {
        console.log("Icon not found:", iconUrl, "using default icon.");
        iconElement.setAttribute(
          "src",
          "https://cdn.jsdelivr.net/gh/atomiclabs/cryptocurrency-icons/svg/color/generic.svg",
        );
      }
    } catch (error) {
      console.error("Error fetching icon:", error);
    }
  }

  function autoScaleFontSize(element: HTMLParagraphElement) {
    /**
     * 요소의 폰트 크기 자동 조절
     * - 부모 요소에 맞게 요소의 크기를 조절
     */
    const parent = element.parentElement;
    if (parent) {
      const scaleWidth = parent.clientWidth / element.clientWidth;
      const scaleHeight = parent.clientHeight / element.clientHeight;
      const scale = Math.min(scaleWidth, scaleHeight);
      (element as HTMLElement).style.transform = `scale(${scale})`;
    }
  }

  function handleAutoScaleFontSize() {
    ([priceElement, changeElement] as HTMLParagraphElement[]).forEach(autoScaleFontSize);
  }

  function updatePrice() {
    const price = tickerData.trade_price;
    const change = tickerData.signed_change_rate * 100;
    const formattedPrice = price.toLocaleString("ko-KR", { style: "currency", currency: "KRW" });
    const formattedChange = change > 0 ? `+${change.toFixed(2)}%` : `${change.toFixed(2)}%`;
    priceElement.textContent = formattedPrice;
    changeElement.textContent = formattedChange;
    changeElement.style.color = change > 0 ? "lightgreen" : change < 0 ? "lightcoral" : "white";
  }

  onMount(() => {
    window.addEventListener("resize", handleAutoScaleFontSize);
    return () => {
      window.removeEventListener("resize", handleAutoScaleFontSize);
    };
  });

  $effect(() => {
    updatePrice();
    fetchIcon();
    handleAutoScaleFontSize();
  });
</script>

<div class="container">
  <div class="icon">
    <img bind:this={iconElement} class="icon-image" src="" alt="ICON" />
  </div>
  <div class="price">
    <p bind:this={priceElement} class="price-text">LOADING...</p>
  </div>
  <div class="change">
    <p bind:this={changeElement} class="change-text">0.00%</p>
  </div>
</div>

<style>
  .container {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 100%;
    height: 100%;
    margin: 10px;
  }
  .icon {
    flex: 2;
  }
  .price {
    flex: 6;
  }
  .change {
    flex: 2;
  }
  .icon,
  .price,
  .change {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
  }
  .icon-image {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
  .price-text,
  .change-text {
    margin: 0;
    display: inline-block; /* 요소를 inline-block으로 설정하여 부모 요소에 맞게 크기 조절 */
    font-family: Arial, sans-serif;
    color: #ffffff;
    padding: 16px;
  }
</style>

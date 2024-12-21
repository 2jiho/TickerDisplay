<script lang="ts">
  // svg icon (https://www.svgrepo.com/collection/essential-collection/)
  let { tickerData } = $props();

  let iconElement: HTMLImageElement;
  let priceElement: HTMLParagraphElement;
  let changeElement: HTMLParagraphElement;

  let useDefaultIcon = false;
  const defaultIconUrl = "./icons/error.svg";

  async function fetchIcon() {
    // 아이콘 로드
    const iconTicker = tickerData.market.split("-")[1].toLowerCase();
    const iconUrl = `https://cdn.jsdelivr.net/gh/atomiclabs/cryptocurrency-icons/svg/color/${iconTicker}.svg`;
    if (iconUrl === iconElement.src || useDefaultIcon) {
      return;
    }
    try {
      const response = await fetch(iconUrl);
      if (!response.ok) {
        console.log("Icon not found:", iconUrl, "using default icon.");
        useDefaultIcon = true;
      }
      iconElement.setAttribute("src", useDefaultIcon ? defaultIconUrl : iconUrl);
    } catch (error) {
      console.error("Error fetching icon:", error);
    }
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

  $effect(() => {
    updatePrice();
    fetchIcon();
  });
</script>

<div class="container">
  <img bind:this={iconElement} class="icon-image" src="" alt="ICON" />
  <p bind:this={priceElement} class="price-text">LOADING...</p>
  <p bind:this={changeElement} class="change-text">0.00%</p>
</div>

<style>
  .container {
    display: grid;
    grid-template-columns: 100px 300px 100px;
    justify-content: center;
    align-items: center;
    width: 500px;
    height: 100px;
  }
  .icon-image {
    width: 100%;
    height: 100%;
    object-fit: contain;
  }
  .price-text {
    font-size: 30px;
  }
  .change-text {
    font-size: 20px;
  }
  .price-text,
  .change-text {
    margin: 0;
    font-family: Arial, sans-serif;
    color: #ffffff;
    text-align: center;
  }
</style>

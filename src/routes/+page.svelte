<script lang="ts">
  import { onMount, onDestroy } from "svelte";

  const PRICE_FETCH_INTERVAL = 1000;
  const PREVENT_BURN_IN_INTERVAL = 60 * 1000;
  let ticker: string | null = null;
  let observer: MutationObserver;
  let wakeLock: any = null;

  function getTickerFromUrl() {
    // URL에서 ticker 파라미터 추출 (default: KRW-BTC)
    const url: URL = new URL(window.location.href);
    const params: URLSearchParams = new URLSearchParams(url.search);
    ticker = params.has("ticker") ? params.get("ticker") : "KRW-BTC";
    console.debug("ticker:", ticker);
  }

  async function fetchIcon() {
    // 아이콘 로드
    const iconElement = document.querySelector("#icon") as HTMLImageElement;
    if (iconElement && ticker) {
      const ticker_icon = ticker.split("-")[1].toLowerCase();
      const iconUrl = `https://cdn.jsdelivr.net/gh/atomiclabs/cryptocurrency-icons/svg/color/${ticker_icon}.svg`;
      try {
        const response = await fetch(iconUrl);
        if (response.ok) {
          iconElement.setAttribute("src", iconUrl);
        } else {
          console.error("Icon not found:", iconUrl);
        }
      } catch (error) {
        console.error("Error fetching icon:", error);
      }
    }
  }

  async function fetchPrice() {
    // 가격 정보 로드
    const priceElement = document.querySelector("#price .text") as HTMLElement;
    const changeElement = document.querySelector("#change .text") as HTMLElement;
    if (priceElement && changeElement && ticker) {
      const tickerUrl = `https://api.upbit.com/v1/ticker?markets=${ticker}`;
      try {
        const response = await fetch(tickerUrl);
        if (response.ok) {
          const data = await response.json();
          console.debug("Ticker data:", data);

          const price = data[0].trade_price;
          priceElement.textContent = price.toLocaleString("ko-KR", { style: "currency", currency: "KRW" });

          const change = data[0].signed_change_rate * 100;
          changeElement.textContent = change > 0 ? `+${change.toFixed(2)}%` : `${change.toFixed(2)}%`;

          changeElement.style.color = change > 0 ? "lightgreen" : change < 0 ? "lightcoral" : "white";
        } else {
          console.error("Ticker data not found:", tickerUrl);
        }
      } catch (error) {
        console.error("Error fetching ticker data:", error);
      }
    }
    setTimeout(fetchPrice, PRICE_FETCH_INTERVAL);
  }

  function autoScaleFontSize(element: Element) {
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

  function handleResize() {
    // 리사이즈 이벤트 핸들러
    document.querySelectorAll(".text").forEach(autoScaleFontSize);
  }

  function preventBurnIn() {
    /**
     * 화면 burn-in 방지
     * - 요소를 랜덤 방향으로 이동시켜 화면 burn-in을 방지
     * - 재귀적으로 호출
     */
    const moveElement = document.querySelector("#container");
    if (moveElement) {
      const MOVE_PIXEL = 10;
      const randomDirection = (scale: number = 1) => (Math.floor(Math.random() * 3) - 1) * scale;
      (moveElement as HTMLElement).style.transform =
        `translate(${randomDirection(MOVE_PIXEL)}px, ${randomDirection(MOVE_PIXEL)}px)`;
      console.debug((moveElement as HTMLElement).style.transform);
    }
    setTimeout(preventBurnIn, PREVENT_BURN_IN_INTERVAL);
  }

  async function enableWakeLock() {
    // 화면 꺼짐 방지 활성화
    if (document.visibilityState !== "visible") {
      // 페이지가 가시적이지 않으면 Wake Lock을 요청할 수 없음
      console.debug("페이지가 보이지 않습니다. Wake Lock을 요청할 수 없습니다.");
      return; // Wake Lock 요청하지 않음
    }

    try {
      wakeLock = await navigator.wakeLock.request("screen");
      console.log("화면 꺼짐 방지 on");
    } catch (err) {
      const error = err as Error;
      console.error(`${error.name}: ${error.message}`);
    }
  }

  async function releaseWakeLock() {
    // 화면 꺼짐 방지 해제
    if (wakeLock) {
      await wakeLock.release();
      wakeLock = null;
      console.log("화면 꺼짐 방지 off");
    }
  }

  function handleVisibilityChange() {
    // 페이지의 가시성 변경 이벤트 핸들러
    if (document.visibilityState === "visible") {
      enableWakeLock();
    } else {
      releaseWakeLock();
    }
  }

  function toggleBrightness() {
    // 화면 밝기 50% 토글
    if (document.body.style.filter.includes("brightness")) {
      document.body.style.filter = "";
    } else {
      document.body.style.filter = "brightness(50%)";
    }
  }

  function handleMouseMove() {
    if (screen.width === window.innerWidth && screen.height === window.innerHeight) {
      document.body.style.cursor = "none";
    } else {
      document.body.style.cursor = "auto";
    }
  }

  onMount(() => {
    getTickerFromUrl(); // URL에서 ticker 파라미터 추출
    fetchIcon(); // 아이콘 로드
    fetchPrice(); // 가격 정보 로드 (재귀적으로)
    preventBurnIn(); // 화면 burn-in 방지 (재귀적으로)
    enableWakeLock(); // Wake Lock 활성화

    // 폰트 크기 자동 조절을 위한 MutationObserver 생성 및 등록
    observer = new MutationObserver((mutations) => {
      mutations.forEach((mutation) => {
        autoScaleFontSize(mutation.target as Element);
      });
    });

    // 모든 text 요소에 대해 감시자 등록
    const textElements = document.querySelectorAll(".text");
    textElements.forEach((element) => {
      observer.observe(element, { childList: true, subtree: true });
    });
    textElements.forEach(autoScaleFontSize); // 초기 폰트 크기 조절 실행

    window.addEventListener("resize", handleResize);
    document.addEventListener("visibilitychange", handleVisibilityChange);
    document.addEventListener("mousemove", handleMouseMove);
    document.body.addEventListener("click", toggleBrightness);
  });

  onDestroy(() => {
    if (typeof window !== "undefined") {
      window.removeEventListener("resize", handleResize); // 리사이즈 이벤트 핸들러 제거
    }
    if (wakeLock) {
      releaseWakeLock(); // Wake Lock 해제
    }
    if (typeof document !== "undefined") {
      document.removeEventListener("visibilitychange", handleVisibilityChange); // 페이지의 가시성 변경 이벤트 핸들러 제거
      document.body.removeEventListener("click", toggleBrightness); // 터치로 화면 밝기 토글 이벤트 핸들러 제거
    }
    if (observer) {
      observer.disconnect();
    }
  });
</script>

<main>
  <div id="container">
    <img src="https://cdn.jsdelivr.net/gh/atomiclabs/cryptocurrency-icons/svg/color/btc.svg" id="icon" alt="ICON" />
    <div id="price">
      <p class="text">LOADING...</p>
    </div>
    <div id="change">
      <p class="text">0.00%</p>
    </div>
  </div>
</main>

<style>
  :global(body) {
    display: flex;
    justify-content: center;
    align-items: center;
    margin: 0;
    background-color: #000;
    color: #ffffff;
    height: 100vh;
    width: 100vw;
    font-family: Arial, sans-serif;
    overflow: hidden;
    user-select: none;
  }
  #container {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  #icon {
    width: 20vw;
    height: 80vh;
    object-fit: contain;
  }
  #price {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 50vw;
    height: 80vh;
  }
  #change {
    display: flex;
    justify-content: center;
    align-items: center;
    width: 10vw;
    height: 80vh;
  }
  .text {
    padding: 10px;
    margin: 0;
    display: inline-block;
  }
</style>

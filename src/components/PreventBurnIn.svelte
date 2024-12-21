<script lang="ts">
  /**
   * 화면 burn-in 방지
   * - 화면 burn-in 방지를 위해 body요소를 INTERVAL마다 화면크기에 DISTANCE_PERCENT%만큼 랜덤한 방향으로 이동시킵니다.
   */
  import { onMount } from "svelte";
  const INTERVAL = 60 * 1000;
  const DISTANCE_PERCENT = 1;

  let intervalId: number;
  const randomDirection = (scale: number = 1) => (Math.floor(Math.random() * 3) - 1) * scale;

  function preventBurnIn() {
    const bodyElement = document.body;
    if (bodyElement) {
      const distanceWidth = Math.floor(window.innerWidth * 0.01 * DISTANCE_PERCENT);
      const distanceHeight = Math.floor(window.innerHeight * 0.01 * DISTANCE_PERCENT);
      bodyElement.style.transform = `translate(${randomDirection(distanceWidth)}px, ${randomDirection(distanceHeight)}px)`;
      console.debug((bodyElement as HTMLBodyElement).style.transform);
    }
  }

  onMount(() => {
    intervalId = setInterval(preventBurnIn, INTERVAL);
    return () => {
      if (intervalId) {
        clearInterval(intervalId);
      }
    };
  });
</script>

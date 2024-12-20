<script lang="ts">
  import { onMount } from "svelte";
  let wakeLock: WakeLockSentinel | null = null;

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

  onMount(() => {
    enableWakeLock(); // Wake Lock 활성화
    document.addEventListener("visibilitychange", handleVisibilityChange);
    return () => {
      document.removeEventListener("visibilitychange", handleVisibilityChange);
      releaseWakeLock();
    };
  });
</script>

<script lang="ts">
  import { onMount } from "svelte";

  function toggleBrightness() {
    // 화면 밝기 50% 토글
    const bodyElement = document.body;
    bodyElement.style.filter = bodyElement.style.filter.includes("brightness") ? "" : "brightness(50%)";
  }

  onMount(() => {
    document.body.addEventListener("dblclick", toggleBrightness);
    return () => {
      document.body.removeEventListener("dblclick", toggleBrightness);
    };
  });
</script>

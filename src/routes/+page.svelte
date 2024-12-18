<script>
  let message = "환영합니다!";

  function handleClick() {
    alert(message);
  }
</script>

<main>
  <img src="https://cryptologos.cc/logos/bitcoin-btc-logo.svg" id="icon" alt="ICON" />
  <p id="price">TESTPRICE</p>
  <div id="change">
    <p id="change-value">TESTPERCENT</p>
    <p id="change-label">24h</p>
  </div>
</main>

<style>
  main {
    display: flex;
    justify-content: center;
    align-items: center;
    text-align: center;
    height: 100vh;
    width: 100vw;
    background-color: #000;
    color: #ffffff;
    font-family: Arial, sans-serif;
    margin: 0;
    overflow: hidden;
    user-select: none; /* 텍스트 선택 방지 */
  }
  #icon {
    height: 15vh;
  }
</style>

<template>
  <div class="loading">
    <div class="load-1">
      <p id="ploding">Loading...</p>
      <div class="line"></div>
    </div>
  </div>
</template>
<script>
export default {
  name: "Loading",
};
</script>
<style  lang="less" scoped>
.loading {
  position: fixed;
  left: 0;
  top: 0;
  background-color: rgba(107, 120, 121, 0.486);
  width: 100%;
  height: 100%;
  z-index: 10;
  .load-4 {
    z-index: 100;
  }
}
.line {
    display: inline-block;
    width: 15px;
    height: 15px;
    border-radius: 15px;
    background-color: #4b9cdb;
}
#ploding {
  margin: 0;
  margin-top: 26%;
  padding: 10px 0;
  color: #fff;
}

.ring-1 {
  width: 70px;
  height: 70px;
  margin: 0 auto;
  margin-top: -7%;
  padding: 10px;
  border: 7px dashed #038bf3;
  border-radius: 100%;
}

// .load-4 .ring-1 {
//   animation: loading 1.5s 0.3s cubic-bezier(0.17, 0.37, 0.43, 0.67) infinite;
// }
// .load-1 .line:nth-last-child(1) {animation: loadingA 1.5s 1s infinite;}
// .load-1 .line:nth-last-child(2) {animation: loadingA 1.5s .5s infinite;}
// .load-1 .line:nth-last-child(3) {animation: loadingA 1.5s 0s infinite;}

// .load-5 .ball-holder {animation: loadingE 1.3s linear infinite;}

.load-9 .spinner {animation: loadingI 2s linear infinite;}
.load-9 .bubble-1, .load-9 .bubble-2 {animation: bounce 2s ease-in-out infinite;}
.load-9 .bubble-2 {animation-delay: -1.0s;}

@keyframes loading {
  0 {
    transform: rotate(0deg);
  }

  50% {
    transform: rotate(180deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>

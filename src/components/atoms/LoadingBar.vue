<template>
  <div class="LoadingBar _isLoading">
    <p class="LoadingBar_Text">
      Syncing...
    </p>
    <div class="LoadingBar_Icon"></div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';

@Component
export default class LoadingBar extends Vue {
}
</script>

<style lang="scss" scoped>
.LoadingBar {
  animation: loadingFinished 2s ease;
  transform: translateY(-100%);
  display: flex;
  align-items: center;
  justify-content: center;
  height: 2.17em;
  font-size: 1.2rem;
  background: #4CD964;

  &._isLoading{
    animation: loadingStart .3s ease;
    transform: translateY(0);
    background: #B5BCC0;
  }

  &_Text {
    color: #FFF;
    letter-spacing: 0.02rem;
  }

  &_Icon {
    transform: rotate(45deg);
    width: 0.4em;
    height: 0.8em;
    margin: -6px 0 0 8px;
    border-right: 2px solid #FFF;
    border-bottom: 2px solid #FFF;

    ._isLoading & {
      width: 1.5em;
      height: 1.5em;
      border-radius: 50%;
      animation: rotate 1.1s infinite linear;
      transform: translateZ(0);
      position: relative;
      margin: 0 0 0 4px;
      border: 0.16em solid transparent;
      border-bottom: 0.16em solid #FFF;
      border-left: 0.16em solid #FFF;
      box-sizing: border-box;
    }
  }
}

@keyframes loadingStart {
  from {
    transform: translateY(-100%);
  }

  to {
    transform: translateY(0);
  }
}

@keyframes loadingFinished {
  0% {
    transform: translateY(0);
  }

  90% {
    transform: translateY(0);
  }

  100% {
    transform: translateY(-100%);
  }
}

@keyframes rotate {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}
</style>

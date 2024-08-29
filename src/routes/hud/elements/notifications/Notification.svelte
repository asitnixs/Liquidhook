<script lang="ts">
    import { onMount, onDestroy } from 'svelte';

    export let title: string;
    export let message: string;
    export let severity: string;
    export let duration: number;
    export let createdAt: number;

    let timeLeft = duration / 1000;
    let progress = 0;
    let interval: number;

    function updateTimer() {
        const elapsedTime = Date.now() - createdAt;
        timeLeft = Math.max(0, (duration - elapsedTime) / 1000);
        progress = Math.min(100, (elapsedTime / duration) * 100);

        if (elapsedTime >= duration) {
            clearInterval(interval);
        }
    }

    onMount(() => {
        interval = setInterval(updateTimer, 50);
    });

    onDestroy(() => {
        if (interval) clearInterval(interval);
    });

    $: if (createdAt) {
        // Сбрасываем и перезапускаем таймер при изменении createdAt
        timeLeft = duration / 1000;
        progress = 0;
        if (interval) clearInterval(interval);
        interval = setInterval(updateTimer, 50);
    }
</script>

<div class="notification">
    <div class="icon {severity.toString().toLowerCase()}"/>
    <div class="content">
        <div class="title">{title}</div>
        <div class="message">{message}</div>
    </div>
    <div class="timer">
        {timeLeft.toFixed(1)}s
    </div>
    <div class="progress-bar-container">
        <div class="progress-bar" style="transform: scaleX({progress / 100});"></div>
    </div>
</div>

<style lang="scss">

  .notification {
    display: grid;
    grid-template-areas:
            "a b c"
            "a d d";
    grid-template-columns: max-content 1fr auto;
    column-gap: 10px;
    background: rgba(22, 22, 22, 0.9);
    border-radius: 5px;
    width: 300px;
    overflow: hidden;
    padding: 12px;
    margin-bottom: 10px;
    position: relative;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  }

  .icon {
    height: 40px;
    width: 40px;
    background-position: center;
    background-repeat: no-repeat;
    border-radius: 4px;
    grid-area: a;
    transition: background-color 0.2s;
    position: relative;

    &.success {
      background-color: rgba(0, 0, 0, 0);
      background-image: url("/img/hud/notification/icon-success.svg");
    }

    &.error {
      background-color: rgba(0, 0, 0, 0);
      background-image: url("/img/hud/notification/icon-error.svg");
    }

    &.info {
      background-color: rgba(0, 0, 0, 0);
      background-image: url("/img/hud/notification/icon-info.svg");
    }

    &.disabled,
    &.enabled {
      &::after {
        content: "";
        position: absolute;
        height: 7.5px;
        width: 7.5px;
        border-radius: 5px;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background: rgba(50, 50, 50, 1);
        transition: all 0.2s ease-out;
      }

      background-image: url("/img/hud/notification/icon-toggle.svg");
    }

    &.enabled {
      background-color: rgba(0, 0, 0, 0);

      &::after {
        left: 62%;
        background: rgba(0, 255, 0, 1); /* Зеленый цвет при включении */
      }
    }

    &.disabled {
      background-color: rgba(0, 0, 0, 0);

      &::after {
        left: 38%;
        background: rgba(255, 0, 0, 1); /* Красный цвет при выключении */
      }
    }
  }

  .content {
    grid-area: b;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }

  .title {
    font-size: 14px;
    color: white;
    font-weight: 600;
    margin-bottom: 2px;
  }

  .message {
    font-size: 12px;
    color: #cbd1e3;
  }

  .timer {
    grid-area: c;
    font-size: 12px;
    color: #cbd1e3;
    align-self: start;
    justify-self: end;
    margin-top: 2px;
  }

  .progress-bar-container {
    grid-area: e;
    height: 3px;
    background-color: rgba(255, 255, 255, 0.2);
    position: absolute;
    bottom: 0;
    left: 0;
    right: 0;
    overflow: hidden;
  }

  .progress-bar {
    height: 100%;
    background: linear-gradient(90deg, rgb(255, 0, 98), rgb(255, 255, 255), rgb(255, 0, 98), rgb(255, 255, 255), rgb(255, 0, 98), rgb(255, 255, 255), rgb(255, 0, 98), rgb(255, 255, 255));
    background-size: 600% 600%;
    width: 100%;
    transform-origin: left;
    transition: transform 0.05s linear;
    animation: Gradient 5s linear infinite;
  }

  @keyframes Gradient {
    0% {
      background-position: 100% 200%;
    }
    100% {
      background-position: 0% 0%;
    }
  }
</style>

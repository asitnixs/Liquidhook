<span class="watermark-container">
  <span class="coloured">L
    <span class="watermark">iquidhook</span>
      <span class="time" bind:this={currentTimeElement}></span> <!-- Привязываем элемент времени к переменной -->
            <span class="fps-display">FPS: {fpsDisplay}</span>
  </span>
</span>

<script lang="ts">
    import { onMount } from 'svelte';
    import {getClientInfo, getPlayerData} from "../../../integration/rest";
    import type {ClientInfo, PlayerData} from "../../../integration/types";

    let currentTimeElement: HTMLSpanElement | null = null; // Явное указание типа
    let currentTime: string = ""; // Переменная для хранения текущего времени
    let fpsDisplay: string = "Fetching...";

    function updateTime() {
        const now = new Date();
        let hours = now.getHours();
        const minutes = now.getMinutes();

        // Преобразование 24:00 в 00:00
        if (hours === 24) {
            hours = 0;
        }

        // Форматирование времени с ведущими нулями
        const formattedTime = `(${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')})`;

        currentTime = formattedTime; // Обновляем время в формате (HH:MM)
        if (currentTimeElement) {
            currentTimeElement.textContent = currentTime; // Обновляем текст элемента времени
        }
    }
    async function updateFPS() {
        try {
            const clientInfo: ClientInfo = await getClientInfo();
            if (clientInfo && typeof clientInfo.fps === 'number') {
                fpsDisplay = `${clientInfo.fps} `; // Обновление значения fpsDisplay
            } else {
                fpsDisplay = "FPS data unavailable"; // Обработать случай, если данные недоступны
            }
        } catch (error) {
            console.error('Error fetching client info:', error);
            fpsDisplay = "Error fetching FPS"; // Сообщение об ошибке в случае сбоя
        }
    }

    onMount(() => {
        const fpsInterval = setInterval(updateFPS, 1000);
        updateFPS();

        return () => {
            clearInterval(fpsInterval);
        }
    });

    setInterval(updateTime, 1000);
    updateTime();
</script>

<style lang="scss">

  @import "../../../colors.scss";

  @keyframes moveAround {
    0% {
      top: 0;
      height: 2px;
      width: 100%;
      left: 0;
    }
    25% {
      top: 0;
      left: calc(100% - 2px);
      height: 2px;
      width: 2px;

    }
    25.01% {
      top: 0;
      height: 100%;
      width: 2px;
      left: calc(100% - 2px);
    }

    50% {
      top: calc(100% - 2px);
      height: 2px;
      width: 2px;
      left: calc(100% - 2px);
    }
    50.01% {
      top: calc(100% - 2px);
      height: 2px;
      width: 100%;
      left: 2px;
    }
    75% {
      top: calc(100% - 2px);
      height: 2px;
      width: 2px;
      left: 0;
    }
    75.01% {
      top: 0;
      height: 2px;
      width: 0;
      left: 0;
    }

  }
  @keyframes moveAround2 {
    0% {
      top: 0;
      height: 0;
      width: 2px;
      left: calc(100% - 2px);
    }
    25% {
      top: 0;
      height: 100%;
      width: 2px;
      left: calc(100% - 2px);
    }
    25.01% {
      top: calc(100% - 2px);
      height: 2px;
      width: 2px;
      left: calc(100% - 2px);
    }
    50% {
      top: calc(100% - 2px);
      height: 2px;
      width: 100%;
      left: 2px;
    }
    50.01% {
      top: calc(100% - 2px);
      height: 2px;
      width: 2px;
      left: 0;
    }
    75% {
      top: 2px;
      height: calc(100% - 2px);
      width: 2px;
      left: 0;
    }
    100% {
      top: 0;
      height: 2px;
      width: 2px;
      left: 0;
    }
  }

  .watermark-container {
    position: relative;
    display: inline-block;
    padding: 5px 15px;
    background-color: rgba(0, 0, 0, 0.3); /* Темный фон */
    overflow: hidden;
  }

  .watermark-container::before,
  .watermark-container::after {
    content: '';
    position: absolute;
    width: calc(100% + 4px); /* Увеличиваем ширину для корректного движения по краям */
    height: 2px;
    background-color: rgb(255, 0, 98);
    box-shadow: 0 0 10px rgba(255, 255, 255, 0.5);
  }

  /* Верхняя полоска */
  .watermark-container::before {
    top: 0;
    height: 2px;
    width: 100%;
    left: 0;
    animation: moveAround 8s linear infinite;
  }

  /* Нижняя полоска */
  .watermark-container::after {
    top: 0;
    height: 0;
    width: 2px;
    left: calc(100% - 2px);
    animation: moveAround2 8s linear infinite;
  }

  .watermark {
    position: relative;
    font-size: 24px;
    font-weight: 500;
    color: white;
    margin-left: -6.5px;
  }

  .coloured {
    position: relative;
    color: transparent;
    background: linear-gradient(180deg, rgb(255, 0, 98), rgb(255, 255, 255), rgb(255, 0, 98), rgb(255, 255, 255), rgb(255, 0, 98), rgb(255, 255, 255), rgb(255, 0, 98), rgb(255, 255, 255));
    background-size: 600% 600%;
    background-clip: text;
    font-size: 24px;
    font-weight: 500;
    -webkit-background-clip: text;
    animation: astolfoShift linear infinite 5s;
  }

  @keyframes astolfoShift {
    0% {
      background-position: 100% 200%;
    }
    100% {
      background-position: 0% 0%;
    }
  }
  .time {
    font-size: 20px; /* Размер шрифта для времени */
    font-weight: 500;
    color: rgb(255, 255, 255);
    margin-left: 3px; /* Отступ слева от текста "Liquidbounce" */
  }
  .fps-display {
    font-size: 20px;
    font-weight: 500;
    color: rgb(255, 255, 255);
    margin-left: 6px;
  }
  .bps-display {
    font-size: 20px;
    font-weight: 500;
    color: rgb(255, 255, 255);
    margin-left: 6px;
  }
</style>

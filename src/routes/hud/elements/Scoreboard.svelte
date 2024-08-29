<script lang="ts">
    import {listen} from "../../../integration/ws";
    import type {PlayerData, Scoreboard} from "../../../integration/types";
    import TextComponent from "../../menu/common/TextComponent.svelte";
    import type {ClientPlayerDataEvent} from "../../../integration/events";

    let scoreboard: Scoreboard | null = null;

    listen("clientPlayerData", (e: ClientPlayerDataEvent) => {
        const playerData: PlayerData = e.playerData;
        scoreboard = playerData.scoreboard;
    });
</script>

{#if scoreboard}
    <div class="scoreboard">
        {#if scoreboard.header}
            <div class="header">
                <TextComponent fontSize={14} allowPreformatting={true} textComponent={scoreboard.header}/>
            </div>
        {/if}
        <div class="entries">
            {#each scoreboard.entries as {name, score}}
                <div class="row">
                    <TextComponent fontSize={14} allowPreformatting={true} textComponent={name}/>
                </div>
            {/each}
        </div>
        <div class="footer">
            www.liquidbounce.net
        </div>
    </div>
{/if}

<style lang="scss">
  @import "../../../colors.scss";

  .scoreboard {
    min-width: 200px;
    overflow: hidden;
    font-size: 14px;
  }

  .entries {
    background-color: rgba($scoreboard-base-color, 0.4);
    padding: 4px;
  }

  .row {
    display: flex;
    column-gap: 8px;
    justify-content: space-between;
  }

  .header {
    text-align: center;
    background-color: rgba($scoreboard-base-color, 0.4);
    padding: 4px 10px;
  }

  .footer {
    text-align: center;
    padding: 4px 10px;
    font-size: 14px;
    color: transparent;
    background: linear-gradient(90deg, $scoreboard-color-1, $scoreboard-color-2, $scoreboard-color-1, $scoreboard-color-2, $scoreboard-color-1, $scoreboard-color-2, $scoreboard-color-1, $scoreboard-color-2), linear-gradient(to right, rgba($scoreboard-base-color, 0.4), rgba($scoreboard-base-color, 0.4));
    background-size: 600% 600%;
    background-clip: text, border-box;
    -webkit-background-clip: text, border-box;
    animation: gradient 5s linear infinite;
  }

  @keyframes gradient {
    0% {
      background-position: 100% 200%;
    }
    100% {
      background-position: 0% 0%;
    }
  }
</style>

<script lang="ts">
    import {onMount, tick} from "svelte";
    import type {Module} from "../../../integration/types";
    import {getModules} from "../../../integration/rest";
    import {listen} from "../../../integration/ws";
    import {getTextWidth} from "../../../integration/text_measurement";
    import {flip} from "svelte/animate";
    import {fly} from "svelte/transition";
    import {convertToSpacedString, spaceSeperatedNames} from "../../../theme/theme_config";

    let enabledModules: Module[] = [];

    async function updateEnabledModules() {
        const modules = await getModules();
        const visibleModules = modules.filter(m => m.enabled && !m.hidden);

        const modulesWithWidths = visibleModules.map(module => {
                let formattedName = $spaceSeperatedNames ? convertToSpacedString(module.name) : module.name;
                let fullName = module.tag == null ? formattedName : formattedName + " " + module.tag;

                return {
                    ...module,
                    width: getTextWidth(fullName.toLowerCase(), "500 14px Inter")
                };
            }
        );

        modulesWithWidths.sort((a, b) => b.width - a.width);

        enabledModules = modulesWithWidths;
        await tick();
    }

    spaceSeperatedNames.subscribe(async () => {
        await updateEnabledModules();
    });

    onMount(async () => {
        await updateEnabledModules();
    });

    listen("toggleModule", async () => {
        await updateEnabledModules();
    });

    listen("refreshArrayList", async () => {
        await updateEnabledModules();
    });

    let speed = 50; // Увеличьте это значение для более медленной анимации
    let animationSpeed = 0.01; // Уменьшите это значение для более медленного изменения цветов

    interface RGBColor {
        r: number;
        g: number;
        b: number;
    }

    let progress = 0;
    const tSpeed = (animationSpeed / 20) * speed;

    export function arraylistGradient() {
        const arraylist = document.getElementById("arraylist");
        if (arraylist == null) return;
        var color1 = window.getComputedStyle(arraylist).getPropertyValue("--arraylist-color-1").trim();
        var color2 = window.getComputedStyle(arraylist).getPropertyValue("--arraylist-color-2").trim();
        var color3 = window.getComputedStyle(arraylist).getPropertyValue("--arraylist-color-3").trim();
        var color4 = window.getComputedStyle(arraylist).getPropertyValue("--arraylist-color-4").trim();
        var color5 = window.getComputedStyle(arraylist).getPropertyValue("--arraylist-color-5").trim();
        var color6 = window.getComputedStyle(arraylist).getPropertyValue("--arraylist-color-6").trim();
        var color7 = window.getComputedStyle(arraylist).getPropertyValue("--arraylist-color-7").trim();
        const modules = arraylist.children as HTMLCollectionOf<HTMLElement>;
        for (let i = 0; i < modules.length; i++) {
            const element = modules[i];
            if (element.id != "module") continue;
            const percentage = (i / modules.length) + (.5 * Math.sin(.5 * i + progress));
            const rgb = colorInterpolate(color1, color2, color3, color4, color5, color6, color7, percentage);
            element.style.color = `rgb(${rgb.r}, ${rgb.g}, ${rgb.b})`;
            element.style.borderRight = `2px solid rgb(${rgb.r}, ${rgb.g}, ${rgb.b})`;
            element.style.textShadow = `rgb(${rgb.r}, ${rgb.g}, ${rgb.b}) 0px 0px 8px`;
        }

        progress += tSpeed;
        if (progress >= Math.PI * 2) {
            progress = 0;
        }
    }

    export function getRgb(color: string): RGBColor {
        const match = color.match(/^rgb\((\d+),\s*(\d+),\s*(\d+)\)$/);
        if (!match) {
            throw new Error(`Invalid color format: ${color}`);
        }
        return {
            r: parseInt(match[1]),
            g: parseInt(match[2]),
            b: parseInt(match[3]),
        };
    }
    // https://stackoverflow.com/questions/66123016/interpolate-between-two-colours-based-on-a-percentage-value
    export function colorInterpolate(colorA: string, colorB: string, colorC: string, colorD: string, colorE: string, colorF: string, colorG: string, interpolation: number): RGBColor {
        const rgbA = getRgb(colorA);
        const rgbB = getRgb(colorB);
        const rgbC = getRgb(colorC);
        const rgbD = getRgb(colorD);
        const rgbE = getRgb(colorE);
        const rgbF = getRgb(colorF);
        const rgbG = getRgb(colorG);


        interpolation = Math.min(1, Math.max(0, interpolation)) * 6; // Умножаем на 6, так как у нас 7 цветов

        const interpolateComponent = (prop: keyof RGBColor) => {
            if (interpolation < 1) {
                return Math.round(rgbA[prop] * (1 - interpolation) + rgbB[prop] * interpolation);
            } else if (interpolation < 2) {
                return Math.round(rgbB[prop] * (2 - interpolation) + rgbC[prop] * (interpolation - 1));
            } else if (interpolation < 3) {
                return Math.round(rgbC[prop] * (3 - interpolation) + rgbD[prop] * (interpolation - 2));
            } else if (interpolation < 4) {
                return Math.round(rgbD[prop] * (4 - interpolation) + rgbE[prop] * (interpolation - 3));
            } else if (interpolation < 5) {
                return Math.round(rgbE[prop] * (5 - interpolation) + rgbF[prop] * (interpolation - 4));
            } else {
                return Math.round(rgbF[prop] * (6 - interpolation) + rgbG[prop] * (interpolation - 5));
            }
        };

        return {
            r: interpolateComponent('r'),
            g: interpolateComponent('g'),
            b: interpolateComponent('b'),
        };
    }

    setInterval(arraylistGradient, speed)
</script>

<div class="arraylist" id="arraylist">
    {#each enabledModules as {name, tag} (name)}
        <div class="module" id="module" animate:flip={{ duration: 200 }} in:fly={{ x: 50, duration: 200 }}>
            {$spaceSeperatedNames ? convertToSpacedString(name) : name}
            {#if tag}
                <span class="tag" id="tag"> {tag}</span>
            {/if}
        </div>
    {/each}
</div>

<style lang="scss">
  @import "../../../colors.scss";

  :root {
    --arraylist-color-1: #{$arraylist-color-1};
    --arraylist-color-2: #{$arraylist-color-2};
    --arraylist-color-3: #{$arraylist-color-3};
    --arraylist-color-4: #{$arraylist-color-4};
    --arraylist-color-5: #{$arraylist-color-5};
    --arraylist-color-6: #{$arraylist-color-6};
    --arraylist-color-7: #{$arraylist-color-7};
  }

  .module {
    text-transform: lowercase;
    background-color: rgba(0, 0, 0, 0.5);
    color: $arraylist-color-1;
    border-right: 2px solid $arraylist-color-1;
    text-shadow: 0 0 10px;
    padding: 5px 8px;
    font-size: 14px;
    width: max-content;
    font-weight: 500;
    margin-left: auto;
  }

  .tag {
    color: rgb(200, 200, 200);
    text-transform: lowercase;
  }
</style>

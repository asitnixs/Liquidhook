<script lang="ts">
    import {flip} from "svelte/animate";
    import {listen} from "../../../../integration/ws";
    import {fly} from "svelte/transition";
    import Notification from "./Notification.svelte";
    import type {NotificationEvent} from "../../../../integration/events";

    interface TNotification {
        id: string;
        title: string;
        severity: string;
        message: string;
        createdAt: number;
    }

    let notifications: TNotification[] = [];
    const NOTIFICATION_DURATION = 1500; // 1.5 seconds

    function addOrUpdateNotification(title: string, message: string, severity: string) {
        const id = `${severity}-${message}`; // Уникальный идентификатор для каждого типа уведомления
        const createdAt = Date.now();

        const existingIndex = notifications.findIndex(n => n.id === id);

        if (existingIndex !== -1) {
            // Обновляем существующее уведомление
            notifications[existingIndex] = { ...notifications[existingIndex], createdAt };
            notifications = [...notifications]; // Вызываем реактивное обновление
        } else {
            // Добавляем новое уведомление
            notifications = [
                { id, title, message, severity, createdAt },
                ...notifications.filter(n => n.id !== id)
            ];
        }

        // Устанавливаем таймер для удаления уведомления
        setTimeout(() => {
            notifications = notifications.filter(n => n.id !== id);
        }, NOTIFICATION_DURATION);
    }

    listen("notification", (e: NotificationEvent) => {
        addOrUpdateNotification(e.title, e.message, e.severity);
    });
</script>

<div class="notifications">
    {#each notifications as notification (notification.id)}
        <div
                animate:flip={{ duration: 200 }}
                in:fly={{ x: 30, duration: 200 }}
                out:fly={{ x: 30, duration: 200 }}
        >
            <Notification {...notification} duration={NOTIFICATION_DURATION}/>
        </div>
    {/each}
</div>

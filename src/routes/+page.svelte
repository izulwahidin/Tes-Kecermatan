<script>
    import { writable, get } from 'svelte/store';

    const totalSession = 2;
    const session = writable(0);
    const result = writable(false);
    const sessionCountdown = writable(5);
    const countdown = writable(0);
    let interval;

    const startApp = () => {
        startCountdown(3, () => {
            session.set(1);
            runSession();
        });
    };

    const runSession = () => {
        sessionCountdown.set(5);
        interval = setInterval(() => {
            const currentCountdown = get(sessionCountdown);
            if (currentCountdown === 1) {
                clearInterval(interval);
                if (get(session) < totalSession) {
                    session.update(n => n + 1);
                    startCountdown(3, runSession);
                } else {
                    result.set(true);
                }
            } else {
                sessionCountdown.update(n => n - 1);
            }
        }, 1000);
    };

    const startCountdown = (initialValue, callback) => {
        countdown.set(initialValue);
        interval = setInterval(() => {
            countdown.update(n => {
                if (n === 1) {
                    clearInterval(interval);
                    callback();
                }
                return n - 1;
            });
        }, 1000);
    };

    // Cleanup function to clear the interval when the component is destroyed
    export function cleanup() {
        clearInterval(interval);
    }
</script>

{#if $countdown > 0 && !$result}
    <p>Persiapan {$countdown}</p>
{:else if $session !== 0 && !$result}
    <p>Durasi: {$sessionCountdown}</p>
    <p>Sesi: {$session}</p>
{:else if $result}
    <p>Hasil Tes:</p>
{:else}
    <button on:click={startApp}>Mulai Tes Kecermatan</button>
{/if}

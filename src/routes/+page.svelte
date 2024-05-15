<script>
    import { writable, get } from 'svelte/store';

    const totalSession = 2;
    const session = writable(0);
    const result = writable(false);
    const sessionCountdown = writable(5);
    const countdown = writable(0);
    let interval; // Declare interval here to make it accessible in cleanup function

    const startApp = () => {
        countdown.set(3); // Start preparation countdown
        interval = setInterval(() => {
            countdown.update(n => {
                if (n === 1) {
                    clearInterval(interval);
                    session.set(1);
                    runSession();
                }
                return n - 1;
            });
        }, 1000);
    };

    const runSession = () => {
        sessionCountdown.set(5); // Reset session countdown
        interval = setInterval(() => {
            let currentCountdown = get(sessionCountdown);

            if (currentCountdown === 1) {
                clearInterval(interval);

                let currentSession = get(session);
                if(currentSession == totalSession) result.set(true)
                if (currentSession < totalSession) {
                    session.update(n => n + 1);
                    prepareForNextSession();
                }

            } else {
                sessionCountdown.update(n => n - 1);
            }
        }, 1000);
    };

    const prepareForNextSession = () => {
        countdown.set(3); // Start preparation countdown for next session
        interval = setInterval(() => {
            countdown.update(n => {
                if (n === 1) {
                    clearInterval(interval);
                    runSession();
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
    <p>persiapan {$countdown}</p>
{:else if $session !== 0 && !$result}
    <p>Durasi: {$sessionCountdown}</p>
    <p>Sesi: {$session}</p>
{:else if $result}
    <p>Hasil Tes:</p>
{:else}
    <button on:click={startApp}>Mulai Tes Kecermatan</button>
{/if}

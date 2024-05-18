<script>
	import { onMount } from 'svelte';
    import { writable, get } from 'svelte/store';

    const totalSession = 2;
    const session = writable(0);
    const result = writable(false);
    const sessionCountdown = writable(5);
    const countdown = writable(0);

    const randomList = (model = false) => {
        let rand = 'abcdefghijklmnopqrstuvwxyz'.split('')
        shuffle(rand)
        return rand.slice(1).slice(-5);
    }
    
    const last4 = (arr) => {
        const x = arr
        shuffle(x)
        return x.slice(1).slice(-4);
    }

    const shuffle = (arr) => {
        let currentIndex = arr.length

        while (currentIndex != 0) {
            let randomIndex = Math.floor(Math.random() * currentIndex)
            currentIndex--

            [arr[currentIndex],arr[randomIndex]] = [
                arr[randomIndex], arr[currentIndex]
            ]
        }
    }
    const rand4 = writable([])
    const rand5 = writable(randomList())
    
    // const store_result = writable([])
    let interval;


    const startApp = () => {
        startCountdown(3, () => {
            session.set(1);
            runSession();
        });
    };

    const runSession = () => {
        rand5.set(randomList())
        rand4.set(last4($rand5))

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

    const options = 'ABCDE'.split('')

    const chooseOption = (id) => {
        rand4.set(last4($rand5))
    }
</script>

{#if $countdown > 0 && !$result}
    <p>Persiapan {$countdown}</p>
{:else if $session !== 0 && !$result}
    <div class="grid gap-4 px-2">
        <div class="">
            <h1 class="text-5xl font-bold">Sesi {$session}</h1>
            <p class="text-sm font-bold">Durasi: {$sessionCountdown} detik</p>
        </div>
        <div id="soal" class="flex gap-2 items-center">
            <h2 class="text-xl font-bold">Soal</h2>
            <ul class="flex gap-3 font-bold text-center">
                {#each $rand5 as soal, id}
                    <li class="border-2 px-6 py-2">
                        <p class="text-4xl">{soal.toUpperCase()}</p>
                        <small class="text-sm">{options[id]}</small>
                    </li>
                {/each}
            </ul>
        </div>
        <div id="pertanyaan" class="flex gap-2 items-center">
            <h2 class="text-xl font-bold">Pertanyaan</h2>
            <ul class="flex gap-3 font-bold">
                {#each $rand4 as pertanyaan}
                    <li class="border-2 px-3 py-2">{pertanyaan.toUpperCase()}</li>
                {/each}
            </ul>
        </div>
        <div id="pilihan_ganda" class="flex gap-2 items-center">
            <h2 class="text-xl font-bold">Jawaban</h2>
            <ul class="flex gap-3 font-bold">
            {#each options as option}
                <button on:click={() => chooseOption(option)}>
                    <li class="border-2 px-6 py-2" >{option}</li>
                </button>
            {/each}
            </ul>
        </div>
    </div>
{:else if $result}
    <p>Hasil Tes:</p>
{:else}
    <button on:click={startApp}>Mulai Tes Kecermatan</button>
{/if}

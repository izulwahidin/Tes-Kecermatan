<script>
	import { onMount } from 'svelte';
    import { writable, get } from 'svelte/store';

    const totalSession = 3;
    const session = writable(0);
    const result = writable(false);
    const sessionCountdown = writable(10);
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
        result.set(false)
        startCountdown(3, () => {
            session.set(1);
            runSession();
        });
    };

    const runSession = () => {
        rand5.set(randomList())
        rand4.set(last4($rand5))

        sessionCountdown.set(10);
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

<div class="grid gap-4 py-5 px-2 max-w-screen-md border-2 rounded mx-[auto] w-full">
{#if $countdown > 0 && !$result}
    <div class="text-center">
        <h1 class="text-2xl font-bold">Persiapan {$countdown} detik</h1>
    </div>
{:else if $session !== 0 && !$result}
    

    <div class="">
        <h1 class="text-5xl font-bold">Sesi {$session}</h1>
        <p class="text-sm font-bold">Durasi: {$sessionCountdown} detik</p>
    </div>

    <div id="soal" class="gap-2 items-center md:flex">
        <h2 class="text-2xl font-bold">Soal</h2>
        <ul class="flex gap-2 text-2xl font-bold text-center justify-between w-full">
            {#each $rand5 as soal, id}
                <li class="border-4 rounded w-full">
                    <p>{soal.toUpperCase()}</p>
                    <small class="text-xs font-bold">{options[id]}</small>
                </li>
            {/each}
        </ul>
    </div>
    <div id="pertanyaan" class="flex-wrap gap-2 items-center md:flex">
        <h2 class="text-2xl font-bold">Pertanyaan</h2>
        <ul class="flex gap-2 text-xl font-bold text-center justify-start">
            {#each $rand4 as pertanyaan}
            <!-- {#each 'acvd'.split('') as pertanyaan} -->
                <li class="border-2 rounded px-3">{pertanyaan.toUpperCase()}</li>
            {/each}
        </ul>
    </div>
    <div id="pilihan_ganda" class="flex-wrap gap-2 items-center">
        <h2 class="text-2xl font-bold">Jawaban</h2>
        <ul class="flex gap-3 text-xl font-bold justify-between w-full">
        {#each options as option}
            <button on:click={() => chooseOption(option)} class="w-full rounded-md border-2 px-3 py-1 hover:bg-slate-400">
                <li>{option}</li>
            </button>
        {/each}
        </ul>
    </div>
{:else if $result}
    <div class="text-center">
        <h1 class="text-2xl font-bold">Hasil Tes</h1>
        <p class="text-md font-bold">Maintenance!!!!</p>
        <button on:click={startApp} class="px-4 py-2 border-2">Mulai ulang</button>
    </div>
{:else}
    <div class="text-center">
        <h1 class="text-2xl font-bold">Selamat Datang di Tes Kecermatan <u>Reswara Pratama</u></h1>
        <p class="text-md font-bold">Peserta bisa mencoba simulasi Tes Kecermatan pada halaman ini.</p>
        <button on:click={startApp} class="mt-4 px-4 py-2 border-2">Mulai Tes Kecermatan</button>
    </div>
{/if}
</div>

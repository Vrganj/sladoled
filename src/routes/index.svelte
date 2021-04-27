<script>
    import SongSelection from '$lib/SongSelection.svelte';
    import { correctSong, selectedSong } from '$lib/stores';
    import { onMount } from 'svelte';

    let stage = 'loading';

    let songs;
    let audio;
    let choices;
    let songQueue;

    let audioProgress = 0;

    let nextSong;

    onMount(() => {
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
        }

        (async () => {
            songs = (await fetch('/songs/').then(res => res.json())).map(song => song.name);

            songQueue = [...songs];
            shuffleArray(songQueue);

            stage = 'ready';
        })();

        nextSong = () => {
            stage = 'playing';

            if (songQueue.length === 0) {
                stage = 'finished';
                return;
            }

            if ($correctSong) {
                audio.pause();
                audio.currentTime = 0;
            }

            $selectedSong = null;
            $correctSong = null;

            $correctSong = songQueue.pop();
            shuffleArray(songQueue);

            const tmp = [...songQueue.slice(0, 14), $correctSong];
            shuffleArray(tmp);
            choices = tmp;

            audio = new Audio(`/songs/${$correctSong}`);

            audio.onloadedmetadata = () => {
                audio.currentTime = audio.duration * Math.random() * 3/5;
                audio.play();
            }

            audio.ontimeupdate = () => {
                audioProgress = audio.currentTime / audio.duration * 100;
            };
        };
    });
</script>

{#if stage === 'loading'}
    ...
{:else if stage === 'ready'}
    <div class="d-flex justify-content-center mt-3">
        <button type="button" class="btn btn-primary col-8 col-md-6 col-lg-4" on:click={nextSong}>Kreni</button>
    </div>
{:else if stage === 'playing'}
    <div class="container my-2">
        <div class="rounded my-2 p-3" style="background: #f4f4f4">
            <div class="progress" style="height: 3px">
                <div class="progress-bar" role="progressbar" style="width: {audioProgress}%;"></div>
            </div>
        </div>

        {#each choices as song}
            <SongSelection name={song} />
        {/each}

        {#if $selectedSong}
            <div class="d-flex justify-content-center">
                <button type="button" class="btn btn-primary col-6" on:click={nextSong}>Dalje</button>
            </div>
        {/if}
    </div>
{:else if stage === 'finished'}
    bravo
{/if}

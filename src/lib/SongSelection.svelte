<script>
    import { selectedSong, correctSong } from '$lib/stores';

    export let name;

    function select() {
        if ($selectedSong) return;
        $selectedSong = name;
    }
</script>

<div
    class="rounded my-2 p-2"
    style="background: #f4f4f4"
    role="button"
    on:click={select}
    class:bg-primary={$selectedSong !== null && $correctSong === name}
    class:bg-danger={$selectedSong === name && $correctSong !== name}
    class:text-white={$selectedSong === name || ($selectedSong !== null && $correctSong === name)}
>
    {name.slice(0, -4)}
</div>

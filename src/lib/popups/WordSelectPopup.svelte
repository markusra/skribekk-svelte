<script lang="ts">
  import { getContext } from "svelte";
  const { close } = getContext("simple-modal");

  export let words = [];
  export let onWordSelect = (word: string) => null;

  function setSelectedWord(word: string) {
    onWordSelect(word);
    close();
  }
</script>

<div class="modal">
  <h1>Select a word</h1>
  <div class="word-list">
    {#each words as word}
      <button on:click={() => setSelectedWord(word)}>{word}</button>
    {/each}
  </div>
</div>

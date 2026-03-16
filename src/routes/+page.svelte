<script lang="ts">
  import { onMount } from 'svelte';
  import TimerPage from '$lib/TimerPage.svelte';
  import InfoPage from '$lib/InfoPage.svelte';

  type Page = 'timer' | 'info';

  let currentPage = $state<Page | null>(null);

  onMount(() => {
    const visited = localStorage.getItem('visited');
    currentPage = visited ? 'timer' : 'info';
    if (!visited) localStorage.setItem('visited', 'true');
  });
</script>

{#if currentPage === 'timer'}
  <TimerPage onGoToInfo={() => (currentPage = 'info')} />
{:else if currentPage === 'info'}
  <InfoPage onGoToTimer={() => (currentPage = 'timer')} />
{/if}

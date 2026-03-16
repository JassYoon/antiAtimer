<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  import TimerDonut from './TimerDonut.svelte';
  import RestModal from './RestModal.svelte';

  interface Props {
    onGoToInfo: () => void;
  }

  let { onGoToInfo }: Props = $props();

  // 설정값 (localStorage에서 불러옴)
  let sets = $state(4);
  let restAfter = $state(5);
  let sessionCount = $state(0);

  onMount(() => {
    sets = parseInt(localStorage.getItem('sets') ?? '4');
    restAfter = parseInt(localStorage.getItem('restAfter') ?? '5');
    sessionCount = parseInt(localStorage.getItem('sessionCount') ?? '0');
  });

  // 호흡 모드
  let mode = $state<'444' | '478'>('444');
  const phases = $derived<[number, number, number]>(
    mode === '444' ? [4, 4, 4] : [4, 7, 8]
  );
  const totalPerCycle = $derived(phases[0] + phases[1] + phases[2]);
  const totalDuration = $derived(sets * totalPerCycle);

  // 타이머 상태
  let running = $state(false);
  let elapsed = $state(0); // 현재 세션에서 경과한 초 (소수점)
  let showRest = $state(false);

  const cycleElapsed = $derived(elapsed % totalPerCycle);

  // requestAnimationFrame 기반 타이머
  let animFrameId = 0;
  let lastTimestamp: number | null = null;

  function tick(timestamp: number) {
    if (lastTimestamp !== null) {
      const delta = (timestamp - lastTimestamp) / 1000;
      elapsed += delta;
      if (elapsed >= totalDuration) {
        elapsed = 0;
        running = false;
        lastTimestamp = null;
        return;
      }
    }
    lastTimestamp = timestamp;
    animFrameId = requestAnimationFrame(tick);
  }

  function handleStart() {
    if (running) {
      // 타이머 중지
      running = false;
      cancelAnimationFrame(animFrameId);
      elapsed = 0;
      lastTimestamp = null;
      return;
    }

    // 휴식 모드 확인
    if (sessionCount >= restAfter) {
      showRest = true;
      return;
    }

    // 타이머 시작
    sessionCount++;
    localStorage.setItem('sessionCount', String(sessionCount));
    running = true;
    lastTimestamp = null;
    animFrameId = requestAnimationFrame(tick);
  }

  function handleDismissRest() {
    showRest = false;
    sessionCount = 0;
    localStorage.setItem('sessionCount', '0');
  }

  function toggleMode() {
    if (running) return;
    mode = mode === '444' ? '478' : '444';
  }

  onDestroy(() => {
    cancelAnimationFrame(animFrameId);
  });
</script>

<div class="relative flex h-screen w-full flex-col items-center justify-center bg-slate-950 select-none overflow-hidden">

  <!-- 상단 버튼 영역 -->
  <div class="absolute top-6 left-0 right-0 flex items-center justify-between px-6">
    <!-- 모드 전환 버튼 -->
    <button
      onclick={toggleMode}
      disabled={running}
      class="flex h-11 w-11 items-center justify-center rounded-full border border-slate-700 bg-slate-800/60 text-slate-300 transition hover:border-slate-500 hover:text-white disabled:opacity-30 disabled:cursor-not-allowed active:scale-95"
      aria-label="호흡 모드 전환"
      title="모드 전환 ({mode === '444' ? '4-4-4' : '4-7-8'})"
    >
      <svg class="h-5 w-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
        <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
      </svg>
    </button>

    <!-- 현재 모드 표시 -->
    <span class="text-xs font-medium text-slate-500 tracking-widest">
      {mode === '444' ? '4·4·4' : '4·7·8'}
    </span>

    <!-- 정보 버튼 -->
    <button
      onclick={onGoToInfo}
      class="flex h-11 w-11 items-center justify-center rounded-full border border-slate-700 bg-slate-800/60 text-slate-300 transition hover:border-slate-500 hover:text-white active:scale-95"
      aria-label="사용 설명 보기"
    >
      <svg class="h-5 w-5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
        <circle cx="12" cy="12" r="10" />
        <path stroke-linecap="round" stroke-linejoin="round" d="M12 16v-4m0-4h.01" />
      </svg>
    </button>
  </div>

  <!-- 도넛 타이머 -->
  <div class="w-64 h-64 sm:w-72 sm:h-72">
    <TimerDonut {phases} {cycleElapsed} {running} />
  </div>

  <!-- 세션 진행 표시 (실행 중일 때) -->
  {#if running}
    <div class="mt-4 flex gap-1.5">
      {#each Array(sets) as _, i}
        {@const cyclesCompleted = Math.floor(elapsed / totalPerCycle)}
        <div
          class="h-1.5 w-6 rounded-full transition-colors duration-300 {i < cyclesCompleted ? 'bg-slate-400' : i === cyclesCompleted ? 'bg-white' : 'bg-slate-700'}"
        ></div>
      {/each}
    </div>
  {:else}
    <div class="mt-4 h-4"></div>
  {/if}

  <!-- 시작/중지 버튼 -->
  <button
    onclick={handleStart}
    class="mt-8 rounded-2xl px-12 py-4 text-base font-semibold tracking-wide transition active:scale-95
      {running
        ? 'border border-slate-600 bg-slate-800 text-slate-300 hover:border-slate-500 hover:text-white'
        : 'bg-white text-slate-900 shadow-lg hover:bg-slate-100'}"
  >
    {running ? '중지하기' : '시작하기'}
  </button>

  <!-- 휴식 모드 오버레이 -->
  {#if showRest}
    <RestModal {restAfter} onDismiss={handleDismissRest} />
  {/if}
</div>

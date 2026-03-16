<script lang="ts">
  import { onMount } from 'svelte';
  import TimerDonut from './TimerDonut.svelte';

  interface Props {
    onGoToTimer: () => void;
  }

  let { onGoToTimer }: Props = $props();

  // 설정값
  let sets = $state(4);
  let restAfter = $state(5);

  onMount(() => {
    sets = parseInt(localStorage.getItem('sets') ?? '4');
    restAfter = parseInt(localStorage.getItem('restAfter') ?? '5');
  });

  function saveSets(val: number) {
    const v = Math.max(1, Math.min(20, val));
    sets = v;
    localStorage.setItem('sets', String(v));
  }

  function saveRestAfter(val: number) {
    const v = Math.max(1, Math.min(50, val));
    restAfter = v;
    localStorage.setItem('restAfter', String(v));
  }

  // 미리보기용 (정지 상태)
  const previewPhases: [number, number, number] = [4, 4, 4];

  const descriptions = [
    { color: '#60a5fa', label: '파란색', text: '들이쉬기 단계예요. 코로 천천히 숨을 들이마세요.' },
    { color: '#a78bfa', label: '보라색', text: '참기 단계예요. 숨을 그대로 멈춰 유지하세요.' },
    { color: '#34d399', label: '초록색', text: '내쉬기 단계예요. 입으로 천천히 내쉬세요.' },
  ];
</script>

<div class="flex min-h-screen flex-col items-center justify-center bg-slate-950 px-6 py-10 select-none">

  <!-- 상단 콘텐츠: 타이머 이미지 + 설명 -->
  <div class="flex w-full max-w-2xl flex-col items-center gap-8 sm:flex-row sm:items-center">

    <!-- 타이머 이미지 (좌측, 기능 없음) -->
    <div class="w-48 h-48 shrink-0 pointer-events-none">
      <TimerDonut phases={previewPhases} cycleElapsed={0} running={false} />
    </div>

    <!-- 사용 설명 (우측) -->
    <div class="flex flex-col gap-3 text-sm">
      {#each descriptions as desc}
        <div class="flex items-start gap-3">
          <span class="mt-0.5 h-3 w-3 shrink-0 rounded-full" style="background-color: {desc.color}"></span>
          <p class="leading-relaxed text-slate-300">
            <span class="font-medium" style="color: {desc.color}">{desc.label}</span>은
            {desc.text}
          </p>
        </div>
      {/each}

      <div class="mt-1 flex items-start gap-3">
        <span class="mt-0.5 h-3 w-3 shrink-0 rounded-full bg-white opacity-60"></span>
        <p class="leading-relaxed text-slate-300">
          도넛이 <span class="font-medium text-white">한 바퀴</span> 돌면 한 세트가 완료돼요.
        </p>
      </div>
    </div>
  </div>

  <!-- 구분선 -->
  <div class="my-8 w-full max-w-2xl border-t border-slate-800"></div>

  <!-- 설정 칸 -->
  <div class="flex w-full max-w-2xl gap-4">

    <!-- 설정1: 세트 반복 수 -->
    <div class="flex flex-1 items-center gap-2 rounded-xl border border-slate-700 bg-slate-900 px-4 py-3">
      <input
        type="number"
        min="1"
        max="20"
        value={sets}
        onchange={(e) => saveSets(parseInt((e.target as HTMLInputElement).value) || 4)}
        class="w-14 rounded-lg border border-slate-600 bg-slate-800 px-2 py-1.5 text-center text-base font-semibold text-white focus:border-slate-400 focus:outline-none"
      />
      <span class="text-sm text-slate-400">세트 반복</span>
    </div>

    <!-- 설정2: 휴식까지 반복 수 -->
    <div class="flex flex-1 items-center gap-2 rounded-xl border border-slate-700 bg-slate-900 px-4 py-3">
      <span class="text-sm text-slate-400">휴식까지</span>
      <input
        type="number"
        min="1"
        max="50"
        value={restAfter}
        onchange={(e) => saveRestAfter(parseInt((e.target as HTMLInputElement).value) || 5)}
        class="w-14 rounded-lg border border-slate-600 bg-slate-800 px-2 py-1.5 text-center text-base font-semibold text-white focus:border-slate-400 focus:outline-none"
      />
      <span class="text-sm text-slate-400">회 반복</span>
    </div>
  </div>

  <!-- 시작하기 버튼 -->
  <button
    onclick={onGoToTimer}
    class="mt-6 w-full max-w-2xl rounded-2xl bg-white px-12 py-4 text-base font-semibold tracking-wide text-slate-900 shadow-lg transition hover:bg-slate-100 active:scale-95"
  >
    시작하기
  </button>
</div>

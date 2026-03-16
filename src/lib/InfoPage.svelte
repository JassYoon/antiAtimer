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

      <div class="mt-1 flex items-start gap-3">
        <span class="mt-0.5 shrink-0 text-slate-400">
          <svg class="h-3.5 w-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
          </svg>
        </span>
        <p class="leading-relaxed text-slate-300">
          상단의 <span class="font-medium text-white">전환 버튼</span>으로 <span class="text-blue-400">4·4·4</span>와 <span class="text-violet-400">4·7·8</span> 모드를 바꿀 수 있어요. 4·7·8은 불안 완화에 효과적인 호흡법이에요.
        </p>
      </div>

      <div class="mt-1 flex items-start gap-3">
        <span class="mt-0.5 shrink-0 text-emerald-400">
          <svg class="h-3.5 w-3.5" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 3v1m0 16v1M4.22 4.22l.7.7m12.16 12.16.7.7M3 12H4m16 0h1M4.22 19.78l.7-.7M18.36 5.64l.7-.7" />
            <circle cx="12" cy="12" r="4" />
          </svg>
        </span>
        <p class="leading-relaxed text-slate-300">
          아래에서 설정한 횟수만큼 타이머를 사용하면 <span class="font-medium text-emerald-400">휴식 모드</span>가 시작돼요. 잠시 쉰 뒤 계속할 수 있어요.
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

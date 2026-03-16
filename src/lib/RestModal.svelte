<script lang="ts">
  interface Props {
    restAfter: number;
    onDismiss: () => void;
  }

  let { restAfter, onDismiss }: Props = $props();
</script>

<!-- 반투명 배경 오버레이 -->
<div class="absolute inset-0 z-10 flex items-center justify-center" aria-modal="true" role="dialog">
  <div class="absolute inset-0 bg-slate-950/70 backdrop-blur-sm"></div>

  <div class="relative z-10 mx-6 rounded-2xl border border-slate-700 bg-slate-900/95 px-8 py-8 text-center shadow-2xl max-w-sm w-full">
    <!-- 아이콘 -->
    <div class="mx-auto mb-4 flex h-14 w-14 items-center justify-center rounded-full bg-emerald-500/15">
      <svg class="h-7 w-7 text-emerald-400" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8">
        <path stroke-linecap="round" stroke-linejoin="round" d="M12 3v1m0 16v1M4.22 4.22l.7.7m12.16 12.16.7.7M3 12H4m16 0h1M4.22 19.78l.7-.7M18.36 5.64l.7-.7" />
        <circle cx="12" cy="12" r="4" />
      </svg>
    </div>

    <h2 class="mb-2 text-lg font-semibold text-white">잠시 휴식을 권해드려요</h2>
    <p class="mb-1 text-sm text-slate-400">
      타이머를 <span class="text-emerald-400 font-medium">{restAfter}번</span> 사용하셨어요.
    </p>
    <p class="mb-6 text-sm text-slate-400">
      5~10분 정도 몸을 이완시키고 돌아오세요.
    </p>

    <button
      onclick={onDismiss}
      class="w-full rounded-xl bg-emerald-600 px-6 py-3 text-sm font-semibold text-white transition hover:bg-emerald-500 active:scale-95"
    >
      충분히 쉬었어요
    </button>
  </div>
</div>

<script lang="ts">
  interface Props {
    phases: [number, number, number];
    cycleElapsed: number;
    running: boolean;
  }

  let { phases, cycleElapsed, running }: Props = $props();

  const COLORS = ['#60a5fa', '#a78bfa', '#34d399'] as const;
  const LABELS = ['들이쉬기', '참기', '내쉬기'] as const;

  const R = 100;
  const CX = 130;
  const CY = 130;
  const CIRC = 2 * Math.PI * R;

  const total = $derived(phases[0] + phases[1] + phases[2]);

  const segments = $derived(
    phases.map((dur, i) => {
      const prevDur = phases.slice(0, i).reduce((a, b) => a + b, 0);
      return {
        color: COLORS[i],
        label: LABELS[i],
        arcLen: (dur / total) * CIRC,
        startAngle: -90 + (prevDur / total) * 360,
        duration: dur,
      };
    })
  );

  const currentPhase = $derived.by(() => {
    if (!running) return { index: 0, phaseElapsed: 0 };
    let acc = 0;
    for (let i = 0; i < 3; i++) {
      if (cycleElapsed < acc + phases[i]) {
        return { index: i, phaseElapsed: cycleElapsed - acc };
      }
      acc += phases[i];
    }
    return { index: 2, phaseElapsed: phases[2] };
  });

  const completedLens = $derived(
    segments.map((seg, i) => {
      if (!running) return 0;
      const { index, phaseElapsed } = currentPhase;
      if (i < index) return seg.arcLen;
      if (i === index) return (phaseElapsed / seg.duration) * seg.arcLen;
      return 0;
    })
  );

  const timeLeft = $derived(
    running ? Math.ceil(phases[currentPhase.index] - currentPhase.phaseElapsed) : null
  );
</script>

<svg viewBox="0 0 260 260" class="w-full h-full" aria-label="호흡 타이머">
  <!-- 배경 링 -->
  <circle cx={CX} cy={CY} r={R} fill="none" stroke="#1e293b" stroke-width="36" />

  <!-- 단계별 가이드 호 (흐릿하게) -->
  {#each segments as seg, i}
    <circle
      cx={CX}
      cy={CY}
      r={R}
      fill="none"
      stroke={seg.color}
      stroke-width="36"
      stroke-linecap="butt"
      stroke-dasharray="{seg.arcLen} {CIRC}"
      transform="rotate({seg.startAngle} {CX} {CY})"
      opacity="0.22"
    />
  {/each}

  <!-- 진행된 호 (밝게) -->
  {#if running}
    {#each segments as seg, i}
      {#if completedLens[i] > 0.5}
        <circle
          cx={CX}
          cy={CY}
          r={R}
          fill="none"
          stroke={seg.color}
          stroke-width="36"
          stroke-linecap="butt"
          stroke-dasharray="{completedLens[i]} {CIRC}"
          transform="rotate({seg.startAngle} {CX} {CY})"
        />
      {/if}
    {/each}
  {/if}

  <!-- 중앙 텍스트 -->
  {#if running && timeLeft !== null}
    <text
      x={CX}
      y={CY - 14}
      text-anchor="middle"
      fill="white"
      font-size="17"
      font-family="system-ui, sans-serif"
    >{segments[currentPhase.index].label}</text>
    <text
      x={CX}
      y={CY + 26}
      text-anchor="middle"
      fill="white"
      font-size="46"
      font-weight="bold"
      font-family="system-ui, sans-serif"
    >{timeLeft}</text>
  {:else}
    <text
      x={CX}
      y={CY + 8}
      text-anchor="middle"
      fill="#475569"
      font-size="15"
      font-family="system-ui, sans-serif"
    >{phases[0]}-{phases[1]}-{phases[2]}</text>
  {/if}
</svg>

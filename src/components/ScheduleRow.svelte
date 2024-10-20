<script lang="ts">
  import { onMount } from "svelte";

  export let framework: string;
  export let latest: {
    version: string;
    status: string;
    date: string;
  };
  export let upcoming: null | {
    version: string;
    status: string;
    date?: string;
  };

  const ALPHABET = " -.ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
  const FLIP_DELAY_MS = 150;
  const CLACK_SOUND_VOLUME = 0.25;
  const CLACK_SOUND_FREQUENCY = 200;
  const CLACK_SOUND_DURATION_MS = 50;
  const FRAMEWORK_MAX_LENGTH = 15;
  const VERSION_MAX_LENGTH = 4;
  const STATUS_MAX_LENGTH = 8;
  const DATE_MAX_LENGTH = 15;
  const PLACEHOLDER = " ";

  let displayFramework = PLACEHOLDER.repeat(FRAMEWORK_MAX_LENGTH);
  let displayLatest = {
    version: PLACEHOLDER.repeat(VERSION_MAX_LENGTH),
    status: PLACEHOLDER.repeat(STATUS_MAX_LENGTH),
    date: PLACEHOLDER.repeat(DATE_MAX_LENGTH),
  };
  let displayUpcoming = {
    version: PLACEHOLDER.repeat(VERSION_MAX_LENGTH),
    status: PLACEHOLDER.repeat(STATUS_MAX_LENGTH),
    date: PLACEHOLDER.repeat(DATE_MAX_LENGTH),
  };

  class ClackSound {
    #audioCtx: AudioContext;
    #gainNode: GainNode;
    #oscillator: OscillatorNode;
    #envelope: GainNode;

    constructor() {
      this.#audioCtx = new AudioContext();

      this.#gainNode = this.#audioCtx.createGain();
      this.#gainNode.gain.value = CLACK_SOUND_VOLUME;

      this.#oscillator = this.#audioCtx.createOscillator();
      this.#oscillator.type = "triangle";
      this.#oscillator.frequency.value = CLACK_SOUND_FREQUENCY;
      this.#oscillator.connect(this.#gainNode);

      this.#envelope = this.#audioCtx.createGain();
      this.#oscillator.connect(this.#envelope);
      this.#envelope.connect(this.#gainNode);

      this.#gainNode.connect(this.#audioCtx.destination);
    }
    start(durationMs: number): void {
      this.#envelope.gain.value = 1;
      this.#oscillator.start();

      this.#envelope.gain.exponentialRampToValueAtTime(
        0.01,
        this.#audioCtx.currentTime + durationMs / 1000,
      );
    }
    stop(): void {
      this.#oscillator.stop();
    }
    static play(durationMs: number): void {
      const clackSound = new ClackSound();
      clackSound.start(durationMs);
      setTimeout(() => {
        clackSound.stop();
      }, durationMs);
    }
  }

  function animateText(
    finalText: string,
    setDisplayText: (s: string) => void,
  ): void {
    const normalizedFinalText = finalText.toUpperCase();
    let currentText = normalizedFinalText.split("").map(() => PLACEHOLDER);
    const interval = setInterval(() => {
      if (currentText.join("") === normalizedFinalText) {
        clearInterval(interval);
        return;
      }

      for (const [charIndex, currentChar] of currentText.entries()) {
        let currentAlphabetIndex = ALPHABET.indexOf(currentChar);

        if (currentChar !== normalizedFinalText[charIndex]) {
          currentText[charIndex] =
            ALPHABET[(currentAlphabetIndex + 1) % ALPHABET.length];
        }
      }

      ClackSound.play(CLACK_SOUND_DURATION_MS);
      setDisplayText(currentText.join(""));
    }, FLIP_DELAY_MS);
  }

  onMount(() => {
    animateText(
      framework.padEnd(FRAMEWORK_MAX_LENGTH, PLACEHOLDER),
      (val) => (displayFramework = val),
    );
    animateText(
      latest.version.padEnd(VERSION_MAX_LENGTH, PLACEHOLDER),
      (val) => (displayLatest.version = val),
    );
    animateText(
      latest.status.padEnd(STATUS_MAX_LENGTH, PLACEHOLDER),
      (val) => (displayLatest.status = val),
    );
    animateText(
      (latest.date ?? "").padEnd(DATE_MAX_LENGTH, PLACEHOLDER),
      (val) => (displayLatest.date = val),
    );
    if (upcoming != null) {
      animateText(
        PLACEHOLDER.repeat(FRAMEWORK_MAX_LENGTH) +
          upcoming.version.padEnd(VERSION_MAX_LENGTH, PLACEHOLDER),
        (val) => (displayUpcoming.version = val),
      );
      animateText(
        upcoming.status.padEnd(STATUS_MAX_LENGTH, PLACEHOLDER),
        (val) => (displayUpcoming.status = val),
      );
      animateText(
        (upcoming.date ?? "").padEnd(DATE_MAX_LENGTH, PLACEHOLDER),
        (val) => (displayUpcoming.date = val),
      );
    }
  });
</script>

<div class="schedule-row-container">
  <div class="schedule-row">
    {#each displayFramework.split("") as char}
      <span class="flip-char">{char}</span>
    {/each}
    {#each displayLatest.version.split("") as char}
      <span class="flip-char">{char}</span>
    {/each}
    {#each displayLatest.status.split("") as char}
      <span class="flip-char">{char}</span>
    {/each}
    {#each displayLatest.date.split("") as char}
      <span class="flip-char">{char}</span>
    {/each}
  </div>
  {#if upcoming != null}
    <div class="schedule-row">
      {#each displayUpcoming.version.split("") as char}
        <span class="flip-char">{char}</span>
      {/each}
      {#each displayUpcoming.status.split("") as char}
        <span class="flip-char">{char}</span>
      {/each}
      {#each displayUpcoming.date.split("") as char}
        <span class="flip-char">{char}</span>
      {/each}
    </div>
  {/if}
</div>

<style>
  .schedule-row-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
    padding: 0 0.25rem;
  }

  .schedule-row {
    display: flex;
    justify-content: center;
    width: auto;
    background-color: #333;
    white-space: nowrap;
  }

  .schedule-row + .schedule-row {
    margin-top: 1px;
  }

  .flip-char {
    display: inline-block;
    position: relative;
    width: 1ch;
    margin: 0 0.025rem;
    padding: 0 0.025rem;
    border: 0.025rem solid #444;
    border-radius: 0.05rem;
    box-shadow: 0 0 0.05rem rgba(255, 215, 0, 0.5);
    text-align: center;
    color: #ffd700;
    font-family: "Courier New", monospace;
    font-weight: bold;
    font-size: 1.5rem;
    text-transform: uppercase;
    background-color: #222;
  }

  @media (min-width: 480px) {
    .flip-char {
      font-size: 0.8rem;
      margin: 0 0.05rem;
      padding: 0 0.05rem;
      border-width: 0.05rem;
    }
  }

  @media (min-width: 768px) {
    .schedule-row {
      padding: 0.5rem 0;
    }

    .flip-char {
      font-size: 1rem;
      margin: 0 0.075rem;
      padding: 0 0.075rem;
      border-width: 0.075rem;
    }
  }

  @media (min-width: 1024px) {
    .flip-char {
      font-size: 1.2rem;
      margin: 0 0.1rem;
      padding: 0 0.1rem;
      border-width: 0.1rem;
    }
  }
</style>

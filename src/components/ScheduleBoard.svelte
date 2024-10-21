<script lang="ts">
  import { onMount } from "svelte";
  import ScheduleRow from "./ScheduleRow.svelte";
  import releaseData from "../data/releases.json";

  let scheduleItems: typeof releaseData = $state([]);

  onMount(async () => {
    scheduleItems = releaseData.toSorted((a, b) =>
      a.framework.localeCompare(b.framework),
    );
  });
</script>

<div class="schedule-board">
  {#each scheduleItems as item}
    <ScheduleRow {...item} />
  {/each}
</div>

<style>
  .schedule-board {
    display: inline-block;
    background-color: #222;
    border-radius: 10px;
    padding: 1rem;
    font-family: "Courier New", monospace;
    overflow-x: auto;
  }

  @media (min-width: 768px) {
    .schedule-board {
      max-width: 90vw;
      padding: 1.5rem;
    }
  }

  @media (min-width: 1024px) {
    .schedule-board {
      max-width: 80vw;
      padding: 2rem;
    }
  }
</style>

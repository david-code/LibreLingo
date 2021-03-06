<script>
  import sound from "../media/sound"
  import DeckChallenge from "./DeckChallenge"
  import OptionChallenge from "./OptionChallenge"
  import ShortInputChallenge from "./ShortInputChallenge"
  import ListeningChallenge from "./ListeningChallenge"
  import ChipsChallenge from "./ChipsChallenge"
  import FanfareScreen from "./FanfareScreen"
  import ProgressBar from "./ProgressBar"
  import shuffle from "lodash.shuffle"
  import { fade, scale } from "svelte/transition"
  import Button from "lluis/Button"

  export let rawChallenges
  export let languageName
  export let languageCode
  export let specialCharacters
  export let sortChallengeGroups
  export let courseURL
  export let skillId
  let challenges = sortChallengeGroups(shuffle(rawChallenges))
  let remainingChallenges = [...challenges]
  let currentChallenge = remainingChallenges.shift()
  let solvedChallenges = []
  let progress = 0
  let stats = {
    correct: 0,
    incorrect: 0,
    skipped: 0,
  }

  const preloadImage = (imageName) => {
    if (typeof Image === "undefined") return
    new Image().src = `images/${imageName}`
  }

  challenges
    .filter(({ type }) => type === "card")
    .map(({ pictures }) => pictures.map(preloadImage))

  $: alternativeChallenges =
    currentChallenge &&
    challenges.filter(({ id }) => id !== currentChallenge.id)

  $: registerResult = (isCorrect) => {
    if (isCorrect) {
      stats.correct++
      sound.correct.play()
      solvedChallenges.push(currentChallenge)
    } else {
      stats.incorrect++
      sound.wrong.play()
      remainingChallenges.push(currentChallenge)
    }
  }

  $: progress = (solvedChallenges.length + stats.skipped) / challenges.length

  $: resolveChallenge = () => {
    if (remainingChallenges) {
      currentChallenge = remainingChallenges.shift()
    }
  }

  $: skipChallenge = () => {
    stats.skipped++
    resolveChallenge()
  }
</script>

{#if currentChallenge}

  <div class="container" in:scale>
    <section class="section">
      <ProgressBar value="{progress}" />
      {#each challenges as challenge, i (challenge.id)}
        {#if challenge.id === currentChallenge.id}
          <div
            class="challenge"
            in:fade|local="{{ duration: 300, delay: 350 }}"
            out:fade|local="{{ duration: 300 }}">
            {#if challenge.type === 'cards'}
              <DeckChallenge
                {skipChallenge}
                {currentChallenge}
                {alternativeChallenges}
                {resolveChallenge}
                {registerResult} />
            {/if}
            {#if challenge.type === 'options'}
              <OptionChallenge
                {skipChallenge}
                {currentChallenge}
                {alternativeChallenges}
                {resolveChallenge}
                {registerResult} />
            {/if}
            {#if challenge.type === 'shortInput'}
              <ShortInputChallenge
                {skipChallenge}
                {languageName}
                {languageCode}
                {specialCharacters}
                {registerResult}
                {resolveChallenge}
                {challenge} />
            {/if}
            {#if challenge.type === 'listeningExercise'}
              <ListeningChallenge
                {skipChallenge}
                {languageCode}
                {specialCharacters}
                {registerResult}
                {resolveChallenge}
                {challenge} />
            {/if}
            {#if challenge.type === 'chips'}
              <ChipsChallenge
                {registerResult}
                {resolveChallenge}
                {challenge}
                {skipChallenge} />
            {/if}
          </div>
        {/if}
      {/each}
    </section>
  </div>
{/if}

{#if !currentChallenge}
  <div class="container">
    <FanfareScreen {courseURL} {rawChallenges} {skillId} {stats} />
  </div>
{/if}

<style>
  .section {
    padding: 1.5em;
  }
</style>

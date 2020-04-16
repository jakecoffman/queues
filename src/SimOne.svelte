<script>
  import {quintOut} from 'svelte/easing';
  import {crossfade} from 'svelte/transition';

  const [send, receive] = crossfade({
    duration: d => Math.sqrt(d * 200),

    fallback(node, params) {
      const style = getComputedStyle(node);
      const transform = style.transform === 'none' ? '' : style.transform;

      return {
        duration: 600,
        easing: quintOut,
        css: t => `transform: ${transform} scale(${t}); opacity: ${t}`
      };
    }
  });

  let work = [];

  function mark(item, done) {
    item.done = done;
    work = work.filter(t => t !== item);
    work = work.concat(item);
  }

  function start() {
    setTimeout(tick, 1000)
  }

  const lines = 2

  function reset() {
    let uid = 1;
    work = [
      {id: uid++, done: false, line: 0, ticks: 2},
      {id: uid++, done: false, line: 0, ticks: 2},
      {id: uid++, done: false, line: 0, ticks: 2},
      {id: uid++, done: false, line: 1, ticks: 2},
      {id: uid++, done: false, line: 1, ticks: 2},
      {id: uid++, done: false, line: 1, ticks: 2},
    ]
  }

  reset();

  function tick() {
    for (let i=0; i<lines; i++) {
      const index = work.findIndex(item => item.line === i && !item.done)
      if (index && work[index]) {
        const item = work[index]
        item.ticks--
        if (item.ticks === 0) {
          mark(work[index], true)
        } else {
          work[index] = item
        }
      }
    }

    if (work.filter(i => !i.done).length > 0) {
      setTimeout(tick, 1000);
    }
  }
</script>

<button on:click={start}>Start</button>
<button on:click={reset}>Reset</button>

<div class=line>
  <div>
    <h2>line 1</h2>
    <p class="small"></p>
      {#each work.filter(t => t.line === 0 && !t.done) as item (item.id)}
        <label class="flex" in:receive="{{key: item.id}}" out:send="{{key: item.id}}" on:click="{() => mark(item, true)}">
          <span class="flex-1">Customer {item.id}</span>
          <span>{item.ticks} left</span>
        </label>
      {/each}
  </div>

  <div>
    <h2>line 2</h2>
      {#each work.filter(t => t.line === 1 && !t.done) as item (item.id)}
        <label class="flex" in:receive="{{key: item.id}}" out:send="{{key: item.id}}" on:click="{() => mark(item, true)}">
          <span class="flex-1">Customer {item.id}</span>
          <span>{item.ticks} left</span>
        </label>
      {/each}
  </div>
</div>

<div class="dones">
<h2>done</h2>
{#each work.filter(t => t.done) as item (item.id)}
  <label class="done" in:receive="{{key: item.id}}" out:send="{{key: item.id}}" on:click="{() => mark(item, false)}">
    <span class="flex-1">Customer {item.id}</span>
  </label>
{/each}
</div>

<style>
  .line {
    display: grid;
    grid-template-columns: 1fr 1fr;
    grid-gap: 1em;
    max-width: 36em;
    margin: 0 auto;
  }

  .dones {
    max-width: 18em;
    margin: 0 auto;
  }

  h2 {
    font-size: 2em;
    font-weight: 200;
    user-select: none;
    margin: 0 0 0.5em 0;
  }

  label {
    position: relative;
    line-height: 1.2;
    padding: 0.5em;
    margin: 0 0 0.5em 0;
    border-radius: 2px;
    user-select: none;
    border: 1px solid hsl(240, 8%, 70%);
    background-color: hsl(240, 8%, 93%);
    color: #333;
  }

  input[type="checkbox"] {
    position: absolute;
    left: 0.5em;
    top: 0.6em;
    margin: 0;
  }

  .done {
    border: 1px solid hsl(240, 8%, 90%);
    background-color: hsl(240, 8%, 98%);
  }

  .flex {
    display: flex;
  }

  .flex-1 {
    flex-grow: 1;
  }
</style>

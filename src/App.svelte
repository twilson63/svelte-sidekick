<script>
  import service from "./app";
  import { marked } from "marked";
  import { Wave } from "svelte-loading-spinners";

  let openSubmitModal = false;

  const send = $service.send;
  $: current = $service.machine.current;
  $: context = $service.context;

  $: {
    if (current === "submitting") {
      openSubmitModal = true;
    } else {
      openSubmitModal = false;
      setTimeout(() => window.hljs.highlightAll(), 100);
    }
  }

  async function handleSubmit(e) {
    send({ type: "query", question: e.target.question.value });
    e.target.reset();
  }
</script>

{#if current !== "error"}
  <div class="hero min-h-screen">
    <div class="hero-content">
      <div class="flex flex-col items-center w-full md:min-w-[700px]">
        <img class="h-16 w-16 text-center" src="svelte.svg" alt="svelte" />
        <h1 class="text-3xl font-mono mb-16 text-center">Svelte Sidekick</h1>
        <form
          on:submit|preventDefault={handleSubmit}
          class="flex flex-col space-y-2 w-full"
        >
          <input
            name="question"
            type="text"
            class="input input-bordered w-full"
            placeholder="👋🏻 Dev, Ask me a question about the Svelte?"
            required
          />
          <button class="btn btn-outline">Submit</button>
          {#if context.hx.length > 0}
            <button
              class="btn btn-secondary"
              type="button"
              on:click={() => send("clear")}>Clear</button
            >
          {/if}
        </form>
        <div class="mt-2 text-sm font-light font-mono text-center">
          Powered by <a class="link" href="https://otherwill.com">Otherwill</a>
          - Instant Answers - Just Add Docs -
          <a class="link" href="https://svelte.dev">Svelte Docs</a>
        </div>
        <div class="mx-auto">
          {#each context.hx as response}
            <div class="card mt-16">
              <div class="card-title">{response.question}</div>
              <div class="my-4 prose">
                {@html marked(response.completion)}
              </div>
            </div>
          {/each}
        </div>
      </div>
    </div>
  </div>
{:else if current === "error"}
  <div class="hero">
    <div class="hero-content">
      <div class="bg-error text-white p-16 flex flex-col space-y-4">
        <div>An Error Occurred processing question.</div>

        <button class="btn btn-outline" on:click={() => send("tryAgain")}
          >Try Again</button
        >
      </div>
    </div>
  </div>
{/if}
<input
  type="checkbox"
  id="loading"
  class="modal-toggle"
  bind:checked={openSubmitModal}
/>
<div class="modal">
  <div class="modal-box">
    <div class="grid items-center justify-center bg-[#f2f3f4] py-16">
      <div class="flex items-center justify-center"><Wave size="60" /></div>
      <div class="mt-4 font-mono">Generating a Response...</div>
    </div>
  </div>
</div>

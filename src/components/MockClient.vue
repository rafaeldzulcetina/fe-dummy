<script setup lang="ts">
import { ref } from "vue";
// @ts-ignore
import PolicyEngine from "pr-policy-evaluator";

defineProps<{ msg: string }>();

const response = ref("");
const result = ref("");
const count = ref(0);
const loading = ref(false);
const loadingEngine = ref(false);
const server = ref("http://localhost:3000");
const endpoint = ref("pipeline");
const id = ref("1");
/**
 * @function fetchPipeline
 * @param {string} id
 * @return {Promise<string>}
 * @description Method to fetch data from a remote server
 */
function fetchPipeline() {
  loading.value = true;
  response.value = "";
  fetch(`${server.value}/${endpoint.value}/${id.value}`)
    .then((res) => res.json())
    .then((data) => {
      console.log(data.data.items);
      response.value = data.data.items;
      loading.value = false;
    });
}

async function runEngine() {
  loading.value = true;
  result.value = "";
  result.value = await PolicyEngine.startPolicyEngine(response.value);
  loading.value = false;
}
</script>

<template>
  <main class="container">
    <div class="grid">
      <div>
        <article>
          <header>{{ msg }}</header>
          <form>
            <label for="id">
              Id:
              <input
                id="id"
                name="id"
                placeholder="Identificador del pipeline"
                v-model="id"
                type="text"
                required
              />
            </label>
            <button
              :aria-busy="loading"
              type="submit"
              @click.prevent="fetchPipeline()"
            >
              Send request 🚀
            </button>
          </form>
          <footer>
            <p>
              Endpoint:
              <code>[server]/{{ endpoint }}/{{ id }}</code>
            </p>
          </footer>
        </article>

        <article>
          <summary>Response</summary>
          <details>
            <pre>
              <code>{{ response }}</code>
            </pre>
          </details>
        </article>
        <article>
          <header>Ejecutar engine ⚙️</header>
          <button
            :disabled="response === ''"
            :aria-busy="loadingEngine"
            type="button"
            @click.prevent="runEngine()"
          >
            Run engine 🚀
          </button>
          <footer>
            <pre>
              <code>{{ result }}</code>
            </pre>
          </footer>
        </article>
      </div>
    </div>
  </main>
</template>

<style scoped></style>

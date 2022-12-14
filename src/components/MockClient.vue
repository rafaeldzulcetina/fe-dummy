<script setup lang="ts">
export type ResponseApi = {
  id: string;
  policies: any;
  expressions?: any;
};
import { computed, onMounted, ref, watch } from "vue";
// @ts-ignore
import PolicyEngine from "pr-policy-evaluator";

defineProps<{ msg: string }>();

const el = ref();

onMounted(() => {
  // @ts-ignore
  const theme = document.getElementById("html").dataset.theme;
  console.log(theme);
  // document.querySelector("html").data("theme", "dark");
});

const defaultResponse: ResponseApi = {
  id: "",
  policies: {},
  expressions: {},
};

const response = ref(defaultResponse);

const darkMode = ref(false);
const result = ref("");
const count = ref(0);
const loading = ref(false);
const loadingEngine = ref(false);
const apiError = ref(false);
const apiOk = ref(false);
// const server = ref("http://localhost:3000");
const server = ref("https://ix-json-server.onrender.com");
const endpoint = ref("pipeline");
const id = ref("1");

const someReactiveRef = ref(null);

const disableRun = computed(() => {
  console.log(response.value);
  console.log(typeof response.value);

  if (response.value.id) {
    return false;
  }

  return true;
});

watch(
  () => darkMode.value,
  (value, prevValue) => {
    // @ts-ignore
    document.getElementById("html").dataset.theme = value ? "dark" : "light";
  }
);

function clearAlerts() {
  apiError.value = false;
  apiOk.value = false;
  result.value = "";
}

/**
 * @function fetchPipeline
 * @param {string} id
 * @return {Promise<string>}
 * @description Method to fetch data from a remote server
 */
function fetchPipeline() {
  clearAlerts();

  loading.value = true;
  response.value = defaultResponse;

  fetch(`${server.value}/${endpoint.value}/${id.value}`)
    .then((res) => res.json())
    .then((data) => {
      console.log(data.data);
      response.value = data.data;
      loading.value = false;

      if (!data.data.id) {
        apiError.value = true;
      } else {
        apiOk.value = true;
      }
    });
}

async function runEngine() {
  loadingEngine.value = true;
  result.value = "";
  result.value = await PolicyEngine.startPolicyEngine(response.value);
  loadingEngine.value = false;
}
</script>

<template>
  <main class="container">
    <div class="grid">
      <div>
        <article>
          <fieldset>
            <label for="switch">
              <input
                type="checkbox"
                id="switch"
                v-model="darkMode"
                name="switch"
                role="switch"
              />
              Dark mode
            </label>
          </fieldset>
          <header>
            <kbd>{{ msg }}</kbd>
          </header>
          <form>
            <label for="id">
              Id:
              <input
                @keyup="clearAlerts()"
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
              Send request ????
            </button>
          </form>
          <footer>
            <p v-show="apiError">
              <mark>
                No se encontr?? un pipeline con el identificador: {{ id }}
              </mark>
            </p>
            <p v-show="apiOk">
              <ins>
                ??? Se encontr?? un pipeline con el identificador: {{ id }}
              </ins>
            </p>

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
          <header>
            <kbd>Ejecutar engine ??????</kbd>
          </header>
          <button
            :disabled="disableRun"
            :aria-busy="loadingEngine"
            type="button"
            @click.prevent="runEngine()"
          >
            Run engine ????
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

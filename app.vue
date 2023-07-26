<script setup lang="ts">
import { MeiliSearch } from 'meilisearch'

useHead({
  title: 'Recipe Search',
});

const client = new MeiliSearch({
  host: 'https://search.matteopolak.com',
  apiKey: 'b95ce79f288f660717c49e35f0667ce8a8bb500dd32259cc0de0c4a12ac8b31c'
});

const COLOURS = new Map<string, string>();

type Recipe = {
  id: number;
  title: string;
  quantities: string[];
  ingredients: string[];
  directions: string[];
}

let ctx = 0;

let search = ref('');
let results = ref<Recipe[]>([]);
let activeRecipe = ref<Recipe>();

const setSearch = debounce(async (s: string) => {
  const context = ++ctx;
  const result = await client.index('recipes').search(s, {
    limit: 100,
  });

  if (context === ctx) {
    results.value = result.hits as Recipe[];
  }
}, 50);

watch(search, setSearch);

function debounce(fn: Function, delay: number) {
  let timeout: NodeJS.Timeout | null = null;

  return function (this: any, ...args: any[]) {
    if (timeout !== null) clearTimeout(timeout);

    timeout = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  }
}

function random(high: number): number {
  return Math.floor(Math.random() * high);
}

function toColour(str: string) {
  if (!COLOURS.has(str)) {
    COLOURS.set(str, `hsl(${random(360)}, ${random(20) + 80}%, 50%)`);
  }

  return COLOURS.get(str)!;
}

function openRecipe(recipe: Recipe) {
  activeRecipe.value = recipe;

  // @ts-expect-error - `recipeModal` is defined by the dialog
  recipeModal.showModal();
}
</script>

<template>
  <dialog id="recipeModal" class="modal">
    <form method="dialog" class="modal-box prose">
      <h1 class="font-bold">{{ activeRecipe?.title }}</h1>

      <h2>Ingredients</h2>
      <ul>
        <li v-for="quantity in activeRecipe?.quantities">
          {{ quantity }}
        </li>
      </ul>

      <h2>Directions</h2>
      <ol>
        <li v-for="direction in activeRecipe?.directions">
          {{ direction }}
        </li>
      </ol>

      <div class="modal-action">
        <button class="btn btn-error">Close</button>
      </div>
    </form>

    <form method="dialog" class="modal-backdrop">
      <button>close</button>
    </form>
  </dialog>

  <div class="my-8 flex flex-col items-center gap-8">
    <div class="grid relative w-full max-w-lg">
      <svg class="w-8 h-8 absolute left-3 fill-current pointer-events-none place-self-center" viewBox="0 0 24 24">
        <path
          d="M15.5 14h-.79l-.28-.27C15.41 12.59 16 11.11 16 9.5 16 5.91 13.09 3 9.5 3S3 5.91 3 9.5 5.91 16 9.5 16c1.61 0 3.09-.59 4.23-1.57l.27.28v.79l5 4.99L20.49 19l-4.99-5zm-6 0C7.01 14 5 11.99 5 9.5S7.01 5 9.5 5 14 7.01 14 9.5 11.99 14 9.5 14z">
        </path>
      </svg>

      <input type="text" id="search" placeholder="Search for a recipe, ingredients, or instructions…"
        class="input input-bordered w-full max-w-lg rounded-full pl-12" v-model="search" />
    </div>

    <div class="flex flex-wrap flex-row gap-3 justify-center" v-if="results.length">
      <div class="card w-96 bg-base-200 hover:shadow-xl card-bordered transition-all duration-300 cursor-pointer"
        @click="openRecipe(result)" v-for="result in results">
        <div class="card-body prose">
          <h2 class="card-title">
            {{ result.title }}
          </h2>

          <ul class="line-clamp-4">
            <li v-for="quantity in result.quantities">
              {{ quantity }}
            </li>
          </ul>

          <div class="card-actions gap-1">
            <div v-for="ingredient in result.ingredients" class="badge badge-outline" :style="{
              color: toColour(ingredient)
            }">
              {{ ingredient }}</div>
          </div>
        </div>
      </div>
    </div>
    <div v-else class="w-full grid justify-center place-items-center">
      No recipes found. 😢
    </div>
  </div>
</template>

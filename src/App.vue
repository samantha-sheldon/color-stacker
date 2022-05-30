<script setup lang="ts">
  import { onMounted, reactive, ref } from 'vue';
  import ColorBottle from './components/ColorBottle.vue';

  interface State {
    contents: Array<Array<number>>,
    liftedBottle: null | number,
    bottleSize: number,
    bottlesPerColor: number,
    numColors: number,
    playerMoves: number,
    wonGame: boolean,
    showGame: boolean
  }

  const colors = [
    { name: 'red', class: 'bg-red-500' },
    { name: 'blue', class: 'bg-blue-500' },
    { name: 'green', class: 'bg-green-500' },
    { name: 'purple', class: 'bg-purple-500' },
    { name: 'yellow', class: 'bg-yellow-400' },
    { name: 'orange', class: 'bg-orange-500' },
    { name: 'pink', class: 'bg-pink-500' },
    { name: 'cyan', class: 'bg-cyan-400' },
  ];

  const state: State = reactive({
    contents: [],
    liftedBottle: null,
    bottleSize: 4,
    bottlesPerColor: 2,
    numColors: 4,
    playerMoves: 0,
    wonGame: false,
    showGame: true
  });

  function clickBottle(b: number) {
    if (state.liftedBottle == null) {
      // lift the clicked bottle (but only if it's got contents)
      if (isLiftable(b)) {
        state.liftedBottle = b;
      }
    } else {
      // if we click on the currently lifted bottle, we put it down
      if (b == state.liftedBottle) {
        state.liftedBottle = null;
        return;
      }
      // if there's no room in the clicked bottle, do nothing
      if (!isPourable(b)) {
        return;
      }
      // put top of lifted bottle's stack into clicked bottle and unlift
      let s: number = state.contents[state.liftedBottle].pop() ?? -1;
      state.contents[b].push(s);
      state.liftedBottle = null;
      state.playerMoves++;
      // check if this was a win
      checkWin();
    }
  }

  function isLiftable(b: number) {
    if (state.contents[b].length > 0) {
      return !isComplete(b);
    }
    return false;
  }

  function isPourable(b: number) {
    if (state.liftedBottle !== null) {
      if (!isComplete(b)) {
        return state.contents[b].length < state.bottleSize;
      }
    }
    return false;
  }

  function isComplete(b: number) {
    if (state.contents[b].length == state.bottleSize) {
      return state.contents[b].every((val, idx, arr) => val === arr[0]);
    }
    return false;
  }

  function checkWin() {
    state.wonGame = state.contents.every((val, idx) => isComplete(idx) || val.length === 0);
    return state.wonGame;
  }

  function startGame() {
    // hide the game so there's no awkward popping
    state.showGame = false;
    // generate contents based on the puzzle parameters
    // TODO: optimize this
    // first, make an array of all the contents, not sorted into bottles
    let items: Array<number> = [];
    for (let ii = 0; ii < state.numColors; ii++) {
      items = items.concat(new Array(state.bottlesPerColor * state.bottleSize).fill(ii));
    }
    console.log(items);
    // next, shuffle this array
    // (reference for different sorts here: https://stackoverflow.com/questions/2450954/how-to-randomize-shuffle-a-javascript-array)
    items = items
      .map(value => ({ value, sort: Math.random() }))
      .sort((a, b) => a.sort - b.sort)
      .map(({ value }) => value);
    // now, put these contents into bottles
    // TODO: shuffle around so there might not be an empty bottle at the start
    state.contents = [];
    for (let ii = 0; ii < state.bottlesPerColor * state.numColors; ii++) {
      state.contents[ii] = items.slice(ii * state.bottleSize, (ii * state.bottleSize) + state.bottleSize);
    }
    state.contents.push([]);
    // reset the remaining game data and reshow the game
    state.liftedBottle = null;
    state.wonGame = false;
    state.playerMoves = 0;
    state.showGame = true;
  }

  onMounted(() => {
    startGame();
  });
</script>

<template>
  <div class="container mx-auto flex flex-col min-h-screen bg-white gap-4 p-4 pb-8 w-fit">
    <h1 class="text-3xl text-orange-800 font-bold">Color Stacker</h1>
    <p>Pour the colored liquids from one bottle to another until all but one bottle is filled to the top with a single color.</p>
    <div class="flex flex-col gap-4 w-fit mx-auto">
      <div class="grid grid-cols-2 gap-4">
        <!--<label for="bottleSize" class="text-right py-1">Bottle Size:</label>
        <input name="bottleSize" class="border border-gray-300 pl-2 pr-px py-1" type="number" step="1" min="2" v-model="state.bottleSize" />-->
        <label for="numColors" class="text-right py-1"># of Colors:</label>
        <input name="numColors" class="border border-gray-300 pl-2 pr-px py-1" type="number" step="1" min="2" :max="colors.length" v-model="state.numColors" />
        <label for="bottlesPerColor" class="text-right py-1"># of Bottles per Color:</label>
        <input name="bottlesPerColor" class="border border-gray-300 pl-2 pr-px py-1" type="number" step="1" min="1" v-model="state.bottlesPerColor" />
      </div>
      <button @click="startGame()" class="bg-orange-300 hover:bg-orange-400 border border-white active:border-orange-300 p-2 rounded-sm font-bold">New Game</button>
    </div>
    <main class="relative">
      <div v-show="state.wonGame" class="absolute bg-white bg-opacity-80 top-0 right-0 bottom-0 left-0 flex flex-col items-center justify-center gap-4">
        <div class="font-bold text-2xl">You won!</div>
        <div>You completed the puzzle in {{ state.playerMoves }} moves.</div>
      </div>
      <div v-show="state.showGame" class="grid md:grid-cols-8 grid-cols-4 gap-8">
        <ColorBottle @click="clickBottle(index)" v-for="(stack, index) in state.contents" :colors="colors" :size="state.bottleSize" :stack="stack" :lifted="index == state.liftedBottle" :liftable="isLiftable(index)" :pourable="isPourable(index)" :complete="isComplete(index)" />
      </div>
    </main>
  </div>
</template>

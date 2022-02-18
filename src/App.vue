<script setup lang="ts">
import { onMounted, ref } from 'vue';
import type { Ref } from "vue";
const obj_function_string = ref("x*x");
const pop_size = ref(4);
const evo_times = ref(10);
const canvas: Ref<HTMLCanvasElement> = ref(document.createElement('canvas'));
const get_obj_function = () => {
  let s = obj_function_string.value;
  let obj_function = eval("(x)=>{return " + s + "}");
  return obj_function
}
const best_so_far: Ref<number[]> = ref([]);
const best_current: Ref<number[]> = ref([]);
function encode(number: number): number[] {
  // 0  0 0 0 0
  // 16 8 4 2 0
  if (number >= 31) return [1, 1, 1, 1, 1];
  let res = [];
  if (number >= 16) {
    res.push(1)
    number = number - 16;
  } else {
    res.push(0)
  }

  if (number >= 8) {
    res.push(1)
    number = number - 8;
  } else {
    res.push(0)
  }

  if (number >= 4) {
    res.push(1)
    number = number - 4;
  } else {
    res.push(0)
  }

  if (number >= 2) {
    res.push(1)
    number = number - 2;
  } else {
    res.push(0)
  }

  res.push(number);
  return res;
}
function decode(bin: number[]): number {
  return bin[0] * 16 + bin[1] * 8 + bin[2] * 4 + bin[3] * 2 + bin[4];
}
function random_gene(): number {
  return Math.random() > 0.5 ? 1 : 0;
}
function random_chromosomes(): number[] {
  return [
    random_gene(),
    random_gene(),
    random_gene(),
    random_gene(),
    random_gene()
  ]
}
function random_population(poplation_size: number): number[][] {
  let pop = [];
  for (let i = 0; i < poplation_size; i++) {
    pop.push(random_chromosomes());
  }
  return pop;
}

function crossover(p1: number[], p2: number[]): { newp1: number[], newp2: number[] } {
  let crossover_place = Math.floor(Math.random() * 4 + 1);
  let p1t = p1.splice(crossover_place);
  let p2t = p2.splice(crossover_place);
  p1.push(...p2t);
  p2.push(...p1t);
  return {
    newp1: p1, newp2: p2
  }
}
function mutation(ind: number[]): number[] {
  let mutation_place = Math.floor(Math.random() * 5);
  if (ind[mutation_place] === 0) {
    ind[mutation_place] = 1;
  } else {
    ind[mutation_place] = 0;
  }
  return ind;
}
function gen_new_pop(p1: number[], p2: number[], pop_size: number): number[][] {
  let new_pop = [];
  for (let i = 0; i < pop_size / 2; i++) {
    let n = crossover(p1, p2);
    new_pop.push(mutation(n.newp1))
    new_pop.push(mutation(n.newp2))
  }
  if (pop_size % 2 === 1) {
    let n = crossover(p1, p2);
    new_pop.push(mutation(n.newp1))
  }
  return new_pop;
}
const start_evo = () => {
  best_so_far.value = [];
  best_current.value = [];
  let obj_function = get_obj_function();
  let population = random_population(pop_size.value);
  for (let i = 0; i < evo_times.value; i++) {
    // evaluation
    let eva = population.map((chromosomes) => {
      return (obj_function(decode(chromosomes)) as number)
    })

    // select two individuals
    // roulette-wheel selection
    let sum = eva.reduce((p, c) => { return p + c }, 0);
    let dis = eva.map((x) => {
      return x / sum;
    });
    let parent_index = [];
    for (let i = 0; i < 2; i++) {
      let random_number = Math.random();
      for (let j = 0; j < pop_size.value; j++) {
        if (random_number > dis[j]) {
          random_number = random_number - dis[j];
        } else {
          parent_index.push(j)
          break;
        }
      }
    }
    let _best_current = eva[parent_index[0]] > eva[parent_index[1]] ? eva[parent_index[0]] : eva[parent_index[1]];
    best_current.value.push(_best_current);
    if (best_so_far.value.length <= 0 || best_so_far.value[best_so_far.value.length - 1] < _best_current) {
      best_so_far.value.push(_best_current)
    } else {
      best_so_far.value.push(best_so_far.value[best_so_far.value.length - 1])
    }
    // Generate
    let p1 = population[parent_index[0]];
    let p2 = population[parent_index[1]];
    population = gen_new_pop(p1, p2, pop_size.value);
  }
  drawCurve(best_so_far.value, best_current.value);
};

function drawCurve(_best_so_far: number[], _best_current: number[]) {
  var best_so_far = {
    y: _best_so_far,
    type: 'scatter',
    name: "best so far",
  };

  var best_current = {
    y: _best_current,
    type: 'scatter',
    name: "best current",
  };

  var data = [best_so_far, best_current];

  Plotly.newPlot('plot', data, {
    height: 500,
    xaxis: {
      title: 'Generation times',
      showgrid: false,
      zeroline: false
    },
    yaxis: {
      title: 'Fitness value',
      showline: false
    }
  });
}
onMounted(() => {
  let context = canvas.value.getContext("2d");
  if (context !== null) {
    context.fillText("0", 0, canvas.value.height)
    context.moveTo(10, 10);
    context.lineTo(70, 70);
    context.lineTo(10, 70);
    context.lineTo(10, 10);
    context.lineWidth = 1;
    context.strokeStyle = "red";
    // 绘制
    context.stroke();

    // 状态设置
    context.moveTo(20, 10);
    context.lineTo(70, 60);
    context.strokeStyle = "black";
    // 绘制
    context.stroke();
  }

})
</script>

<template>
  <h1>Evolutionary Computation Lab 1</h1>
  <h2>Gesheng</h2>
  <div>
    <div style="margin: 5px;">
      <span>Object Function:</span>
      <input type="text" v-model="obj_function_string" />
    </div>
    <div style="margin: 5px;">
      <span>Population Size:</span>
      <input type="number" v-model="pop_size" />
    </div>
    <div style="margin: 5px;">
      <span>Evolution Times:</span>
      <input type="number" v-model="evo_times" />
    </div>
  </div>
  <div>
    <button @click="start_evo">start evolution</button>
  </div>
  <div id="plot" style="margin-left: 10vw;margin-right: 10vw;"></div>
</template>

<style>
#id {
  margin-top: 5vh;
  width: 600px;
  height: 300px;
  border: 1px solid #2c3e50;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

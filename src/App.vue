<template>
  <div class="app">
    <h1>Weight Tracker</h1>

    <div class="current">
      <span>{{currentWeight.weight}}</span>
      <small>Current Weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight">

      <input type="number" step="0.1" v-model="weightInput">

      <input type="submit" value="Add Weight">

    </form>

    <div v-if="weights&& weights.length>0">
      <h2>Last 7 days</h2>

      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>

      <div class="weight-history">
        <h2>Weight History</h2>
        <ul>
          <li v-for="weight in weights">
            <span>{{weight.weight}}kg</span>
            <small>{{new Date(weight.date).toLocaleDateString()}}</small>
          </li>
        </ul>
      </div>
    </div>
  </div>


</template>

<script setup lang="ts">
import { ref, reactive, shallowRef, watch, computed, nextTick } from 'vue'
import Chart from 'chart.js/auto'
type Weight = {
  weight: number,
  date: number
}
// 体重数据的数组
const weights = ref<Weight[]>([])

// 体重图表元素
const weightChartEl = ref<any>(null)

const weightChart = shallowRef<any>(null)

const weightInput = ref(60.0)

const currentWeight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 }
})

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: new Date().getTime()
  })
}

watch(weights, newWeights => {
  const ws = [...newWeights]
  if (weightChart.value) {
    weightChart.value.data.labels = ws
      .sort((a, b) => a.date - b.date)
      .map(w => new Date(w.date).toLocaleDateString())
      .slice(-7)

    weightChart.value.data.datasets[0].data = ws
      .sort((a, b) => a.date - b.date)
      .map(w => w.weight)
      .slice(-7)

    weightChart.value.update()
  }
  nextTick(() => {
    weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
      type: 'line',
      data: {
        labels: ws
          .sort((a, b) => a.date - b.date)
          .map(w => new Date(w.date).toLocaleDateString()),
        datasets: [
          {
            label: 'Weight',
            data: ws
              .sort((a, b) => a.date - b.date)
              .map(w => w.weight),
            backgroundColor: 'rgba(255,105,180,0.2)',
            borderColor: 'rgb(255,105,180)',
            borderWidth: 1,
            fill: true
          }
        ]

      },
      options: {
        responsive: true,
        maintainAspectRatio: false
      }
    })
  })
}, { deep: true })

</script>
<style lang='less'>
* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
  font-family: "montserrat", sans-serif;
}

body {
  background-color: rgb(241, 241, 241);
}

.app {
  padding: 1.5rem;

  h1 {
    margin-bottom: 2rem;
    font-size: 2em;
    text-align: center;
  }

  h2 {
    margin-bottom: 1rem;
    color: #888;
    font-weight: 400;
  }

  .current {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 200px;
    height: 200px;
    margin: 0 auto 2rem;
    text-align: center;
    background-color: #fff;
    box-shadow: 0 4px 12px rgba(0, 0, 0, .1);
    border: 5px solid hwb(330 41% 0%);
    border-radius: 999px;

    span {
      display: block;
      margin-bottom: .5rem;
      font-size: 2em;
      font-weight: bold;
    }

    small {
      color: #888;
      font-style: italic;
    }
  }

  form {
    display: flex;
    overflow: hidden;
    margin-bottom: 2rem;
    border: 1px solid #aaa;
    border-radius: .5rem;
    transition: 200ms linear;
  }

  form:focus-within,
  form:hover {
    border-color: hotpink;
    border-width: 2px;
  }

  form input[type="number"] {
    // flex: grow shrink basis
    flex: 1 0 0;
    padding: 1rem 1.5rem;
    font-size: 1.25rem;
    appearance: none;
    background-color: #fff;
    border: none;
    outline: none;
  }

  form input[type="submit"] {
    padding: .5rem 1rem;
    color: white;
    font-size: 1.25rem;
    font-weight: 700;
    appearance: none;
    background-color: hotpink;
    border: none;
    border-left: 3px solid transparent;
    outline: none;
    transition: 200ms linear;
    cursor: pointer;
  }

  form input[type="submit"]:hover {
    color: hotpink;
    background-color: #fff;
    border-left-color: hotpink;
  }

  .canvas-box {
    width: 100%;
    max-width: 720px;
    padding: 1rem;
    margin-bottom: 2rem;
    background-color: #fff;
    box-shadow: 0 4px 12px rgba(0, 0, 0, .1);
    border-radius: .5rem;
  }

  .weight-history ul {
    padding: 0;
    margin: 0;
    list-style: none;

    li {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: .5rem;
      border-radius: .5rem;
      cursor: pointer;
    }

    li:nth-child(odd) {
      background-color: #dfdfdf;
    }

    li:hover {
      background-color: #f8f8f8;
    }

    li:last-of-type {
      border-bottom: none;
    }

    li span {
      display: block;
      margin-right: 1rem;
      font-size: 1.25rem;
      font-weight: 700;
    }

    li small {
      color: #888;
      font-style: italic;
    }
  }
}
</style>
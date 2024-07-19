<script setup lang="ts">
import { ref, watch } from 'vue'

interface Car {
  name: string
  wheels: string
}

const searchText = ref('')

const carList = ref<Car[]>([])
const car = ref<Car>()

const carFounded = ref(false)
const isAddableCar = ref(false)

const newWheels = ref('')
const message = ref('')

const searchCarByWheelsText = ref('')

const BASE_API_URL = import.meta.env.VITE_API_BASE_URL

async function getCarByName() {
  isAddableCar.value = false

  if (searchText.value === 'all') {
    try {
      const response = await fetch(`${BASE_API_URL}/cars`)
      carList.value = await response.json()
    } catch (error) {
      message.value = 'There is not a car in the list!'
    }
  } else
    try {
      const response = await fetch(`${BASE_API_URL}/${searchText.value}`)
      car.value = await response.json()

      if (car.value) {
        carFounded.value = true
      }
    } catch (error) {
      isAddableCar.value = true
      carFounded.value = false
    }
}

async function getCarByWheels() {
  try {
    const response = await fetch(`${BASE_API_URL}/cars/wheels/${searchCarByWheelsText.value}`)
    carList.value = await response.json()

    console.log(carList.value)
  } catch (error) {
    isAddableCar.value = true
    carFounded.value = false
  }
}

async function addNewCar() {
  try {
    const requestOptions = {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ name: searchText.value, wheels: newWheels.value })
    }

    const response = await fetch(`${BASE_API_URL}/cars/add`, requestOptions)
    const res = await response.json()

    if (res) {
      message.value = 'Thanks. I updated the information.'
    }
  } catch (error) {
    console.log(error)
  }
}

watch(searchText, () => {
  isAddableCar.value = false
  message.value = ''
  carList.value = []
})

watch(searchCarByWheelsText, () => {
  carList.value = []
})
</script>

<template>
  <div>
    <form @submit.prevent="getCarByName">
      <input type="text" v-model="searchText" required placeholder="Find car by name!" />
      <button type="submit">Search</button>
    </form>

    <form @submit.prevent="getCarByWheels">
      <input
        type="text"
        v-model="searchCarByWheelsText"
        required
        placeholder="Find car by wheels!"
      />
      <button type="submit">Get car by wheels</button>
    </form>

    <span v-if="carFounded">“{{ car?.name }}” has {{ car?.wheels }} wheels.</span>

    <form @submit.prevent="addNewCar" v-if="isAddableCar">
      <h2>{{ searchText }} is not on my list. Please input number of wheels below:</h2>
      <input type="text" v-model="newWheels" required placeholder="Number of wheels?" />
      <button type="submit">Add New Car</button>
    </form>

    <span v-if="message !== ''">{{ message }} </span>

    <div v-if="!!carList.length">
      <div v-for="car in carList" :key="car.name">
        {{ car.name }}
      </div>
    </div>
  </div>
</template>

<style scoped></style>

<template>
  <UCard>
    <template #header>
      <Placeholder class="h-8" />
      <label>{{ title }}</label>
    </template>

    <Placeholder class="h-32" />

    <UCard v-for="item in data" :key="item.id">
      <div class="flex justify-between gap-10">
        <div>
          <span>
            {{ item.title }}
          </span>
        </div>
        <div class="flex gap-2">
          <UButton
            v-if="title !== 'TODO'"
            color="red"
            @click="emit('moveLeft', title, item.id)"
            ><i class="fa-solid fa-caret-left"></i
          ></UButton>
          <UButton color="amber" @click="emit('openModal', isOpen, item.id)"
            ><i class="fa-solid fa-pencil"></i
          ></UButton>
          <UButton
            v-if="title !== 'DONE'"
            color="primary"
            @click="emit('moveRight', title, item.id)"
            ><i class="fa-solid fa-caret-right"></i
          ></UButton>
        </div>
      </div>
      <UButton v-if="item.complete" class="mt-2 flex items-center gap-4"
        ><i class="fa-regular fa-square-check"></i>
        <span>
          {{ formattedDate(item.due_date) }}
        </span>
      </UButton>
    </UCard>

    <UCard v-if="add">
      <UInput v-model="cardTitle" @keyup.enter="handleSubmit" />
    </UCard>

    <div class="flex justify-center">
      <UButton class="my-2" @click="handleAdd" v-if="!add"
        ><i class="fa-solid fa-plus"></i> Add a card</UButton
      >
      <UButton class="my-2" @click="handleClose" v-if="add"
        ><i class="fa-solid fa-xmark"></i
      ></UButton>
    </div>
  </UCard>
</template>

<script setup>
import moment from "moment";
const { title, data, isOpen } = defineProps(["title", "data", "isOpen"]);

const add = ref(false);
const cardTitle = ref("");

//format date day and month only
const formattedDate = (date) => {
  return moment(date).format("DD MMM");
};
const handleAdd = () => {
  add.value = true;
};

const handleClose = () => {
  add.value = false;
  cardTitle.value = "";
};

const emit = defineEmits(["openModal", "addData", "moveRight", "moveLeft"]);

const handleSubmit = () => {
  emit("addData", title, cardTitle.value);
  add.value = false;
  cardTitle.value = "";
};
</script>

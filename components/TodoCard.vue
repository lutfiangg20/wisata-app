<template>
  <UCard class="dark:border-green-600 dark:border-2 w-max">
    <template #header>
      <Placeholder class="h-8" />
      <label class="font-bold">{{ title }}</label>
    </template>

    <Placeholder class="h-32" />
    <div class="flex justify-end gap-2 mb-5">
      <label for="">Sort</label>
      <USelect v-model="selected" :options="options" @change="handleSort" />
    </div>

    <UCard
      draggable="true"
      v-for="item in data"
      :key="item.id"
      @dragstart="startDrag($event, item)"
      @drop="endDrag($event, item)"
      @dragover="handleDragOver($event)"
      class="mb-2 dark:border-green-600 dark:border-2"
    >
      <div class="flex justify-between gap-10">
        <div>
          <span class="truncate">
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
      <UButton
        v-if="item.complete"
        class="mt-2 flex items-center gap-4 rounded-sm"
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
      <UButton class="my-5" @click="handleAdd" v-if="!add"
        ><i class="fa-solid fa-plus"></i> Add a card</UButton
      >
      <div class="flex gap-2" v-if="add">
        <UButton class="my-5" @click="handleClose"
          ><i class="fa-solid fa-xmark"></i
        ></UButton>
        <UButton class="my-5" @click="handleSubmit"
          ><i class="fa-solid fa-check"></i
        ></UButton>
      </div>
    </div>
  </UCard>
</template>

<script setup>
import moment from "moment";
const { title, data, isOpen } = defineProps(["title", "data", "isOpen"]);

const add = ref(false);
const cardTitle = ref("");
const selected = ref("");
const options = ref(["Name", "Created", "Due Date"]);

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

const startDrag = (e, item) => {
  console.log("start", item);
  e.dataTransfer.dropEffect = "move";
  e.dataTransfer.effectAllowed = "move";
  e.dataTransfer.setData("itemId", item.id);
};

const endDrag = (e, item) => {
  const itemId = e.dataTransfer.getData("itemId");
  const toDrag = item;
  emit("endDrag", toDrag, itemId);
  /* if (itemId !== item.id) {
    emit("endDrag", title, selected.value.toLocaleLowerCase());
  } */
};

const handleDragOver = (e) => {
  e.preventDefault();
};

const emit = defineEmits([
  "openModal",
  "addData",
  "moveRight",
  "moveLeft",
  "handleSort",
  "endDrag",
]);

const handleSubmit = () => {
  emit("addData", title, cardTitle.value);
  add.value = false;
  cardTitle.value = "";
};

const handleSort = () => {
  emit("handleSort", title, selected.value.toLocaleLowerCase());
};
</script>

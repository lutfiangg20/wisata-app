<template>
  <UContainer>
    <div class="flex flex-row gap-5">
      <TodoCard
        title="TODO"
        :data="todo"
        isOpen="todo"
        v-on:openModal="openModal"
        @add-data="addData"
        @move-right="moveRight"
      />
      <TodoCard
        title="DOING"
        :data="doing"
        isOpen="doing"
        v-on:openModal="openModal"
        @add-data="addData"
        @move-right="moveRight"
        @move-left="moveLeft"
      />
      <TodoCard
        title="DONE"
        :data="done"
        isOpen="done"
        v-on:openModal="openModal"
        @add-data="addData"
        @move-left="moveLeft"
      />
    </div>
  </UContainer>
  <UModal v-model="isOpenModal">
    <form @submit.prevent="handleUpdate">
      <div class="p-4">
        <div class="flex justify-end">
          <UButton color="red" @click="handleDelete"
            ><i class="fa-solid fa-trash"></i
          ></UButton>
        </div>
        <Placeholder class="h-48" />
        <div class="mb-2">
          <label for=""> Title </label>
          <UInput v-model="formEdit.title" :value="formEdit.title" />
        </div>
        <div class="mb-2">
          <label for="">Description</label>
          <UTextarea
            v-model="formEdit.description"
            :value="formEdit.description"
          />
        </div>
        <div class="flex justify-end mt-2">
          <UButton type="submit">Submit</UButton>
        </div>
      </div>
    </form>
  </UModal>
</template>

<script setup>
import { createClient } from "@supabase/supabase-js";
const todo = ref([]);
const doing = ref([]);
const done = ref([]);
const edit = ref();
const formEdit = reactive({
  id: "",
  title: "",
  description: "",
});

const isOpenModal = ref(false);

const supabaseUrl = useRuntimeConfig().public.apiURL;
const supabaseKey = useRuntimeConfig().public.apiKey;
const supabase = createClient(supabaseUrl, supabaseKey);

const getData = async () => {
  const { data, error } = await supabase
    .from("todo")
    .select("*")
    .order("updated_at");
  if (error) {
    console.log(error);
  } else {
    todo.value = await data.filter((item) => {
      return item.status === "todo";
    });
    doing.value = await data.filter((item) => {
      return item.status === "doing";
    });
    done.value = await data.filter((item) => {
      return item.status === "done";
    });
  }
};

const openModal = (value, id) => {
  isOpenModal.value = true;

  if (value === "todo") {
    const filter = todo.value.filter((item) => item.id === id);
    edit.value = filter[0];
  }
  if (value === "doing") {
    const filter = doing.value.filter((item) => item.id === id);
    edit.value = filter[0];
  }
  if (value === "done") {
    const filter = done.value.filter((item) => item.id === id);
    edit.value = filter[0];
  }

  formEdit.id = edit.value.id;
  formEdit.title = edit.value.title;
  formEdit.description = edit.value.description;
};

const addData = async (title, cardTitle) => {
  await supabase
    .from("todo")
    .insert({
      title: cardTitle,
      status: title.toLowerCase(),
    })
    .then(() => {
      getData();
    });
};

const handleUpdate = async () => {
  await supabase
    .from("todo")
    .update({
      title: formEdit.title,
      description: formEdit.description,
    })
    .eq("id", edit.value.id)
    .then(() => {
      getData();
    });
  isOpenModal.value = false;
};

const handleDelete = async () => {
  await supabase
    .from("todo")
    .delete()
    .eq("id", formEdit.id)
    .then(() => {
      getData();
      isOpenModal.value = false;
    });
};

const moveRight = async (title, id) => {
  await supabase
    .from("todo")
    .update({
      status: title === "TODO" ? "doing" : "done",
      updated_at: new Date(),
    })
    .eq("id", id)
    .then(() => {
      getData();
    });
};

const moveLeft = async (title, id) => {
  await supabase
    .from("todo")
    .update({
      status: title === "DONE" ? "doing" : "todo",
      updated_at: new Date(),
    })
    .eq("id", id)
    .then(() => {
      getData();
    });
};

onMounted(() => {
  getData();
});
</script>

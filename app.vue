<template>
  <div class="h-screen dark:accent-slate-900">
    <div class="flex justify-between p-2">
      <a href="/" class="text-3xl font-bold">Todo App</a>
      <div class="flex justify-end gap-2">
        <label for="">Theme</label>
        <USelect
          v-model="selected"
          :options="options"
          @change="setColorTheme(selected)"
        >
          <template #leading v-if="selected === 'Light'"
            ><i class="fa-regular fa-sun"></i
          ></template>
          <template #leading v-else><i class="fa-solid fa-moon"></i></template>
        </USelect>
      </div>
    </div>
    <UContainer class="w-fit">
      <div class="flex gap-2 justify-end mb-5">
        <label for="">Search</label>
        <UInput v-model="search" @keyup="handleSearch" />
      </div>
      <div class="flex flex-row gap-5 justify-between">
        <TodoCard
          title="TODO"
          :data="todo"
          isOpen="todo"
          v-on:openModal="openModal"
          @add-data="addData"
          @move-right="moveRight"
          @handle-sort="handleSort"
          @end-drag="endDrag"
        />
        <TodoCard
          title="DOING"
          :data="doing"
          isOpen="doing"
          v-on:openModal="openModal"
          @add-data="addData"
          @move-right="moveRight"
          @move-left="moveLeft"
          @handle-sort="handleSort"
          @end-drag="endDrag"
        />
        <TodoCard
          title="DONE"
          :data="done"
          isOpen="done"
          v-on:openModal="openModal"
          @add-data="addData"
          @move-left="moveLeft"
          @handle-sort="handleSort"
          @end-drag="endDrag"
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
          <div class="mb-2">
            <label for="">Due Date</label>
            <div class="flex gap-2 items-center">
              <UCheckbox @change="checkDate" :model-value="formEdit.complete" />
              <UInput
                type="date"
                v-if="formEdit.complete"
                v-model="formEdit.due_date"
                :value="new Date(formEdit.due_date).toISOString().split('T')[0]"
              />
              <UInput
                type="date"
                v-else
                v-model="formEdit.due_date"
                :value="formEdit.due_date"
              />
              <UButton v-if="formEdit.complete"
                ><i class="fa-regular fa-square-check"></i
              ></UButton>
              <UButton color="amber" v-else
                ><i class="fa-regular fa-clock"></i
              ></UButton>
            </div>
          </div>
          <div class="flex justify-end mt-2">
            <UButton type="submit">Submit</UButton>
          </div>
        </div>
      </form>
    </UModal>
  </div>
</template>

<script setup>
import { createClient } from "@supabase/supabase-js";

useHead({
  title: "Todo App",
});

const setColorTheme = (NewTheme) => {
  console.log(NewTheme);
  useColorMode().preference = NewTheme.toLowerCase();
};

const selected = ref("Light");
const todo = ref([]);
let todoFilter = [];
const doing = ref([]);
let doingFilter = [];
const done = ref([]);
let doneFilter = [];
const edit = ref();
const options = ref(["Light", "Dark"]);
const search = ref("");
const formEdit = reactive({
  id: "",
  title: "",
  description: "",
  complete: false,
  due_date: "",
});
const date = ref(new Date());

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
    todoFilter = await todo.value;

    doing.value = await data.filter((item) => {
      return item.status === "doing";
    });
    doingFilter = await doing.value;

    done.value = await data.filter((item) => {
      return item.status === "done";
    });
    doneFilter = await done.value;
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
  formEdit.due_date = edit.value.due_date;
  formEdit.complete = edit.value.complete;
  console.log(formEdit);
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

const handleSearch = () => {
  todo.value = todoFilter.filter((item) =>
    item.title.toLowerCase().includes(search.value.toLowerCase())
  );

  doing.value = doingFilter.filter((item) =>
    item.title.toLowerCase().includes(search.value.toLowerCase())
  );

  done.value = doneFilter.filter((item) =>
    item.title.toLowerCase().includes(search.value.toLowerCase())
  );
};

const handleUpdate = async () => {
  await supabase
    .from("todo")
    .update({
      title: formEdit.title,
      description: formEdit.description,
      due_date: formEdit.due_date,
      complete: formEdit.complete,
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

const handleSort = async (title, selected) => {
  let select;

  if (selected === "name") {
    select = "title";
  }
  if (selected === "created") {
    select = "created_at";
  }
  if (selected === "due date") {
    select = "due_date";
  }

  const { data, error } = await supabase
    .from("todo")
    .select("*")
    .order(`${select}`);

  if (title === "TODO") {
    todo.value = await data.filter((item) => {
      return item.status === "todo";
    });
  }
  if (title === "DOING") {
    doing.value = await data.filter((item) => {
      return item.status === "doing";
    });
  }
  if (title === "DONE") {
    done.value = await data.filter((item) => {
      return item.status === "done";
    });
  }
};

const checkDate = () => {
  formEdit.complete = !formEdit.complete;
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

const endDrag = async (toDrag, itemId) => {
  console.log("toDrag", toDrag.status);
  console.log("itemId", itemId);
  await supabase
    .from("todo")
    .update({
      status: toDrag.status,
      updated_at: new Date(),
    })
    .eq("id", itemId)
    .then(() => {
      getData();
    });
};

onMounted(() => {
  getData();
});
</script>

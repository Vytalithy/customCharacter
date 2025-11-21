<script setup lang="ts">
import { ref, reactive, onMounted, computed } from 'vue'
import OBR from "@owlbear-rodeo/sdk";
import Vdrag from 'vue3-draggable-resizable';
// import { Checkbox, CheckboxGroup } from 'primevue';


if (OBR.isAvailable) {
  OBR.notification.show(`count is ${1}`);
}
const mousex = ref(0)
const mousey = ref(0)
const isChecked = ref(false)
const class_options: Array<Object> = reactive([])
const current_class = ref("")
const dict_array: Array<Object> = reactive([])
const current_index = ref(0)
const default_width: number = 1280
const default_height: number = 720
interface BoxProperties {
  x: number,
  y: number,
  w: number,
  h: number,
  type: string,
  value: string | number,
  class: string,
  checkbox: boolean,
  checkboxval: string | number,
  active: boolean,
}
document.addEventListener("mousemove", (event: MouseEvent) => {
  mousex.value = event.pageX
  mousey.value = event.pageY
})

const get_option = (index: number, active: boolean = false) => {
  return {
    label: class_options[index],
    value: class_options[index],
    disabled: active,
  }
}
const get_option_array = computed(() => {
  let array: Array<object> = []
  for (let i = 0; i < class_options.length; i++) {
    array[i] = get_option(i)
  }
  return array
})
const is_editing = ref(false)
// TODO: create different types like:
//   -labels
//   -descriptions
//   -numbers
//   -checks X
//   -radials X

let input: BoxProperties = {
  x: 0,
  y: 0,
  w: 0,
  h: 0,
  type: "text",
  value: "",
  class: "",
  checkbox: false,
  checkboxval: "",
  active: true,
}
const input_dict: { [key: string]: BoxProperties } = reactive({
  "input": {...input}
})

async function fetch_data(path: string) {
  const data = await fetch(path)
  .then((res) => res.json())
  .catch(error => {
    console.error('Error fetching JSON:', error);
  });
  Object.keys(data).forEach((key) => add_element(data[key], key))
}

const add_element = (data: BoxProperties = {...input}, key: string = `${"input"}-${Object.keys(input_dict).length}`
) => {
  const new_array = Object.keys(data)
  input_dict[key] = {...input}
  data.x = data.x !== 0 ? data.x : mousex.value - 25
  data.y = data.y !== 0 ? data.y : mousey.value - 25
  input_dict[key] = data
  console.log('Key is: '+key+' Data is: '+JSON.stringify(input_dict[key]))
}
const viewport: Object = reactive({
  "x": window.innerWidth,
  "y": window.innerHeight,
})
const updated_viewport = () => {
  return {
  "x": window.innerWidth,
  "y": window.innerHeight,
  }
}

// function selectFile() {
//   this.$refs.fileInput.click();
// }
// function handleFileChange(event) {
//   const file = event.target.files[0];
//   if (file) {
//     this.fileName = file.name;
//   }
// }

onMounted(async () => {
  await fetch_data("../initial_position.json")
  input.type = "checkbox"
  for (let i = 0; i < 0; i++) {
    add_element()
  }
})
const onrestop = (top:number,left:number,width:number,height: number) => {
  console.log('New resize is: '+JSON.stringify({
    x:left,
    y:top,
    w:width,
    h:height,
  }))
}
const add_class = () => {
  const value: string | null = prompt("Enter the new class name")
  if (typeof value !== 'string'){
    console.log("Invalid value")
    return
  }
  current_class.value = value
  class_options[class_options.length] = value
  
  console.log("classes array is: " + get_option_array.value)
}
const load_file = async () => {
  const file = document.createElement('input')
  file.type = 'file'
  file.click()
}

const download_json = () => {
  const a = document.createElement("a")
  a.href = URL.createObjectURL(new Blob([JSON.stringify(input_dict)], {type: "application/json"}))
  a.download = "initial_position.json"
  a.click()
}
let toggle = ref({
  read: true,
  write: false,
})
const custom_func = () => {
  Object.keys(input_dict).forEach((key: string) => {
    if (input_dict[key] && input_dict[key].type === 'label') {
      input_dict[key].type = 'text'
    } 
  })
}
</script>

<template>
  <nav class="fixed-nav">
    <button @click="custom_func()" disabled></button>
    <button class="fixed-btn" @click="add_class()" ><ion-icon class="" name="bookmarks"></ion-icon></button>
    <!-- TODO: -->
    <button
      v-show="is_editing"
      class="plus-btn fixed-btn" 
      @click="input.type='text';
      add_element()"
    >
      <svg height="24px" id="Layer_1" style="enable-background:new 0 0 32 32;" version="1.1" viewBox="0 0 32 32" width="24px" xml:space="preserve" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><path fill="currentColor" d="M28,14H18V4c0-1.104-0.896-2-2-2s-2,0.896-2,2v10H4c-1.104,0-2,0.896-2,2s0.896,2,2,2h10v10c0,1.104,0.896,2,2,2  s2-0.896,2-2V18h10c1.104,0,2-0.896,2-2S29.104,14,28,14z"/></svg>
    </button>
    <button class="edit-btn fixed-btn text-center" @click="is_editing = !is_editing">
      <svg v-show="!is_editing" class="text-blue-400 feather feather-edit-3" fill="none" height="24" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M12 20h9"/><path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/></svg>
      <svg v-show="is_editing" class="text-red-700" stroke-width="" id="Icons" viewBox="0 0 24 24" width="24px" height="24px" xmlns="http://www.w3.org/2000/svg"><path fill="currentColor" stroke-width="" d="M12,0A12,12,0,1,0,24,12,12.013,12.013,0,0,0,12,0Zm0,22A10,10,0,1,1,22,12,10.011,10.011,0,0,1,12,22Z"/><path fill="currentColor" d="M16.707,7.293a1,1,0,0,0-1.414,0L12,10.586,8.707,7.293A1,1,0,1,0,7.293,8.707L10.586,12,7.293,15.293a1,1,0,1,0,1.414,1.414L12,13.414l3.293,3.293a1,1,0,0,0,1.414-1.414L13.414,12l3.293-3.293A1,1,0,0,0,16.707,7.293Z"/></svg>
    </button>
    <button class="dwn-btn fixed-btn" @click="download_json()">
      <svg class="" fill="currentColor" height="24" viewBox="0 0 24 24" width="24">
        <path stroke="currentColor" fill="currentColor" d="M22,16 L22,20 C22,21.1045695 21.1045695,22 20,22 L4,22 C2.8954305,22 2,21.1045695 2,20 L2,16 L4,16 L4,20 L20,20 L20,16 L22,16 Z M13,12.5857864 L16.2928932,9.29289322 L17.7071068,10.7071068 L12,16.4142136 L6.29289322,10.7071068 L7.70710678,9.29289322 L11,12.5857864 L11,2 L13,2 L13,12.5857864 Z"
          fill-rule="evenodd"/>
      </svg>
    </button>
    <button class="up-btn fixed-btn" @click="load_file()">
      <svg class="text-teal-400 feather feather-upload" fill="currentColor" height="24" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" width="24" xmlns="http://www.w3.org/2000/svg"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" x2="12" y1="3" y2="15"/></svg>
    </button>
  </nav>
  <Vdrag
    v-for="(val,el) in adapt_input" :key="el"
    v-model:x="val.x"
    v-model:y="val.y"
    v-model:w="val.w"
    v-model:h="val.h"
    v-model:active="val.active"
    :init-w="val.w > 0 ? val.w : 50"
    :init-h="val.h > 0 ? val.h : 50"
    class="input-container"
    :draggable="is_editing"
    :resizable="is_editing"
    @resizestop="onrestop"
  >
    <n-checkbox
      v-show="val.type === 'checkbox' && !is_editing" :disabled="is_editing"
      class="input-check" :name="val.class" :checked="val.checkbox" :value="val.checkboxval"
      @update:checked="(check: boolean) => {val.checkbox=check}"/>
    <n-select
      v-show="(val.type === 'checkbox' || val.type === 'radio') && is_editing"
      class="input-select"
      filterable tag
      :value="val.class"
      :placeholder="val.class ? val.class : 'none'"
      :options="get_option_array"
    />
    <n-radio v-show="val.type === 'radio'" class="input-check" :name="val.class" :value="val.checkboxval" @update:checked="(check: boolean) => {val.checkbox=check}"/>
    <input v-show="val.type === 'text'" v-model="val.value" type="text" class="input-box" >
    <textarea v-show='val.type === "textarea"' v-model="val.value"></textarea>
  </Vdrag>
  <div v-for="num in 3" :key="num" :class="`bg-img bg${num}`"/>
  <!-- <label for="checking"> -->
  <!--   <input id="checking" value="read" type="checkbox" v-model="toggle.read" class="input-check"  /> -->
  <!--   {{ toggle }} -->
  <!-- </label> -->
</template>
<style scoped>

#checking0{
  position: absolute;
  top: 200px;
  left: 200px;
  width: 200px;
  height: 200px;
  
}
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>


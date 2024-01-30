<script setup>
import { ref, onMounted } from 'vue';
const peopleBox = ref([]);
const people = ["Ataraxia", "carrie 🍄", "CPing", "hsuhsusophie", "Ivy", "jeremyChan", "jia yu", "PayRoom", "WA", "銀光菇"];
const waitArea = ref([]);
const colors = ref({});
const allBoxes = ref([]);
const animatePerson = ref([]);
const groupNum = ref(0)
// 把拖動目標資料抓出來儲存
function dragged(e) {
  e.dataTransfer.setData("text/plain", e.target.textContent)
}
// 在等待分組區域新增該 person
function drop(e) {
  cancelDefault(e);
  // 放下的地方錯誤的話，不作用
  if (e.target.id !== "waitArea") return
  const name = e.dataTransfer.getData('text/plain');
  if (waitArea.value.includes(name) || allBoxes.value.flat().includes(name)) return
  waitArea.value.push(name)
  // e.target.appendChild(peopleBox.value.find(person => person.textContent === name))
}
// 取消預設行為
function cancelDefault(e) {
  e.preventDefault();
  e.stopPropagation();
  return false
}
// 給每個人一個隨機的背景色
function randomColor() {
  const r = Math.floor((Math.random() * 255));
  const g = Math.floor((Math.random() * 255));
  const b = Math.floor((Math.random() * 255));
  const color = `rgba(${r},${g},${b},0.5)`
  return color;
}
function divideGroup() {
  // 目前有多少人
  let length = waitArea.value.length;
  // 如果沒人就不能分
  if (!length) return
  if (allBoxes.value.length) {
    alert("請先清除！！(・`ω´・)")
    return
  }
  // 總共幾組
  groupNum.value = Math.ceil(waitArea.value.length / 2)
  // 第幾組
  let box = 0;
  // 還有人剩下沒被分到
  while (length > 0) {
    // 隨機選一人，並取得他的名字
    const random = Math.floor(Math.random() * length);
    const el = waitArea.value[random];
    // 如果第 X 組 已經有人就 push進去 ，沒有的話要先建立陣列再放進去
    if (allBoxes.value[box % groupNum.value]) {
      allBoxes.value[box % groupNum.value].push(el)
    } else {
      allBoxes.value[box % groupNum.value] = [el]
    }
    // 被分完之後要從 waitArea 移除
    cancelPick(el)
    // 補上動畫的 class
    setTimeout(() => {
      animatePerson.value.push(el);
    }, box * 500)
    box++;
    length--;
  }
}
function allPick() {
  waitArea.value = [...people]
}
function cancelPick(person) {
  const index = waitArea.value.indexOf(person);
  if (index === -1) return
  waitArea.value.splice(index, 1)
}
function reset() {
  waitArea.value = [];
  allBoxes.value = [];
  animatePerson.value = [];
  groupNum.value = 0;
}
function defineGroupsNum() {
  const num = groupNum.value;
  allBoxes.value = [];
  for (let i = 0; i < num; i++) {
    allBoxes.value.push([]);
  }
  console.log(allBoxes.value)
}
onMounted(() => {
  people.forEach((person) => {
    colors.value[person] = randomColor()
  })
})
</script>
<template>
  <div class="container mx-auto">
    <h1>分組選擇器</h1>
    <p>使用方法：將要分組的人拖曳進等待分組區，選擇好組數後，按下分組按鈕即可。</p>
    <p>※點擊可以取消</p>
    <h2>請選擇要分組的人員:</h2>
    <button type="button"
      class="mb-4 p-2 bg-transparent border-(2 solid t-red r-gray l-gray b-blue) hover:(cursor-pointer bg-#eee)"
      @click="allPick">我全都要！！(´≖◞౪◟≖)</button>
    <div class="flex gap-4">
      <template v-if="Object.keys(colors).length">
        <div ref="peopleBox" class="person p-4 flex rd hover:cursor-pointer border border-solid h-50px items-center"
          draggable="true" @dragstart="dragged" v-for="person in people" :key="person"
          :style="{ 'background-color': colors[person] }"
          :class="{ 'op-20': waitArea.includes(person) || allBoxes.flat().includes(person) }">
          {{ person }}
        </div>
      </template>
    </div>
    <h3>等待分組區域</h3>
    <h4>目前人數:{{ waitArea.length }}</h4>
    <div class="flex gap-4 w-80% px-10 h-200px py-20 border border-solid border-black" @drop="drop"
      @dragenter="cancelDefault" @dragover="cancelDefault" id="waitArea">
      <div v-for="person in waitArea"
        class="person p-4 flex rd hover:cursor-pointer border border-solid h-50px items-center"
        :style="{ 'background-color': colors[person] }" @click="cancelPick(person)">{{ person }}</div>
    </div>
    <div class="mt-4">
      <button type="button"
        class="px-10 py-4 hover:cursor-pointer me-4 rd border border-solid bg-transparent hover:bg-gray hover:text-white"
        @click="divideGroup()">分組</button>
      <button type="button"
        class="px-4 py-2 hover:cursor-pointer me-4 rd border border-solid border-red bg-transparent color-red hover:bg-red hover:text-white"
        @click="reset()">清除</button>
    </div>
    <div class="flex justify-center">
      <div v-for="(num, index) in groupNum" :key="index" class="groups flex flex-col gap-4 me-10">
        <h3 class="text-center">分組{{ num }}</h3>
        <div v-for="person in allBoxes[index]"
          class="person p-4 flex rd hover:cursor-pointer border border-solid h-50px items-center"
          :style="{ 'background-color': colors[person] }" :class="{ 'animate': animatePerson.includes(person) }">{{ person
          }}
        </div>
      </div>
    </div>
  </div>
</template>
<style>
* {
  box-sizing: border-box;
}

.groups .person {
  transform: scale(0);
  transition: 0.5s;

  &.animate {
    transform: scale(1);
  }
}</style>
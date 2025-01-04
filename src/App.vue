<script setup lang="ts">
import {ref} from "vue";

const revealed = ref([] as string[])
const opened = ref([] as number[])
const songs = ref([] as { name: string, bonus: boolean, guessed:boolean }[])

const char = ref("")
const close = ref(false) // 关字母
const show = ref(false) // show answer
const edit = ref(true)

function unreveal(val: string) {
  if (!close.value) return
  revealed.value = revealed.value.filter(v => v !== val)
}

function reveal() {
  for (let i = 0; i < char.value.length; i++) {
    if (revealed.value.includes(char.value[i])) continue
    revealed.value.push(char.value[i])
  }
  char.value = ""
}

function addSong() {
  songs.value.push({name: "", bonus: false, guessed:false})
}

function encrypt(val: string, index: number) {
  if (opened.value.includes(index)) return val
  const ed = val.replace(/./g, (v) => {
    if (v == " ") return " "
    return revealed.value.includes(v) ? v : "*"
  })
  if (ed == val) return [val, true]
  return [ed, false]
}

function isAsciiOnly(val: string) {
  return /^[\x00-\x7F]*$/.test(val)
}

function guess(val: string) {
  const x = songs.value.indexOf(songs.value.find(v => v.name == val) as any)
  if (x >= 0) songs.value[x].guessed = true
}
function unguess(val:string) {
  const x = songs.value.indexOf(songs.value.find(v => v.name == val) as any)
  if (x >= 0) songs.value[x].guessed = false
}
function deleteSong(val: string) {
  if (!edit.value) return
  const x = songs.value.indexOf(songs.value.find(v => v.name == val) as any)
  if (x >= 0) songs.value.splice(x, 1)
}
</script>

<template>
  <div class="wrapper">
    <div class="opened-w">
      <div class="func">
        <label>
          <button class="btn-reveal" @click="reveal" style="width: 1.6rem;">开</button>
          <input v-model="char" placeholder="输入！" class="inp" @keydown.enter="reveal">
        </label>
        <label>
          <input type="checkbox" class="checkbox" v-model="close">
          关字符模式：{{ close ? "开启" : "关闭" }}（点击字母以关闭）
        </label>

          <label>
            <input type="checkbox" v-model="edit">
            编辑模式
          </label>
          <label>
            <input type="checkbox" v-model="show" :disabled="edit">
            显示答案
          </label>
          <button class="btn-reveal" @click="addSong">加一首</button>

      </div>
    </div>
    <div class="line-w">
      <div class="content">
        <div style="text-wrap: nowrap">已开（含空格）：</div>
        <div class="list">
          <div class="char" v-for="str in revealed"
               :class="close ? 'char-close':''" @click="() => unreveal(str)">{{ str }}
          </div>
        </div>
      </div>

      <div class="slst">
        <div v-for="(value, index) of songs" class="song-line">
          <button class="btn-reveal" @click="() => deleteSong(value.name)" v-if="edit">删</button>
          <button class="btn-reveal" @click="() => guess(value.name)" v-else-if="!value.guessed">开</button>
          <button class="btn-reveal" @click="() => unguess(value.name)" v-else>关</button>

          <div class="ascii"><span v-if="!isAsciiOnly(value.name)">含非Ascii</span></div>
          <div>
            <label :class="songs[index].bonus ? 'bonus' : ''" v-if="edit">
              Bonus:
              <input type="checkbox" v-model="songs[index].bonus">
            </label>
            <span v-if="value.bonus && !edit" class="bonus">Bonus!</span>
          </div>
          <div v-html="index"/>
          <input class="song-input" type="text" v-model="songs[index].name" v-if="edit" placeholder="输入歌名">
          <span v-else-if="value.guessed" class="green song-name">{{value.name}}</span>
          <span v-else class="song-name" :class="encrypt(value.name,index)[1] ? 'green' : ''">
            {{ encrypt(value.name, index)[0] }}
          </span>

        </div>
      </div>
    </div>

  </div>
</template>
<style scoped>
.bonus {
  color: yellow;
  animation: a-rainbow-text 2s linear infinite;
}

.song-input {
  border: none;
  border-bottom: 1px solid #b8dcee;
  outline: none;
  background-color: transparent;
  margin-left: 4px;
  color: white;
  margin-bottom: 3px;
}

.song-name {
  text-align: left;
}

.song-line {
  display: grid;
  grid-template-columns: 1.6em 6em 5em 20px 1fr;
  text-align: center;
  margin: 1px;
}

.line-w {
  width: 100%;
  border: #b8dcee 2px solid;
  border-radius: 5px;
  min-height: 15px;
  box-sizing: border-box;
  padding: 15px;
}

.l-func {
  display: flex;
  flex-direction: row;
  border-bottom: #b8dcee 1px solid;
  padding-bottom: 3px;
  gap: 15px;
}

.slst {
  display: flex;
  flex-direction: column;
  gap: 3px;
  padding: 5px;
  margin-top: 15px;
}
</style>

<style scoped>
* {
  user-select: none;
}
.green {
  color: #47bb49;
}

.wrapper {
  position: relative;
  width: 80%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 25px;
  left: 10%;
}

.opened-w {
  position: relative;
  width: 100%;
  padding: 15px;
  border: 2px #b8dcee solid;
  border-radius: 5px;
  box-sizing: border-box;
}

.content {
  width: 100%;
  display: flex;
  flex-direction: row;
  border-bottom: 1px solid #b8dcee;
  padding-bottom: 3px;
  margin-bottom: 3px;
}

.list {
  display: flex;
  width: 100%;
}

.char {
  color: white;
  width: 1em;
  transition: 0.2s all linear;
  text-align: center;
}

.char-close {
  color: black;
  background: #7cdcf4;
  cursor: pointer;
  width: 1.5em;
  font-weight: bold;
}

.func {
  width: 100%;
}

.btn-reveal {
  background: transparent;
  border: 2px solid #b8dcee;
  border-radius: 3px;
  cursor: pointer;
  color: #b8dcee;
  text-align: center;
  padding: 1px 2px;
}

.inp:focus-visible {
  outline: none;
}

.inp {
  margin-left: 3px;
  border: none;
  border-bottom: #b8dcee 1px solid;
  background-color: transparent;
  max-width: 5em;
  color: #b8dcee;
  margin-right: 10px;
}

.checkbox {
  height: 1em;
  width: 1em;
  vertical-align: center;
}
</style>

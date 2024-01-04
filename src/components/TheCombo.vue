<template>
  <div>
    <input v-model="searchText" @input="searchTags" placeholder="Search or add a tag" />

    <div v-if="equalTags.length == 0 && searchText.length > 0">
      <ul class="tagList">
        <li class="neuTag" @click="newTag(searchText)">
          {{ searchText }}
        </li>
      </ul>
    </div>

    <div>
      <ul class="tagList" v-if="matchingTags.length > 0 && searchText.length > 0">
        <li class="altTag" v-for="(tag, index) in matchingTags" :key="index" @click="selectTag(tag)">
          {{ tag }}
        </li>
      </ul>
    </div>

    <div>
      <ul class="selectedTags">
        <li v-for="tag in myTags" :key="tag" @click="removeTag(tag)">{{ tag }}</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const tags = ref(['abcd', 'asdf', 'sdfg']); // Initial list of tags
const searchText = ref('');
const myTags = ref([]);

const matchingTags = computed(() => {
  let matching = tags.value.filter(tag => tag.includes(searchText.value));
  let filtered = matching.filter(element => !myTags.value.includes(element));
  return filtered;
});

const equalTags = computed(() => {
  let equalUnselected = tags.value.filter(tag => tag == searchText.value);
  let euqalSelected = myTags.value.filter(tag => tag == searchText.value);
  return equalUnselected.concat(euqalSelected);
});

function searchTags() {
  console.log("equalTags: " + equalTags.value);
  if (!tags.value.filter(element => element.includes(searchText.value))) {
    // wenn kein Tag den Suchtext enthält...
    tags.value.push(searchText.value);
    searchText.value = '';
  }
}

function selectTag(tag) {
  searchText.value = '';

  myTags.value.push(tag)
}

function newTag(tag) {
  console.log("tag: " + tag)
  tags.value.push(tag)
  myTags.value.push(tag)
  searchText.value = ''
}

function removeTag(tag) {
  var sodeli = myTags.value.filter(element => element !== tag)
  myTags.value = sodeli
}


</script>

<style scoped>
.neuTag {
  background-color: lightpink
}

.altTag {
  background-color: lightgreen;
}

.selectedTags li {
  display: inline-block;
  margin-right: 1em;
  background-color: lightblue
}

.tagList {
  display: inline-block;
  width: 50%;
  list-style-type: none;
  padding-left: 0;
  margin-top: 0;
  margin-bottom: 0
}

.selectedTags {
  list-style-type: none;
  padding-left: 0
}
</style>
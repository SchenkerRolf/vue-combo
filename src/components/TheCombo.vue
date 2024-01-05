<template>
  <div>
    <input 
      v-model="searchText" 
      @input="searchTags" 
      @keydown="handleKeydown" 
      placeholder="Search or add a tag" 
    />

    <div>
      <ul 
        class="tagList"
        v-if="showOptions"
      >
        <li 
          class="neuTag"
          @click="newTag(searchText)"
          v-if="equalTags.length == 0 && searchText.length > 0"
          :class="selectedOption(-1)"
        >
          <span class="tag">
            <span class="tagCat">neu</span>
            <span class="tagDot">.</span>
            <span class="tagTag">{{ searchText }}</span>
          </span>
        </li>

        <li 
          class="altTag" 
          v-for="(tag, index) in matchingTags" :key="index" 
          @click="selectTag(tag)"
          :class="selectedOption(index)"
        >
          <span class="tag">
            <span class="tagCat">{{ tag.cat }}</span>
            <span class="tagDot">.</span>
            <span class="tagTag">{{ tag.tag }}</span>
          </span>
        </li>
      </ul>
    </div>

    <div>
      <ul class="selectedTags">
        <li 
          v-for="tag in myTags" 
          :key="tag"
          @click="removeTag(tag)"
        >{{ tag.cat }}.{{ tag.tag }}</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const tags = ref([{ cat: 'prj', tag: 'abcd' }, { cat: 'lala', tag: 'asdf' }, { cat: 'abc', tag: 'sdfg' }]); // Initial list of tags
const searchText = ref('');
const myTags = ref([]);
const selectedIndex = ref(-1);
const showOptions = ref(true);

const matchingTags = computed(() => {
  if (searchText.value.length > 0) {
    // tag includes search text
    let matching = tags.value.filter(
      tag => (tag.cat + "." + tag.tag).includes(searchText.value)
    );
    // tag is not selected
    let filtered = []
    if (myTags.value.length > 0) {
      filtered = matching.filter(
        a => !myTags.value.some(b => (a.cat === b.cat) & (a.tag === b.tag))
      )
    } else {
      filtered = matching
    }
    return filtered.sort((a, b) => b.cat - a.cat)
  } else {
    return [];
  }
});

const equalTags = computed(() => {
  // array contains 
  let equalUnselected = tags.value.filter(
    tag => (tag.cat + "." + tag.tag) == searchText.value || (tag.cat + "." + tag.tag) == "neu." + searchText.value
  );
  let equalSelected = myTags.value.filter(
    tag => (tag.cat + "." + tag.tag) == searchText.value || (tag.cat + "." + tag.tag) == "neu." + searchText.value
  );
  return equalUnselected.concat(equalSelected);
});

const optionsLength = computed(() => {
  if (equalTags.length == 0 && searchText.length > 0) {
    return matchingTags.value.length
  } else {
    return matchingTags.value.length + 1
  }
})

function searchTags() {
  showOptions.value = true
}

function selectTag(tag) {
  searchText.value = '';

  myTags.value.push(tag)
  selectedIndex.value = -1
}

function newTag(tag) {
  var newTagObj = { "cat": "neu", "tag": searchText.value }
  tags.value.push(newTagObj)
  myTags.value.push(newTagObj)
  searchText.value = ''
  selectedIndex.value = -1
}

function removeTag(tag) {
  var sodeli = myTags.value.filter(element => (element.cat !== tag.cat || element.tag !== tag.tag))
  myTags.value = sodeli
}

function handleKeydown(event) {
  const key = event.key;
  if (key === 'ArrowUp') {
    selectedIndex.value--;
    if (selectedIndex.value < 0) {
      selectedIndex.value = optionsLength.value - 1;
    }
  } else if (key === 'ArrowDown') {
    selectedIndex.value++;
    if (selectedIndex.value >= optionsLength.value) {
      selectedIndex.value = 0;
    }
  } else if (key === 'Enter') {
    if (equalTags.value.length == 0 && searchText.value.length > 0) { // newTag vorhanden
      if (selectedIndex.value > 0) {
        selectTag(matchingTags.value[selectedIndex.value - 1]);
      } else {
        newTag(searchText.value)
      }
    } else {
      selectTag(matchingTags.value[selectedIndex.value]);
    }
  } else if (key === 'Escape') {
    showOptions.value = !showOptions.value;
  }
}

function selectedOption(index) {

  if (equalTags.value.length == 0 && searchText.value.length > 0) {
    if (selectedIndex.value == index + 1) {
      return "selected"
    }
  } else {
    if (selectedIndex.value == index) {
      return "selected"
    }
  }
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

.selected {
  background-color: #ccc;
}
</style>

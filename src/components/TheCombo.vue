<template>
  <div>
    <input 
      v-model="searchText" 
      @input="searchTags" 
      @keydown="handleKeydown" 
      placeholder="Search or add a tag" 
    />

    <div class="tagListContainer">
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
        >
          <span class="tag">
            <span class="tagCat">{{ tag.cat }}</span>
            <span class="tagDot">.</span>
            <span class="tagTag">{{ tag.tag }}</span>
            <span class="tagDelete">x</span>
          </span>
        </li>
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


// list of existing tags to display
// (an eventual new tag is not part of this list)
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
    filtered.sort((a, b) => {
      if (a.cat !== b.cat) {
        return a.cat.localeCompare(b.cat)
      } else {
        return a.tag.localeCompare(b.tag)
      }
    })
    return filtered
  } else {
    return [];
  }
});


// is this tag already selected?
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


// how many options should be displayed?
const optionsLength = computed(() => {
  if (equalTags.length == 0 && searchText.length > 0) {
    return matchingTags.value.length
  } else {
    return matchingTags.value.length + 1
  }
})


// when text in search field changes
function searchTags() {
  showOptions.value = true
}


// existing tag is selected
function selectTag(tag) {
  searchText.value = '';

  myTags.value.push(tag)
  selectedIndex.value = -1
}


// new tag is selected
function newTag(tag) {
  var newTagObj = { "cat": "neu", "tag": searchText.value }
  tags.value.push(newTagObj)
  myTags.value.push(newTagObj)
  searchText.value = ''
  selectedIndex.value = -1
}


// selected tag should be removed
function removeTag(tag) {
  var sodeli = myTags.value.filter(element => (element.cat !== tag.cat || element.tag !== tag.tag))
  myTags.value = sodeli
}


// keyboard control
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


// class for selected option in list
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
.selectedTags li .tag {
  display: inline-block;
  margin-right: 1em;
}

.tagListContainer {
  height: 21ex;
}
.tagList {
  display: inline-block;
  width: 50%;
  list-style-type: none;
  padding-left: 0;
  margin-top: 1em;
  margin-bottom: 0;
  height: 20ex;
}

ul.tagList li {
  height: 5ex;
}

.tagList .tag {
    position: relative;
    top: 0.5em;  
}

.selectedTags {
  list-style-type: none;
  padding-left: 0
}

.selectedTags li {
  display: inline-block
}

.selected {
  background-color: #ccc;
}
.tag {
  cursor: pointer
}
.tagCat {
  background-color: lightpink;
  border-top-left-radius: 1em;
  border-bottom-left-radius: 1em;
  padding: 0.5em 0 0.5em 1em;
}
.tagDot {
  background-color: lightpink;
  padding: 0.5em 0;
}
.tagTag {
  background-color: lightgreen;
  border-top-right-radius: 1em;
  border-bottom-right-radius: 1em;
  padding: 0.5em 1em 0.5em 0;
}
ul.selectedTags li span.tag span.tagTag {
  padding: 0.5em 0.5em 0.5em 0;
  border-radius: 0;
}
.tagDelete {
  background-color: lightgrey;
  border-top-right-radius: 1em;
  border-bottom-right-radius: 1em;
  padding: 0.5em 1em 0.5em 0.5em;
}
</style>

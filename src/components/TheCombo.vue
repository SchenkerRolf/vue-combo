<template>
  <div>

    <input 
      v-model="searchText" 
      @input="searchTags" 
      @keydown="handleKeydown" 
      placeholder="Search or add a tag" 
      class="searchText"
      ref="searchRef"
    />

    <div
      class="tagListContainer"
    >
      <ul 
        class="tagList"
        v-if="showOptions && ((equalTags.length == 0 && searchText.length > 0) || matchingTags.length > 0)"
      >
        <li 
          class="neuTag"
          v-if="equalTags.length == 0 && searchText.length > 0"
          :class="selectedOption(-1)"
        >
          <span
            class="tag"
            @click="newTag(searchText)"
          >
            <span class="tagCat">neu</span>
            <span class="tagDot">.</span>
            <span class="tagTag">{{ searchText }}</span>
          </span>
        </li>

        <li 
          class="altTag" 
          v-for="(tag, index) in matchingTags" :key="index" 
          :class="selectedOption(index)"
        >
          <span 
            class="tag"
            @click="selectTag(tag)"
          >
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
import { ref, computed } from 'vue'

// Initial list of tags
const tags = ref([{ cat: 'prj', tag: 'abcd' }, { cat: 'prj', tag: 'eaa' }, { cat: 'prj', tag: 'aaa' }, { cat: 'lala', tag: 'asdf' }, { cat: 'abc', tag: 'sdfg' }])
const searchText = ref('')
const myTags = ref([])
const selectedIndex = ref(-1)
const showOptions = ref(true)


// list of existing tags to display
// (an eventual new tag is not part of this list)
const matchingTags = computed(() => {
  if (searchText.value.length > 0) {
    // tag includes search text
    let matching = tags.value.filter(
      tag => (tag.cat + "." + tag.tag).includes(searchText.value)
    )
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
    return []
  }
})


// is this tag already selected?
const equalTags = computed(() => {
  // array contains 
  let equalUnselected = tags.value.filter(
    tag => (tag.cat + "." + tag.tag) == searchText.value || (tag.cat + "." + tag.tag) == "neu." + searchText.value
  )
  let equalSelected = myTags.value.filter(
    tag => (tag.cat + "." + tag.tag) == searchText.value || (tag.cat + "." + tag.tag) == "neu." + searchText.value
  )
  return equalUnselected.concat(equalSelected)
})


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
  searchText.value = ''

  myTags.value.push(tag)
  selectedIndex.value = -1

  focusSearch()
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
  focusSearch()
}


// keyboard control
function handleKeydown(event) {
  const key = event.key
  if (key === 'ArrowUp') {
    selectedIndex.value--
    if (selectedIndex.value < 0) {
      selectedIndex.value = optionsLength.value - 1
    }
  } else if (key === 'ArrowDown') {
    selectedIndex.value++
    if (selectedIndex.value >= optionsLength.value) {
      selectedIndex.value = 0
    }
  } else if (key === 'Enter') {
    if (equalTags.value.length == 0 && searchText.value.length > 0) { // newTag vorhanden
      if (selectedIndex.value > 0) {
        selectTag(matchingTags.value[selectedIndex.value - 1])
      } else {
        newTag(searchText.value)
      }
    } else {
      selectTag(matchingTags.value[selectedIndex.value])
    }
  } else if (key === 'Escape') {
    showOptions.value = !showOptions.value
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


// set focus to search field
const searchRef = ref(null)

const focusSearch = () => {
  if (searchRef.value) {
    searchRef.value.focus()
  }
}

</script>

<style scoped>
.searchText {
  box-sizing: border-box;
  width: 150px;
}
.selectedTags li .tag {
  display: inline-block;
  margin-right: 1em;
}

.tagListContainer {
  height: 5ex;
  position: relative;
  z-index: 5;
}
.tagList {
  display: inline-block;
  list-style-type: none;
  padding: 0.5em;
  padding-top: 0.2em;
  margin: 0;
  margin-left: 2px;
  max-height: 20ex;
  overflow-y: auto;
  border-radius: 2px;
  box-sizing: border-box;
  width: 146px;
  background-color: white;
  box-shadow: 0 10px 16px 0 rgba(0,0,0,0.2), 0 6px 20px 0 rgba(0,0,0,0.19) !important;
  -webkit-user-select: none;
  -ms-user-select: none; 
  user-select: none; 
}

ul.tagList li {
  height: 4.1ex;
}

.tagList .tag {
    position: relative;
    top: 6px;  
}

.selectedTags {
  list-style-type: none;
  padding-left: 0;
  position: relative;
  z-index: 1;
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
  padding: 0.3em 0 0.3em 0.6em;
}
.neuTag .tagCat {
  background-color: purple;
  color: white
}
.tagDot {
  background-color: lightpink;
  padding: 0.3em 0;
}
.neuTag .tagDot {
  background-color: purple;
  color: white
}
.tagTag {
  background-color: lightgreen;
  border-top-right-radius: 1em;
  border-bottom-right-radius: 1em;
  padding: 0.3em 0.6em 0.3em 0;
}
.neuTag .tagTag {
  background-color: green;
  color: white
}
ul.selectedTags li span.tag span.tagTag {
  padding: 0.3em 0.3em 0.3em 0;
  border-radius: 0;
}
.tagDelete {
  background-color: lightgrey;
  border-top-right-radius: 1em;
  border-bottom-right-radius: 1em;
  padding: 0.3em 0.6em 0.3em 0.3em;
}
</style>

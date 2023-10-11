<template>
    <div class="min-w-screen min-h-screen bg-gray-50 flex items-center justify-center px-3 py-32">
        <div class="w-full max-w-lg relative" @click.outside="onBlur">
            <textarea class="w-full bg-white border-2 border-gray-300 shadow-lg px-3 py-2 rounded-lg focus:outline-none focus:border-indigo-500" placeholder="Type @ or # to trigger the mention" @input="onInput()" @keydown="onKeyDown" ref="input" v-html="textAreaText"></textarea>
    
  <div class="absolute z-30 "  :style="`top: ${caretPosition.top-5}px; left: ${caretPosition.left-10}px`" v-show="showPopover" >

<span class="absolute top-0 left-0 w-2 h-2 bg-white transform rotate-45 -mt-1 ml-3 border-gray-300 border-r border-b z-20"></span>
<div class="bg-white overflow-auto rounded-lg shadow-md z-10 py-2 border border-gray-300 text-gray-800 text-xs absolute bottom-full">
    <ul class="list-reset">
        <!-- v-if="!displayedItems.length" -->
        <div v-if="displayedItems.length == 0">
            <li>
                <span class="block px-4 py-1 text-gray-500 text-nowrap whitespace-nowrap">No results</span>
            </li>
        </div>
        <div v-if="displayedItems.length > 0">
        <!--  -->
            <div v-for="(item, index) in displayedItems" :key="index"> 
                <li @click.prevent="applyMention(index)" class="px-4 py-1 flex no-underline hover:no-underline transition-colors duration-100 text-nowrap whitespace-nowrap d-flex justify-content-between align-item-center" :class="selectedIndex == index ? 'bg-indigo-500 text-white' : 'hover:bg-gray-100'"  >
                    <div style="width: 45px; height: 45px">
                        <img src="./public/images/Image_created_with_a_mobile_phone.png" class="w-full h-full rounded-full" alt="image"/>
                    </div>
                    <div  class="font-bold ml-2" v-text="item.label"></div>
                </li>
            </div>
        </div>
    </ul>
</div>
</div>
       
        </div>
    </div>
</template>
<script setup>
   const keys = [ '@'];
//    const keyDataVal = ref({});
   const users = [
            {
                label: "Rodolfo Kidd",
                value: "rodolfo.kidd",
                img: "Image_created_with_a_mobile_phone.png"
            },
            {
                label: "Sheldon Lindsey",
                value: "sheldonlindsey43",
                img: "pexels-andrea-piacquadio-774909.jpg"
            },
            {
                label: "Adan Best",
                value: "adan_best",
                img: "pexels-andrea-piacquadio-3779760.jpg"
            },
            {
                label: "Rosemary Hurley",
                value: "rosemaryhurley22",
                img: "pexels-jack-winbow-1559486.jpg"
            },
            {
                label: "Allyson Livingston",
                value: "_allyson_livingston_",
                img: "pexels-masha-raymers-2726111.jpg"
            },
            {
                label: "Carolina Gray",
                value: "carolinagray",
                img: "pexels-matheus-bertelli-1906157.jpg"
            },
            {
                label: "Howard Tran",
                value: "howardtran923",
                img: "pexels-pixabay-220453 (1).jpg"
            },
        ];
const items = ref([]);
const placement = ref('top-start');
        const omitKey = ref(false);
        const filteringDisabled = ref(false);
       const  insertSpace = ref(false);
        const mapInsert = ref(null);
        const limit = ref(4);
        const key = ref(null);
       const keyIndex = ref(null);
        const oldKey = ref(null);
        const searchText = ref(null);
        const caretPosition = ref({top: 0, left: 0});
        const selectedIndex = ref(0);
        const input = ref(null);
        const showPopover = ref(false);
        const displayedItems = ref([]);
        const cancelKeyCode = ref(null);
        const cancelKeyUp = ref(null);
        const textAreaText = ref('');
        const mentionUsers = ref([]);

      function  onBlur() {
          closeMenu();
      }
      function closeMenu () {
        if (key.value != null) {
                oldKey.value = key.value;
                showPopover.value = false;
                key.value = null;
            }
      }
      function onInput(e) {
         checkKey();
      } 
   
      function getSelectionStart() {
        return input.value.selectionStart;
      }
      function getLastSearchText(caretIndex, keyIndexInner) {
        if(keyIndexInner !== -1 ) {
            const searchTextInner = getValue().substring(keyIndexInner + 1, caretIndex)
                // If there is a space we close the menu
                if (!/\s/.test(searchTextInner)) {
                    return searchTextInner
                }
            }
            return null;
      }
      function regexMentionUsers  () {
        const regex = /@[^ ]+/g;
const mentions = input.value.value.match(regex);
console.log(mentionUsers.value)
if(mentionUsers.value.length > 0 ) {
  if(!mentions) {
   return  mentionUsers.value = [];
  }
  return mentionUsers.value = mentionUsers.value.filter(user =>
  mentions.includes(`@${user.value}`)
);
}
      }
      const checkKey = () => {
        regexMentionUsers();
        const index = getSelectionStart();
            if (index >= 0) {
                const { keyInner, keyIndexInner } = getLastKeyBeforeCaret(index);
                
                const searchTextInner = getLastSearchText(index, keyIndexInner)
                if (!(keyIndexInner < 1 || /\s/.test(getValue()[keyIndexInner - 1]))) {
                    return false
                }
                if (searchTextInner != null) {
                    openMenu(keyInner, keyIndexInner);
                    searchText.value = searchTextInner;
                    updateDisplayedItems();
                    return true
                }
            }
            closeMenu();
            return false
      }
      function updateDisplayedItems() {
        return displayedItems.value = filteredItems().slice(0,limit.value);
      }
      function filteredItems() {
        if (!searchText.value || filteringDisabled.value) {
            return items.value;
      }
      const searchTextInner = searchText.value.toLowerCase()
          return items.value.filter(item => {
            let text = '';
            if (item.searchText) {
              text = item.searchText
            } else if (item.label) {
              text = item.label
            } else {
              text = ''
              for (const key in item) {
                text += item[key]
              }
            }
            return text.toLowerCase().includes(searchTextInner)
          })
    }
      function openMenu(keyInner, keyIndexInner) {
            if (key.value !== keyInner) {
                key.value = keyInner
                keyIndex.value = keyIndexInner;
                updateCaretPosition()
                selectedIndex.value = 0
                showPopover.value = true;
                const usersNotInMention = users.filter(user => !mentionUsers.value.find(mentionUser => mentionUser.label === user.label));

                items.value = key.value === '@' ? usersNotInMention : [] ;
            } 
        }

        function updateCaretPosition() {
            if(key.value) {
                caretPosition.value = getCaretPosition(input.value, keyIndex.value);
            }
        }
         function getCaretPosition(element, position) {
            var mirrorDiv, computed, style;
            var properties = [
                'boxSizing',
                'width', // on Chrome and IE, exclude the scrollbar, so the mirror div wraps exactly as the textarea does
                'height',
                'overflowX',
                'overflowY', // copy the scrollbar for IE

                'borderTopWidth',
                'borderRightWidth',
                'borderBottomWidth',
                'borderLeftWidth',

                'paddingTop',
                'paddingRight',
                'paddingBottom',
                'paddingLeft',

                // https://developer.mozilla.org/en-US/docs/Web/CSS/font
                'fontStyle',
                'fontVariant',
                'fontWeight',
                'fontStretch',
                'fontSize',
                'lineHeight',
                'fontFamily',

                'textAlign',
                'textTransform',
                'textIndent',
                'textDecoration', // might not make a difference, but better be safe

                'letterSpacing',
                'wordSpacing'
            ];
            // mirrored div
            mirrorDiv = document.getElementById(element.nodeName + '--mirror-div');

            if (!mirrorDiv) {
                mirrorDiv = document.createElement('div');
                mirrorDiv.id = element.nodeName + '--mirror-div';
                document.body.appendChild(mirrorDiv);
            }

            style = mirrorDiv.style;
            computed = getComputedStyle(element);

            // default textarea styles
            style.whiteSpace = 'pre-wrap';
            if (element.nodeName !== 'INPUT')
                style.wordWrap = 'break-word'; // only for textarea-s

            // position off-screen
            style.position = 'absolute'; // required to return coordinates properly
            style.top = element.offsetTop + parseInt(computed.borderTopWidth) + 'px';
            style.left = "400px";
            style.visibility = 'hidden'; // not 'display: none' because we want rendering

            // transfer the element's properties to the div
            properties.forEach(function(prop) {
                style[prop] = computed[prop];
            });
            let test= true;
            if (!test) {
                style.width = parseInt(computed.width) - 2 + 'px' // Firefox adds 2 pixels to the padding - https://bugzilla.mozilla.org/show_bug.cgi?id=753662
                // Firefox lies about the overflow property for textareas: https://bugzilla.mozilla.org/show_bug.cgi?id=984275
                if (element.scrollHeight > parseInt(computed.height))
                    style.overflowY = 'scroll';
            } else {
                style.overflow = 'hidden'; // for Chrome to not render a scrollbar; IE keeps overflowY = 'scroll'
            }

            mirrorDiv.textContent = element.value.substring(0, position);
            // the second special handling for input type="text" vs textarea: spaces need to be replaced with non-breaking spaces - http://stackoverflow.com/a/13402035/1269037
            if (element.nodeName === 'INPUT')
                mirrorDiv.textContent = mirrorDiv.textContent.replace(/\s/g, "\u00a0");

            var span = document.createElement('span');
         
            span.textContent = element.value.substring(position) || '.'; // || because a completely empty faux span doesn't render at all
            mirrorDiv.appendChild(span);

            var coordinates = {
                top: span.offsetTop + parseInt(computed['borderTopWidth']) - element.scrollTop,
                left: span.offsetLeft + parseInt(computed['borderLeftWidth']),
            };
            return coordinates;
        }
      function getValue () {
        return input.value.value;
      }
      function getLastKeyBeforeCaret (index) {
        const [keyData] =keys.map(keyInner => (
            {
                keyInner,
                keyIndexInner: getValue().lastIndexOf(keyInner, index - 1),
            })).sort((a, b) => b.keyIndexInner - a.keyIndexInner)
            return keyData
      }

      function onKeyDown(e) {
        if (key.value) {
                if( keys.includes(e.key) ) {
                    return cancelEvent(e);
                }
                updateDisplayedItems();
                if (e.key === 'ArrowDown' || e.keyCode === 40) {
                    selectedIndex.value++;
                    if (selectedIndex.value >= displayedItems.value.length) {
                        selectedIndex.value = 0
                    }
                    cancelEvent(e)
                }
                if (e.key === 'ArrowUp' || e.keyCode === 38) {
                    selectedIndex.value-- ;
                    if (selectedIndex.value < 0) {
                        selectedIndex.value = displayedItems.value.length - 1
                    }
                    cancelEvent(e);
                }
                if ((e.key === 'Enter' || e.key === 'Tab' || e.keyCode === 13 || e.keyCode === 9) &&
                    displayedItems.value.length > 0) {
                    applyMention(selectedIndex.value)
                    cancelEvent(e)
                }
                if (e.key === 'Escape' || e.keyCode === 27) {
                    closeMenu()
                    cancelEvent(e)
                }
            }
      }
      function applyMention(itemIndex) {
        const item = displayedItems.value[itemIndex]
      mentionUsers.value.push(item);
            const value = (omitKey.value ? '' : key.value || '') + String(mapInsert.value ? mapInsert.value(item, key.value) : item.value) + (insertSpace.value ? ' ' : '')
            setValue(replaceText(getValue(), searchText.value, value, keyIndex.value))
            setCaretPosition(keyIndex.value + value.length);
            closeMenu()
        }
        import './node_modules/highlight-in-textarea/src/highlight-in-textarea.css'
        import './node_modules/highlight-in-textarea/src/highlight-in-textarea.js'
        function setValue(value) {
            input.value.value = value+ ' '
            const regex = /@[^ ]+/g;
          const mentions = input.value.value.match(regex);
          
mentions.forEach(word => {
    const startIndex = input.value.value.indexOf(word);
    if (startIndex !== -1) {
        const endIndex = startIndex + word.length;
        input.value.focus();
        input.value.setSelectionRange(startIndex, endIndex);
    }
});
        }
        function setCaretPosition(index) {
                input.value.selectionEnd = index
        }
        function replaceText(text, searchText, newText, index) {
            return text.slice(0, index) + newText + text.slice(index + searchText.length + 1, text.length)
        }
        
      function cancelEvent(e) {
        e.preventDefault()
            e.stopPropagation()
            cancelKeyUp.value = e.key
            cancelKeyCode.value = e.keyCode
      }
</script>

```vue
<template>
  <div>
    <input
      ref="input"
      v-model="input"
      @input="onInput"
      @blur="onBlur"
      @keydown="onKeyDown"
      @keyup="onKeyUp"
      @scroll="onScroll"
    />
    <div
      v-if="showPopover"
      :style="{ top: caretPosition.top + 'px', left: caretPosition.left + 'px' }"
    >
      <ul>
        <li
          v-for="(item, index) in displayedItems"
          :key="index"
          :class="{ selected: index === selectedIndex }"
          @click="applyMention(index)"
        >
          {{ item.label }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      keys: ['#', '@'],
      users: [
        {
          label: "Rodolfo Kidd",
          value: "rodolfo.kidd",
        },
        {
          label: "Sheldon Lindsey",
          value: "sheldonlindsey43",
        },
        {
          label: "Adan Best",
          value: "adan_best",
        },
        {
          label: "Rosemary Hurley",
          value: "rosemaryhurley22",
        },
        {
          label: "Allyson Livingston",
          value: "_allyson_livingston_",
        },
        {
          label: "Carolina Gray",
          value: "carolinagray",
        },
        {
          label: "Howard Tran",
          value: "howardtran923",
        },
      ],
      tags: [
        {
          label: "alpine",
          value: "alpine",
        },
        {
          label: "alpinestars",
          value: "alpinestars",
        },
        {
          label: "alpinebabes",
          value: "alpinebabes",
        },
        {
          label: "alpinestar",
          value: "alpinestar",
        },
        {
          label: "alpinelake",
          value: "alpinelake",
        },
        {
          label: "alpinewhite",
          value: "alpinewhite",
        },
        {
          label: "alpineclimbing",
          value: "alpineclimbing",
        },
        {
          label: "alpineloop",
          value: "alpineloop",
        },
        {
          label: "alpinebreak",
          value: "alpinebreak",
        },
        {
          label: "alpineskiing",
          value: "alpineskiing",
        },
        {
          label: "alpinea110",
          value: "alpinea110",
        },
        {
          label: "alpineski",
          value: "alpineski",
        },
        {
          label: "alpinelakeswilderness",
          value: "alpinelakeswilderness",
        },
        {
          label: "alpinevillage",
          value: "alpinevillage",
        },
        {
          label: "alpinevogue",
          value: "alpinevogue",
        },
      ],
      items: [],
      placement: 'top-start',
      omitKey: false,
      filteringDisabled: false,
      insertSpace: false,
      mapInsert: null,
      limit: 4,
      key: null,
      oldKey: null,
      searchText: null,
      caretPosition: {
        top: 0,
        left: 0,
      },
      selectedIndex: 0,
      input: null,
      showPopover: false,
    };
  },
  computed: {
    filteredItems() {
      if (!this.searchText || this.filteringDisabled) {
        return this.items;
      }
      const searchText = this.searchText.toLowerCase();
      return this.items.filter((item) => {
        let text;
        if (item.searchText) {
          text = item.searchText;
        } else if (item.label) {
          text = item.label;
        } else {
          text = '';
          for (const key in item) {
            text += item[key];
          }
        }
        return text.toLowerCase().includes(searchText);
      });
    },
    displayedItems() {
      this.selectedIndex = 0;
      return this.filteredItems.slice(0, this.limit);
    },
  },
  methods: {
    isIe() {
      const userAgent = typeof window !== 'undefined' ? window.navigator.userAgent : '';
      return userAgent.indexOf('MSIE ') !== -1 || userAgent.indexOf('Trident/') !== -1;
    },
    isFirefox() {
      return !(window.mozInnerScreenX == null);
    },
    setInput(el) {
      this.input = el;
    },
    getCaretPosition(element, position) {
      var mirrorDiv, computed, style;
      var properties = [
        'boxSizing',
        'width',
        'height',
        'overflowX',
        'overflowY',
        'borderTopWidth',
        'borderRightWidth',
        'borderBottomWidth',
        'borderLeftWidth',
        'paddingTop',
        'paddingRight',
        'paddingBottom',
        'paddingLeft',
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
        'textDecoration',
        'letterSpacing',
        'wordSpacing',
      ];
      mirrorDiv = document.getElementById(element.nodeName + '--mirror-div');
      if (!mirrorDiv) {
        mirrorDiv = document.createElement('div');
        mirrorDiv.id = element.nodeName + '--mirror-div';
        document.body.appendChild(mirrorDiv);
      }
      style = mirrorDiv.style;
      computed = getComputedStyle(element);
      style.whiteSpace = 'pre-wrap';
      if (element.nodeName !== 'INPUT') style.wordWrap = 'break-word';
      style.position = 'absolute';
      style.top = element.offsetTop + parseInt(computed.borderTopWidth) + 'px';
      style.left = "400px";
      style.visibility = 'hidden';
      properties.forEach(function (prop) {
        style[prop] = computed[prop];
      });
      if (this.isFirefox()) {
        style.width = parseInt(computed.width) - 2 + 'px';
        if (element.scrollHeight > parseInt(computed.height)) style.overflowY = 'scroll';
      } else {
        style.overflow = 'hidden';
      }
      mirrorDiv.textContent = element.value.substring(0, position);
      if (element.nodeName === 'INPUT') mirrorDiv.textContent = mirrorDiv.textContent.replace(/\s/g, "\u00a0");
      var span = document.createElement('span');
      span.textContent = element.value.substring(position) || '.';
      mirrorDiv.appendChild(span);
      var coordinates = {
        top: span.offsetTop + parseInt(computed['borderTopWidth']) - element.scrollTop,
        left: span.offsetLeft + parseInt(computed['borderLeftWidth']),
      };
      return coordinates;
    },
    onInput(e) {
      this.checkKey();
    },
    onBlur() {
      this.closeMenu();
    },
    onKeyDown(e) {
      if (this.key) {
        if (this.keys.includes(e.key)) {
          return this.cancelEvent(e);
        }
        this.updateDisplayedItems();
        if (e.key === 'ArrowDown' || e.keyCode === 40) {
          this.selectedIndex++;
          if (this.selectedIndex >= this.displayedItems.length) {
            this.selectedIndex = 0;
          }
          this.cancelEvent(e);
        }
        if (e.key === 'ArrowUp' || e.keyCode === 38) {
          this.selectedIndex--;
          if (this.selectedIndex < 0) {
            this.selectedIndex = this.displayedItems.length - 1;
          }
          this.cancelEvent(e);
        }
        if ((e.key === 'Enter' || e.key === 'Tab' || e.keyCode === 13 || e.keyCode === 9) &&
          this.displayedItems.length > 0) {
          this.applyMention(this.selectedIndex);
          this.cancelEvent(e);
        }
        if (e.key === 'Escape' || e.keyCode === 27) {
          this.closeMenu();
          this.cancelEvent(e);
        }
      }
    },
    onKeyUp(e) {
      if (this.cancelKeyUp && (e.key === this.cancelKeyUp || e.keyCode === this.cancelKeyCode)) {
        this.cancelEvent(e);
      }
      this.cancelKeyUp = null;
      this.cancelKeyCode = null;
    },
    cancelEvent(e) {
      e.preventDefault();
      e.stopPropagation();
      this.cancelKeyUp = e.key;
      this.cancelKeyCode = e.keyCode;
    },
    onScroll() {
      this.updateCaretPosition();
    },
    getSelectionStart() {
      return this.input.selectionStart;
    },
    setCaretPosition(index) {
      this.$nextTick(() => {
        this.input.selectionEnd = index;
      });
    },
    getValue() {
      return this.input.value;
    },
    setValue(value) {
      this.input.value = value;
    },
    checkKey() {
      const index = this.getSelectionStart();
      if (index >= 0) {
        const { key, keyIndex } = this.getLastKeyBeforeCaret(index);
        const searchText = this.lastSearchText = this.getLastSearchText(index, keyIndex);
        if (!(keyIndex < 1 || /\s/.test(this.getValue()[keyIndex - 1]))) {
          return false;
        }
        if (searchText != null) {
          this.openMenu(key, keyIndex);
          this.searchText = searchText;
          this.updateDisplayedItems();
          return true;
        }
      }
      this.closeMenu();
      return false;
    },
    getLastKeyBeforeCaret(caretIndex) {
      const [keyData] = this.keys.map(key => ({
        key,
        keyIndex: this.getValue().lastIndexOf(key, caretIndex - 1),
      })).sort((a, b) => b.keyIndex - a.keyIndex);
      return keyData;
    },
    getLastSearchText(caretIndex, keyIndex) {
      if (keyIndex !== -1) {
        const searchText = this.getValue().substring(keyIndex + 1, caretIndex);
        if (!/\s/.test(searchText)) {
          return searchText;
        }
      }
      return null;
    },
    openMenu(key, keyIndex) {
      if (this.key !== key) {
        this.key = key;
        this.keyIndex = keyIndex;
        this.updateCaretPosition();
        this.selectedIndex = 0;
        this.showPopover = true;
        this.items = this.key === '@' ? this.users : this.tags;
      }
    },
    closeMenu() {
      if (this.key != null) {
        this.oldKey = this.key;
        this.showPopover = false;
        this.key = null;
      }
    },
    updateCaretPosition() {
      if (this.key) {
        this.caretPosition = this.getCaretPosition(this.input, this.keyIndex);
      }
    },
    applyMention(itemIndex) {
      const item = this.displayedItems[itemIndex];
      const value = (this.omitKey ? '' : this.key || '') + String(this.mapInsert ? this.mapInsert(item, this.key) : item.value) + (this.insertSpace ? ' ' : '');
      this.setValue(this.replaceText(this.getValue(), this.searchText, value, this.keyIndex));
      this.setCaretPosition(this.keyIndex + value.length);
      this.closeMenu();
    },
    replaceText(text, searchText, newText, index) {
      return text.slice(0, index) + newText + text.slice(index + searchText.length + 1, text.length);
    },
  },
};
</script>
```
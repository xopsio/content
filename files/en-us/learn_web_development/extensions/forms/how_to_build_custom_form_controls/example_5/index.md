---
title: Example 5
slug: Learn_web_development/Extensions/Forms/How_to_build_custom_form_controls/Example_5
page-type: learn-module-chapter
sidebar: learnsidebar
---

This is the last example that explains [how to build custom form widgets](/en-US/docs/Learn_web_development/Extensions/Forms/How_to_build_custom_form_controls).

## Change states

### HTML

```html
<form class="no-widget">
  <select name="myFruit" aria-label="Fruit">
    <option>Cherry</option>
    <option>Lemon</option>
    <option>Banana</option>
    <option>Strawberry</option>
    <option>Apple</option>
  </select>

  <div
    class="select"
    role="combobox"
    aria-label="Fruit"
    aria-haspopup="listbox"
    aria-expanded="false">
    <span class="value">Cherry</span>
    <ul class="optList hidden" role="listbox">
      <li class="option" role="option" aria-selected="true">Cherry</li>
      <li class="option" role="option" aria-selected="false">Lemon</li>
      <li class="option" role="option" aria-selected="false">Banana</li>
      <li class="option" role="option" aria-selected="false">Strawberry</li>
      <li class="option" role="option" aria-selected="false">Apple</li>
    </ul>
  </div>
</form>
```

### CSS

```css
.widget select,
.no-widget .select {
  display: none;
}

/* --------------- */
/* Required Styles */
/* --------------- */

.select {
  position: relative;
  display: inline-block;
}

.select.active,
.select:focus {
  box-shadow: 0 0 3px 1px #227755;
  outline-color: transparent;
}

.select .optList {
  position: absolute;
  top: 100%;
  left: 0;
}

.select .optList.hidden {
  max-height: 0;
  visibility: hidden;
}

/* ------------ */
/* Fancy Styles */
/* ------------ */

.select {
  font-size: 0.625em; /* 10px */
  font-family: "Verdana", "Arial", sans-serif;

  box-sizing: border-box;

  padding: 0.1em 2.5em 0.2em 0.5em; /* 1px 25px 2px 5px */
  width: 10em; /* 100px */

  border: 0.2em solid black; /* 2px */
  border-radius: 0.4em; /* 4px */

  box-shadow: 0 0.1em 0.2em rgb(0 0 0 / 45%); /* 0 1px 2px */

  background: linear-gradient(0deg, #e3e3e3, #fcfcfc 50%, #f0f0f0);
}

.select .value {
  display: inline-block;
  width: 100%;
  overflow: hidden;

  white-space: nowrap;
  text-overflow: ellipsis;
  vertical-align: top;
}

.select::after {
  content: "▼";
  position: absolute;
  z-index: 1;
  height: 100%;
  width: 2em; /* 20px */
  top: 0;
  right: 0;

  padding-top: 0.1em;

  box-sizing: border-box;

  text-align: center;

  border-left: 0.2em solid black;
  border-radius: 0 0.1em 0.1em 0;

  background-color: black;
  color: white;
}

.select .optList {
  z-index: 2;

  list-style: none;
  margin: 0;
  padding: 0;

  background: #f0f0f0;
  border: 0.2em solid black;
  border-top-width: 0.1em;
  border-radius: 0 0 0.4em 0.4em;

  box-shadow: 0 0.2em 0.4em rgb(0 0 0 / 40%);

  box-sizing: border-box;

  min-width: 100%;
  max-height: 10em; /* 100px */
  overflow-y: auto;
  overflow-x: hidden;
}

.select .option {
  padding: 0.2em 0.3em;
}

.select .highlight {
  background: black;
  color: white;
}
```

### JavaScript

```js
// -------------------- //
// Function definitions //
// -------------------- //

function deactivateSelect(select) {
  if (!select.classList.contains("active")) {
    return;
  }

  const selectedOption = select.querySelectorAll(".option")[getIndex(select)];
  if (selectedOption) {
    highlightOption(select, selectedOption);
  }

  const optList = select.querySelector(".optList");

  optList.classList.add("hidden");
  select.classList.remove("active");
  select.setAttribute("aria-expanded", "false");
  select.removeAttribute("aria-activedescendant");
}

function deactivateOtherSelects(select, selectList) {
  selectList.forEach((other) => {
    if (other !== select) {
      deactivateSelect(other);
    }
  });
}

function toggleOptList(select) {
  const optList = select.querySelector(".optList");
  const willOpen = optList.classList.contains("hidden");

  if (!willOpen) {
    deactivateSelect(select);
    return;
  }

  optList.classList.remove("hidden");
  select.classList.add("active");
  select.setAttribute("aria-expanded", "true");

  const selected = select.querySelector('.option[aria-selected="true"]');
  if (selected) {
    select.setAttribute("aria-activedescendant", selected.id);
  }
}

function highlightOption(select, option) {
  const optionList = select.querySelectorAll(".option");

  optionList.forEach((other) => {
    other.classList.remove("highlight");
  });

  option.classList.add("highlight");

  if (select.getAttribute("aria-expanded") === "true") {
    select.setAttribute("aria-activedescendant", option.id);
  }
}

function updateValue(select, index) {
  const nativeWidget = select.previousElementSibling;
  const value = select.querySelector(".value");
  const optionList = select.querySelectorAll(".option");

  nativeWidget.selectedIndex = index;
  value.textContent = optionList[index].textContent;

  optionList.forEach((option, optionIndex) => {
    const isSelected = optionIndex === index;
    option.classList.toggle("highlight", isSelected);
    option.setAttribute("aria-selected", String(isSelected));

    if (isSelected) {
      if (select.getAttribute("aria-expanded") === "true") {
        select.setAttribute("aria-activedescendant", option.id);
      } else {
        select.removeAttribute("aria-activedescendant");
      }
    }
  });
}

function getIndex(select) {
  const nativeWidget = select.previousElementSibling;

  return nativeWidget.selectedIndex;
}

// This function returns the index of the currently active option in the listbox
// when the custom select is expanded. While expanded, hover and keyboard
// navigation can temporarily move `aria-activedescendant` away from the
// committed selection, so keyboard navigation should continue from the
// currently active highlighted option rather than from the committed
// selection. If the custom select is collapsed, or if the active descendant
// is missing or no longer matches an option, we fall back to the committed
// selection returned by `getIndex()`.
// It takes two parameters:
// select     : the DOM node with the class `select` related to the native control
// optionList : the list of options for the given custom control
function getActiveIndex(select, optionList) {
  if (select.getAttribute("aria-expanded") === "true") {
    const activeId = select.getAttribute("aria-activedescendant");
    const index = [...optionList].findIndex((option) => option.id === activeId);

    if (index !== -1) {
      return index;
    }
  }

  return getIndex(select);
}

// ------------- //
// Event binding //
// ------------- //

const form = document.querySelector("form");

const selectList = form.querySelectorAll(".select");

selectList.forEach((select, selectIndex) => {
  const optionList = select.querySelectorAll(".option");
  const selectedIndex = getIndex(select);

  select.tabIndex = 0;

  const optList = select.querySelector(".optList");
  const listboxId = `custom-select-${selectIndex}-listbox`;
  optList.id = listboxId;
  select.setAttribute("aria-controls", listboxId);

  optionList.forEach((option, optionIndex) => {
    option.id = `custom-select-${selectIndex}-option-${optionIndex}`;
  });

  updateValue(select, selectedIndex);

  optionList.forEach((option, index) => {
    option.addEventListener("mousedown", (event) => {
      event.preventDefault();
    });

    option.addEventListener("mouseover", () => {
      highlightOption(select, option);
    });

    option.addEventListener("click", (event) => {
      event.stopPropagation();
      updateValue(select, index);
      deactivateSelect(select);
      select.focus();
    });
  });

  select.addEventListener("click", () => {
    toggleOptList(select);
  });

  select.addEventListener("focus", () => {
    deactivateOtherSelects(select, selectList);
  });

  select.addEventListener("blur", () => {
    deactivateSelect(select);
  });

  select.addEventListener("keydown", (event) => {
    let index = getActiveIndex(select, optionList);

    switch (event.key) {
      case "ArrowDown":
        event.preventDefault();

        if (select.getAttribute("aria-expanded") !== "true") {
          toggleOptList(select);
          break;
        }

        if (index < optionList.length - 1) {
          index++;
          updateValue(select, index);
        }
        break;

      case "ArrowUp":
        event.preventDefault();

        if (select.getAttribute("aria-expanded") !== "true") {
          toggleOptList(select);
          break;
        }

        if (index > 0) {
          index--;
          updateValue(select, index);
        }
        break;

      case "Home":
        event.preventDefault();

        if (select.getAttribute("aria-expanded") !== "true") {
          toggleOptList(select);
          break;
        }

        updateValue(select, 0);
        break;

      case "End":
        event.preventDefault();

        if (select.getAttribute("aria-expanded") !== "true") {
          toggleOptList(select);
          break;
        }

        updateValue(select, optionList.length - 1);
        break;

      case "Enter":
      case " ":
        event.preventDefault();
        toggleOptList(select);
        break;

      case "Escape":
        event.preventDefault();
        deactivateSelect(select);
        break;
      default:
        // Ignore all other keys
        return;
    }
  });
});

form.classList.remove("no-widget");
form.classList.add("widget");
```

### Result

{{ EmbedLiveSample('Change_states') }}

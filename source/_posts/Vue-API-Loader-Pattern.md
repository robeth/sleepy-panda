---
title: API Call Pattern in Vue
date: 2020-08-22 20:41:23
tags:
  - Vue
  - API
  - Pattern
categories:
  - Frontend
---

## Overview

One powerful feature of modern frontend javascript framework such as React and Vue is the ability to create a custom component. A component could contains the HTML structure, style (CSS), and logic (javascript) that will be rendered to the webpage. Once component created, we could reuse it to implement feature faster with minimal maintenance.

## Use Case: API Call

In interactive web apps such as single page application (SPA), it's common that frontend communicate with backend for fetching data or submitting form using API/AJAX.

However, as API call requires some time to complete and not always return succesfully, we should consider several state and show it to user for great UX.

Typical API call state is illustrated below:

{% img /images/api-loader-state.png 400 '"API Call State" "api-call-state"'%}

1. Loading State: API call is initiated. Application show loading message
2. Success state: API call return success response. Application render the response
3. Error state: API call return error response. Application show error message and optionally present retry button. When pressed, return to "Loading State"

Demo:

{% img /images/api-loader-demo.gif 400 '"API Call Demo" "api-call-demo"'%}

## Without component

Implementing 3 states for 1 API call using basic Vue require some steps:

1. Setup `loading` state. Flag indicating the API call still active
2. Setup `error` state. Details of errors if API call is failed
3. Setup `responseData` state. Result if API call is success
4. Call API and update various state accordingly
5. Create conditional rendering based on loading, error, and success state.

```js
const app = Vue.createApp({
  setup() {
    // ...
    // Step 1, 2, 3
    const loading = Vue.ref(false);
    const state = Vue.reactive({ products: [], error: null });

    //Step 4
    function fetch() {
      loading.value = true;
      state.products = [];
      state.error = null;
      axios
        .get(url.value)
        .then((response) => {
          loading.value = false;
          state.products = response.data.products;
        })
        .catch((error) => {
          loading.value = false;
          state.error = error;
        });
    }
    //...
  },
});
```

```html
<div id="q-app">
  <!-- ... -->
  <div class="q-ma-md">
    <!-- Step 5: Conditional Rendering -->
    <div v-if="loading">Loading...</div>
    <div v-else-if="state.error">
      <!-- ... -->
    </div>
    <div v-else-if="state.products.length > 0">
      <div v-for="product in state.products" :key="product.id">
        <!-- .... -->
      </div>
    </div>
  </div>
</div>
```

Full source code: [Codepen](https://codepen.io/robet/pen/yLGNajg)

As the number of API call grow, we need an abstraction so maintenance and development process easier to manage.

## With component <api-loader>

Fortunately, we could encapsulate majority of API call state process using component, so developer could focus on what API to call and how to render it.

Compared to previous code, using component `<api-loader>` _cut the development step from 5 to 1.5_.

1. ~~Setup `loading` state. Flag indicating the API call still active~~
2. ~~Setup `error` state. Details of errors if API call is failed~~
3. ~~Setup `responseData` state. Result if API call is success~~
4. Call API ~~and update various state accordingly~~
5. Create conditional rendering based on loading, error, and success state.

Sample usage of `<api-loader>`:

```js
const app = Vue.createApp({
  setup() {
    //...

    // Step 4: Define API to be called
    function getProducts() {
      console.log('get products');
      return axios.get(url.value, {
        params: {
          limit: 5,
          skip: 0,
        },
      });
    }
    return {
      // ...,
      getProducts,
    };
  },
});
```

```html
<div id="q-app">
  <!-- ... -->
  <div class="q-ma-md">
    <!-- ... -->
    <!-- Step 4: Provide API to be called by component-->
    <api-loader ref="apiLoaderRef" :api-call="getProducts">
      <!-- Step 5: Conditional rendering using vue slot -->
      <template #success="slotProps">
        <div v-for="product in slotProps.responseData.products" :key="product.id">
          <!-- ... -->
        </div>
        <q-btn @click="slotProps.retry" icon="refresh" size="xs"></q-btn>
      </template>
    <api-loader />
  </div>
  <!-- ... -->
</div>
```

Full source code: [Codepen](https://codepen.io/robet/pen/ZEVGBrj?editors=1010)

## Further Usage

As we successfully create `<api-loader>` component, we could use that for more complex use case: paralel and sequential API call.

For paralel API call, we simply use multiple `<api-loader>`:

{% img /images/api-loader-parallel.png 400 '"Get profile and products at the same time" "parallel-api-call"'%}

```html
<!-- First API Call: Get User Profile -->
<api-loader ref="apiLoaderRef1" :api-call="getUserProfile">
  <template #success="slotProps">
    <!-- render user profile -->
  </template>
  <api-loader
/></api-loader>

<!-- Second API Call: Get Products -->
<api-loader ref="apiLoaderRef2" :api-call="getProducts">
  <template #success="slotProps">
    <!-- render products -->
  </template>
  <api-loader
/></api-loader>
```

For sequential API call, we could use nested `<api-loader>`:

{% img /images/api-loader-sequential.png 400 '"Get all users then relevant message" "Sequential-api-call"'%}

```html
<!-- First API Call: Get Users -->
<api-loader ref="apiLoaderRef1" :api-call="getUsers">
  <template #success="slotProps">
    <!-- Render Users -->
    <!-- Second API Call: Get Chat of the first user -->
    <api-loader ref="apiLoaderRef2" :api-call="getChats">
      <template #success="slotProps">
        <!-- render chats with first user-->
      </template>
      <api-loader
    /></api-loader>
  </template>
  <api-loader
/></api-loader>
```

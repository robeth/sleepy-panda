---
title: Vue-API-Loader-Pattern
date: 2020-04-26 16:41:23
tags:
  - Vue
  - API
  - Pattern
categories:
  - Frontend
---

When part of your apps are doing expensive operation, such as fetching or sending data to API, you may need to show the user a loading state/indicator. By doing so, user will perceive your apps is responsive and running properly. Not only loading indicator, your apps should communicate whether the operation is success or fail properly.

In vue application, this is easily achieved by

1. Introducing a `loading` state. Set it before API call and clear it after.
2. Show any loading message/indicator when `loading` state active.
3. If API success show the result. Otherwise, inform the user why operation failed.
4. Optional: give user option to retry operation.

Imagine employee administration menu in a human resource application. We need to show list of registered employee which is retrieved from API. You could try and experiment this in [codepen]()

{% codeblock lang:html %}

<div id="app">
    <h3>Employee List</h3>
    <div v-if="!loading">
        <template v-if="success">
        <ol>
            <li v-for="employee in employees" :key="employee.id">
                { { employee.name } }
            </li>
        </ol>
        </template>
        <div v-else>
            Request Error! <button @click="fetchEmployees">Retry</button>
        </div>
    </div>
    <div v-else>
        Loading....
    </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue"></script>

<script>
new Vue({
  el: '#app',
  data: function () {
    return {
      loading: false,
      success: false,
      errorMessage: null,
      employees: null
    }
  },
  mounted() {
      this.fetchEmployees();
  },
  methods: {
      fetchEmployees: function() {
          this.loading = true;
          this.success = false;
          this.employees = null;

          axios.get('https://jsonplaceholder.typicode.com/users')
            .then(response => {
                this.loading = false;
                this.success = true;
                this.employees = response.data;
            })
            .catch(error => {
                this.loading = false;
                this.errorMessage = error.message;
            });
      }
  }
})
</script>

{% endcodeblock %}

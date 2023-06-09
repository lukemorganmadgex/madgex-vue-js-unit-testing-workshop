# Madgex FED Vue.js unit testing workshop

A tooling workshop for Madgex front-end developers. Looks at NPM Workspaces, Vue, Vite, Vitest, Vue Test Utils and Mock Service Worker.

## Lesson 1 - npm workspaces

1. run `npm init -w api`
2. run `npm i express -w api`
3. create an `index.js` file in `/api`
4. add the following to your `/api/index.js` to create a simple hello world express app

```js
const express = require("express");
const app = express();
const port = 3000;

const jobs = [
  {
    id: 1,
    title: "Web developer",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "25,000.00",
  },
  {
    id: 2,
    title: "Senior Web developer",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "35,000.00",
  },
  {
    id: 3,
    title: "Web designer",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "25,000.00",
  },
  {
    id: 4,
    title: "Senior Web designer",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "35,000.00",
  },
  {
    id: 5,
    title: "QA Tester",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "25,000.00",
  },
];

app.get("/", (req, res) => {
  res.send("Hello World!");
});

app.get("/api/jobs", (req, res) => {
  if (req.query["jobCount"]) {
    const jobSlice = jobs.slice(0, parseInt(req.query["jobCount"], 10));
    res.json({
      status: "success",
      data: jobSlice,
    });
  }

  res.json({
    status: "success",
    data: jobs,
  });
});

app.listen(port, () => {
  console.log(`Example app listening on port ${port}`);
});
```

5. in `/api/package.json` define a start command

```json
  "scripts": {
    "start": "node index.js"
  },
```

6. in monorepo root, add a command to start the api `npm run start -w api`

```json
  "scripts": {
      "start:api": "npm run start -w api"
  },
```

## Lesson 2 - vite + vue

[vue](https://vuejs.org/)
[vite](https://vitejs.dev/)

1. Create a vue project using: `npm create vite@latest`. Add the directory to our workspaces in `package.json`

```json
 "workspaces": [
    "vite-project",
    "api"
  ],
```

2. define an proxy in our vite config file for our api domain so that we can make requests using '/api'

add the following to the `/vite-project/vite.config.js`

```js
server: {
    proxy: {
      '/api': {
        target: '"http://localhost:3000"',
        changeOrigin: true,
      },
    },
  },
```

3. define an alias so that `@` resolves to the `/src` of our `/vite-project/src`

add the following to the `/vite-project/vite.config.js`

```js
import { fileURLToPath, URL } from "node:url";
```

```js
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),
    },
  },
```

4. Add a script to the monorepo root to start our vite project dev-server

```json
"scripts": {
    "dev:vite-project": "npm run dev -w vite-project",
  },
```

## Lesson 3 - vitest & vue test utils

[vitest](https://vitest.dev/)
[vue test utils](https://test-utils.vuejs.org/)

1. install vitest into the package you want to test `npm i -D vitest -w vite-project`.

2. install `jsdom` so we can mock a browser environment `npm i -D jsdom -w vite-project`.

3. Add a test block to your `vite.config.js`

```js
test: {
    environment: 'jsdom',
  },
```

4. install vue test utils `npm i -D @vue/test-utils -w vite-project`.

5. create a test spec file that will live next to our component `HelloWorld.spec.js`.

```js
import { describe, it, expect } from "vitest";
import { mount } from "@vue/test-utils";
import HelloWorld from "./HelloWorld.vue";
```

6. write a unit test that will render the title in a h1 tag.

[mounting a vue context](https://test-utils.vuejs.org/api/#mount)
[finding elements in the DOM](https://test-utils.vuejs.org/api/#find)
[writing a test statement](https://vitest.dev/api/expect.html)

```js
describe("Hello World", () => {
  it("Displays the title in a H1 tag", () => {
    const hello = mount(HelloWorld, {
      props: {
        msg: "Unit test",
      },
    });

    const title = hello.find("h1");
    expect(title.text()).toBe("Unit test");
  });
});
```

7. write an npm script that will run all our tests for us.

```json
"scripts": {
    "test:vite-project": "npm run test -w vite-project",
  },
```

8. write a unit test that will trigger the button click and check the value of the `count` state ref.

9. modify your `HelloWorld.vue` so that the interpolated count is in a span with an id we can target in a test.

```html
<button type="button" @click="count++">
  count is <span id="counter-text">{{ count }}</span>
</button>
```

10. write a unit test that will ensure the text inside the span with the id `counter-text` upates.

```javascript
import { mount, flushPromises } from "@vue/test-utils";
```

## Lesson 4 - Mock service worker

1. create an async hello world component that will speak to our api from lesson 01

```bash
npm i axios -w vite-project
```

```vue
<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const jobCount = ref(3);
const jobs = ref([]);

async function getJobs() {
  const res = await axios.get(`/api/jobs?jobCount=${jobCount.value}`);
  if (res.status === 200) {
    jobs.value = res.data.data;
  }
}

onMounted(() => {
  getJobs();
});
</script>

<template>
  <form style="margin-bottom: 2rem;" @submit.prevent>
    <div
      style="margin: 0 auto; width: 300px; margin-bottom: 2rem; display: flex; flex-direction: column;"
    >
      <label for="jobs" style="margin-bottom: 1rem;"
        >How Many Jobs Do you Want to See?</label
      >
      <input type="number" name="jobs" id="jobs" v-model="jobCount" />
    </div>
    <button type="submit" @click="getJobs()">Get jobs</button>
  </form>
  <div class="results">
    <div class="job" v-for="(job, index) in jobs" :key="index">
      <h3>{{ job.title }}</h3>
      <p>{{ job.description }}</p>
      <small>salrary: £{{ job.salary }} gbp p/a</small>
    </div>
  </div>
</template>

<style scoped></style>
```

2. Lets see why we need mock service worker - write a unit test file to check that modifying the number input updates our view model.

```js
import { describe, it, expect } from "vitest";
import { mount, flushPromises } from "@vue/test-utils";
import AsyncHelloWorld from "./AsyncHelloWorld.vue";

describe("AsyncHelloWorld", () => {
  it("Updates the job count ref when the number input is changed", () => {
    const hello = mount(AsyncHelloWorld);
    const jobCountInput = hello.find('input[name="jobs"]');
    jobCountInput.setValue("4");
    jobCountInput.trigger("change");
    expect(hello.vm.jobCount).toBe(4);
  });
});
```

Temporarily console log out the html of the component inside that test we just wrote - you can see there are no job cards being rendered

```
console.log(hello.html());
```

3. install the mock service worker package in your vite project

```bash
npm i msw -D -w vite-project
```

4. create a `/test` directory sat adjacent to our `/src` and create an `index.js` file to live inside of your `/test` dir

5. in our setup file - lets init msw following the [vitest docs exmaple](https://vitest.dev/guide/mocking.html#configuration)

```js
import { beforeAll, afterAll, afterEach, vi } from "vitest";
import { setupServer } from "msw/node";

const server = setupServer();

// Start server before all tests
beforeAll(() => server.listen({ onUnhandledRequest: "error" }));

//  Close server after all tests
afterAll(() => server.close());

// Reset handlers after each test `important for test isolation`
afterEach(() => server.resetHandlers());
```

6. now we need to tell vitest to run that setup file before every test run. modify your `vite.config.js`

```js
test: {
    environment: 'jsdom',
    setupFiles: ['./test/index.js'],
  },
```

7. now lets mock an api response - create a folder inside `/test` called `/mocks`

8. inside `test/mocks` create a file called `api.js`

```js
import { rest } from "msw";

const BASE_URL = "/api";

const jobs = [
  {
    id: 1,
    title: "Web developer",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "25,000.00",
  },
  {
    id: 2,
    title: "Senior Web developer",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "35,000.00",
  },
  {
    id: 3,
    title: "Web designer",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "25,000.00",
  },
  {
    id: 4,
    title: "Senior Web designer",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "35,000.00",
  },
  {
    id: 5,
    title: "QA Tester",
    description:
      "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Cras quis tellus lorem. Nam convallis porta augue sit amet aliquet. Aenean.",
    salary: "25,000.00",
  },
];

export const apiRequests = [
  rest.get(`${BASE_URL}/jobs`, (req, res, ctx) => {
    const count = req.url.searchParams.get("jobCount");
    let data = count && count > 0 ? jobs.slice(0, parseInt(count, 10)) : jobs;
    return res(
      ctx.status(200),
      ctx.json({
        data,
      })
    );
  }),
];
```

9. now we need to bring in that handler function to our server in our setup file

```js
import { apiRequests } from "./mocks/api";

const server = setupServer(...apiRequests);
```

10. Write a unit test that renders a card for each job once the component has mounted

```js
it("renders a card for each job", async () => {
  const hello = mount(AsyncHelloWorld);
  await flushPromises();
  const jobCards = hello.findAll(".job");
  expect(jobCards.length).toBe(3);
});
```

## Lesson 05 - Spies

[mocking and spying](https://vitest.dev/guide/mocking.html#functions)

1. Import the vi object from vitest

```js
import { describe, it, expect, vi } from "vitest";
```

2. Write a unit test that lets us spy on the `getJobs` function

```js
it("calls the getJobs fn when the button is click", async () => {
  const hello = mount(AsyncHelloWorld);
  const spy = vi.spyOn(hello.vm, "getJobs");
  const button = hello.find("button");
  await button.trigger("click");
  await flushPromises();
  expect(spy).toHaveBeenCalled();
});
```

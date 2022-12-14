<script setup>
import { computed } from "vue";
import Fuse from "fuse.js";

/**
 * busStops have key values that are numerical which stands for the bus stop ID,
 * each ID is an array with * 3 numerical key values which are lat, lon and name of bus stop.
 */
import busStops from "../assets/formattedData.json";
import { useRouter } from "vue-router";
import { useStore } from "../stores/counter";

const router = useRouter();
const store = useStore();

/**
 * Original way of implementing autocomplete search function before importing fuse.
 * Search results of the bus stop name, will be a list of bus stop objects
 */
// const results = computed(function () {

/**
 * Lowercase the query once to use for search, rather than calling
 * the `toLowerCase` method repeatedly.
 */
// const queryString = store.query.toLowerCase();
// const listOfBustStopsThatMatch = [];
// for (const busStop of busStops)
//   if (
//     busStop.Name.toLowerCase().includes(queryString) ||
//     busStop.ID.includes(queryString)
//   ) {
//     listOfBustStopsThatMatch.push(busStop);
//     // Return the array early if there is already 10 matches or more
//     if (listOfBustStopsThatMatch.length >= 10)
//       return listOfBustStopsThatMatch;
//   }
// return listOfBustStopsThatMatch;
// });

// Options to make search results more accurate and return less results if the search is more specific.
const options = {
  threshold: 0.3,
  distance: 20,
  includeMatches: true,
  includeScore: true,
  keys: ["ID", "Name"],
};

const fuse = new Fuse(busStops, options);

const results = computed(() => {
  return fuse.search(store.query, { limit: 10 });
});

const selectResult = (result) => {
  // Originlly the router push method was using path, that way is wrong as it does not go to the timing page but we should use the name instead like what we used in the router.js file.

  // Redirect to the Timing view and pass results as busStopID prop
  router.push({
    name: "Timing",
    params: { busStopID: result.ID, busStopName: result.Name },
  });
};

// Gets the road name base on lat and lon location. What I need now is one call that gets me all of the road names in order of the json files so i can add them in altogether.

async function testApi() {
  for (const busStop of busStops) {
    const street = await fetch(
      `https://developers.onemap.sg/privateapi/commonsvc/revgeocode?location=${busStop.Latitude},${busStop.Longitude}&token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOjk2NDUsInVzZXJfaWQiOjk2NDUsImVtYWlsIjoiemhlbmdzaGFvYmluMDBAZ21haWwuY29tIiwiZm9yZXZlciI6ZmFsc2UsImlzcyI6Imh0dHA6XC9cL29tMi5kZmUub25lbWFwLnNnXC9hcGlcL3YyXC91c2VyXC9zZXNzaW9uIiwiaWF0IjoxNjcyOTEzMDM3LCJleHAiOjE2NzMzNDUwMzcsIm5iZiI6MTY3MjkxMzAzNywianRpIjoiOTNmNjY2MGQ4Mzc0ZmUyYWYzYjI1NjA2Y2U3Mzk1OWYifQ.OgQfeSMoIyDPclqCKjpG58y8Ga1CKIdY5RWrIab0Dzo
`
    ).then((res) => res.json());
    console.log(street);
  }
}

const requestBody = JSON.stringify(busStops);

console.log(requestBody);
</script>

<template>
  <div class="field has-addons has-addons-centered">
    <router-link :to="{ name: 'home' }" class="icon pt-4 pr-5">
      <i class="fa fa-arrow-left" aria-hidden="true"></i>
    </router-link>

    <!--
          Binding this input element to a reactive property called query. When the user
          types into the input field, the computed function will listen for changes in the query,
          which fetches a list of suggestions based on the current value of query. The suggestions 
          are then displayed in a list below the input field.
        -->
    <div class="control has-icons-left has-icons-right">
      <input
        class="input"
        type="text"
        placeholder="Search buses and bus stops..."
        v-model="store.query"
        style="width: 400px"
      />
      <span class="icon is-left">
        <i class="fas fa-search"></i>
      </span>
      <!-- Show x icon only if query has value. -->
      <span v-if="store.query !== ''" class="icon is-right">
        <i @click="store.clearInput" class="fa-regular fa-x"></i>
      </span>
    </div>
  </div>

  <div class="columns is-mobile">
    <div class="column">
      <button class="button">Bus Stops</button>
    </div>
    <div class="column">
      <button class="button">Bus Service only</button>
    </div>
    <div class="column">
      <button class="button">Filter by smth else</button>
    </div>
  </div>

  <!-- 
      Only show the list of items if the query is not an empty string, which will change if
      there is input in the search box. To ensure the list of results is usable, we want the
      user to click on one of the result and show that value as the chosen one. 
    -->
  <div v-if="store.query !== ''" class="container">
    <p class="header is-5 has-text-centered">Bus Stop Name</p>

    <!-- On the left side of the "in" keyword, it contains the result element of the results aray 
      which is on the right side. What I did not look at was the shape of the results array. 
      The output was in an 'item' which contains the data I need (ID & Name). So without thinking I went
      to give the results array a dot item which should be remembered forever as something I should 
      never do again. To access the values in an object we use dot notation. Since the ID & name data 
      is stored in the item key, we can simply just .item the result element to access it. -->
    <div
      class="columns is-mobile has-background-link-light mb-4 mx-1"
      v-for="(result, i) in results"
      :key="i"
      @click="selectResult(result.item)"
    >
      <div class="column is-4">
        <p class="title is-4">{{ result.item.ID }}</p>
      </div>
      <div class="column">
        <p class="title is-5">{{ result.item.Name }}</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.fa-regular {
  pointer-events: all;
  cursor: pointer;
}

.button {
  width: 100%;
}
</style>

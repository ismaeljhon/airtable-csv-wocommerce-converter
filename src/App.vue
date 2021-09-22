<template>
  <div id="app">
    <vue-csv-import v-model="jsonData" :map-fields="mapFields">
      <vue-csv-toggle-headers></vue-csv-toggle-headers>
      <vue-csv-errors></vue-csv-errors>
      <vue-csv-input></vue-csv-input>
      <vue-csv-map :auto-match="false"></vue-csv-map>
    </vue-csv-import>
    <div v-if="forWordpressJsonData.length">
      <hr />
      <download-csv :data="forWordpressJsonData">
        <button>Download Data</button>
      </download-csv>
      <hr />
      <table border="1">
        <thead>
          <tr>
            <th v-for="(column, index) in mapFields" :key="index">
              {{ column }}
            </th>
            <th>Tags</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in forWordpressJsonData" :key="index">
            <td>{{ item.sku }}</td>
            <td>{{ item.name }}</td>
            <td>{{ item.description }}</td>
            <td>
              <ul>
                <li v-for="imageName in item.images" :key="imageName">
                  {{ imageName }}
                </li>
              </ul>
            </td>
            <td>{{ item.colors }}</td>
            <td>{{ item.categories }}</td>
            <td>{{ item.regular_price }}</td>
            <td>{{ item.wholesale_price }}</td>
            <td>{{ item.tags }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { VueCsvImport } from "vue-csv-import";
import JsonCSV from "vue-json-csv";
import _startCase from "lodash.startcase";

export default {
  name: "App",
  components: {
    VueCsvImport,
    downloadCsv: JsonCSV,
  },
  watch: {
    jsonData(data) {
      const forWordpressJsonData = [];

      data.forEach((item) => {
        const {
          name = "",
          colors = "",
          images = "",
          regular_price = "",
          wholesale_price = "",
        } = item;

        if (!name) return;

        const colorArray = colors.split(",");

        const updateItem = {
          ...item,
          images: images.split(","),
          regular_price: regular_price.replace(/\$/g, ""),
          wholesale_price: wholesale_price.replace(/\$/g, ""),
        };

        if (!colorArray) {
          forWordpressJsonData.push(updateItem);
          return;
        }

        colorArray.forEach((colorName) => {
          const beautifyColorName = _startCase(colorName.toLowerCase());

          forWordpressJsonData.push({
            ...updateItem,
            name: `${name.replace(/\./g, "")} - ${beautifyColorName}`,
            tags: beautifyColorName,
          });
        });
      });

      this.forWordpressJsonData = forWordpressJsonData;
    },
  },
  data: () => ({
    jsonData: null,
    forWordpressJsonData: [],
    mapFields: [
      "sku",
      "name",
      "description",
      "images",
      "colors",
      "categories",
      "regular_price",
      "wholesale_price",
    ],
  }),
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

<template>
  <div class="q-pa-md">
    <q-checkbox v-model="checked">Checked</q-checkbox>

    <q-table
      :title="title"
      :rows="rows"
      :columns="columns"
      row-key="name"
      :filter="filter"
      :filter-method="filterMethod"
      :rows-per-page-options="rowsPerPageOptions"
    >
      <template v-slot:top-right>
        <q-input
          borderless
          dense
          debounce="300"
          v-model="search"
          placeholder="Search"
        >
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </template>
    </q-table>
  </div>
</template>

<script>
import { defineComponent, ref } from "vue";
// import EssentialLink from "components/EssentialLink.vue";
import { faker } from "@faker-js/faker";

const columns = [
  {
    name: "desc",
    required: true,
    label: "Dessert (100g serving)",
    align: "left",
    field: (row) => row.name,
    format: (val) => `${val}`,
    sortable: true,
  },
  { name: "checked", label: "Checked", field: "checked" },
  {
    name: "calories",
    align: "center",
    label: "Calories",
    field: "calories",
    sortable: true,
  },
  { name: "fat", label: "Fat (g)", field: "fat", sortable: true },
  { name: "carbs", label: "Carbs (g)", field: "carbs" },
  { name: "random1", label: "random1", field: "random1" },
  { name: "random2", label: "random2", field: "random2" },
  { name: "random3", label: "random3", field: "random3" },
  { name: "random4", label: "random4", field: "random4" },
];

const rows = [
  {
    name: "Frozen Yogurt",
    checked: false,
    calories: 159,
    fat: 6.0,
    carbs: 24,
    random1: 1,
    random2: 2,
    random3: 3,
    random4: 4,
  },
];

export default defineComponent({
  name: "MainLayout",

  components: {},
  data() {
    return {
      checked: false,
      rowsPerPageOptions: [0],
      search: "",
      columns: columns,
      rows: rows,
      filteredRows: [],
    };
  },
  setup() {
    for (var n = 0; n < 5000; ++n) {
      rows.push({
        name: faker.name.fullName(),
        calories: Math.floor(Math.random() * 10),
        checked: Math.random() > 0.5,
        fat: Math.floor(Math.random() * 10),
        carbs: Math.floor(Math.random() * 10),
        random1: faker.name.fullName(),
        random2: faker.name.fullName(),
        random3: faker.name.fullName(),
        random4: faker.name.fullName(),
      });
    }

    return {};
  },
  computed: {
    title() {
      return `Title ${this.filteredRows.length}`;
    },
    filter() {
      return {
        checked: this.checked,
        search: this.search,
      };
    },
  },
  methods: {
    filterMethod(rows, terms, cols, getCellValue) {
      let filteredRows = rows;
      if (terms.checked) {
        filteredRows = rows.filter((row) => row.checked == terms.checked);
      }
      if (terms.search) {
        filteredRows = rows.filter((row) => row.name.includes(terms.search));
      }
      this.filteredRows = filteredRows;
      return filteredRows;
    },
  },
});
</script>

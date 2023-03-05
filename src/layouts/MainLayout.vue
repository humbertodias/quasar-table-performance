<template>
  <div class="q-pa-md">
    Page size
    <input type="number" v-model="pageSize" />
    <q-checkbox v-model="grid">Grid</q-checkbox>
    <br />
    <q-checkbox
      v-model="checkedTmp"
      @update:model-value="
        (value, evt) =>
          updateModelAfterTransitionEnd(
            value,
            evt,
            () => (this.checked = value)
          )
      "
      >Checked Async</q-checkbox
    >
    <q-checkbox v-model="checked">Checked Sync</q-checkbox>

    <q-table
      :title="title"
      :rows="rows"
      :columns="columns"
      row-key="id"
      :filter="filter"
      :filter-method="filterMethod"
      :rows-per-page-options="rowsPerPageOptions"
      selection="multiple"
      v-model:selected="selected"
      ref="myTable"
      :grid="grid"
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
// import { getCurrentInstance } from "vue";

const columns = [
  {
    name: "id",
    required: true,
    field: "id",
    label: "id",
  },
  {
    name: "desc",
    required: true,
    label: "Dessert (100g serving)",
    align: "left",
    field: (row) => row.name,
    format: (val) => `${val}`,
    sortable: true,
  },
  {
    name: "checked",
    label: "Checked",
    field: (row) => row.checked,
    format: (val) => `${val}`,
    sortable: true,
  },
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
    id: 0,
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
      grid: true,
      checked: false,
      checkedTmp: false,
      rowsPerPageOptions: [0],
      search: "",
      columns: columns,
      rows: rows,
      selected: [],
      n: 0,
      refChebox: "",
      totalPage: 0,
      pageSize: 100,
    };
  },
  setup() {
    for (var n = 1; n < 5000; ++n) {
      rows.push({
        id: n,
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
  created() {
    this.$bus.on("some-event", this.changeChecked);
  },
  mounted() {
    this.handleInfiniteScroll("myTable");
  },
  beforeUnmount() {
    this.unHandleInfiniteScroll("myTable");
  },
  computed: {
    title() {
      return `Treats ${this.rows.length}`;
    },
    filter() {
      return {
        checked: this.checked,
        search: this.search,
      };
    },
  },
  methods: {
    handleInfiniteScroll(tableRefName) {
      const self = this;
      window.addEventListener(
        "scroll",
        this.handleTotalPage(tableRefName, self)
      );
    },
    unHandleInfiniteScroll(tableRefName) {
      console.log("Main Vue destroyed");

      const self = this;
      window.removeEventListener(
        "scroll",
        this.handleTotalPage(tableRefName, self)
      );
    },
    handleTotalPage(tableRefName, self) {
      return () => {
        const ref = self.$refs[tableRefName];
        const selector = ref.grid ? ".q-table__grid-item" : "tr";
        const items = ref.$el.querySelectorAll(selector);

        if (items?.length > 0) {
          const lastItem = items[items.length - 1];
          if (self.isInViewport(lastItem)) self.totalPage++;
        }
      };
    },
    isInViewport(element) {
      const rect = element.getBoundingClientRect();
      return (
        rect.top >= 0 &&
        rect.left >= 0 &&
        rect.bottom <=
          (window.innerHeight || document.documentElement.clientHeight) &&
        rect.right <=
          (window.innerWidth || document.documentElement.clientWidth)
      );
    },
    addObersable(tableRefname) {
      this.$nextTick(function () {
        const options = {
          root: null,
          rootMargin: "15px",
          threshold: 1.0,
        };

        const observer = new IntersectionObserver((entries, observer) => {
          entries.forEach((entry) => {
            if (entry.isIntersecting) {
              this.totalPage++;
              console.log(entry.target);
              observer.unobserve(entry.target);
            }
          });
        }, options);
        const ref = this.$refs[tableRefname];
        const selector = ref.grid ? ".q-table__grid-item" : "tr";
        const items = ref.$el.querySelectorAll(selector);
        if (items?.length > 1) observer.observe(items[items.length - 1]);
      });
    },
    filterMethod(rows, terms, cols, getCellValue) {
      console.log("filterMethod");
      // this.addObersable("myTable");
      if (terms.checked) {
        return this.pagination(
          rows.filter((row) => row.checked == terms.checked)
        );
      }
      if (terms.search) {
        return this.pagination(
          rows.filter((row) => this.hasRowValue(row, terms.search))
        );
      }
      return this.pagination(rows);
    },

    pagination(rows) {
      return rows.slice(0, (this.totalPage + 1) * this.pageSize);
    },

    hasRowValue(row, search) {
      const values = Object.values(row);
      const lowerSearch = ("" + search).toLocaleUpperCase();
      for (const value of values) {
        const lowerValue = ("" + value).toLocaleUpperCase();
        if (lowerValue.includes(lowerSearch)) return true;
      }
      return false;
    },
    updateModelAfterTransitionEnd(value, evt, callback) {
      console.log("updateModelAfterTransitionEnd", value, evt);
      const fn = (event) => {
        console.log("Transition ended");
        event.currentTarget.removeEventListener("transitionend", fn);
        callback();
      };
      evt.currentTarget.addEventListener("transitionend", fn, { once: true });
    },
  },
});
</script>

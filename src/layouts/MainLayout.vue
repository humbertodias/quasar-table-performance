<template>
  <div class="q-pa-md">
    Page size
    <input type="number" v-model="pageSize" />
    Total page {{ this.totalPage }}
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
      style="height: 500px"
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
<style></style>
<script>
import { defineComponent, ref } from "vue";
// import EssentialLink from "components/EssentialLink.vue";
import { faker } from "@faker-js/faker";
import { date } from "quasar";

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
    name: "hour",
    label: "Hour",
    field: (row) => row.hour,
    format: (val) => `${val}`,
    sortable: true,
  },
  {
    name: "time",
    label: "Time",
    field: (row) => row.time,
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
const rows = [];

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
      pageSize: 10,
      rowsPaginationCount: 0,
    };
  },
  setup() {
    return {};
  },
  watch: {
    grid: function (value) {
      this.handleInfiniteScroll();
    },
  },
  created() {
    for (var n = 1; n <= 5000; ++n) {
      rows.push(this.createRow(n));
    }
  },
  mounted() {
    this.handleInfiniteScroll("myTable", this.$refs.myTable.$el);
  },
  computed: {
    title() {
      return `Treats ${this.rowsPaginationCount}/${this.rows.length}`;
    },
    filter() {
      return {
        checked: this.checked,
        search: this.search,
      };
    },
  },
  methods: {
    createRow(n) {
      const createdAt = Date.now();
      return {
        id: n,
        name: faker.name.fullName(),
        calories: Math.floor(Math.random() * 10),
        checked: Math.random() > 0.5,
        createdAt: createdAt,
        hour: date.formatDate(createdAt, "YYYY-MM-DD"),
        time: date.formatDate(createdAt, "HH:mm:ss.SSSZ"),
        fat: Math.floor(Math.random() * 10),
        carbs: Math.floor(Math.random() * 10),
        random1: faker.name.fullName(),
        random2: faker.name.fullName(),
        random3: faker.name.fullName(),
        random4: faker.name.fullName(),
      };
    },
    handleInfiniteScroll() {
      console.log("handleInfiniteScroll");
      this.$nextTick(() => {
        const qtableScrollElem = document.getElementsByClassName(
          "q-table__middle scroll"
        );
        const elementToScroll =
          qtableScrollElem.length > 0 ? qtableScrollElem[0] : window;

        const fnAddScroll = (event) => {
          if (elementToScroll == window) {
            let documentHeight = document.body.scrollHeight;
            let currentScroll = window.scrollY + window.innerHeight;
            let rowHeight = 40;
            if (currentScroll + rowHeight > documentHeight) {
              console.log("[Grid] You are at the bottom!");
              this.totalPage++;
            }
          } else {
            const { scrollHeight, scrollTop, clientHeight } = event.target;
            if (Math.abs(scrollHeight - clientHeight - scrollTop) < 1) {
              console.log("[Table] You are at the bottom!");
              this.totalPage++;
            }
          }
        };
        console.log("current.elementToScroll", elementToScroll);
        window.removeEventListener("scroll", fnAddScroll);
        elementToScroll.removeEventListener("scroll", fnAddScroll);
        elementToScroll.addEventListener("scroll", fnAddScroll);
      });
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
      if (terms.search) {
        return this.pagination(
          rows
            .filter((row) => this.hasRowValue(row, terms.search))
            .filter((row) => row.checked == terms.checked)
        );
      }
      return this.pagination(
        rows.filter((row) => row.checked == terms.checked)
      );
    },
    pagination(rows) {
      const rowsPagination = rows.slice(
        0,
        (this.totalPage + 1) * this.pageSize
      );
      this.rowsPaginationCount = rowsPagination.length;
      return rowsPagination.sort(this.sortRowsByCheckedAndDate);
    },
    sortRowsByCheckedAndDate(a, b) {
      return (
        Number(a.checked) - Number(b.checked) ||
        new Date(a.createdAt) - new Date(b.createdAt)
      );
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

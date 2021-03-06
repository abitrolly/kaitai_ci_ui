<template>
  <table class="table">
    <thead>
      <tr>
        <th>Test \ Target</th>
        <th v-for="(key, i) in filteredColumns" :key="i"
          @click="sortBy(key)"
          :class="{ active: sortKey == key }">
          {{ key.replace("/", " ") }}
          <span class="arrow" :class="sortOrders[key] > 0 ? 'asc' : 'dsc'">
          </span>
        </th>
      </tr>
      <tr>
        <th>Last update</th>
        <th v-for="(key, i) in filteredColumns" :key="i">
          {{ meta[key].timestamp | humanTime }}
          <a class="ext-link" v-if="meta[key].ci" :href="meta[key].ci.url">Job</a>
          <a class="ext-link" :href="meta[key].artifactsUrl">Artifacts</a>
        </th>
      </tr>
      <tr>
        <th>Rating</th>
        <th v-for="(key, i) in filteredColumns" :key="i">
          {{ passRating[key] }}%
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="entry in filteredData" :key="entry.name">
        <td>{{ entry.name }}</td>
        <ci-cell
          v-for="key in filteredColumns"
          :key="entry.name + '/' + key"
          :data="entry[key] || {}">
        </ci-cell>
      </tr>
    </tbody>
    <tfoot>
      <tr>
        <th>KST adoption</th>
        <th v-for="key in filteredColumns" :key="key">
          {{ kstRating[key] }}%
        </th>
      </tr>
    </tfoot>
  </table>
</template>

<script>
import CiCell from '@/components/CiCell.vue'

export default {
  name: 'CiGrid',
  components: {
    CiCell
  },
  props: {
    data: Array,
    columns: Array,
    meta: Object,
    filterKey: String,
    filterColumnsKey: String,
    skipPassed: Boolean
  },
  data: function () {
    var sortOrders = {}
    this.columns.forEach(function (key) {
      sortOrders[key] = 1
    })
    return {
      sortKey: 'name',
      sortOrders: sortOrders,
    }
  },
  computed: {
        filteredData: function () {
            var sortKey = this.sortKey
            var filterKey = this.filterKey && this.filterKey.toLowerCase()
            var order = this.sortOrders[sortKey] || 1
            var data = this.data
            if (filterKey) {
                data = data.filter(function (row) {
                    return Object.keys(row).some(function (key) {
                        return String(row[key]).toLowerCase().indexOf(filterKey) > -1
                    })
                })
            }
            if (this.skipPassed) {
                var columns = this.filteredColumns
                data = data.filter(function (row) {
                    return !(columns.every(function(key) {
                        return key == 'name' || (row[key] && row[key].status === 'passed');
                    }))
                })
            }
            if (sortKey) {
                data = data.slice().sort(function (a, b) {
                    a = a[sortKey]
                    b = b[sortKey]
                    return (a === b ? 0 : a > b ? 1 : -1) * order
                })
            }
            return data
    },
    filteredColumns: function() {
      if (this.filterColumnsKey && this.filterColumnsKey != '') {
        var key = this.filterColumnsKey.toLowerCase();
        return this.columns.filter((x) => x.toLowerCase().indexOf(key) > -1);
      } else {
        return this.columns;
      }
    },
    passRating: function () {
      var r = {};
      for (var pair in this.meta) {
        r[pair] = Math.round(this.meta[pair].passed / this.data.length * 100 * 10.0) / 10.0;
      }
      return r;
    },
    kstRating: function () {
      var r = {};
      for (var pair in this.meta) {
        r[pair] = Math.round(this.meta[pair].kst / this.data.length * 100 * 10.0) / 10.0;
      }
      return r;
    },
  },
  filters: {
    capitalize: function (str) {
      return str.charAt(0).toUpperCase() + str.slice(1)
    },
    humanTime: function (d) {
      var sec = (new Date() - d) / 1000;
      if (sec < 60) {
        return "just now";
      } else if (sec < 60 * 60) {
        return Math.round(sec / 60) + "m ago";
      } else if (sec < 24 * 60 * 60) {
        return Math.round(sec / (60 * 60)) + "h ago";
      } else {
        return Math.round(sec / (24 * 60 * 60)) + "d ago";
      }
    }
  },
  methods: {
    sortBy: function (key) {
      this.sortKey = key
      this.sortOrders[key] = this.sortOrders[key] * -1
    }
  }
}
</script>

<style scoped>
a.ext-link {
  display: block;
}
</style>

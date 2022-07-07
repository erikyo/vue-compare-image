<template>
  <input type="search" v-model="filter">
  <table>
    <thead>
    <tr>
      <th @click="sort('quality')">Quality 🔽</th>
      <th @click="sort('similarity')">Similarity 🔽</th>
      <th @click="sort('name')">Name 🔽</th>
      <th @click="sort('sizebyte')">Size 🔽</th>
      <th @click="sort('encodetime')">Time to encode 🔽</th>
    </tr>
    </thead>
    <tbody>
    <tr v-for="image in sortedImages()" v-bind:key="image.name">
      <td>{{ image.quality }}</td>
      <td>{{ parseFloat(image.similarity).toFixed(4) }}</td>
      <td>{{ image.type }} {{ image.quality }}%</td>
      <td>{{ image.filesize }}</td>
      <td>{{ parseFloat(image.encodetime).toFixed(4) }}</td>
    </tr>
    </tbody>
  </table>
  <p>
    <button @click="prevPage">Previous</button>
    <button @click="nextPage">Next</button>
  </p>

  sort={{currentSort}}, dir={{currentSortDir}}, page={{currentPage}}

</template>

<script>
export default {
  name: "CompareTable",
  props: ['tableImagesData'],
  data() {
    return {
      currentSort: 'name',
      currentSortDir: 'asc',
      pageSize: 20,
      currentPage: 1,
      filter: '',
      tableData: {}
    }
  },
  methods: {
    sort:function(s) {
      //if s == current sort, reverse
      if(s === this.currentSort) {
        this.currentSortDir = this.currentSortDir==='asc'?'desc':'asc';
      }
      this.currentSort = s;
      this.tableData = this.sortedImages();
    },
    nextPage:function() {
      if((this.currentPage*this.pageSize) < this.filteredImages.length) this.currentPage++;
    },
    prevPage:function() {
      if(this.currentPage > 1) this.currentPage--;
    },
    updateTable() {
      this.tableData = this.tableImagesData;
      this.tableData = this.sortedImages();
    },
    sortedImages() {
      return this.filteredImages.sort((a,b) => {
        let modifier = 1;
        if(this.currentSortDir === 'desc') modifier = -1;
        if(a[this.currentSort] < b[this.currentSort]) return -1 * modifier;
        if(a[this.currentSort] > b[this.currentSort]) return 1 * modifier;
        return 0;
      }).filter((row, index) => {
        let start = (this.currentPage-1)*this.pageSize;
        let end = this.currentPage*this.pageSize;
        if(index >= start && index < end) return true;
      });
    }
  },
  computed: {
    filteredImages() {
      return Object.values(this.tableImagesData).filter(image => {
        if(!this.filter) return true;
        return image.name.toLowerCase().indexOf(this.filter.toLowerCase()) >= 0;
      })
    },
  },
  watch: {
    filter() {
      this.currentPage = 1;
    },
    tableImagesData : {
      handler: function() {
        this.updateTable()
      }
    }
  },
}
</script>

<style lang="scss" scoped>

div#img-comparator-table {
  max-width: 850px;
  margin: auto;
}
div#img-comparator-table p{
  font-size: 14px;
  text-align: center;
}
div#img-comparator-table table th{
  font-weight: bold;
}

table {
  table-layout: fixed;
  border-collapse: collapse;
  tr {
    td, th {
      color:white;
      border: 1px solid #555;
      padding: 2px 8px;
    }
  }

  width: 850px;
  margin: auto;
}
</style>
<template>
  <div>
    <p style="font-size: 20px;"><strong>Location Table - Searched Places</strong></p>
    <div class="selectClass">
      <div class="leftButtons">
        <button @click="toggleSelectAll">Select All</button>
        <button @click="deleteSelectedLocations">Delete Selected Locations</button>
      </div>

      <div class="rightButtons">
        <button @click="prevPage">Previous</button>
        <span>Page {{ currentPage }} of {{ pagesCount }}</span>
        <button @click="nextPage">Next</button>
      </div>
    </div>

    <table class="locationTable">
      <thead>
        <tr>
          <th style="width: 5%;"></th>
          <th style="width: 15%;">Name</th>
          <th style="width: 15%;">Latitude</th>
          <th style="width: 15%;">Longitude</th>
          <th style="width: 20%;">Time Zone</th>
          <th style="width: 30%;">Local Time</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="location in displayedData" :key="location.name">
          <td><input type="checkbox" v-model="location.isSelected" /></td>
          <td>{{ location.name }}</td>
          <td>{{ location.latitude }}</td>
          <td>{{ location.longitude }}</td>
          <td>{{ location.timeZone }}</td>
          <td>{{ location.localTime }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  props: ['locations'],
  data() {
    return {
      selectAll: false,
      currentPage: 1,
      itemsPerPage: 10,
    };
  },
  computed: {
    displayedData() {
      const startIndex = (this.currentPage - 1) * this.itemsPerPage;
      const endIndex = startIndex + this.itemsPerPage;

      return this.locations.slice(startIndex, endIndex);
    },
    pagesCount() {
      const pages = Math.ceil(this.locations.length / this.itemsPerPage);

      if (pages == 0) {
        return 1;
      } else {
        return pages;
      }
    },
  },
  methods: {
    toggleSelectAll() {
      if (this.selectAll) {
        this.locations.forEach((location) => {
          location.isSelected = false;
        });
      } else {
        this.locations.forEach((location) => {
          location.isSelected = true;
        });
      }

      this.selectAll = !this.selectAll;
    },
    deleteSelectedLocations() {
      const selectedLocations = this.locations.filter((location) => location.isSelected);
      
      if (selectedLocations.length == 0) {
        alert("Select at lease one location for deletion.");
        return;
      } else {
        this.$emit('deleteLocations', selectedLocations);
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    nextPage() {
      if (this.currentPage < this.pagesCount) {
        this.currentPage++;
      }
    },
  },
};
</script>

<style scoped>
.selectClass {
  text-align: left;
  display: flex;
  justify-content: space-between;
 }

.leftButtons button {
  padding: 10px 20px;
  margin-bottom: 10px;  
  background-color: #f3260b;
  color: #fff;
  align-items: center;
 }

.rightButtons button {
  padding: 10px 10px;
  margin: 0 10px;
  background-color: #020202;
  color: #fff;
  align-items: center;
}

.locationTable {
  width: 100%;
  border-collapse: collapse;
  border-spacing: 20;
  text-align: left;
}

.location-table th,
.location-table td {
  border: 1px solid rgb(8, 35, 240);
  padding: 10px;
}
</style>

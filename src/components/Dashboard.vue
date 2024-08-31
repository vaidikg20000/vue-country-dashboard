<template>
  <div class="q-pa-md">
    <q-table
      flat
      bordered
      title="Countries"
      :rows="rows"
      :columns="columns"
      row-key="countryName"
      :filter="filter"
      :pagination="{ rowsPerPage: 15 }"
    >
      <template v-slot:top-right>
        <q-input
          borderless
          dense
          debounce="300"
          v-model="filter"
          placeholder="Search"
        >
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
        <q-btn
          color="primary"
          icon-right="archive"
          label="Export to csv"
          no-caps
          @click="exportTable"
        />
      </template>

      <template v-slot:header="props">
        <q-tr :props="props">
          <q-th auto-width />
          <q-th v-for="col in props.cols" :key="col.name" :props="props">
            {{ col.label }}
          </q-th>
        </q-tr>
      </template>

      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td auto-width>
            <q-btn
              size="sm"
              color="accent"
              round
              dense
              @click="props.expand = !props.expand"
              :icon="props.expand ? 'remove' : 'add'"
            />
          </q-td>
          <q-td v-for="col in props.cols" :key="col.name" :props="props">
            <img
              v-if="col.name === 'flagIcon'"
              :src="col.value"
              :alt="`Flag of ${props.row.countryName}`"
              style="width: 32px; height: auto"
            />
            <div v-else>
              {{ col.value }}
            </div>
          </q-td>
        </q-tr>
        <q-tr v-show="props.expand" :props="props">
          <q-td colspan="100%">
            <div class="row justify-center" style="height: 200px">
              <div
                class="col-12 col-md-6 col-sm-12 text-center rounded-borders"
              >
                <p class="text-weight-bold text-subtitle1">
                  Sub Region:
                  <span>{{
                    props.row.subRegion ? props.row.subRegion : "-"
                  }}</span>
                </p>
              </div>
              <div
                class="col-12 col-md-6 col-sm-12 text-center rounded-borders"
              >
                <p class="text-weight-bold text-subtitle1">
                  Official name:
                  <span v-for="name in props.row.officialName" :key="name">{{
                    name
                  }}</span>
                </p>
              </div>
              <div
                class="col-12 col-md-6 col-sm-12 text-center rounded-borders"
              >
                <p class="text-weight-bold text-subtitle1">
                  UN Membership:
                  <span>{{ props.row.unMember ? "YES" : "NO" }}</span>
                </p>
              </div>
              <div
                class="col-12 col-md-6 col-sm-12 text-center rounded-borders"
              >
                <p class="text-weight-bold text-subtitle1">
                  Border Countries:
                  <span v-for="border in props.row.borders" :key="border"
                    >{{ border }} ,</span
                  >
                </p>
              </div>
              <div
                class="col-12 col-md-6 col-sm-12 text-center rounded-borders"
              >
                <q-btn
                  label="Map"
                  color="primary"
                  @click="openModal(props.row.countryName)"
                />
                
              </div>
            </div>
          </q-td>
        </q-tr>
      </template>
    </q-table>
    

    <!-- Map Modal -->
    <div v-if="icon" class="modal-overlay">
      <div class="modal-content text-center">
        <iframe
            width="100%"
            height="600"
            :src="mapUrl"
            frameborder="0"
            scrolling="no"
            marginheight="0"
            marginwidth="0"
          ></iframe>
        <q-btn color="primary" @click="closeModal">Close Modal</q-btn>
      </div>
    </div>
    
    
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watchEffect } from "vue";
import { exportFile, useQuasar } from 'quasar'
// import.meta.env.VITE_GOOGLE_MAP_API_KEY

const rawCountries = ref([]);
const loading = ref(true);
const error = ref(null);
const filter = ref(""); // Add this line for the search filter
const icon = ref(false);
const currentMapUrl = ref("");
const mapUrl = computed(() => 
  `https://www.google.com/maps/embed/v1/place?key=${import.meta.env.VITE_GOOGLE_MAP_API_KEY}&q=${currentMapUrl.value}`
)



const formatCountryData = (country) => {
  return {
    countryName: country.name.common,
    capital: country.capital ? country.capital[0] : "N/A",
    currencyName: country.currencies
      ? Object.values(country.currencies)[0].name
      : "N/A",
    region: country.region,
    latitude: country.latlng[0],
    longitude: country.latlng[1],
    language: country.languages ? Object.values(country.languages)[0] : "N/A",
    population: country.population,
    flagIcon: country.flags.png,
    subRegion: country.subregion,
    borders: country.borders,
    officialName: country.name.official,
    map: country.maps.openStreetMaps,
    unMember: country.unMember,
  };
};

const countries = computed(() => rawCountries.value.map(formatCountryData));

const rows = computed(() => countries.value);


const openModal = (country) => {
  icon.value = true;
  currentMapUrl.value=country
};



// Function to close the modal
const closeModal = () => {
  icon.value = false
}

const columns = [
  {
    name: "countryName",
    required: true,
    label: "Country",
    align: "left",
    field: "countryName",
    sortable: true,
  },
  { name: "capital", label: "Capital", field: "capital", sortable: true },
  {
    name: "currencyName",
    label: "Currency",
    field: "currencyName",
    sortable: true,
  },
  { name: "region", label: "Region", field: "region", sortable: true },
  { name: "latitude", label: "Latitude", field: "latitude", sortable: true },
  { name: "longitude", label: "Longitude", field: "longitude", sortable: true },
  { name: "language", label: "Language", field: "language", sortable: true },
  {
    name: "population",
    label: "Population",
    field: "population",
    sortable: true,
  },
  { name: "flagIcon", label: "Flag", field: "flagIcon" },
];



function wrapCsvValue (val, formatFn, row) {
  let formatted = formatFn !== void 0
    ? formatFn(val, row)
    : val

  formatted = formatted === void 0 || formatted === null
    ? ''
    : String(formatted)
  formatted = formatted.split('"').join('""')
  return `"${formatted}"`
}

function exportTable () {
  // naive encoding to csv format
  const content = [columns.map(col => wrapCsvValue(col.label))].concat(
    rows.value.map(row => columns.map(col => wrapCsvValue(
      typeof col.field === 'function'
        ? col.field(row)
        : row[col.field === void 0 ? col.name : col.field],
      col.format,
      row
    )).join(','))
  ).join('\r\n')

  const status = exportFile(
    'countries-export.csv',
    content,
    'text/csv'
  )

  if (status !== true) {
    // Assuming you're using Quasar's notify plugin
    Quasar.Notify.create({
      message: 'Browser denied file download...',
      color: 'negative',
      icon: 'warning'
    })
  }
}



const fetchCountries = async () => {
  try {
    const response = await fetch("https://restcountries.com/v3.1/all");
    if (!response.ok) throw new Error("Failed to fetch data");
    rawCountries.value = await response.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

onMounted(fetchCountries);
</script>

<style scoped>
.my-table-details {
  font-size: 0.85em;
  font-style: italic;
  max-width: 200px;
  white-space: normal;
  color: #555;
  margin-top: 4px;
}
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}
</style>

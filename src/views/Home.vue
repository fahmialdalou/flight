<template>
<v-container class="py-0">
  <v-card>
    <v-card-title>
      <h1 class="display-1">Search For Flights</h1>
    </v-card-title>
    <v-card-text>
      <v-form>
        <v-row>
          <v-col cols="12" md="6"
            ><v-autocomplete
              @change="
                getAirports('dep_cities');
                validate();
              "
              @click="validate()"
              v-model="dep_from_country"
              :items="countries"
              placeholder="Country"
              item-value="API"
              prepend-icon="mdi-airplane-takeoff"
              return-object
            ></v-autocomplete
          ></v-col>
          <v-col cols="12" md="6"
            ><v-autocomplete
              @click="validate()"
              @change="validate()"
              v-model="dep_from_city"
              :items="dep_cities"
              placeholder="Airport"
              return-object
            ></v-autocomplete
          ></v-col>
        </v-row>
        <v-row>
          <v-col cols="12" md="6"
            ><v-autocomplete
              @change="
                getAirports('arr_cities');
                validate();
              "
              @click="validate()"
              v-model="arr_to_country"
              :items="countries"
              placeholder="Country"
              prepend-icon="mdi-airplane-landing"
              return-object
            ></v-autocomplete
          ></v-col>
          <v-col cols="12" md="6"
            ><v-autocomplete
              @click="validate()"
              @change="validate()"
              v-model="arr_to_city"
              :items="arr_cities"
              placeholder="Airport"
              return-object
            ></v-autocomplete
          ></v-col>
        </v-row>

        <v-row>
          <v-col cols="12" md="6">
            <v-dialog
              ref="dialog"
              v-model="modal"
              :return-value.sync="date"
              persistent
              width="290px"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="date"
                  label="Departure Date"
                  prepend-icon="mdi-sort-calendar-descending"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker v-model="date" scrollable :min="todaydate">
                <v-spacer></v-spacer>
                <v-btn text color="primary" @click="modal = false">
                  Cancel
                </v-btn>
                <v-btn text color="primary" @click="$refs.dialog.save(date);validate()">
                  OK
                </v-btn>
              </v-date-picker>
            </v-dialog>
          </v-col>
          <v-col cols="12" md="6">
            <v-dialog
              ref="dialog2"
              v-model="modal2"
              :return-value.sync="date2"
              persistent
              width="290px"
            >
              <template v-slot:activator="{ on, attrs }">
                <v-text-field
                  v-model="date2"
                  label="Return Date (optional)"
                  prepend-icon="mdi-sort-calendar-ascending"
                  readonly
                  v-bind="attrs"
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker v-model="date2" scrollable :min="date">
                <v-spacer></v-spacer>
                <v-btn text color="primary" @click="modal2 = false">
                  Cancel
                </v-btn>
                <v-btn text color="primary" @click="$refs.dialog2.save(date2)">
                  OK
                </v-btn>
              </v-date-picker>
            </v-dialog>
          </v-col>
        </v-row>
      </v-form>
    </v-card-text>
    <v-card-actions>
      <v-btn
        color="success x-large"
        :to="{
          name: 'Results',
          query: {
            date2: date2,
            date: date,
            dep_from_city: dep_from_city,
            arr_to_city: arr_to_city,
          },
        }"
        :disabled="!valid"
        >Search</v-btn
      >
    </v-card-actions>
  </v-card>
</v-container>
</template>

<script>
export default {
  name: "Home",
  data: () => ({
    valid: false,
    todaydate: new Date().toISOString().slice(0, 10),
    date: new Date().toISOString().substr(0, 10),
    modal: false,
    date2: "",
    modal2: false,
    countries: [],
    dep_cities: [],
    arr_cities: [],
    dep_from_country: "",
    dep_from_city: "",
    arr_to_country: "",
    arr_to_city: "",
    value: null,
  }),
  created() {
    this.getCountries();
    this.date = this.$route.query.date;
    this.date2 = this.$route.query.date2;
    this.dep_from_city = this.$route.query.dep_from_city;
    this.arr_to_city = this.$route.query.arr_to_city;
  },
  methods: {
    validate() {
      // console.log(this.arr_to_city);
      if (
        this.dep_from_country == "" ||
        this.dep_from_city == "" ||
        this.arr_to_country == "" ||
        this.arr_to_city == "" ||
        typeof this.date === "undefined"
      ) {
        this.valid = false;
      } else {
        this.valid = true;
      }
    },
    getCountries() {
      fetch(
        "https://skyscanner-skyscanner-flight-search-v1.p.rapidapi.com/apiservices/reference/v1.0/countries/en-US",
        {
          method: "GET",
          headers: {
            "x-rapidapi-host":
              "skyscanner-skyscanner-flight-search-v1.p.rapidapi.com",
            "x-rapidapi-key":
              "61c3564d5cmsha37736d660c1f55p15dc49jsn4ddaa5d018b6",
          },
        }
      )
        .then((response) => response.json())
        .then((data) => {
          // console.log(data.Countries);
          for (var i = 0; i < data.Countries.length; i++) {
            this.countries.push(data.Countries[i].Name);
          }
        })
        .catch((err) => {
          console.log(err);
        });
    },
    getPlacesArray(places, sel_array) {
      if (sel_array == "dep_cities") {
        this.dep_cities = [];
        this.dep_from_city = "";
        for (var i = 0; i < places.length; i++)
          this[sel_array].push(places[i].PlaceId);
      } else {
        this.arr_cities = [];
        this.arr_to_city = "";
        for (var k = 0; k < places.length; k++)
          this[sel_array].push(places[k].PlaceId);
      }
    },
    getAirports(selected_autocomplete) {
      var selected_country = "";
      if (selected_autocomplete == "dep_cities") {
        selected_country = this.dep_from_country;
      } else {
        selected_country = this.arr_to_country;
      }
      fetch(
        "https://skyscanner-skyscanner-flight-search-v1.p.rapidapi.com/apiservices/autosuggest/v1.0/AE/GBP/en-GB/?query=" +
          selected_country,
        {
          method: "GET",
          headers: {
            "x-rapidapi-host":
              "skyscanner-skyscanner-flight-search-v1.p.rapidapi.com",
            "x-rapidapi-key":
              "61c3564d5cmsha37736d660c1f55p15dc49jsn4ddaa5d018b6",
          },
        }
      )
        .then((response) => response.json())
        .then((data) => {
          // console.log(data.Places);
          this.getPlacesArray(data.Places, selected_autocomplete);
        })
        .catch((err) => {
          console.log(err);
        });
    },
  },
};
</script>


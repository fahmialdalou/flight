<template>
  <v-container class="py-0">
    <h2>Departing Flights:</h2>
    <v-row align="center" justify="space-around">
      <v-col cols="12" sm="4" md="6">
        <v-text-field
          type="number"
          label="A Flight Max Price"
          v-model="dep_max_price"
        ></v-text-field>
        <v-btn color="success" @click="FilterQuotes()" depressed>
          Filter Flights By Price
        </v-btn>
        <v-btn @click="ResetFilterQuotes()" depressed> Reset </v-btn>
      </v-col>

      <v-btn color="success" @click="SortByCost()"> Sort By Cost </v-btn>
      <v-btn color="success" @click="SortByDate()"> Sort By Date </v-btn>
      <v-checkbox
        @change="showDirectFlights()"
        v-model="direct_flights"
        :label="`Show Direct Flights Only?`"
      ></v-checkbox>
    </v-row>

    <v-simple-table>
      <template v-slot:default>
        <thead>
          <tr>
            <th>Airline</th>
            <th>From</th>
            <th>To</th>
            <th>Date</th>
            <th>Price</th>
            <th>Direct Flight?</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="Quote in Quotes"
            :key="Quote.QuoteId"
            @click="SelectDepFlight(Quote.MinPrice)"
          >
            <td>{{ getAirlineName(Quote.OutboundLeg.CarrierIds, "dep") }}</td>
            <td>{{ getPlaceName(Quote.OutboundLeg.OriginId, "dep") }}</td>
            <td>{{ getPlaceName(Quote.OutboundLeg.DestinationId, "dep") }}</td>
            <td>{{ Quote.OutboundLeg.DepartureDate }}</td>
            <td>{{ Quote.MinPrice }} AED</td>
            <td>{{ Quote.Direct }}</td>
          </tr>
        </tbody>
      </template>
    </v-simple-table>
    <div v-if="return_flight == true">
      <h2>Return Flights</h2>
      <v-simple-table>
        <template v-slot:default>
          <thead>
            <tr>
              <th>Airline</th>
              <th>From</th>
              <th>To</th>
              <th>Date</th>
              <th>Price</th>
              <th>Direct Flight?</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="Quote in Quotes2"
              :key="Quote.QuoteId"
              @click="SelectArrFlight(Quote.MinPrice)"
            >
              <td>{{ getAirlineName(Quote.OutboundLeg.CarrierIds, "dep") }}</td>
              <td>{{ getPlaceName(Quote.OutboundLeg.OriginId, "dep") }}</td>
              <td>
                {{ getPlaceName(Quote.OutboundLeg.DestinationId, "dep") }}
              </td>
              <td>{{ Quote.OutboundLeg.DepartureDate }}</td>
              <td>{{ Quote.MinPrice }} AED</td>
              <td>{{ Quote.Direct }}</td>
            </tr>
          </tbody>
        </template>
      </v-simple-table>
    </div>
    <div class="pricce">
      <h3>Total Flights Price: {{total_result}} AED</h3>
    </div>
  </v-container>
</template>
<style>
.pricce {
  position: fixed;
  bottom: 0;
  left: 0;
  background: #fff;
}
tr.active_node,
tr.active_node2 {
  background: #4caf50 !important;
  color: #fff;
}
</style>
<script>
// import router from "../router";

export default {
  name: "Results",
  data() {
    return {
      dep_max_price: 0,
      dep_min_price: 0,
      date: "",
      date2: "",
      dep_from_city: "",
      arr_to_city: "",
      Quotes: "",
      temp_quotes: "",
      temp_quotes_direct: "",
      Carriers: "",
      Places: "",
      Quotes2: "",
      Carriers2: "",
      Places2: "",
      dep_price: 0,
      arr_price: 0,
      return_flight: false,
      direct_flights: false,
    };
  },
  computed: {
    total_result: function () {
      return this.dep_price + this.arr_price;
    },
  },
  created() {
    this.date = this.$route.query.date;
    this.date2 = this.$route.query.date2;
    this.dep_from_city = this.$route.query.dep_from_city;
    this.arr_to_city = this.$route.query.arr_to_city;

    this.searchForFlights("dep");
    if (this.date2 != "" && typeof this.date2 !== "undefined") {
      this.searchForFlights("arr");
      this.return_flight = true;
    }
  },
  methods: {
    searchForFlights(flight_mode) {
      var from_city = "";
      var to_city = "";
      var dep_date = "";
      var arr_date = "";
      if (flight_mode == "dep") {
        from_city = this.dep_from_city;
        to_city = this.arr_to_city;
        dep_date = this.date;
        arr_date = this.date2;
      } else {
        from_city = this.arr_to_city;
        to_city = this.dep_from_city;
        dep_date = this.date2;
        arr_date = this.date;
      }
      //   fetch("https://rapidapi.p.rapidapi.com/apiservices/browseroutes/v1.0/AE/AED/en-US/DXBA-sky/CAI-sky/2021-10-23?inboundpartialdate=2021-11-01", {
      fetch(
        "https://rapidapi.p.rapidapi.com/apiservices/browseroutes/v1.0/AE/AED/en-US/" +
          from_city +
          "/" +
          to_city +
          "/" +
          dep_date +
          "?inboundpartialdate=" +
          arr_date,
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
          //console.log(data);
          if (flight_mode == "dep") {
            this.Quotes = data.Quotes;
            this.Carriers = data.Carriers;
            this.Places = data.Places;
          } else {
            this.Quotes2 = data.Quotes;
            this.Carriers2 = data.Carriers;
            this.Places2 = data.Places;
          }
        })
        .catch((err) => {
          console.error(err);
        });
    },
    getAirlineName(id, type) {
      if (type == "dep") {
        for (var i = 0; i < this.Carriers.length; i++) {
          if (this.Carriers[i].CarrierId == id) {
            return this.Carriers[i].Name;
          }
        }
      } else {
        for (var k = 0; k < this.Carriers2.length; k++) {
          if (this.Carriers2[k].CarrierId == id) {
            return this.Carriers2[k].Name;
          }
        }
      }
    },
    getPlaceName(id, type) {
      if (type == "dep") {
        for (var i = 0; i < this.Places.length; i++) {
          if (this.Places[i].PlaceId == id) {
            return this.Places[i].Name;
          }
        }
      } else {
        //console.log(this.Places2);
        for (var k = 0; k < this.Places2.length; k++) {
          if (this.Places2[k].PlaceId == id) {
            return this.Places2[k].Name;
          }
        }
      }
    },
    FilterQuotes() {
      this.ResetFilterQuotes();
      if (this.temp_quotes == "") this.temp_quotes = this.Quotes;
      this.Quotes = this.Quotes.filter(
        (Quotes) => Quotes.MinPrice < this.dep_max_price
      );
    },
    ResetFilterQuotes() {
      if (this.temp_quotes != "") {
        this.Quotes = this.temp_quotes;
        this.temp_quotes = "";
      }
    },
    showDirectFlights() {
      if (this.temp_quotes_direct == "") this.temp_quotes_direct = this.Quotes;
      //if (this.dep_max_price > 0) this.FilterQuotes();
      if (this.direct_flights == true) {
        this.Quotes = this.Quotes.filter((Quotes) => Quotes.Direct == true);
      } else {
        this.Quotes = this.temp_quotes_direct;
      }
    },
    SortByCost() {
      this.Quotes.sort(function (a, b) {
        return a.MinPrice - b.MinPrice;
      });
      this.Quotes2.sort(function (a, b) {
        return a.MinPrice - b.MinPrice;
      });
    },
    SortByDate() {
      this.Quotes.sort(
        (a, b) =>
          new Date(a.OutboundLeg.DepartureDate) -
          new Date(b.OutboundLeg.DepartureDate)
      );

      this.Quotes2.sort(
        (a, b) =>
          new Date(a.OutboundLeg.DepartureDate) -
          new Date(b.OutboundLeg.DepartureDate)
      );
    },
    SelectDepFlight(selected_data) {
      var els = document.querySelectorAll(".active_node2");
      for (var i = 0; i < els.length; i++) {
        els[i].classList.remove("active_node2");
      }
      event.target.parentElement.classList.add("active_node2");
      this.dep_price = selected_data;
    },
    SelectArrFlight(selected_data) {
      var els = document.querySelectorAll(".active_node");
      for (var i = 0; i < els.length; i++) {
        els[i].classList.remove("active_node");
      }
      event.target.parentElement.classList.add("active_node");
      this.arr_price = selected_data;
    },
  },
};
</script>
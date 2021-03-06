<template>
  <div class="flex justify-center bg-white min-height w-full my-5 xs:my-3">
    <div class="flex justify-center max-width w-full">
      <div v-if="error" class="pt-5">Store not found</div>
      <ItemNew v-if="individual._id" @getItems="getItems" />
      <Loader v-if="isLoading" />
      <div
        class="flex flex-col w-5/6 items-center mt-10  fade-in"
        v-if="!isLoading && !error"
      >
        <div class="flex justify-between items-start w-full">
          <div v-if="!isEditingName" class="flex items-start">
            <h2 class="text-2xl">
              {{ store.name }}
            </h2>
            <div v-if="isAbleToEdit" class="text-accent-medium px-5 pt-1">
              {{ storeStatusText }}
            </div>
          </div>
          <form
            v-else
            class="w-full"
            @submit.prevent="putStore(store, 'isEditingName')"
          >
            <input
              v-model="store.name"
              class="border border-accent-dark pl-2 w-4/6 mt-4"
            />
          </form>
          <ButtonEdit
            class="pt-1"
            v-if="isAbleToEdit"
            :document="store"
            :fields="['name']"
            fieldName="isEditingName"
            :isEditing.sync="isEditingName"
            @callback="putStore"
            @toggleEdit="toggleEdit"
          />
        </div>
        <hr class="w-full" />
        <div class="flex flex-col justify-start items-center w-full mb-5 mt-5">
          <div class="flex flex-col justify-between w-full">
            <div class="flex justify-between items-center">
              <h4 class="text-xl">Location</h4>
              <ButtonEdit
                v-if="isAbleToEdit"
                :document="store"
                :fields="['addressLine1', 'addressLine2', 'postcode', 'city']"
                fieldName="isEditingLocation"
                :isEditing.sync="isEditingLocation"
                @callback="putStore"
                @toggleEdit="toggleEdit"
              />
            </div>
            <div v-if="!isEditingLocation">
              <p v-if="store.addressLine1">{{ store.addressLine1 }},</p>
              <p v-if="store.addressLine2 && !isEditingLocation">
                {{ store.addressLine2 }},
              </p>
              <p v-if="store.postcode">{{ store.postcode }},</p>
              <p v-if="store.city">{{ store.city }}</p>
            </div>
            <div v-else class="flex flex-col">
              <input
                class="border border-accent-dark pl-2 my-1"
                v-model="store.addressLine1"
              />
              <input
                class="border border-accent-dark pl-2 my-1"
                v-model="store.addressLine2"
              />
              <input
                class="border border-accent-dark pl-2 my-1"
                v-model="store.postcode"
              />
              <input
                class="border border-accent-dark pl-2 my-1"
                v-model="store.city"
              />
            </div>
          </div>
          <GmapMap
            v-if="storePosition"
            :center="storePosition"
            :zoom="8"
            map-type-id="terrain"
            class="single-map mt-5"
          >
            <GmapMarker :position="storePosition" :clickable="true" />
          </GmapMap>
        </div>
        <hr class="w-full" />
        <div class="flex flex-col justify-start w-full mt-3 mb-5">
          <div class="flex justify-between items-center">
            <h4 class="text-xl">Description</h4>
            <ButtonEdit
              v-if="isAbleToEdit"
              :document="store"
              :fields="['description']"
              fieldName="isEditingDescription"
              :isEditing.sync="isEditingDescription"
              @callback="putStore"
              @toggleEdit="toggleEdit"
            />
          </div>
          <p v-if="!isEditingDescription">{{ store.description }}</p>
          <textarea
            v-else
            v-model="store.description"
            class="border border-accent-dark pl-1 w-full"
          />
        </div>
        <hr class="w-full" />
        <div class="flex flex-col justify-start w-full mb-5 mt-3">
          <div class="flex justify-between">
            <h4 class="text-xl">Dates</h4>
            <button
              class="underline text-accent-dark"
              @click="toggleDateNew"
              v-if="!isDateNewActive && isAbleToEdit"
            >
              Add date
            </button>
            <div v-if="isDateNewActive" class="store__add-date-btns">
              <button
                class="underline text-accent-dark p-3"
                @click="saveDateNew"
              >
                Save
              </button>
              <button class="underline text-accent-dark" @click="toggleDateNew">
                Cancel
              </button>
            </div>
          </div>
          <DateNew
            v-if="isDateNewActive"
            :newDate.sync="newDate"
            :newStartTime.sync="newStartTime"
            :newEndTime.sync="newEndTime"
          />
          <div class="mt-2">
            <div
              class="flex justify-between"
              v-for="(date, index) in store.dates"
              :key="index"
            >
              <p>
                {{ date.formatted.date }}, {{ date.formatted.startTime }} -
                {{ date.formatted.endTime }}
              </p>
              <p
                v-if="isAbleToEdit"
                class="underline text-accent-medium cursor-pointer text-sm"
                @click="deleteDate(date.id)"
              >
                Remove
              </p>
            </div>
          </div>
        </div>
        <hr class="w-full" />
        <div class="flex flex-col justify-start w-full mb-5 mt-3">
          <div class="flex justify-between">
            <h4 class="text-xl">Items</h4>
            <button
              v-if="isAbleToEdit"
              class="underline text-accent-dark"
              @click="$modal.show('newItemModal')"
            >
              Add
            </button>
          </div>
          <div class="flex flex-wrap justify-center pb-10">
            <div
              class="store__other flex flex-col items-center m-5 xs:m-2 relative"
              v-for="item in items"
              :key="item._id"
            >
              <img
                class="store__other-thumbnail cursor-pointer"
                v-bind:src="'data:image/jpeg;base64,' + item.images[0].buffer"
                @click="selectItem(item._id)"
              />
              <Close
                v-if="isAbleToEdit"
                class="store__other-thumbnail--close"
                @click="deleteItem(item._id)"
              />
              <div class="flex justify-start items-center mt-1 store__other">
                <div>
                  <div>£{{ item.price }}</div>
                  <div v-if="isAbleToEdit" class="text-accent-medium">
                    {{ getItemStatusText(item._id) }}
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import moment from "moment";
import Loader from "@/components/Loader.vue";
import DateNew from "@/components/DateNew.vue";
import ButtonEdit from "@/components/ButtonEdit.vue";
import ItemNew from "@/components/ItemNew.vue";
import setIndividual from "@/lib/individual";
import Close from "vue-material-design-icons/Close.vue";

export default {
  name: "Store",
  components: {
    Loader,
    DateNew,
    ItemNew,
    ButtonEdit,
    Close,
  },
  data() {
    return {
      isLoading: true,
      individual: {},
      store: null,
      items: [],
      isDateNewActive: false,
      newDate: new Date().toISOString(),
      newStartTime: new Date().toISOString(),
      newEndTime: new Date().toISOString(),
      isEditingName: false,
      isEditingLocation: false,
      isEditingDescription: false,
      error: false,
      isAbleToEdit: false,
      storePosition: null,
    };
  },
  computed: {
    isLoggedIn: function() {
      return !!this.$store.getters.token;
    },
    storeStatusText: function() {
      if (this.isAbleToEdit) {
        return this.store.status === "approved"
          ? "Approved"
          : "Pending approval";
      } else {
        return null;
      }
    },
  },
  async mounted() {
    if (this.isLoggedIn) {
      this.individual = await setIndividual();
    }
    await this.getStore();
    await this.getItems();
  },

  methods: {
    getItemStatusText: function(id) {
      const item = this.items.find((item) => item._id == id);

      if (this.isAbleToEdit) {
        return item.status === "approved" ? "Approved" : "Pending approval";
      } else {
        return null;
      }
    },
    async getStore() {
      try {
        const { data: store } = await axios.get(
          `stores/${this.$route.params.storeId}`
        );
        if (
          store.status !== "approved" &&
          this.individual._id !== store.userId
        ) {
          throw Error();
        }
        this.store = store;
        this.isAbleToEdit = this.individual._id === this.store.userId;
        const assignStorePosition = (position) =>
          (this.storePosition = position);

        const addressObj = {
          address_line_1: store.addressLine1,
          address_line_2: store.addressLine2,
          city: store.city,
          postal_code: store.postcode,
        };
        try {
          this.$geocoder.send(addressObj, async (response) => {
            const position = response.results[0].geometry.location;
            assignStorePosition(position);
          });
        } catch (err) {
          console.log("Geocoder error", err);
        }
      } catch {
        this.error = true;
      }
    },
    async getItems() {
      try {
        const {
          data: { items },
        } = await axios.get(
          `items?sortCriterion=1&storeId=${this.$route.params.storeId}`
        );
        this.items = items;
        this.isLoading = false;
      } catch (err) {
        console.log(err);
        this.isLoading = false;
      }
    },
    toggleDateNew() {
      this.isDateNewActive = !this.isDateNewActive;
    },
    saveDateNew() {
      this.addDate();
      this.isDateNewActive = false;
    },
    async deleteDate(id) {
      const newDates = this.store.dates.filter((date) => date.id !== id);
      try {
        await axios.put(`stores/${this.$route.params.storeId}`, {
          dates: newDates,
        });
        await this.getStore();
      } catch (err) {
        console.log(err);
      }
    },
    toggleEdit(field, val) {
      this[field] = val;
    },
    selectItem(id) {
      this.$router.push("/item/" + id);
      window.scrollTo({ top: 0, behavior: "smooth" });
    },
    async putStore(data, field) {
      const addressObj = {
        address_line_1: this.store.addressLine1,
        address_line_2: this.store.addressLine2,
        city: this.store.city,
        postal_code: this.store.postcode,
      };

      this.$geocoder.send(addressObj, async (response) => {
        const position = response.results[0].geometry.location;
        data.position = position;
        try {
          await axios.put(`stores/${this.$route.params.storeId}`, data);
          if (field) {
            this.toggleEdit(field, false);
          }
          await this.getStore();
        } catch (err) {
          console.log(err);
        }
      });
    },
    async addDate() {
      const startTime =
        new Date(this.newStartTime).getTime() -
        new Date(this.newStartTime).setHours(0, 0, 0, 0);

      const endTime =
        new Date(this.newEndTime).getTime() -
        new Date(this.newEndTime).setHours(0, 0, 0, 0);

      const startDate = new Date(this.newDate).getTime() + startTime;
      const endDate = new Date(this.newDate).getTime() + endTime;

      const formattedDate = moment(startDate).format("ddd Do MMM, YYYY");
      const formattedStartTime = moment(startDate).format("HH:mm");
      const formattedEndTime = moment(endDate).format("HH:mm");

      const date = {
        id: Math.random() * 100000000,
        iso: {
          start: new Date(startDate).toISOString(),
          end: new Date(endDate).toISOString(),
        },
        formatted: {
          date: formattedDate,
          startTime: formattedStartTime,
          endTime: formattedEndTime,
        },
      };

      const dates = this.store.dates;
      dates.push(date);
      await axios.put(`stores/${this.$route.params.storeId}`, { dates });
    },
    async deleteItem(id) {
      if (window.confirm("Are you sure you want to delete this item?")) {
        try {
          await axios.delete(`items/${id}`);
          this.getItems();
        } catch (err) {
          console.log(err);
        }
      }
    },
  },
};
</script>

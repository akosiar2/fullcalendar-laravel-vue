<template>
  <div class="container">
    <div class="row justify-content-center">
      <div class="col-md-8">
        <form @submit.prevent>
          <div class="form-group">
            <label for="event_name">Event Name</label>
            <input type="text" id="event_name" class="form-control" v-model="newEvents.title">
          </div>
          <div class="row">
            <div class="col-md-6">
              <div class="form-group">
                <label for="start_date">Start Date</label>
                <input
                  type="date"
                  id="start_date"
                  class="form-control"
                  v-model="newEvents.start_date"
                >
              </div>
            </div>
            <div class="col-md-6">
              <div class="form-group">
                <label for="end_date">End Date</label>
                <input type="date" id="end_date" class="form-control" v-model="newEvents.end_date">
              </div>
            </div>
            <div class="col-md-6 mb-4" v-if="addingMode">
              <button class="btn btn-sm btn-primary" @click="addnewEvents">Save Event</button>
            </div>
            <template v-else>
              <div class="col-md-6 mb-4">
                <button class="btn btn-sm btn-success" @click="updateEvent">Update</button>
                <button class="btn btn-sm btn-danger" @click="deleteEvent">Delete</button>
                <button class="btn btn-sm btn-secondary" @click="addingMode = !addingMode">Cancel</button>
              </div>
            </template>
          </div>
        </form>
      </div>
      <div class="col-md-8">
        <Fullcalendar :options="calendarOptions"/>
      </div>
    </div>
  </div>
</template>

<script>
import Fullcalendar from "@fullcalendar/vue";
import dayGridPlugin from "@fullcalendar/daygrid";
import interactionPlugin from "@fullcalendar/interaction";
import axios from "axios";

export default {
  components: {
    Fullcalendar
  },
  data: function() {
    return {
        calendarOptions:{
            plugins: [dayGridPlugin, interactionPlugin],
            initialView: 'dayGridMonth',
            //initialEvents: this.getEvents, // alternatively, use the `events` setting to fetch from a feed
            editable: true,
            select: this.showEvent,
            eventAdd: this.addnewEvents,
            eventChange: this.updateEvent,
            eventRemove: this.deleteEvent,
        },
        events: [],
        newEvents:[],
        addingMode: true,
        indexToUpdate: ""
    };
  },
  created() {
    this.getEvents();
  },
  methods: {
    addnewEvents() {
      axios
        .post("/api/calendar", {
          ...this.newEvents
        //   event_name = this.newEvents.title,
        //   start_date = this.newEvents.start,
        //   end_date = this.newEvents.end
        })
        .then(data => {
          this.getEvents(); // update our list of events
          this.resetForm(); // clear newEvents properties (e.g. title, start_date and end_date)
        })
        .catch(err =>
          console.log("Unable to add new event!", err.response.data)
        );
    },
    showEvent(arg) {
      this.addingMode = false;
      const { id, title, start, end, days } = this.events.find(
        event => event.id === +arg.event.id
      );
      this.indexToUpdate = id;
      this.newEvents = {
        title: title,
        start_date: start,
        end_date: end,
        //days: "all"
      };
    },
    updateEvent() {
      axios
        .put("/api/calendar/" + this.indexToUpdate, {
            ...this.newEvents
            // event_name = this.newEvents.title,
            // start_date = this.newEvents.start,
            // end_date = this.newEvents.end
        })
        .then(resp => {
          this.resetForm();
          this.getEvents();
          this.addingMode = !this.addingMode;
        })
        .catch(err =>
          console.log("Unable to update event!", err.response.data)
        );
    },
    deleteEvent() {
      axios
        .delete("/api/calendar/" + this.indexToUpdate)
        .then(resp => {
          this.resetForm();
          this.getEvents();
          this.addingMode = !this.addingMode;
        })
        .catch(err =>
          console.log("Unable to delete event!", err.response.data)
        );
    },
    getEvents() {
      axios
        .get("/api/calendar")
        .then(resp => (this.events = resp.data.data))
        .catch(err => console.log(err.response.data));

        console.log("get event:");
        //console.log(this.events);
    },
    resetForm() {
      Object.keys(this.newEvents).forEach(key => {
        return (this.newEvents[key] = "");
      });
    }
  },
  watch: {
    indexToUpdate() {
      return this.indexToUpdate;
    }
  }
};
</script>

<style lang="css">
@import "~@fullcalendar/common/main.css";
@import "~@fullcalendar/daygrid/main.css";

.fc-title {
  color: #fff;
}

.fc-title:hover {
  cursor: pointer;
}
</style>


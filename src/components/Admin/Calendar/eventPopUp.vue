<template>
  <div id="event-pop-up" :class="popUpStyleClass()" :style="stylePositionString">

    <div class="content">

      <h1>{{ date.monthName }} {{ date.dayNum }}, {{ date.year }}</h1>
      <br/>

      <div class="event-info">
        <div class="row">
          <div class="col col-label">
            Start Time:
          </div>
          <!-- Adjust time for new events -->
          <div v-if="!eventData.content" class="col col-content">
            <vue-timepicker
              v-model="startTime"
              hide-clear-button
              :format="timeFormat"
              :minute-interval="minuteInterval">
            </vue-timepicker>
          </div>
          <!-- Display non-adjustable time on existing events -->
          <div v-if="eventData.content" class="col col-content">
            {{ eventData.content.confirmed_time | momentOnlyTime }}
          </div>
        </div>

        <!-- Rows existing events -->
        <div class="row" v-if="eventData.content" v-for="(item, key) in filteredContent" :key="key">
          <div class="col col-label">
            {{ key | underscoresAreSpaces | capitalize }}
          </div>
          <div class="col col-content">
            {{ item }}
          </div>
        </div>

        <div class="row" v-if="eventData.content">
          <div class="col col-label">
            Link
          </div>
          <div class="col col-content">
            <router-link :to=jobURL>
              {{ jobURL }}
            </router-link>
          </div>
        </div>

        <!-- Rows creating new events -->
        <div class="row" v-if="!eventData.content" v-for="(item, index) in content" :key="index">
          <div class="col col-label">
            {{ item.key | underscoresAreSpaces | capitalize }}
          </div>
          <div class="col col-content">
            <input
              type="text"
              v-model="content[index].value"
              required
            ></input>
          </div>
        </div>

        <!-- Submit button displayed when creating new events -->
        <div class="custom-btn ok-btn" v-if="!eventData.content" v-on:click="handleClickOkay">Ok</div>
      </div>

    </div>

  </div>
</template>


<script>
import VueTimepicker from 'vue2-timepicker'
import uuidv4 from 'uuid'

export default {
  props: ['stylePosition', 'date', 'eventData', 'jobRequestData'],
  components: { VueTimepicker },

  data() {
    return {
      timeFormat: "hh:mm a",
      minuteInterval: 15,
      startTime: {
        hh: '00',
        mm: '00',
        a: 'am'
      },
      endTime: {
        hh: '00',
        mm: '00',
        a: 'am'
      },
      content: [
        // TO DO: externalize list of wanted columns so don't have to duplicate these everywhere
        { key: 'description', value: '' },
        { key: 'address', value: '' },
        { key: 'admin_notes', value: '' },
        { key: 'phone', value: '' },
        { key: 'email', value: '' },
        { key: 'customer_first_name', value: '' },
        { key: 'customer_last_name', value: '' },
      ],
    }
  },

  methods: {
    uuidv4() {
      return uuidv4()
    },

    handleClickOkay: function() {

      // Handling new event data inputted by admin
      // This gets re-emitted in parent to outside of calendar
      this.$emit('handle-click-okay-in-pop-up', {
        date: this.date,
        startTime: this.startTime,
        endTime: this.endTime,
        description: this.content[0].value,
        address: this.content[1].value,
        adminNotes: this.content[2].value,
        phone: this.content[3].value,
        email: this.content[4].value,
        customerFirstName: this.content[5].value,
        customerLastName: this.content[6].value,
      })
    },

    popUpStyleClass: function() {
      const { sideToDisplay } = this.stylePosition
      // For left or right side styling
      switch (sideToDisplay) {
        case ('right'): return 'arrow-box-on-right'
        case ('left'): return 'arrow-box-on-left'
        default: break
      }
    }
  },


  computed: {
    // For getting data from an existing event
    filteredContent: function() {
      if (this.eventData.content) {
        const {
          description,
          address,
          admin_notes,
          employee_notes,
          phone,
          email,
          customer_first_name,
          customer_last_name,
        } = this.eventData.content

        return {
          description,
          address,
          admin_notes,
          employee_notes,
          phone,
          email,
          customer_first_name,
          customer_last_name,
        }
      }
    },

    stylePositionString: function() {
      const { top, left } = this.stylePosition
      return `top: ${top}px; left: ${left}px;`
    },

    jobURL() {
      return `/admin/jobs/${this.eventData.content.id}`
    },
  },

  mounted() {
    console.log("Event Data", this.jobRequestData)

    const { startTime, endTime, content } = this.eventData
    if (this.eventData.content) {
      this.startTime = {
        hh: startTime.slice(0, 2),
        mm: startTime.slice(3, 5),
        a: startTime.slice(-2),
      }
      this.endTime = {
        hh: endTime.slice(0, 2),
        mm: endTime.slice(3, 5),
        a: endTime.slice(-2),
      }
    }

    const { jobRequestData } = this
    if (jobRequestData) {
      this.content[0].value = jobRequestData.description
      this.content[1].value = jobRequestData.address
      this.content[3].value = jobRequestData.guest_phone
      this.content[4].value = jobRequestData.guest_email
      this.content[5].value = jobRequestData.guest_first_name
      this.content[6].value = jobRequestData.guest_last_name
    }
  },
}
</script>


<style lang="scss" scoped>
@import "../../../colors.scss";

$border-color: gainsboro;
$fill-color: rgb(248, 248, 248);
$border-radius: 4px;


#event-pop-up {
  z-index: 3;
}

.arrow-box-on-right {
	position: absolute;
	background: $fill-color;
  border: 1px solid $border-color;
  border-radius: $border-radius;

  &:after, &:before {
    right: 100%;
    top: 200px;
    border: solid transparent;
    content: " ";
    height: 0;
    width: 0;
    position: absolute;
    pointer-events: none;
  }

  &:after {
    border-color: rgba(0, 0, 0, 0);
    border-right-color: $fill-color;
    border-width: 10px;
    margin-top: -10px;
  }

  &:before {
    border-color: rgba(0, 0, 0, 0);
    border-right-color: $border-color;
    border-width: 11px;
    margin-top: -11px;
  }
}

.arrow-box-on-left {
	position: absolute;
	background: $fill-color;
  border: 1px solid $border-color;
  border-radius: $border-radius;

  &:after, &:before {
    left: 100%;
    top: 200px;
    border: solid transparent;
    content: " ";
    height: 0;
    width: 0;
    position: absolute;
    pointer-events: none;
  }

  &:after {
    border-color: rgba(0, 0, 0, 0);
    border-left-color: $fill-color;
    border-width: 10px;
    margin-top: -10px;
  }

  &:before {
    border-color: rgba(0, 0, 0, 0);
    border-left-color: $border-color;
    border-width: 11px;
    margin-top: -11px;
  }
}

.content {
  width: 500px;
  height: min-content;
  background-color: transparent;
  padding: 6px 8px;
  border-radius: $border-radius;

  overflow-y: scroll;

  h1 {
    font-size: 1.2em;
  }

  h2 {
    font-size: 1em;
  }

  .event-info {

    .row {
      width: 100%;
      display: grid;
      grid-template-columns: [col] 25% [col] 75%;

      .col {
        padding: 0.5em;
      }

      .col-label {
        font-weight: bold;
        display: flex;
        align-items: flex-start;
        justify-content: flex-end;
        text-align: right;
        padding: 0.5em;
        border-right: 1px solid #d2d2d2
      }

      .col-content {
        display: flex;
        align-items: center;
        justify-content: flex-start;
        text-align: left;

        input {
          padding: 0.25em;
          margin: 0;
          border: 1px solid #d2d2d2;
          width: 100%;
        }
      }
    }
  }
}

.ok-btn {
  background-color: gainsboro;
  height: fit-content;
  padding: 0.5em 1em;
  width: 6em;
  cursor: pointer;
  float: right;
}
</style>

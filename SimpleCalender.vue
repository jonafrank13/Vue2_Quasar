<template>
  <div class="container">
  	<div class="calendar full-width">
  		<div class="calendar-navigation full-width">
  			<div class="calendar-prev" @click="calendarPrevious" >&lt;</div>
  	        <div class="calendar-title">{{visitingDateDisplay}}</div>
  	        <div class="calendar-next" @click="calendarNext" >&gt;</div>
  		</div>
  		<div :class="{ 'calendar-hide':hideCalendar() }" class="calendar-days">
  			<div v-for="(day, index) in days" :class="{'calendar-sunday': index === 6 }">{{day}}</div>
  		</div>
  		<div v-if="showReset()" @click="resetCalendar" class="calendar-gotoday">
  			{{today}}
  		</div>
  		<div class="calendar-week" :class="{ 'calendar-hide':hideCalendar() }" v-for="week in calendarMatrix" v-touch-swipe="swipeHandler">
  			<div class="calendar-date" v-for="(day, index) in week" v-on:click="selectDate(day)" :class="{ 'calendar-current-date':isCurrentDate(day), 'calendar-selected-date':isSelectedDate(day), 'calendar-diff-month':isDiffMonth(day), 'calendar-has-event':hasEvent(day), 'calendar-sunday': index === 6}">{{day.getDate()}}</div>
  		</div>
  	</div>
    <slot :selectedDate="selectedDate" :calendarStartDate="calendarStartDate" :calendarStopDate="calendarStopDate" ></slot>
  </div>
</template>
<script>
import moment from 'moment'

export default{
  props: {
    eventDates: {
      type: Array,
      default: function () {
        return []
      }
    },
    type: {
      type: String,
      default: function () {
        return 'full'
      }
    }
  },
  data () {
    return {
      selectedDate: new Date(),
      visitingDate: new Date(),
      today: new Date().getDate(),
      moveCount: 0,
      days: ['Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa', 'Su']
    }
  },
  methods: {
    calendarPrevious: function () {
      this.visitingDate = (this.type === 'full') ? this.prevMonth(this.visitingDate) : (this.type === 'week') ? this.prevWeek(this.visitingDate) : this.prevDay(this.visitingDate)
      this.moveCount -= 1
      this.selectedDate = this.visitingDate
    },
    prevMonth: function (date) {
      let tmpMonth = date.getMonth() - 1
      return new Date(date.setMonth(tmpMonth))
    },
    prevWeek: function (date) {
      let tmpWeek = date.getDate() - 7
      return new Date(date.setDate(tmpWeek))
    },
    prevDay: function (date) {
      let tmpDay = date.getDate() - 1
      return new Date(date.setDate(tmpDay))
    },
    calendarNext: function () {
      this.visitingDate = (this.type === 'full') ? this.nextMonth(this.visitingDate) : (this.type === 'week') ? this.nextWeek(this.visitingDate) : this.nextDay(this.visitingDate)
      this.moveCount += 1
      this.selectedDate = this.visitingDate
    },
    nextMonth: function (date) {
      let tmpMonth = date.getMonth() + 1
      return new Date(date.setMonth(tmpMonth))
    },
    nextWeek: function (date) {
      let tmpWeek = date.getDate() + 7
      return new Date(date.setDate(tmpWeek))
    },
    nextDay: function (date) {
      let tmpDay = date.getDate() + 1
      return new Date(date.setDate(tmpDay))
    },
    selectDate: function (date) {
      this.selectedDate = date
    },
    isCurrentDate: function (date) {
      return date != null && moment(date).format('MMM Do YY') === moment(new Date()).format('MMM Do YY')
    },
    isSelectedDate: function (date) {
      return date === this.selectedDate
    },
    isDiffMonth: function (date) {
      return moment(date).format('MMM') !== moment(this.visitingDate).format('MMM')
    },
    hideCalendar: function () {
      return this.type === 'day'
    },
    hasEvent: function (date) {
      if (this.eventDates.length > 0) {
        for (let eventDateKey in this.eventDates) {
          let formattedEventDate = moment(this.eventDates[eventDateKey]).format('MMM Do YY')
          let formattedCurrDate = moment(date).format('MMM Do YY')
          if (formattedEventDate === formattedCurrDate) {
            return true
          }
        }
      }
      return false
    },
    getCalendarMatrix: function (date) {
      let calendarMatrixLocal = [] // Initialize an empty calendar matrix

      let startDate = (this.type === 'full') ? new Date(date.getFullYear(), date.getMonth(), 1) : new Date(date) // If its full calendar set the start day of month else set it as the current date
      let stopDate = (this.type === 'full') ? new Date(date.getFullYear(), date.getMonth() + 1, 0) : new Date(date) // Similar to the above either last day of month or the current date

      let startDow = (startDate.getDay() + 6) % 7 // Start the month in the correct day so correct the left over days
      let endDow = (stopDate.getDay() + 6) % 7

      startDate.setDate(startDate.getDate() - startDow)
      stopDate.setDate(stopDate.getDate() + (6 - endDow))
      let week = []

      while (startDate <= stopDate) { // Iterate from start date to end date and fill up the matrix
        week.push(new Date(startDate))
        if (week.length === 7) {
          calendarMatrixLocal.push(week)
          week = []
        }
        startDate = new Date(startDate.setDate(startDate.getDate() + 1))
      }

      return calendarMatrixLocal
    },
    resetCalendar: function () {
      this.visitingDate = new Date()
      this.moveCount = 0
      this.selectedDate = this.visitingDate
    },
    showReset: function () {
      return this.moveCount !== 0
    },
    swipeHandler: function (event) {
      if (event.direction === 'right' || event.direction === 'down') {
        this.calendarPrevious()
      }
      else if (event.direction === 'left' || event.direction === 'up') {
        this.calendarNext()
      }
    }
  },
  computed: {
    calendarMatrix: function () {
      return this.getCalendarMatrix(this.visitingDate)
    },
    visitingDateDisplay: function () {
      return (this.type !== 'day') ? moment(this.visitingDate).format('MMM YYYY') : moment(this.visitingDate).format('dddd MMMM Do YYYY')
    },
    calendarStartDate: function () {
      return new Date(this.calendarMatrix[0][0])
    },
    calendarStopDate: function () {
      return new Date(this.calendarMatrix[this.calendarMatrix.length - 1][6])
    }
  }
}
</script>
<style scoped>
.container{
  display: flex;
  flex-flow: column;
}
.calendar{
	color: #31353b;
	position: relative;
	padding: 0px 30px 10px 20px;
  border-bottom: 1px solid #999;
}
.calendar-gotoday{
	position: absolute;
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background: #4476bb;
  right: 5px;
  text-align: center;
  line-height: 28px;
  top: 11px;
  /*font-size: 0.7em;*/
  color: white;
}
.calendar-navigation{
	display: flex;
	flex-flow: row;
	justify-content: space-between;
	line-height: 50px;
}
.calendar-prev{
	color: #999999;
	margin-left: 4%;
	/*font-size: 25px;*/
	font-weight: lighter;
	padding: 0px 5px 0px 5px;
}
.calendar-title{
	font-weight: bolder;
}
.calendar-next{
	color: #999999;
	margin-right: 4%;
	/*font-size: 25px;*/
	font-weight: lighter;
	padding: 0px 5px 0px 5px;
}
.calendar-days{
	display: flex;
	flex-flow: row;
	justify-content: space-between;
	line-height: 25px;
}
.calendar-days>div{
	/*font-size: 0.7em;*/
	width: 35px;
	height: 35px;
	text-align: center;
}
.calendar-week{
	display: flex;
	flex-flow: row;
	justify-content: space-between;
	line-height: 25px;
}
.calendar-date{
	text-align: center;
	width: 35px;
	height: 35px;
	line-height: 30px;
	border-radius: 50%;
	position: relative;
	vertical-align: middle;
	border: 1px solid transparent;
}
.calendar-diff-month{
	color: #999999;
}
.calendar-has-event:after{
	content: '.';
  color: red;
  font-size: 30px;
  bottom: -2px;
  left: 15px;
  position: absolute;
}
.calendar-current-date{
	background: #4476bb;
	color: white;
}
.calendar-selected-date, .calendar-selected-date:active{
	color: #5898e3;
	border: 1px solid #5898e3;
}
.calendar-current-date:after{
  color: white !important;
}
.calendar-hide{
	display: none;
}
.calendar-sunday{
  color: red;
}
</style>

<template>
  <div>
    <div class="is-wrap">
      <ul class="is-list">
        <li v-for="(row, i) in rows_displayed" :key="i">{{ row }}</li>
      </ul>
      <div class="is-scroll"></div>
    </div>
    <!-- reverse button action -->
    <!-- <button class="is-button" @click="scrollUp()">Scroll Up</button> -->
    <!-- normal button action -->
    <button class="is-button" @click="scrollDown()">Scroll Up</button>
    <br />
    <!-- reverse button action -->
    <!-- <button class="is-button" @click="scrollDown()">Scroll Down</button> -->
    <!-- normal button action -->
    <button class="is-button" @click="scrollUp()">Scroll Down</button>
  </div>
</template>

<script>
import axios from "axios";
import { gsap } from "gsap";

export default {
  data() {
    return {
      // config fields
      rows_to_display: 13, // number of rows which we will display
      buffer: 5, // buffer can not be smaller then 1

      // private fields
      start: 1,
      end: 0, // will imediatelly equal to total_rows
      requested_data: [], // all rows, buffer + rows we see, from response
      rows_displayed: [], // rows that we actually see
      total_rows: 0, // total number of all rows, buffer + rows we see
      mnd: 0, // minimum for new data load
      step: 1,
      total_clicks: 0,
      blocked_scroll: false, // blocks scroll while data is loading

      total_count_DB: 0,

      sb_percent: 0,
      wp: "",
      wp_height: 0,
      sb_height: 0,
      sb_factor: 0,
    };
  },
  created() {
    this.total_rows = this.rows_to_display + this.buffer * 2;
    this.end = this.total_rows;
    this.mnd = this.total_rows - this.rows_to_display + 1;
    this.requestData();
  },
  mounted() {
    this.wheel(".is-wrap", 1, 10000);
  },
  methods: {
    requestData() {
      console.log("loading...");
      this.blocked_scroll = true;
      axios
        .get(
          `http://localhost:3004/data?_start=${this.start - 1}&_end=${this.end}`
        )
        .then((response) => {
          this.requested_data = response.data;
          this.total_count_DB = response.headers["x-total-count"];
          this.showData();
          console.log("done loading!", this.requested_data);
          this.blocked_scroll = false;
          // set scroll bar leveraging event loop
          setTimeout(() => {
            this.setScrollBar();
          });
        })
        .catch((error) => console.log(error));
    },
    sliceData(array, start, end) {
      start -= 1;
      let slice = start + end;
      return array.slice(start, slice);
    },
    showData() {
      this.rows_displayed = this.sliceData(
        this.requested_data,
        this.step,
        this.rows_to_display
      );
    },

    // magic starts here ;)
    scrollUp() {
      // stop when certain point is reached (no need for total number, but will jump one place more)
      // if (this.rows_to_display > this.rows_displayed.length) return;

      // stop when certain point is reached (last array item is in view), we need to have the total number of rows_to_display
      if (this.total_clicks === this.total_count_DB - this.rows_to_display)
        return;

      this.total_clicks++;
      this.step++;

      if (this.step === this.mnd) {
        this.start += this.buffer;
        this.end += this.buffer;
        this.step = this.buffer + 1;
        this.requestData();
        return;
      }
      this.showData();
    },
    scrollDown() {
      this.total_clicks > 0 && this.total_clicks--;
      this.step > 1 && this.step--;

      if (this.total_clicks > 0 && this.step === 1) {
        this.start -= this.buffer;
        this.end -= this.buffer;
        this.step += this.buffer;
        this.requestData();
        return;
      }
      this.showData();
    },

    /**
     * Increment and decrement a value on mouse wheel event
     * @selector : string _ can be class, id, etc...
     * @min : int & @max : int _ set the limits
     */
    wheel(selector, min, max) {
      let timer = null;
      // select an input and set initial value
      const scrollable_element = document.querySelector(selector);

      // add wheel, direction recognition & set wheel limits
      scrollable_element.addEventListener("wheel", (e) => {
        // reverse mouse wheel
        // if (e.deltaY > min) {
        //   if (this.step > min && !this.blocked_scroll) {
        //     this.scrollDown();
        //   }
        // } else {
        //   if (this.step < max && !this.blocked_scroll) {
        //     this.scrollUp();
        //   }
        // }

        // normal mouse wheel
        if (e.deltaY > min) {
          if (this.step < max && !this.blocked_scroll) {
            this.scrollUp();
          }
        } else {
          if (this.step > min && !this.blocked_scroll) {
            this.scrollDown();
          }
        }
        gsap.to(".is-scroll", { duration: 0.5, opacity: 1 });

        if (timer !== null) {
          clearTimeout(timer);
        }
        timer = setTimeout(() => {
          gsap.to(".is-scroll", { duration: 1, opacity: 0 });
        }, 250);
      });
    },
    setScrollBar() {
      this.wp = document.querySelector(".is-wrap");
      this.wp_height = this.wp.clientHeight;
      this.sb_percent = (this.rows_to_display / this.total_count_DB) * 100;
      this.sb_height = (this.sb_percent / 100) * this.wp_height;
      this.sb_factor = this.wp_height / this.total_count_DB;

      const el = document.querySelector(".is-scroll");
      el.style.height = this.sb_height + "px";
    },
  },
  watch: {
    // move the scroll bar
    total_clicks() {
      const el = document.querySelector(".is-scroll");
      el.style.top = this.total_clicks * this.sb_factor + "px";
    },
  },
};
</script>

<style scoped>
.is-wrap {
  border: 1px solid red;
  margin-bottom: 45px;
  position: relative;
  user-select: none;
}
.is-list {
  list-style-type: none;
  margin: 0;
  padding: 0;
}
.is-button {
  width: 100px;
  padding: 5px 0;
  margin-bottom: 10px;
  user-select: none;
}
.is-scroll {
  width: 7px;

  box-sizing: border-box;
  opacity: 0;
  position: absolute;
  right: 0;
  top: 0; /* will be calculated */

  background: rgba(128, 0, 128, 0.5);
}
</style>
<template>
  <div>
    <div id="gantt-header" class="h-12 p-2 flex items-center">
      <h1 class="text-xl font-bold">ガントチャート</h1>
    </div>
    <div id="gantt-content" class="flex">
      <div id="gantt-task">
        <div
          id="gantt-task-title"
          class="flex items-center bg-green-600 text-white h-20"
          ref="task"
        >
          <div
            class="
              border-t border-r border-b
              flex
              items-center
              justify-center
              font-bold
              text-xs
              w-48
              h-full
            "
          >
            タスク
          </div>
          <div
            class="
              border-t border-r border-b
              flex
              items-center
              justify-center
              font-bold
              text-xs
              w-24
              h-full
            "
          >
            開始日
          </div>
          <div
            class="
              border-t border-r border-b
              flex
              items-center
              justify-center
              font-bold
              text-xs
              w-24
              h-full
            "
          >
            完了期限日
          </div>
          <div
            class="
              border-t border-r border-b
              flex
              items-center
              justify-center
              font-bold
              text-xs
              w-16
              h-full
            "
          >
            担当
          </div>
          <div
            class="
              border-t border-r border-b
              flex
              items-center
              justify-center
              font-bold
              text-xs
              w-12
              h-full
            "
          >
            進捗
          </div>
        </div>
        <div
          id="gantt-task-list"
          class="overflow-y-hidden"
          :style="`height:${calendarViewHeight}px`"
        >
          <div
            v-for="(task, index) in displayTasks"
            :key="index"
            class="flex h-10 border-b"
            @dragstart="dragTask(task)"
            @dragover.prevent="dragTaskOver(task)"
            draggable="true"
          >
            <template v-if="task.cat === 'category'">
              <div
                class="flex items-center font-bold w-full text-sm pl-2 flex justify-between items-center bg-teal-100"
              >
                <span>{{ task.name }}</span>
                <div class="pr-4" @click="toggleCategory(task.id)">
                  <span v-if="task.collapsed">
                    <svg
                      class="w-4"
                      xmlns="http://www.w3.org/2000/svg"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M9 5l7 7-7 7"
                      />
                    </svg>
                  </span>
                  <span v-else>
                    <svg
                      class="w-4"
                      xmlns="http://www.w3.org/2000/svg"
                      fill="none"
                      viewBox="0 0 24 24"
                      stroke="currentColor"
                    >
                      <path
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M19 9l-7 7-7-7"
                      />
                    </svg>
                  </span>
                </div>
              </div>
            </template>
            <template v-else>
              <div
                class="border-r flex items-center font-bold w-48 text-sm pl-4"
              >
                {{ task.name }}
              </div>
              <div
                class="border-r flex items-center justify-center w-24 text-sm"
              >
                {{ task.start_date }}
              </div>
              <div
                class="border-r flex items-center justify-center w-24 text-sm"
              >
                {{ task.end_date }}
              </div>
              <div
                class="border-r flex items-center justify-center w-16 text-sm"
              >
                {{ task.incharge_user }}
              </div>
              <div class="flex items-center justify-center w-12 text-sm">
                {{ task.percentage }}%
              </div>
            </template>
          </div>
        </div>
      </div>
      <div
        id="gantt-calendar overflow-y-hidden border-l"
        class="overflow-x-scroll"
        :style="`width:${calendarViewWidth}px`"
        ref="calendar"
      >
        <div id="gantt-date" class="h-20">
          <div id="gantt-year-month" class="relative h-8">
            <div v-for="(calendar, index) in calendars" :key="index">
              <div
                class="
                  bg-indigo-700
                  text-white
                  border-b border-r border-t
                  h-8
                  absolute
                  font-bold
                  text-sm
                  flex
                  items-center
                  justify-center
                "
                :style="
                  `width:${calendar.calendar *
                    block_size}px;left:${calendar.start_block_number *
                    block_size}px`
                "
              >
                {{ calendar.date }}
              </div>
            </div>
          </div>
          <div id="gantt-day" class="relative h-12">
            <div v-for="(calendar, index) in calendars" :key="index">
              <div v-for="(day, index) in calendar.days" :key="index">
                <div
                  class="
                    border-r border-b
                    h-12
                    absolute
                    flex
                    items-center
                    justify-center
                    flex-col
                    font-bold
                    text-xs
                  "
                  :class="{
                    'bg-blue-100': day.dayOfWeek === '土',
                    'bg-red-100': day.dayOfWeek === '日',
                    'bg-red-600 text-white':
                      calendar.year === today.year() &&
                      calendar.month === today.month() &&
                      day.day === today.date(),
                  }"
                  :style="
                    `width:${block_size}px;left:${day.block_number *
                      block_size}px`
                  "
                >
                  <span>{{ day.day }}</span>
                  <span>{{ day.dayOfWeek }}</span>
                </div>
              </div>
            </div>
          </div>
          <div id="gantt-height" class="relative">
            <div v-for="(calendar, index) in calendars" :key="index">
              <div v-for="(day, index) in calendar.days" :key="index">
                <div
                  class="border-r border-b absolute"
                  :class="{
                    'bg-blue-100': day.dayOfWeek === '土',
                    'bg-red-100': day.dayOfWeek === '日',
                  }"
                  :style="
                    `width:${block_size}px;left:${day.block_number *
                      block_size}px;height:${calendarViewHeight}px`
                  "
                ></div>
              </div>
            </div>
          </div>
        </div>
        <div
          id="gantt-bar-area"
          class="relative"
          :style="`width:${calendarViewWidth}px;height:${calendarViewHeight}px`"
        >
          <div v-for="(bar, index) in taskBars" :key="index">
            <div
              :style="bar.style"
              class="rounded-lg absolute h-5 bg-yellow-100"
              v-if="bar.task.cat === 'task'"
              @mousedown="mouseDownMove(bar.task)"
            >
              <div class="w-full h-full" style="pointer-events: none;">
                <div
                  class="h-full bg-yellow-500 rounded-l-lg"
                  style="pointer-events: none;"
                  :style="`width:${bar.task.percentage}%`"
                  :class="{ 'rounded-r-lg': bar.task.percentage === 100 }"
                ></div>
              </div>
              <div
                class="absolute w-2 h-2 bg-gray-300 border border-black"
                style="top:6px;left:-6px;cursor:col-resize"
                @mousedown.stop="mouseDownResize(bar.task, 'left')"
              ></div>
              <div
                class="absolute w-2 h-2 bg-gray-300 border border-black"
                style="top:6px;right:-6px;cursor:col-resize"
                @mousedown.stop="mouseDownResize(bar.task, 'right')"
              ></div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from "moment";

export default {
  name: "Gantt",
  data() {
    return {
      GANTT_HEADER_HEIGHT: 48,
      SCROLLBAR_HEIGHT: 20,
      position_id: 0,
      inner_width: "",
      inner_height: "",
      task_width: "",
      task_height: "",
      start_month: "2021-06",
      end_month: "2021-08",
      today: moment(),
      block_size: 30,
      block_number: 0,
      dragging: false,
      pageX: "",
      elememt: "",
      left: "",
      task_id: "",
      width: "",
      leftResizing: false,
      rightResizing: false,
      calendars: [],
      categories: [
        {
          id: 1,
          name: "テストA",
          collapsed: false,
        },
        {
          id: 2,
          name: "テストB",
          collapsed: false,
        },
      ],
      tasks: [
        {
          id: 1,
          category_id: 1,
          name: "テスト1",
          start_date: "2021-07-18",
          end_date: "2021-07-20",
          incharge_user: "鈴木",
          percentage: 100,
        },
        {
          id: 2,
          category_id: 1,
          name: "テスト2",
          start_date: "2021-07-19",
          end_date: "2021-07-23",
          incharge_user: "佐藤",
          percentage: 90,
        },
        {
          id: 3,
          category_id: 1,
          name: "テスト3",
          start_date: "2021-07-19",
          end_date: "2021-08-04",
          incharge_user: "鈴木",
          percentage: 40,
        },
        {
          id: 4,
          category_id: 1,
          name: "テスト4",
          start_date: "2021-07-21",
          end_date: "2021-07-30",
          incharge_user: "山下",
          percentage: 60,
        },
        {
          id: 5,
          category_id: 1,
          name: "テスト5",
          start_date: "2021-07-25",
          end_date: "2021-08-04",
          incharge_user: "佐藤",
          percentage: 5,
        },
        {
          id: 6,
          category_id: 2,
          name: "テスト6",
          start_date: "2021-07-28",
          end_date: "2021-08-08",
          incharge_user: "佐藤",
          percentage: 0,
        },
      ],
      task: "",
    };
  },
  methods: {
    getWindowSize() {
      this.inner_width = window.innerWidth;
      this.inner_height = window.innerHeight;
      // this.task_width = this.$refs.task.offsetWidth;
      this.task_width = 496;
      this.task_height = this.$refs.task.offsetHeight;
    },
    windowSizeCheck() {
      let height = this.lists.length - this.position_id;
      if (window.event.deltaY > 0 && height * 40 > this.calendarViewHeight) {
        this.position_id++;
      } else if (window.event.deltaY < 0 && this.position_id !== 0) {
        this.position_id--;
      }
    },
    getDays(year, month, block_number) {
      const dayOfWeek = ["日", "月", "火", "水", "木", "金", "土"];
      let days = [];
      let date = moment(`${year}-${month}-01`);
      let num = date.daysInMonth();
      for (let i = 0; i < num; i++) {
        days.push({
          day: date.date(),
          dayOfWeek: dayOfWeek[date.day()],
          block_number,
        });
        date.add(1, "day");
        block_number++;
      }
      return days;
    },
    getCalendar() {
      let block_number = 0;
      let days;
      let start_month = moment(this.start_month);
      let end_month = moment(this.end_month);
      let between_month = end_month.diff(start_month, "months");
      for (let i = 0; i <= between_month; i++) {
        days = this.getDays(
          start_month.year(),
          start_month.format("MM"),
          block_number
        );
        this.calendars.push({
          date: start_month.format("YYYY年MM月"),
          year: start_month.year(),
          month: start_month.month(),
          start_block_number: block_number,
          calendar: days.length,
          days: days,
        });
        start_month.add(1, "months");
        block_number = days[days.length - 1].block_number;
        block_number++;
      }
      return block_number;
    },
    todayPosition() {
      this.$refs.calendar.scrollLeft = this.scrollDistance;
    },
    mouseDownMove(task) {
      this.dragging = true;
      this.pageX = window.event.pageX;
      this.element = window.event.target;
      this.left = window.event.target.style.left;
      this.task_id = task.id;
    },
    mouseDownResize(task, direction) {
      direction === "left"
        ? (this.leftResizing = true)
        : (this.rightResizing = true);
      this.pageX = window.event.pageX;
      this.width = window.event.target.parentElement.style.width;
      this.left = window.event.target.parentElement.style.left;
      this.element = window.event.target.parentElement;
      this.task_id = task.id;
    },
    mouseMove() {
      if (this.dragging) {
        let diff = this.pageX - event.pageX;
        this.element.style.left = `${parseInt(this.left.replace("px", "")) -
          diff}px`;
      }
    },
    mouseResize() {
      if (this.leftResizing) {
        let diff = this.pageX - event.pageX;
        if (parseInt(this.width.replace("px", "")) + diff > this.block_size) {
          this.element.style.width = `${parseInt(this.width.replace("px", "")) +
            diff}px`;
          this.element.style.left = `${this.left.replace("px", "") - diff}px`;
        }
      }
      if (this.rightResizing) {
        let diff = this.pageX - event.pageX;
        if (parseInt(this.width.replace("px", "")) - diff > this.block_size) {
          this.element.style.width = `${parseInt(this.width.replace("px", "")) -
            diff}px`;
        }
      }
    },
    stopDrag() {
      if (this.dragging) {
        let diff = this.pageX - event.pageX;
        let days = Math.ceil(diff / this.block_size);
        if (days !== 0) {
          let task = this.tasks.find((task) => task.id === this.task_id);
          let start_date = moment(task.start_date).add(-days, "days");
          let end_date = moment(task.end_date).add(-days, "days");
          task["start_date"] = start_date.format("YYYY-MM-DD");
          task["end_date"] = end_date.format("YYYY-MM-DD");
        } else {
          this.element.style.left = `${this.left.replace("px", "")}px`;
        }
      }
      if (this.leftResizing) {
        let diff = this.pageX - event.pageX;
        let days = Math.ceil(diff / this.block_size);
        if (days !== 0) {
          let task = this.tasks.find((task) => task.id === this.task_id);
          let start_date = moment(task.start_date).add(-days, "days");
          let end_date = moment(task.end_date);
          if (end_date.diff(start_date, "days") <= 0) {
            task["start_date"] = end_date.format("YYYY-MM-DD");
          } else {
            task["start_date"] = start_date.format("YYYY-MM-DD");
          }
        } else {
          this.element.style.width = this.width;
          this.element.style.left = `${this.left.replace("px", "")}px`;
        }
      }
      if (this.rightResizing) {
        let diff = this.pageX - event.pageX;
        let days = Math.ceil(diff / this.block_size);
        if (days === 1) {
          this.element.style.width = `${parseInt(
            this.width.replace("px", "")
          )}px`;
        } else if (days <= 2) {
          days--;
          let task = this.tasks.find((task) => task.id === this.task_id);
          let end_date = moment(task.end_date).add(-days, "days");
          task["end_date"] = end_date.format("YYYY-MM-DD");
        } else {
          let task = this.tasks.find((task) => task.id === this.task_id);
          let start_date = moment(task.start_date);
          let end_date = moment(task.end_date).add(-days, "days");
          if (end_date.diff(start_date, "days") < 0) {
            task["end_date"] = start_date.format("YYYY-MM-DD");
          } else {
            task["end_date"] = end_date.format("YYYY-MM-DD");
          }
        }
      }
      this.dragging = false;
      this.leftResizing = false;
      this.rightResizing = false;
    },
    dragTask(dragTask) {
      this.task = dragTask;
    },
    dragTaskOver(overTask) {
      let deleteIndex;
      let addIndex;
      if (this.task.cat !== "category") {
        if (overTask.cat === "category") {
          let updateTask = this.tasks.find((task) => task.id === this.task.id);
          updateTask["category_id"] = overTask["id"];
        } else {
          if (overTask.id !== this.task.id) {
            this.tasks.map((task, index) => {
              if (task.id === this.task.id) deleteIndex = index;
            });
            this.tasks.map((task, index) => {
              if (task.id === overTask.id) addIndex = index;
            });
            this.tasks.splice(deleteIndex, 1);
            this.task["category_id"] = overTask["category_id"];
            this.tasks.splice(addIndex, 0, this.task);
          }
        }
      }
    },
    toggleCategory(task_id) {
      let category = this.categories.find(
        (category) => category.id === task_id
      );
      category["collapsed"] = !category["collapsed"];
    },
  },
  computed: {
    calendarViewWidth() {
      return this.inner_width - this.task_width;
    },
    calendarViewHeight() {
      return (
        this.inner_height -
        this.task_height -
        this.GANTT_HEADER_HEIGHT -
        this.SCROLLBAR_HEIGHT
      );
    },
    scrollDistance() {
      let start_date = moment(this.start_month);
      let between_days = this.today.diff(start_date, "days");
      return between_days * this.block_size;
    },
    lists() {
      let lists = [];
      this.categories.map((category) => {
        lists.push({ cat: "category", ...category });
        this.tasks.map((task) => {
          if (task.category_id === category.id && !category.collapsed) {
            lists.push({ cat: "task", ...task });
          }
        });
      });
      return lists;
    },
    taskBars() {
      let start_date = moment(this.start_month);
      let top = 10;
      let left;
      let between;
      let start;
      let style;
      return this.displayTasks.map((task) => {
        style = {};
        if (task.cat === "task") {
          let date_from = moment(task.start_date);
          let date_to = moment(task.end_date);
          between = date_to.diff(date_from, "days");
          between++;
          start = date_from.diff(start_date, "days");
          left = start * this.block_size;
          style = {
            top: `${top}px`,
            left: `${left}px`,
            width: `${this.block_size * between}px`,
          };
        }
        top = top + 40;
        return {
          style,
          task,
        };
      });
    },
    displayTasks() {
      let display_task_number = Math.floor(this.calendarViewHeight / 40);
      return this.lists.slice(
        this.position_id,
        this.position_id + display_task_number
      );
    },
  },
  created() {},
  mounted() {
    this.getCalendar();
    this.getWindowSize();
    this.$nextTick(() => {
      this.todayPosition();
    });
    window.addEventListener("resize", this.getWindowSize);
    window.addEventListener("wheel", this.windowSizeCheck);
    window.addEventListener("mousemove", this.mouseMove);
    window.addEventListener("mouseup", this.stopDrag);
    window.addEventListener("mousemove", this.mouseResize);
  },
};
</script>

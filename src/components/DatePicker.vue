<script setup lang="ts">
import { computed, ref } from "vue";

type Day = {
  date: Date;
  isSelected: boolean;
  isShadowed: boolean;
  isToday: boolean;
};

const { date = new Date() } = defineProps<{
  date?: Date | string;
}>();

const clearDate = (date: Date) => {
  return new Date(date.getFullYear(), date.getMonth(), date.getDate());
};

const selectedDate = ref<Date>(clearDate(new Date(date)));
const todayDate = ref<Date>(clearDate(new Date()));
const viewingMonthYear = ref<Date>(clearDate(new Date(date)));
const localeLang = ref<"ru" | "en">("en");

const days = computed<Day[]>(() => {
  const firstDay = new Date(
    viewingMonthYear.value.getFullYear(),
    viewingMonthYear.value.getMonth(),
    1,
  );
  const lastDay = new Date(
    viewingMonthYear.value.getFullYear(),
    viewingMonthYear.value.getMonth() + 1,
    0,
  );
  const firstWeekDay = firstDay.getDay();
  const result: Day[] = [];
  let gapFromPrevMonth = 0;
  switch (localeLang.value) {
    case "ru":
      gapFromPrevMonth = firstWeekDay != 0 ? firstWeekDay - 2 : 5;
      break;
    case "en":
      gapFromPrevMonth = firstWeekDay > 1 ? firstWeekDay - 1 : 0;
      break;
  }

  if (gapFromPrevMonth != firstWeekDay)
    for (let i = 0; i <= gapFromPrevMonth; i++) {
      result.push({
        date: new Date(
          viewingMonthYear.value.getFullYear(),
          viewingMonthYear.value.getMonth(),
          -(gapFromPrevMonth - i),
        ),
        isSelected: false,
        isShadowed: true,
        isToday: false,
      });
    }

  for (let i = 1; i <= lastDay.getDate(); i++) {
    const date = new Date(
      viewingMonthYear.value.getFullYear(),
      viewingMonthYear.value.getMonth(),
      i,
    );

    result.push({
      date: date,
      isSelected: +date == +selectedDate.value ? true : false,
      isShadowed: false,
      isToday: +date == +todayDate.value ? true : false,
    });
  }

  return result;
});

const formattedMonthYear = computed(() =>
  new Intl.DateTimeFormat(localeLang.value, {
    month: "short",
    year: "numeric",
  }).format(viewingMonthYear.value),
);

const setPrevMonth = () => {
  viewingMonthYear.value = new Date(
    viewingMonthYear.value.setMonth(viewingMonthYear.value.getMonth() - 1),
  );
};

const setNextMonth = () => {
  viewingMonthYear.value = new Date(
    viewingMonthYear.value.setMonth(viewingMonthYear.value.getMonth() + 1),
  );
};
</script>

<template>
  <div class="date-picker-container">
    <div class="date-picker-header">
      <span class="date-picker-change-month-button" @click="setPrevMonth()">
        <
      </span>
      <span>{{ formattedMonthYear }}</span>
      <span class="date-picker-change-month-button" @click="setNextMonth">
        >
      </span>
    </div>
    <table class="month-table" v-if="days.length > 0">
      <thead>
        <tr>
          <td v-for="day of days.slice(0, 7)" :key="day.date.getDate()">
            {{
              new Intl.DateTimeFormat(localeLang, {
                weekday: "short",
              }).format(day.date)
            }}
          </td>
        </tr>
      </thead>
      <tbody>
        <tr v-for="i in 6" :key="`week-row-${i}`">
          <td
            v-for="day of days.slice((i - 1) * 7, i * 7)"
            :key="`day-timestamp-${day.date.getTime()}`"
          >
            {{ !day.isShadowed ? day.date.getDate() : null }}
          </td>
        </tr>
      </tbody>
    </table>
    <div>
      <span>{{ localeLang }}</span>
    </div>
  </div>
</template>

<style scoped>
.date-picker-container {
  display: flex;
  flex-direction: column;
  border: 1px solid;
  width: 400px;
  padding: 10px;
}

.date-picker-header {
  width: 100%;
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}

.date-picker-change-month-button {
  cursor: pointer;
}

td {
  width: 50px;
  height: 50px;
  text-align: center;
}
</style>


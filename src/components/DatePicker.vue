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

const emit = defineEmits<{
  selectedDate: [selectedDate: Date];
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

const selectNewDate = (date: Date) => {
  selectedDate.value = date;
  emit("selectedDate", selectedDate.value);
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
    <table class="date-picker-container-calendar" v-if="days.length > 0">
      <thead>
        <tr>
          <th v-for="day of days.slice(0, 7)" :key="day.date.getDate()">
            {{
              new Intl.DateTimeFormat(localeLang, {
                weekday: "short",
              }).format(day.date)
            }}
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="i in 6" :key="`week-row-${i}`">
          <td
            v-for="day of days.slice((i - 1) * 7, i * 7)"
            :key="`day-timestamp-${day.date.getTime()}`"
            :class="{
              'date-picker-date-cell': true,
              'date-picker-date-selected': day.isSelected,
              'date-picker-date-today': day.isToday,
              'date-picker-date-not-shadowed': !day.isShadowed,
            }"
            @click="selectNewDate(day.date)"
          >
            {{ !day.isShadowed ? day.date.getDate() : null }}
          </td>
        </tr>
      </tbody>
    </table>
    <div>
      <select v-model="localeLang">
        <option value="en">EN-en</option>
        <option value="ru">RU-ru</option>
      </select>
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

.date-picker-container-calendar {
  table-layout: fixed;
  width: 100%;
  height: 100%;
}

.date-picker-change-month-button {
  cursor: pointer;
}

.date-picker-date-cell {
  width: 50px;
  height: 50px;
  text-align: center;
}

.date-picker-date-selected {
  border: 2px solid #48d4ff;
  height: 46px;
}

.date-picker-date-today {
  color: red;
}

.date-picker-date-not-shadowed {
  cursor: pointer;
}
</style>


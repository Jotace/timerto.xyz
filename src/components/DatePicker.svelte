<script>
  import cn from "clsx";
  import moment from "moment";
  import { onMount, createEventDispatcher } from "svelte";
  import Dropdown from "./Dropdown.svelte";

  // -----------------------
  // Props
  // -----------------------
  let className;
  export let title;
  export let minDate;
  export let maxDate;
  export let value;
  export { className as class };

  // -----------------------
  // Internal
  // -----------------------
  const dispatcher = createEventDispatcher();

  // -----------------------
  // Methods
  // -----------------------

  let state = { day: 0, month: 0, year: 0 };

  function setState(newState, callback) {
    state = { ...state, ...newState };
    callback && callback(state);
  }

  function getDate() {
    const { year, month, day } = state;
    return moment([year, month, day]).startOf("day");
  }

  function getYearSpan() {
    return {
      min: moment(minDate),
      max: moment(maxDate)
    };
  }

  function isValid(m1, m2, yearSpan = undefined) {
    yearSpan = yearSpan || getYearSpan();
    return (
      m1 &&
      m2 &&
      m1.isValid() &&
      m2.isValid() &&
      m2.isAfter(yearSpan.min) &&
      m1.isBefore(yearSpan.max)
    );
  }

  function getValidMonths(state) {
    let months = moment.months(true);
    const { year } = state;
    if (year) {
      const yearSpan = getYearSpan();
      return months.filter((_, index) => {
        let m = moment([year, index, 1]);
        return isValid(
          m.clone().startOf("month"),
          m.clone().endOf("month"),
          yearSpan
        );
      });
    }
    return months;
  }

  function getValidDays(state) {
    const days = new Array(32).fill(0).map((_, index) => index + 1);
    const { year, month } = state;
    if (year && month != null) {
      const yearSpan = getYearSpan();
      return days.filter(day => {
        let m = moment([year, month, day]);
        return isValid(
          m.clone().startOf("day"),
          m.clone().endOf("day"),
          yearSpan
        );
      });
    }
    return days;
  }

  function getYears() {
    const yearSpan = getYearSpan();
    const minYear = yearSpan.min.year();
    return new Array(yearSpan.max.year() - yearSpan.min.year() + 1)
      .fill(0)
      .map((_, index) => minYear + index)
      .map(year => year.toString());
  }

  function onYearChange() {
    const m = getDate();
    if (!isValid(m, m)) {
      setState({ month: 0 }, onMonthChange);
    } else {
      onMonthChange();
    }
  }

  function onMonthChange() {
    const m = getDate();
    if (!isValid(m, m)) {
      setState({ day: 1 }, onDateChange);
    } else {
      onDateChange();
    }
  }

  function onDateChange() {
    const m = getDate();
    if (isValid(m, m)) {
      dispatcher("change", { date: m.toDate() });
    }
  }

  function mapMonthsToOptions(months) {
    return months.map((month, index) => ({
      label: month[0].toUpperCase() + month.substr(1),
      value: index.toString()
    }));
  }

  function mapDaysToOptions(days) {
    return days.map(day => ({
      label: day.toString(),
      value: day.toString()
    }));
  }

  // -----------------------
  // Lifecycle
  // -----------------------
  onMount(() => {
    const m = moment(value);
    if (m.isValid()) {
      setState({ year: m.year(), month: m.month(), day: m.day() });
    }
  });
</script>

<div class={cn('datepicker', className)}>
  <label
    class="block uppercase tracking-wide text-gray-700 text-xs font-bold mb-2"
    htmlFor="grid-last-name">
    {title}
  </label>
  <div class="flex flex-row">
    <Dropdown
      class="relative w-3/12 mr-2"
      options={mapDaysToOptions(getValidDays(state))}
      selectedValue={state.day.toString()}
      on:change={evt => setState({ day: parseInt(evt.target.value) }, onDateChange)} />

    <Dropdown
      class="relative w-6/12 mr-2"
      options={mapMonthsToOptions(getValidMonths(state))}
      selectedValue={state.month.toString()}
      on:change={evt => setState({ month: parseInt(evt.target.value) }, onMonthChange)} />

    <Dropdown
      class="relative w-3/12"
      options={getYears().map(year => ({ label: year, value: year }))}
      selectedValue={state.year.toString()}
      on:change={evt => setState({ year: parseInt(evt.target.value) }, onYearChange)} />
  </div>
</div>
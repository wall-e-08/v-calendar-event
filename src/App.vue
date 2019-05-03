<template>
  <div id="app">
    <SimpleCalendar/>
  </div>
</template>

<script>
import SimpleCalendar from './components/Calendar'

export default {
    name: 'App',
    components: {
        SimpleCalendar
    }
}
</script>


<style src="./assets/scss/default.scss" lang="scss"></style>

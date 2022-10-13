<template>
  <div>
    <index-task v-if="indexStyle==3"></index-task>
  </div>
</template>

<script>
  import IndexTask from "./IndexTask"

  export default {
    name: "Analysis",
    components: {
      IndexTask
    },
    data() {
      return {
        indexStyle:3
      }
    },
    created() {

    },
    methods: {

    }
  }
</script>
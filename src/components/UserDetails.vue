<script>
export default {
  props: {
    header: {
      type: String,
      default: 'Details'
    },
    id: Object,
    name: Object,
    picture: String,
    phone: String,
    cell: String,
    location: Object
  }
}
</script>

<template>
  <div class="details border rounded p-4 shadow-2xl">
    <h3 class="text-xl font-bold">{{header}}</h3>
    <ul>
      <li><img class="rounded my-4" :src="picture" :alt="`${name.first} ${name.last}`" /></li>
      <li><span class="font-bold">id:</span> {{id}}</li>
      <li><span class="font-bold">location:</span> <span>{{location.street.name}}</span> <span>{{location.street.number}}</span>,
        <span>{{location.city}}</span>, <span>{{location.postcode}}</span>, 
        <span>{{location.state}}</span>, <span>{{location.country}}</span>
      </li>
      <li><span class="font-bold">phone:</span> {{phone}}</li>
      <li><span class="font-bold">cell:</span> {{cell}}</li>
    </ul>
  </div>
</template>
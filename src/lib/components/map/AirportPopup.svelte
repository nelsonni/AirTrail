<script lang="ts">
  import NumberFlow from '@number-flow/svelte';

  import { pluralize } from '$lib/utils';
  import { formatAsDate } from '$lib/utils/datetime';

  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  let { data }: { data: any } = $props();
</script>

<div class="min-w-[18rem]">
  <div class="flex flex-col px-3 pt-3">
    <h3 class="font-thin text-muted-foreground">Airport</h3>
    <h4 class="flex items-center text-lg">
      <img
        src="https://flagcdn.com/{data.country.toLowerCase()}.svg"
        alt={data.country}
        class="w-8 h-5 mr-2"
      />
      {data.iata ?? data.code} - {data.name}
    </h4>
  </div>
  <div class="h-[1px] bg-muted my-3" />
  <div class="grid grid-cols-[repeat(3,_1fr)] px-3">
    <h4 class="font-semibold">
      <NumberFlow value={data.departures} />
      <span class="font-thin text-muted-foreground"
        >{pluralize(data.departures, 'departure')}</span
      >
    </h4>
    <h4 class="font-semibold">
      <NumberFlow value={data.arrivals} />
      <span class="font-thin text-muted-foreground"
        >{pluralize(data.arrivals, 'arrival')}</span
      >
    </h4>
    <h4 class="font-semibold">
      <NumberFlow value={data.airlines.length} />
      <span class="font-thin text-muted-foreground"
        >{pluralize(data.airlines.length, 'airline')}</span
      >
    </h4>
  </div>
  <div class="h-[1px] bg-muted my-3" />
  <div class="px-3 pb-3">
    <div class="grid grid-cols-[repeat(3,_1fr)]">
      <h3 class="font-semibold">Route</h3>
      <h3 class="font-semibold">Date</h3>
      <h3 class="font-semibold">Airline</h3>
    </div>
    {#each data.flights
      .slice(0, 5)
      .sort((a, b) => b.date.getTime() - a.date.getTime()) as flight}
      <div class="grid grid-cols-[repeat(3,_1fr)]">
        <h4 class="font-thin">{flight.route}</h4>
        <h4 class="font-thin">{formatAsDate(flight.date, true, true)}</h4>
        <h4 class="font-thin">{flight.airline}</h4>
      </div>
    {/each}
    {#if data.flights.length > 5}
      <h4 class="font-thin text-muted-foreground">
        +{data.flights.length - 5} more
      </h4>
    {/if}
  </div>
</div>

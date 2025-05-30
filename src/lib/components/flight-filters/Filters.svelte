<script lang="ts">
  import DateFilter from './DateFilter.svelte';
  import SelectFilter from './SelectFilter.svelte';

  import {
    defaultFilters,
    type FlightFilters,
  } from '$lib/components/flight-filters/types';
  import { Button } from '$lib/components/ui/button';
  import type { Airport } from '$lib/db/types';
  import type { FlightData } from '$lib/utils';

  let {
    flights = $bindable(),
    filters = $bindable(),
  }: {
    flights: FlightData[];
    filters: FlightFilters;
  } = $props();

  const showClear = $derived.by(
    () =>
      filters.departureAirports.length ||
      filters.arrivalAirports.length ||
      filters.fromDate ||
      filters.toDate,
  );

  const uniqueAirports = (
    flights: FlightData[],
    airportSelector: (f: FlightData) => Airport,
  ) => {
    const seenICAO = new Set();
    return flights
      .map(airportSelector)
      .filter((airport) => {
        if (seenICAO.has(airport.code)) {
          return false;
        } else {
          seenICAO.add(airport.code);
          return true;
        }
      })
      .map((airport) => ({
        value: airport.code,
        label: `${airport.iata ?? airport.code} | ${airport.name}`,
      }));
  };

  const departureAirports = $derived.by(() => {
    if (!flights) return [];
    return uniqueAirports(flights, (f) => f.from);
  });

  const arrivalAirports = $derived.by(() => {
    if (!flights) return [];
    return uniqueAirports(flights, (f) => f.to);
  });

  const getAircraftRegistrationsByFrequency = (flights: FlightData[]) => {
    if (!flights) return [];

    const regFrequencyMap = flights.reduce<Map<string, number>>(
      (acc, flight) => {
        if (flight.aircraftReg) {
          acc.set(flight.aircraftReg, (acc.get(flight.aircraftReg) || 0) + 1);
        }
        return acc;
      },
      new Map(),
    );

    return Array.from(regFrequencyMap.entries())
      .map(([registration, count]) => ({
        registration,
        flightCount: count,
      }))
      .sort((a, b) => b.flightCount - a.flightCount)
      .map(({ registration }) => ({
        value: registration,
        label: registration,
      }));
  };

  const aircraftRegs = $derived.by(() =>
    getAircraftRegistrationsByFrequency(flights),
  );
</script>

<SelectFilter
  bind:filterValues={filters.departureAirports}
  title="Departure Airport"
  placeholder="Search departure airports"
  disabled={flights.length === 0}
  options={departureAirports}
/>
<SelectFilter
  bind:filterValues={filters.arrivalAirports}
  title="Arrival Airport"
  placeholder="Search arrival airports"
  disabled={flights.length === 0}
  options={arrivalAirports}
/>
<DateFilter
  bind:date={filters.fromDate}
  title="From"
  iconDirection="up"
  disabled={flights.length === 0}
/>
<DateFilter
  bind:date={filters.toDate}
  title="To"
  iconDirection="down"
  disabled={flights.length === 0}
/>
<SelectFilter
  bind:filterValues={filters.aircraftRegs}
  title="Tail Number"
  placeholder="Search tail numbers"
  disabled={!aircraftRegs.length}
  options={aircraftRegs}
/>
{#if showClear}
  <Button
    variant="ghost"
    class="h-8 px-2 lg:px-3"
    onclick={() => {
      filters = defaultFilters;
    }}
  >
    Clear Filters
  </Button>
{/if}

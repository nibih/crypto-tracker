<script lang="ts">
  import './app.css';
  import Select from 'svelte-select';
  //   import apolloclient
  import ApolloClient, { gql } from 'apollo-boost';

  // set client and query data with typescript
  const client: any = new ApolloClient({
    uri: 'https://countries.trevorblades.com/',
  });
  let countries: any = [];
  //   # get flag, name and currency
  //   #   get china, us, india, japan, indonesia
  // filter by code
  client
    .query({
      query: gql`
        query {
          countries(filter: { code: { in: ["CN", "ID", "IN", "US", "JP"] } }) {
            name
            emoji
            currency
          }
        }
      `,
    })
    .then((result: any) => {
      countries = result.data.countries;
    });

  //   items have value of currency and label of name
  $: items = countries.map((country: any) => {
    return {
      value: country.currency,
      label: `${country.emoji} ${country.name}`,
    };
  });

  let value = { value: 'USD', label: 'Select a Country' };

  //   get BTC/USDT, ETH/USDT, DOGE/USDT FROM BINANCE WEBSOCKET
  let binance = new WebSocket(
    'wss://stream.binance.com:9443/ws/btcusdt@ticker/ethusdt@ticker/dogeusdt@ticker/btcusd@ticker/btceur@ticker/etheur@ticker/dogeeur@ticker'
  );
  let binanceData: any = [];
  let btc = {};
  let eth = {};
  let doge = {};

  binance.onmessage = (event) => {
    binanceData = JSON.parse(event.data);
    if (binanceData.s === 'BTCUSDT') {
      btc = binanceData;
    } else if (binanceData.s === 'ETHUSDT') {
      eth = binanceData;
    } else if (binanceData.s === 'DOGEUSDT') {
      doge = binanceData;
    }
    // append to each variable the value of the euro price
    if (binanceData.s === 'BTCEUR') {
      btc['eur'] = binanceData.c;
    } else if (binanceData.s === 'ETHEUR') {
      eth['eur'] = binanceData.c;
    } else if (binanceData.s === 'DOGEEUR') {
      doge['eur'] = binanceData.c;
    }

    let eur_usd = fetch(
      'https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/eur/usd.json'
    )
      .then((response) => response.json())
      .then((data) => {
        if (data) {
          btc['usd'] = data.usd * btc['eur'];
          eth['usd'] = data.usd * eth['eur'];
          doge['usd'] = data.usd * doge['eur'];
        }
      });
    let eur_cny = fetch(
      'https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/eur/cny.json'
    )
      .then((response) => response.json())
      .then((data) => {
        btc['cny'] = data.cny * btc['eur'];
        eth['cny'] = data.cny * eth['eur'];
        doge['cny'] = data.cny * doge['eur'];
      });
    let eur_jpy = fetch(
      'https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/eur/jpy.json'
    )
      .then((response) => response.json())
      .then((data) => {
        btc['jpy'] = data.jpy * btc['eur'];
        eth['jpy'] = data.jpy * eth['eur'];
        doge['jpy'] = data.jpy * doge['eur'];
      });
    let eur_idr = fetch(
      'https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/eur/idr.json'
    )
      .then((response) => response.json())
      .then((data) => {
        btc['idr'] = data.idr * btc['eur'];
        eth['idr'] = data.idr * eth['eur'];
        doge['idr'] = data.idr * doge['eur'];
      });
    let eur_inr = fetch(
      'https://cdn.jsdelivr.net/gh/fawazahmed0/currency-api@1/latest/currencies/eur/inr.json'
    )
      .then((response) => response.json())
      .then((data) => {
        btc['inr'] = data.inr * btc['eur'];
        eth['inr'] = data.inr * eth['eur'];
        doge['inr'] = data.inr * doge['eur'];
      });
  };

  function handleSelect(event) {
    //log first 3 letters of currency
    value.value = event.detail.value.slice(0, 3);
  }
</script>

<main>
  <Select {items} bind:value on:select={handleSelect} />
  <div class="flex justify-around my-2 space-x-4">
    <div>
      <div class="flex justify-between border p-5 drop-shadow-xl">
        <div>
          BTC/USDT:
          <h2 class="font-bold">
            {btc.c}
          </h2>
          <h3 class="font-semibold">volume: {btc.q}</h3>
        </div>
        {#if btc.P < 0}
          <p class="text-red-500">
            {btc?.P}%
          </p>
        {:else if btc.P > 0}
          <p class="text-green-500">
            {btc?.P}%
          </p>
        {:else}
          <p>
            {btc?.P}%
          </p>
        {/if}
      </div>
      {value.value}: {btc[value.value.toLowerCase()]}
    </div>
    <div>
      <div class="flex justify-between border p-5 drop-shadow-xl">
        <div>
          ETH/USDT:
          <h2 class="font-bold">
            {eth.c}
          </h2>
          <h3 class="font-semibold">volume: {eth.q}</h3>
        </div>
        {#if eth.P < 0}
          <p class="text-red-500">
            {eth?.P}%
          </p>
        {:else if eth.P > 0}
          <p class="text-green-500">
            {eth?.P}%
          </p>
        {:else}
          <p>
            {eth?.P}%
          </p>
        {/if}
      </div>
      {value.value}: {eth[value.value.toLowerCase()]}
    </div>
    <div>
      <div class="flex justify-between border p-5 drop-shadow-xl">
        <div>
          DOGE/USDT:
          <h2 class="font-bold">
            {doge.c}
          </h2>
          <h3 class="font-semibold">volume: {doge.q}</h3>
        </div>
        <!-- text is red if negative -->
        {#if doge.P < 0}
          <p class="text-red-500">
            {doge?.P}%
          </p>
        {:else if doge.P > 0}
          <p class="text-green-500">
            {doge?.P}%
          </p>
        {:else}
          <p>
            {doge?.P}%
          </p>
        {/if}
      </div>
      {value.value}: {doge[value.value.toLowerCase()]}
    </div>
  </div>
</main>

<style>
  /* make main 1/2 of screen */
  main {
    width: 80%;
    margin: 5rem auto;
  }
</style>

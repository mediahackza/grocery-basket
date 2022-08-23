<script>
  import { fly, fade } from 'svelte/transition'
  import { onMount } from 'svelte'
  let promise = false
  let loaddata = false
  let data = []
  let dates = []
  let reports = []
  let dropdown = []
  let curData = []
  let totals = []
  let selected = null

  function formatDate(date) {
    let d = date.split('-')
    let year = d[0]
    let month = d[1]
    return `${month} ${year}`
  }
  async function getData() {
    await fetch(
      'https://api.datadesk.co.za/csvjson.php?table=dd_groceries_basket_1026889'
    )
      .then((response) => response.json())
      .then((response) => {
        console.log(response)

        data = response.filter(
          (d) => (d.published === 'yes') & (d.included === 'yes')
        )
        dates = [...new Set(data.map((x) => x.report_date))]
        reports = [...new Set(data.map((x) => x.report_period))]

        dates.forEach((d, i) => {
          dropdown.push({
            date: d,
            name: reports[i],
          })

          console.log(dropdown)
        })

        curData = data.filter((d) => d.report_date === dates[dates.length - 1])
        selected = dates[dates.length - 1]
        promise = true
        loaddata = true
      })
  }
  function changeData() {
    loaddata = false
    setTimeout(() => {
      curData = data.filter((d) => d.report_date === selected)
      loaddata = true
    }, 500)
  }

  function total(curData, store) {
    let val = 0
    curData.forEach((d) => {
      val += +d[store]
    })

    return val.toFixed(2)
  }

  onMount(() => {
    getData().then(() => {
      console.log(data)
      console.log(dates)
    })
  })
</script>

<main>
  <h1>Grocery Basket</h1>

  {#if !promise}
    <div class="placeholder">
      <div class="basket-icon">
        <img src="./images/basket.svg" alt="Shopping basket icon" />
      </div>
      Hang on a sec, fetching the basket data...
    </div>
  {/if}

  {#if promise}
    <div class="dropdown">
      Choose a report: <select
        bind:value={selected}
        on:change={() => changeData()}
      >
        {#each dropdown as d, i}
          <option value={d.date}>{d.name}</option>
        {/each}
      </select>
    </div>

    <div class="table">
      <table>
        <tr
          ><th>Report Date</th><th>Item</th><th>Checkers</th><th>Pick n Pay</th
          ><th>Spar</th><th>Woolworths</th><th>Shoprite</th></tr
        >
        {#if loaddata}
          {#each curData as cd}
            <tr transition:fade={{ duration: 500 }}
              ><td>{cd.report_date}</td><td>{cd.type}</td><td>R{cd.checkers}</td
              ><td>R{cd.spar}</td><td>R{cd.pick_n_pay}</td><td
                >R{cd.woolworths}</td
              ><td>R{cd.shoprite}</td></tr
            >
          {/each}
          {#if curData}
            <tr
              ><td /><td />
              <td class="total">{total(curData, 'checkers')}</td>
              <td class="total">{total(curData, 'pick_n_pay')}</td>
              <td class="total">{total(curData, 'spar')}</td>
              <td class="total">{total(curData, 'woolworths')}</td>
              <td class="total">{total(curData, 'shoprite')}</td>
            </tr>
          {/if}
        {/if}
      </table>
    </div>
  {/if}
</main>

<style>
  main {
    width: 90%;
    max-width: 900px;
    margin-left: auto;
    margin-right: auto;
  }
  .dropdown {
    /* margin-bottom: 20px; */
    background: #eee;
    font-size: 0.8rem;
    padding: 5px 20px 0px 20px;
  }
  .total {
    background: #b72b2c;
    color: #fff;
  }
  .placeholder {
    margin-top: 20vh;
    width: 100%;
    text-align: center;
    font-family: 'Titillium Web', Arial, Helvetica, sans-serif;
    font-size: 1.6rem;
    font-weight: 700;
    color: #b72b2c;
  }
  .basket-icon img {
    width: 50px;
  }
</style>

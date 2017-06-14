<template>
  <div class="hello">
    <div class="container">
      <div class="row">
        <h2 class="title">Realtime Chart with Vue and Pusher</h2>
        <h3 class="subtitle">Expense and Income Tracker</h3>
        <line-chart :chart-data="datacollection"></line-chart>
      </div>
    </div>
    <div class="container">
      <div class="row">
        <form class="form" @submit.prevent="addExpenses">
          <h4>Add New Entry</h4>
          <div class="form-group">
            <label>Expenses</label>
            <input class="form-control" placeholder="How much did you spend?" type="number" v-model="expenseamount" required>
          </div>
          <div class="form-group">
            <label>Income</label>
            <input class="form-control" placeholder="How much did you earn?" type="number" v-model="incomeamount" required>
          </div>
          <div class="form-group">
            <label>Date</label>
            <input class="form-control" placeholder="Date" type="date" v-model="entrydate" required>
          </div>
          <div class="form-group">
            <button class="btn btn-primary">Add New Entry</button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'
  import moment from 'moment'
  import Pusher from 'pusher-js'
  import LineChart from '@/components/LineChart'

  const socket = new Pusher('3e6b0e8f2442b34330b7', {
    cluster: 'eu',
    encrypted: true
  })
  const channel = socket.subscribe('finance')

  export default {
    name: 'home',
    components: {LineChart},
    data () {
      return {
        expense: null,
        income: null,
        date: null,
        expenseamount: null,
        incomeamount: null,
        datacollection: null,
        entrydate: null
      }
    },
    created () {
      this.fetchData()
      this.fillData()
    },
    mounted () {
      this.fillData()
    },
    methods: {
      fillData () {
        axios.get('/finances')
          .then(response => {
            console.log(response.data.data)
            let results = response.data.data
            let dateresult = results.map(a => a.date)
            let expenseresult = results.map(a => a.expense)
            let incomeresult = results.map(a => a.income)

            this.expense = expenseresult
            this.income = incomeresult
            this.date = dateresult

            this.datacollection = {
              labels: this.date,
              datasets: [
                {
                  label: 'Expense',
                  backgroundColor: '#f87979',
                  data: this.expense
                },
                {
                  label: 'Income',
                  backgroundColor: '#5bf8bf',
                  data: this.income
                }
              ]
            }
          })
          .catch(error => {
            console.log(error)
          })
      },
      addExpenses () {
        let expense = this.expenseamount
        let income = this.incomeamount
        let today = moment(this.entrydate).format('MMMM Do YYYY')

        axios.post('/expense/add', {
          expense: expense,
          income: income,
          date: today
        })
          .then(response => {
            this.expenseamount = ''
            this.incomeamount = ''
            this.entrydate = ''
            channel.bind('new-expense', function (data) {
              let results = data.newExpense.data

              let dateresult = results.map(a => a.date)
              let expenseresult = results.map(a => a.expense)
              let incomeresult = results.map(a => a.income)

              this.expense = expenseresult
              this.income = incomeresult
              this.date = dateresult

              this.datacollection = {
                labels: this.date,
                datasets: [
                  {
                    label: 'Expense',
                    backgroundColor: 'transparent',
                    pointBorderColor: '#f87979',
                    data: this.expense
                  },
                  {
                    label: 'Income',
                    backgroundColor: 'transparent',
                    pointBorderColor: '#5bf8bf',
                    data: this.income
                  }
                ]
              }
            })
          })
          .catch(error => {
            console.log(error)
          })
      },

      fetchData () {
        channel.bind('new-expense', data => {
          let _results = data.newExpense.data
          let dateresult = _results.map(a => a.date)
          let expenseresult = _results.map(a => a.expense)
          let incomeresult = _results.map(a => a.income)
          this.expense = expenseresult
          this.income = incomeresult
          this.date = dateresult
          this.datacollection = {
            labels: this.date,
            datasets: [
              {
                label: 'Expense Charts',
                backgroundColor: '#f87979',
                data: this.expense
              },
              {
                label: 'Income Charts',
                backgroundColor: '#5bf8bf',
                data: this.income
              }
            ]
          }
        })
      }
    }
  }
</script>


<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

  .title {
    text-align: center;
    margin-top: 40px;
  }
  .subtitle {
    text-align: center;
  }
  .form {
    max-width: 600px;
    width: 100%;
    margin: 20px auto 0 auto;
  }
  .form h4 {
    text-align: center;
    margin-bottom: 30px;
  }

  h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>

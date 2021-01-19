<template>
  <div>
    <Preloader v-if="loading"/>
    <div class="container pt-3 text-center" v-if="accounts.length">
      <div>
        <button class="btn btn-dark mb-2" @click="createAccount">Создать счет</button>
      </div>
      <div class="error text-danger border  border-danger d-inline-block p-2 rounded" v-if="error">
        {{ error }}
      </div>
      <table class="table">
        <thead>
        <tr>
          <th>#</th>
          <th>Баланс</th>
          <th></th>
        </tr>
        </thead>
        <tbody>
        <tr class="align-middle" v-for="account in paginatedAccounts" :key="account.id">
          <td>{{ account.id }}</td>
          <td>{{ parseInt(account.balance) }}</td>
          <td>
            <button type="button" class="btn btn-success me-2 mb-2 mt-2" @click="clearError" data-bs-toggle="modal"
                    :data-bs-target="'#modal-add-balance-'+account.id">
              Пополнить счет
            </button>
            <div class="modal fade" :id="'modal-add-balance-'+account.id" tabindex="-1"
                 :aria-labelledby="'modal-add-balance-'+account.id+'Label'" aria-hidden="true">
              <div class="modal-dialog">
                <div class="modal-content">
                  <form @submit.prevent="addAmount(account.id,$event)">
                    <div class="modal-body">
                      <label class="form-label">
                        Сумма
                        <input class="form-control" type="number" min="1" v-model="amount" required>
                      </label>
                    </div>
                    <div class="modal-footer">
                      <button class="btn btn-dark" type="submit">Пополнить</button>
                      <button type="button" class="btn btn-secondary js-close-modal" data-bs-dismiss="modal">Close
                      </button>
                    </div>
                  </form>
                </div>
              </div>
            </div>
            <button type="button" class="btn btn-light me-2 mb-2 mt-2" data-bs-toggle="modal" @click="clearError"
                    :data-bs-target="'#modal-spend-balance-'+account.id">
              Потратить со счета
            </button>
            <div class="modal fade" :id="'modal-spend-balance-'+account.id" tabindex="-1"
                 :aria-labelledby="'modal-spend-balance-'+account.id+'Label'" aria-hidden="true">
              <div class="modal-dialog">
                <div class="modal-content">
                  <form @submit.prevent="addTransaction(account.id,$event)">
                    <div class="modal-body">
                      <div>
                        <label class="form-label">
                          Название
                          <input class="form-control" type="text" v-model="name" required>
                        </label>
                      </div>
                      <div>
                        <label class="form-label">
                          Сумма
                          <input class="form-control" type="number" min="1" v-model="transaction" required>
                        </label>
                      </div>
                    </div>
                    <div class="modal-footer">
                      <button class="btn btn-dark" type="submit">Потратить</button>
                      <button type="button" class="btn btn-secondary js-close-modal" data-bs-dismiss="modal">Close
                      </button>
                    </div>
                  </form>
                </div>
              </div>
            </div>
            <button class="btn btn-info me-2  mb-2 mt-2" type="button" @click="getTransactions(account.id,$event)"
                    data-bs-toggle="modal" :data-bs-target="'#modal-transaction-'+account.id">
              История счета
            </button>
            <div class="modal fade" :id="'modal-transaction-'+account.id" tabindex="-1"
                 :aria-labelledby="'modal-transaction-'+account.id+'Label'" aria-hidden="true">
              <div class="modal-dialog">
                <div class="modal-content">
                  <div class="modal-body">
                    <table class="table">
                      <thead>
                      <tr>
                        <th>#</th>
                        <th>Дата</th>
                        <th>Название</th>
                        <th>Сумма</th>
                      </tr>
                      </thead>
                      <tbody>
                      <tr v-for="historyItem in  history">
                        <td>{{ historyItem.id }}</td>
                        <td>{{ toDate(historyItem.date) }}</td>
                        <td>{{ historyItem.merchant }}</td>
                        <td>{{ parseInt(historyItem.amount) }}</td>
                      </tr>
                      </tbody>
                    </table>
                  </div>
                  <div class="modal-footer">
                    <button type="button" class="btn btn-secondary js-close-modal" data-bs-dismiss="modal">Close
                    </button>
                  </div>
                </div>
              </div>
            </div>
            <button class="btn btn-danger mb-2  mt-2" type="button" @click="removeAccount(account.id)">
              Удалить счет
            </button>
          </td>
        </tr>
        </tbody>
      </table>
      <nav aria-label="Page navigation"
           v-if="paginatedAccounts.length>=page.size || page.length === pageCount -1 && page.length !==0">
        <ul class="pagination justify-content-center">
          <li class="page-item">
            <button class='page-link' :disabled="page.length ===0" @click="prevPage">Previous</button>
          </li>
          <li class="page-item">
            <button @click="nextPage" :disabled="page.length >= pageCount -1" class='page-link'>Next</button>
          </li>
        </ul>
      </nav>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import Preloader from '@/components/Preloader';

const token = '54e68fd62e66cbde2881e68a124aab98b6c5d220';
const baseUrl = 'http://localhost:8000/api/bank';

export default {
  name: 'App',
  components: {Preloader},
  data() {
    return {
      accounts: [],
      loading: true,
      amountForm: false,
      amount: 1,
      transaction: 1,
      name: '',
      history: [],
      error: null,
      page: {
        length: 0,
        size:3
      }
    }
  },
  created() {
    this.getAccounts();
  },
  computed: {
    paginatedAccounts() {
      const start = this.page.length * this.page.size,
          end = start + this.page.size;
      return this.accounts.slice(start, end)
    },
    pageCount() {
      let l = this.accounts.length,
          s = this.page.size;
      return Math.ceil(l / s);
    }
  },
  methods: {
    createAccount() {
      axios.post(`${baseUrl}/account/`, {}, {headers: {Authorization: 'Token ' + token}})
          .then(res => {
            this.accounts.push(res.data);
          })
    },
    getAccounts() {
      axios.get(`${baseUrl}/account/`, {
        headers: {Authorization: 'Token ' + token}
      })
          .then(res => {
            this.loading = false;
            this.accounts = res.data;
          })
    },
    addAmount(accountID, $event) {
      axios.post(`${baseUrl}/action/`, {
        amount: this.amount,
        account: accountID
      }, {headers: {Authorization: 'Token ' + token}})
          .then(res => {
            for (let i = 0; i < this.accounts.length; i++) {
              if (this.accounts[i].id === res.data.account) {
                this.accounts[i].balance = parseInt(this.accounts[i].balance) + parseInt(res.data.amount);
                break;
              }
            }
            this.amount = 1;
          })
          .finally(() => {
            const modal = document.getElementById(`modal-add-balance-${accountID}`);
            modal.querySelector(".js-close-modal").click()
          })
    },
    getAmount() {
      axios.get(`${baseUrl}/action/`, {
        headers: {Authorization: 'Token ' + token}
      })
    },
    addTransaction(accountID) {
      axios.post(`${baseUrl}/transaction/`, {
        amount: this.transaction,
        account: accountID,
        merchant: this.name
      }, {headers: {Authorization: 'Token ' + token}})
          .then(res => {
            for (let i = 0; i < this.accounts.length; i++) {
              if (this.accounts[i].id === res.data.account) {
                this.accounts[i].balance = parseInt(this.accounts[i].balance) - parseInt(res.data.amount);
                break;
              }
            }
          })
          .catch(e => {
            this.error = e.response.data.error;
            console.error(e.response.data.error)
          })
          .finally(() => {
            this.transaction = 1;
            this.name = '';
            const modal = document.getElementById(`modal-spend-balance-${accountID}`);
            modal.querySelector(".js-close-modal").click()
          })
    },
    getTransactions(accountID) {
      this.history = [];
      this.error = null;
      this.loading = true;

      function getTransactions() {
        return axios.get(`${baseUrl}/transaction/`, {headers: {Authorization: 'Token ' + token}});
      }

      function getActions() {
        return axios.get(`${baseUrl}/action/`, {headers: {Authorization: 'Token ' + token}});
      }

      Promise.all([getTransactions(), getActions()])
          .then(results => {
            let arr = [];
            arr.push(...results[0].data);
            arr.push(...results[1].data);
            const filteredArr = arr.filter(item => item.account === accountID)
            if (filteredArr.length) {
              this.history = filteredArr;
            }
            this.loading = false;
          });
    },
    removeAccount(id) {
      axios.delete(`${baseUrl}/account/${id}`, {headers: {Authorization: 'Token ' + token}})
          .finally(() => {
            const index = this.paginatedAccounts.findIndex(item => item.id === id);
            return this.accounts.splice(index, 1)
          })
    },
    clearError() {
      this.error = null;
    },
    toDate(date) {
      function isoFormatDMY(currentDate) {
        function pad(n) {
          return (n < 10 ? '0' : '') + n
        }

        return pad(currentDate.getUTCDate()) + '/' + pad(currentDate.getUTCMonth() + 1) + '/' + currentDate.getUTCFullYear();
      }

      function parseISOString(s) {
        const b = s.split(/\D+/);
        return new Date(Date.UTC(b[0], --b[1], b[2], b[3], b[4], b[5], b[6]));
      }

      const dateIso = parseISOString(date);

      return isoFormatDMY(dateIso)
    },
    nextPage() {
      this.page.length++;
    },
    prevPage() {
      this.page.length--;
    },
  }
}
</script>

<style lang="scss">
.hide {
  display: none;
}
</style>

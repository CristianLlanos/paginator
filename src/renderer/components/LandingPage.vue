<template>
  <div id="wrapper">
    <h1>Paginator</h1>
    <form class="form">

      <div class="field">
        <label class="label">Request</label>
        <div class="control">
          <textarea rows="8" class="input" type="text" placeholder="Request" v-model="request"></textarea>
        </div>
      </div>

      <div class="field">
        <label class="label">Next page</label>
        <div class="control">
          <input class="input" type="text" placeholder="Next page" v-model="nextPage">
        </div>
      </div>
      <div class="field">
        <label class="label">Take</label>
        <div class="control">
          <input class="input" type="text" placeholder="Take" v-model="take">
        </div>
      </div>
      <div class="field">
        <label class="label">Stop at</label>
        <div class="control">
          <input class="input" type="text" placeholder="Stop at" v-model="stopAt">
        </div>
      </div>

      <div class="field is-grouped">
        <div class="control">
          <button class="button is-outlined is-primary" @click="start()">Start</button>
        </div>
        <div class="control">
          <button class="button is-outlined is-primary" @click="stop()">Stop</button>
        </div>
        <div class="control">
          <button class="button is-outlined is-primary" @click="next()">Next</button>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
  function status (response) {
    if (response.status >= 200 && response.status < 300) {
      return Promise.resolve(response)
    } else {
      return Promise.reject(new Error(response.statusText))
    }
  }

  export default {
    name: 'landing-page',
    data () {
      return {
        nextPage: 1,
        take: 1000,
        stopAt: 1,
        pause: true,
        processed: {},
        request: JSON.stringify({
          url: 'http://localhost:8888',
          method: 'get',
          params: {
            page: '{page}',
            take: '{take}'
          },
          data: {},
          headers: {}
        }, null, 4)
      }
    },
    methods: {
      start () {
        this.pause = false

        if (this.nextPage > this.stopAt) {
          return
        }

        this.next().then(() => {
          if (this.pause) {
            return
          }
          this.start()
        })
      },
      stop () {
        this.pause = true
      },
      next () {
        return this.hit(this.request, this.nextPage, this.take)
      },
      hit (request, page, take) {
        return this.$http.request(this.settings(page, take)).then(status).then((response) => {
          console.log(response)
          let current = page
          let key = page + '-' + take
          this.nextPage = page + 1
          this.take = take
          this.request = request
          if (!this.processed[key]) {
            this.$set(this.processed, key, {
              page: current,
              take,
              tries: []
            })
          }

          this.processed[key].tries.push({
            data: response.data,
            status: response.status,
            statusText: response.statusText,
            request
          })

          if (response.status === 230) {
            this.pause = true
          }
        })
      },
      settings (page, take) {
        let settings = this.request.replace('{page}', page).replace('{take}', take)
        return JSON.parse(settings)
      }
    }
  }
</script>

<style>

  #wrapper {
    padding: 10px;
  }

  #wrapper textarea {
    height: auto;
  }

</style>

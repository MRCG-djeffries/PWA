<template>
  <div>
    <h1>Randomisation Home Page</h1>

    <div class="random">
      <div class="box">
        <button @click="doSomething">Randomise</button>
        <p>Counter: {{ counter }}</p>
        <p>Message: {{ message }}</p>
        <p>Group: {{ group.replace('group ', '') }}</p>
        <p>syncMess: {{ syncMess }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import PouchDB from 'pouchdb'

const db = new PouchDB('edrandodatabase')
const remoteDB = new PouchDB(
  'https://admin:Fajara2017@stats4.mrc.gm:6984/edrandodatabase',
)

export default {
  name: 'Randomisation',
  data: () => {
    return {
      counter: 0,
      group: '',
      message: 'Hello world!',
      pouchDbSyncActiveEvent: false,
      pouchDbSyncChangeEvent: false,
      syncMess: '',
    }
  },
  computed: {
    id() {
      return this.counter.toString()
    },
  },
  methods: {
    async doSomething() {
      this.counter += 1
      this.message = 'Doing something...'

      let doc = await this.getDocumentById(this.id)
      this.group = doc['groupof']
      console.log('\nInitial doc:')
      console.table(doc)

      doc.allocated = 'yes'
      await db.put(doc)

      doc = await this.getDocumentById(this.id)
      console.log('\nUpdated doc:')
      console.table(doc)

      // sync the databases in both directions
      // Not sure about the cancel - see https://pouchdb.com/errors.html#event_emitter_limit
      db.sync(remoteDB, { live: true, retry: true }).cancel()

      this.message = 'Done it!'
    },
    getDocumentById(id) {
      return db.get(id).catch(err => console.log(err))
    },
    async init() {
      const info = await db.info()
      console.log('\nDB info:')
      console.table(info)

      this.message = 'Syncing with remote database...'
      db.sync(remoteDB, { live: true, retry: true })

        .on('active', change => {
          console.log('\nactive:')
          console.log(change)
          this.pouchDbSyncActiveEvent = true
        })

        .on('change', change => {
          console.log('\nchange:')
          console.log(change)
          this.pouchDbSyncChangeEvent = true
        })

        .on('paused', info => {
          console.log('\npaused:')
          console.log(info)
          if (this.pouchDbSyncActiveEvent && !this.pouchDbSyncChangeEvent) {
            this.syncMess = 'Error'
          } else {
            this.syncMess = 'Syncing with remote database.'
          }

          this.pouchDbSyncActiveEvent = false
          this.pouchDbSyncChangeEvent = false
        })

      this.message = 'Syncing with remote database completed!'
    },
  },
  created() {
    this.init()
  },
}
</script>

<style scoped>
.random {
  display: flex;
  align-items: center;
  flex-direction: column;
}
button {
  padding: 10px;
}
.box {
  text-align: left;
  width: 400px;
}
</style>

<!--
   Licensed to the Apache Software Foundation (ASF) under one or more
   contributor license agreements.  See the NOTICE file distributed with
   this work for additional information regarding copyright ownership.
   The ASF licenses this file to You under the Apache License, Version 2.0
   (the "License"); you may not use this file except in compliance with
   the License.  You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.
-->

<template>
  <b-container fluid>
    <b-row class="mt-4">
      <b-col cols="12">
        <h2>Tasks</h2>
        <b-card no-body>
          <b-card-header header-tag="nav">
            <Menu />
          </b-card-header>
          <b-card-body>
            <div>
              <b-row>
                <b-col>
                  <table-listing
                    :fields="fields"
                    :items="tasksReversed"
                    :is-busy="isBusy"
                    placeholder="Filter tasks"
                    empty-text="No tasks created"
                    empty-filtered-text="No tasks found"
                  >
                  </table-listing>
                </b-col>
              </b-row>
           </div>
          </b-card-body>
        </b-card>
      </b-col>
    </b-row>
  </b-container>
</template>

<script>
import Menu from '@/components/manage/Menu'
import TableListing from '@/components/dataset/TableListing'

export default {
  name: 'ManageTasks',

  data () {
    return {
      polling: null,
      tasks: [],
      isBusy: true,
      fields: [
        {
          key: 'task',
          label: 'task',
          sortable: true,
          sortDirection: 'asc'
        },
        {
          key: 'taskId',
          label: 'task ID',
          sortable: true,
          sortDirection: 'asc'
        },
        {
          key: 'started',
          label: 'started',
          sortable: true,
          sortDirection: 'asc'
        },
        {
          key: 'finished',
          label: 'finished',
          sortable: true,
          sortDirection: 'asc'
        }
      ]
    }
  },

  computed: {
    tasksReversed () {
      if (!this.tasks) {
        return []
      }
      return [...this.tasks].reverse()
    }
  },

  components: {
    Menu,
    'table-listing': TableListing
  },

  async beforeRouteEnter (to, from, next) {
    next(vm => {
      vm.$fusekiService
        .getTasks()
        .then(response => {
          vm.tasks = response.data
          vm.isBusy = false
        })
    })
  },

  mounted () {
    const vm = this
    this.polling = setInterval(async () => {
      vm.isBusy = true
      vm.$fusekiService
        .getTasks()
        .then(response => {
          vm.tasks = response.data
          vm.isBusy = false
        })
    }, 10000)
  },

  beforeRouteLeave (to, from, next) {
    if (this.polling) {
      clearInterval(this.polling)
    }
    next()
  }
}
</script>

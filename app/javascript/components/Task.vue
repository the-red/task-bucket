<template>
  <div>
    <div>
      <div v-if="editable" @click="toggle">
        <form
          id="edit_task_157"
          class="simple_form edit_task"
          novalidate="novalidate"
          action="/tasks/157"
          accept-charset="UTF-8"
          data-remote="true"
          method="post"
        >
          <input type="hidden" name="_method" value="patch"><div class="my-2">
            <input
              id="task_title"
              class="js-task-input form-control"
              placeholder="タスク名"
              type="text"
              value="vue"
              name="task[title]"
            >
          </div>
          <div class="form-inline">
            <input
              id="task_start_at"
              value="20:28"
              class="js-timepicker js-task-input form-control mr-2"
              placeholder="開始"
              type="text"
              name="task[start_at]"
            >
            <input
              id="task_end_at"
              class="js-timepicker js-task-input form-control mr-2"
              placeholder="終了"
              type="text"
              name="task[end_at]"
            >
            <input
              id="task_estimate"
              class="js-task-input form-control mr-2"
              placeholder="予測"
              type="number"
              name="task[estimate]"
            >
            0
            <div class="form-group select optional task_project">
              <select id="task_project_id" class="form-control select optional js-task-input" name="task[project_id]">
                <option value="" />
                <option value="3">
                  NR
                </option>
                <option value="1">
                  WNI
                </option>
                <option value="2">
                  家庭
                </option>
                <option value="5">
                  クルマ
                </option>
                <option value="4">
                  スタンド
                </option>
              </select>
            </div>
          </div>
          <div class="my-2">
            <textarea
              id="task_comment"
              class="js-task-input form-control"
              placeholder="ふりかえり"
              name="task[comment]"
            />
          </div>
          <div>
            <input
              type="submit"
              name="commit"
              value="更新する"
              class="btn btn-secondary js-submit-task"
            >
            <a
              data-confirm="Are you sure?"
              class="btn"
              data-remote="true"
              rel="nofollow"
              data-method="delete"
              href="/tasks/157"
            ><svg
              class="svg-inline--fa fa-trash fa-w-14"
              aria-hidden="true"
              focusable="false"
              data-prefix="fas"
              data-icon="trash"
              role="img"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 448 512"
              data-fa-i2svg=""
            ><path fill="currentColor" d="M432 32H312l-9.4-18.7A24 24 0 0 0 281.1 0H166.8a23.72 23.72 0 0 0-21.4 13.3L136 32H16A16 16 0 0 0 0 48v32a16 16 0 0 0 16 16h416a16 16 0 0 0 16-16V48a16 16 0 0 0-16-16zM53.2 467a48 48 0 0 0 47.9 45h245.8a48 48 0 0 0 47.9-45L416 128H32z" /></svg><!-- <i class="fas fa-trash"></i> --></a>
          </div>
        </form>
      </div>
      <div v-else>
        <div class="my-2" @click="toggle">
          {{ task.title }}
        </div>
        <div class="form-inline">
          <span v-if="task.start_at">
            {{ task.start_at | timeFormat }}
          </span>
          <span v-if="task.end_at">
            〜
            {{ task.end_at | timeFormat }}
          </span>
          <span v-if="task.completed">
            実績:
            {{ actual }}
          </span>
        </div>
        <div class="my-2">
          <template v-if="task.start_at && !task.end_at" class="my-2">
            <button
              class="mx-2 btn btn-danger"
              @click="complete(task)"
            >
              <i class="fas fa-stop" />
            </button>
            <button
              class="mx-2 btn"
              @click="pause(task)"
            >
              <i class="fas fa-pause" />
            </button>
          </template>
          <template v-else-if="!task.start_at && !task.end_at" class="my-2">
            <div class="btn-group">
              <button class="btn btn-success" @click="start_from_now(task)"><i class="fas fa-play" /></button>
              <button aria-expanded="false" aria-haspopup="true" class="btn btn-success dropdown-toggle" data-toggle="dropdown" type="button">
                <span class="caret"></span>
                <span class="sr-only">Toggle Dropdown</span>
              </button>
              <ul class="dropdown-menu">
                <li class="dropdown-item"><a @click="start_from_just_before(task)">直前のタスクの終了時刻から開始</a></li>
                <li class="dropdown-item"><a @click="postpone(task)">明日に延期</a></li>
              </ul>
            </div>
          </template>
          <button
            class="mx-2 btn"
            @click="toggle"
          >
            <i class="fas fa-cog" />
          </button>
          <button
            class="mx-2 btn"
            @click="clone(task)"
          >
            <i class="fas fa-clone" />
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import { csrfToken } from '@rails/ujs'
import { format as formatDate, parseISO } from 'date-fns'
import { startApiV1TaskPath, completeApiV1TaskPath, cloneApiV1TaskPath, pauseApiV1TaskPath, postponeApiV1TaskPath } from '../javascripts/rails-routes'

axios.defaults.headers.common['X-CSRF-Token'] = csrfToken()

export default {
  filters: {
    timeFormat(time) {
      return formatDate(parseISO(time), 'HH:mm')
    },
  },
  props: {
    task: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      editable: false,
    }
  },
  computed: {
    actual() {
      return Math.round((parseISO(this.task.end_at) - parseISO(this.task.start_at)) / 1000 / 60)
    },
  },
  mounted() {
  },
  methods: {
    toggle() {
      this.editable = !this.editable
    },
    start_from_now(task) {
      axios.put(startApiV1TaskPath({ id: task.id })).then((response) => {
        this.updateTask(response.data)
      }, (error) => {
      })
    },
    start_from_just_before(task) {
      axios.put(startApiV1TaskPath({ id: task.id, from_just_before: true })).then((response) => {
        this.updateTask(response.data)
      }, (error) => {
      })
    },
    complete(task) {
      axios.put(completeApiV1TaskPath({ id: task.id })).then((response) => {
        this.updateTask(response.data)
      }, (error) => {
      })
    },
    pause(task) {
      axios.put(pauseApiV1TaskPath({ id: task.id })).then((response) => {
        this.updateTask(response.data)
      }, (error) => {
      })
    },
    clone(task) {
      axios.put(cloneApiV1TaskPath({ id: task.id })).then((response) => {
        this.updateTask(response.data)
      }, (error) => {
      })
    },
    postpone(task) {
      axios.put(postponeApiV1TaskPath({ id: task.id })).then((response) => {
        this.removeTask(response.data)
      }, (error) => {
      })
    },
    updateTask(task) {
      this.$emit('task-updated', task)
    },
    removeTask(task) {
      this.$emit('task-removed', task)
    },
  },
}
</script>

<style scoped lang="sass">
</style>

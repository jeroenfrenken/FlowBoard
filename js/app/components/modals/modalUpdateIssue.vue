<template>

  <modal
    :close="close"
  >

    <div slot="header">

      <h3>Update issue</h3>

    </div>

    <div slot="body" >

      <div class="issue-content" v-if="!loading">

        <p
          v-if="error !== null"
          class="error-box"
        >
          {{ error }}
        </p>

        <div class="field-set">

          <div class="field-title">
            <p>Title</p>
          </div>

          <div class="field-value">
            <input
              v-model="issue.title"
              placeholder="title"
            />
          </div>

        </div>

        <div class="field-set">

          <div class="field-title">
            <p>Type</p>
          </div>

          <div class="field-value">
            <custom-select
              v-model="issue.status"
              :options="options"
              :show-img="true"
            />
          </div>

        </div>

        <div class="field-set">

          <div class="field-title">
            <p>Summary</p>
          </div>

          <div class="field-value">
            <textarea
              v-model="issue.comment"
              placeholder="issue discription"
            />
          </div>

        </div>

        <div class="field-set">

          <div class="field-title">
            <p>Assignee</p>
          </div>

          <div class="field-value">
            <custom-user-select
              v-if="projectUsers !== false"
              v-model="issue.assigned"
              :options="projectUsers"
              :show-img="true"
            />
            <p v-else>
              Loading users...
            </p>
          </div>

        </div>

        <button
          @click="updateIssue($event)"
        >
          Update issue
        </button>

      </div>

      <div
        v-else
        class="loading"
      >

        <p class="loading-text">

          <i class="fas fa-spinner fa-spin" /> Loading...

        </p>

      </div>

    </div>

  </modal>

</template>
<script>
    import Vue from 'vue/dist/vue';
    import api from '../../store/axios';
    import Modal from "../ui/modal.vue";
    import CustomSelect from "../ui/customSelect.vue";
    import CustomUserSelect from "../ui/customUserSelect.vue";
    import mutations from '../../consts/mutationConsts';

    export default {
        components: {
            Modal,
            CustomSelect,
            CustomUserSelect
        },
        props: {
            update: {
                type: Function,
                default: () => {
                    return 0;
                }
            },
            payload: {
                type: Object,
                default() {}
            },
            close: {
                type: Function,
                default: () => {
                    return 0;
                }
            }
        },
        created () {

            this.issue = JSON.parse(JSON.stringify(this.payload));

            this.issue.status = this.issue.status.name;

            Vue.set(this.issue, 'assigned', (this.issue.assignee === null ? "" : this.issue.assignee.email));

            if (this.projectUsers === false) {
                this.$store.dispatch(mutations.GET_PROJECT_USERS, this.payload.project);
            }

        },
        computed: {

            projectUsers() {
                return this.$store.getters.getProjectUsers;
            }

        },
        data () {
            return {
                loading: false,
                error: null,
                options: [
                    {
                        value: 'task',
                        img: '/img/issue/type-task.png'
                    },
                    {
                        value: 'bug',
                        img: '/img/issue/type-bug.png'
                    },
                    {
                        value: 'epic',
                        img: '/img/issue/type-epic.png'
                    }
                ],
                issue: {
                    status: 'task',
                    title: '',
                    comment: null,
                    assigned: null
                }
            }
        },
        methods: {

            updateIssue (e) {
                e.preventDefault();

                this.error = null;
                this.loading = true;

                if (this.issue.assigned === "") this.issue.assigned = null;

                api(this.$store.state.token)
                    .put('/' + this.$store.state.org + '/' + this.payload.project + '/issue/update',
                        this.issue
                    )
                    .then((res) => {
                        this.loading = false;
                        this.showIssueCreateSuccess();
                        this.$store.commit(mutations.SAVE_PROJECT, false);
                        this.$store.dispatch(mutations.GET_PROJECT, this.payload.project);
                        this.close();
                    }).catch((res) => {
                        this.loading = false;
                        if (res.response.status === 400) {
                            this.error = res.response.data.data.message;
                        } else {
                            this.error = 'Unknown error'
                        }
                    }
                );

            }

        },
        notifications: {
            showIssueCreateSuccess: {
                title: 'Issue updated',
                message: 'You\'re issue is updated',
                type: 'success'
            }
        }
    }
</script>
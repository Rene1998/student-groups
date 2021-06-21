<template>
	<div class="container">
		<b-row>
			<b-col>
				<div class="a-profile-subpage-title">
					<b-col cols="10">
						<div class="d-flex align-items-center">
							<div class="z-icon-wrapper border cp" @click="$router.push(`/triedy/${$route.params.id}`)">
								<svg-arrow-left-icon/>
							</div>
							<div>
								<h1 v-if="task" class="ml-4">{{task.course.name}}</h1>
							</div>
						</div>
					</b-col>
					<b-col>
						<div class="d-flex flex-md-row-reverse">
							<p class="name-p light-blue" v-if="task">Trieda: <b class="group-name fw-bold cp" @click="$router.replace({path: `/triedy/${$route.params.id}`})">{{task.group.name}} - {{task.group.second_name}}</b></p>
						</div>
					</b-col>
				</div>
			</b-col>
		</b-row>
		<b-tabs class="mb-5 mt-3">
			<b-tab title="Informácie o úlohe">
				<b-row class="mt-4" v-if="task">
					<b-col class="cell-border mt-3 mr-3">
						<p class="name-p mb-0">Stav</p>
						<p v-if="task.is_active" class="status-p text-success m-0"><svg-bullet-icon /> Aktívny</p>
						<p v-else class="status-p text-danger m-0"><svg-bullet-icon /> Neaktívny</p>
					</b-col>
					<b-col class="cell-border mt-3 mr-3">
						<p class="name-p mb-0">Dátum úlohy</p>
						<b class="m-0">{{formatDate(task.created_at)}}</b>
					</b-col>
					<b-col class="cell-border mt-3 mr-3">
						<p class="name-p mb-0">Do kedy splniť?</p>
						<b class=" m-0">{{formatDate(task.deadline)}}</b>
					</b-col>
					<b-col class="mt-3">
						<b-button class="mt-3" variant="outline-primary" v-if="task"
							@click="$refs['edit-task-modal'].showModal()">Upraviť</b-button>
					</b-col>
				</b-row>
			</b-tab>
		</b-tabs>
		<div class="d-flex align-items-center pt-5">
			<div class="z-icon-wrapper border">
				<svg-graduation-cap-icon/>
			</div>
			<div>
				<h1 class="ml-4 mb-0">Zoznam študentov</h1>
			</div>
		</div>
		<b-row>
			<z-group-table v-if="task"
				:tableItems="task.members_responses" :tableFields="studentListFields"/>
		</b-row>
		<a-edit-task-modal ref="edit-task-modal"
			@editTask="editTask" @removeTask="$refs['remove-task-modal'].showModal(task.course)"/>
		<a-remove-task-modal ref="remove-task-modal" v-if="task"
			:modalConfig="modalConfig" @remove="removeTask"/>
	</div>
</template>

<script>
import moment from 'moment'
import sortBy from 'lodash/sortBy'
import wAxios from '@/plugins/w/axios'

const api = {
	getTask: (gid, tid) => wAxios.get_auth_data(`v1/group-admin/groups/${gid}/tasks/${tid}`),
	editTask: (gid, tid, task) => wAxios.patch_auth_data(`v1/group-admin/groups/${gid}/tasks/${tid}`, task),
	removeTask: (gid, tid) => wAxios.delete_auth_data(`v1/group-admin/groups/${gid}/tasks/${tid}`)
}

export default {
	layout: 'profile',

	components : {
		'z-group-table': () => import('@/plugins/lib@profile/groups/z-group-table.vue'),
		'a-edit-task-modal': () => import('@/plugins/lib@profile/groups/modals/a-edit-task-modal.vue'),
		'a-remove-task-modal': () => import('@/plugins/lib@profile/groups/modals/a-remove-group-modal.vue'),
		'svg-arrow-left-icon': () => import('@/plugins/appzmudri/_theme/icon/arrow-left.svg?inline'),
		'svg-bullet-icon': () => import('@/plugins/appzmudri/_theme/icon/bullet.svg?inline'),
		'svg-graduation-cap-icon': () => import('@/plugins/appzmudri/_theme/icon/graduation-cap.svg?inline')
	},

	data() {
		return {
			task: null,
			studentListFields: [
				{key: 'name', label: 'Meno', class: 'float-left '},
				{key: 'exams_count', label: 'Počet získaných bodov', class: ''}
			],
			modalConfig: {
				adminHeaderText: 'Odstrániť úlohu',
				adminBodyText: 'Ste si istý, že chcete odstrániť túto úlohu',
			}
		}
	},

	async created() {
		await this._loadTask()
	},

	methods: {
		formatDate(t) {
			return moment(t).format('L')
		},
		async removeTask() {
			this.$wToast.clear()
			try {
				this.$wToast.success()
				await api.removeTask(this.$route.params.id, this.$route.params.taskId)
				this.$router.push(`/triedy/${this.$route.params.id}`)
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async editTask(task) {
			this.$wToast.clear()
			try {
				this.$wToast.success()
				await api.editTask(this.$route.params.id, this.$route.params.taskId, {
					course_id: this.task.course.id,
					deadline: task.deadline
				})
				this.$nextTick(async () => {
					await this._loadTask()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async _loadTask() {
			this.task = await api.getTask(this.$route.params.id ,this.$route.params.taskId)
			sortBy(this.task.members_responses, e => e.user.name)
		},
	}
}
</script>

<style lang="scss" scoped>
	.cp {
		cursor: pointer;
	}

	.a-profile-subpage-title{
		padding: 0;
	}

	.container{
		width: 90%;
	}

	h1 {
		font-size: 24px;
		color: #2B2E4A;
		line-height: normal;
		display: flex;
	}

	button {
		width: 100%;
		color: #86B2CD !important;
		border-color: #86B2CD !important;
		font-size: small;
		font-weight: bold;
		&:hover {
			background-color: #FB2564;
			color: #FFFFFF !important;
			border-color: #FB2564 !important;
		}
	}

	.cp {
		cursor: pointer;
	}

	.name-p {
		color: #2B2E4A;
		font-weight: lighter;
		font-family: Poppins;
		font-style: normal;
		font-weight: normal;
		font-size: 14px;
		line-height: 21px;
	}

	.status-p {
		font-family: Poppins;
		font-style: normal;
		font-weight: normal;
		font-size: 14px;
		line-height: 40px;
	}

	b {
		font-family: Poppins;
		font-style: normal;
		font-weight: bold;
		font-size: 14px;
		line-height: 40px;
	}
	.cell-border {
		border-bottom: 1px solid #CADEEA;
	}

	.light-blue {
		color: #86B2CD;
	}

	.group-name {
		color: #333333,
	}
</style>
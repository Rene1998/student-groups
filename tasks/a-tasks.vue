<template>
	<div>
		<div class="mt-4">
			<svg-active-tasks-icon class="p-0" />
			<h1 class="ml-4 mb-0" v-if="tasks">Súčasné úlohy ({{tasks[0].length}})</h1>
		</div>
		<z-group-table v-if="tasks"
			:tableItems="tasks[0]" :tableFields="isAdmin ? adminActiveTaskFields : memberActiveTaskFields"/>
		<div class="mt-5">
			<svg-inactive-tasks-icon/>
			<h1 class="ml-4 p-0" v-if="tasks">Neaktívne úlohy ({{tasks[1].length}})</h1>
		</div>
		<z-group-table v-if="tasks"
			:class="isAdmin && 'inactive-tasks'" :tableItems="tasks[1]" :tableFields="isAdmin ? adminInactiveTaskFields : memberInactiveTaskFields" :tableHeader="'d-none'"
			@itemToReactivate="reactivateTask" @groupToRemove="$refs['remove-task-modal'].showModal($event)"/>
		<a-remove-task-modal ref="remove-task-modal" v-if="tasks"
			:modalConfig="modalConfig" @remove="removeTask"/>
	</div>
</template>

<script>
import moment from 'moment'
import { partition, sortBy } from 'lodash'
import { mapGetters } from 'vuex'
import wAxios from '@/plugins/w/axios'

const api = {
	admin: {
		getTasks: (gid) => wAxios.get_auth_data(`v1/group-admin/groups/${gid}/tasks`),
		reactivateTask: (gid, tid, newDeadline) => wAxios.patch_auth_data(`v1/group-admin/groups/${gid}/tasks/${tid}`, newDeadline),
		removeTask: (gid, tid) => wAxios.delete_auth_data(`v1/group-admin/groups/${gid}/tasks/${tid}`)
	},
	member: {
		getTasks: (gid) => wAxios.get_auth_data(`v1/group-member/groups/${gid}/tasks`),
	}
}

export default {
	components: {
		'z-group-table': () => import('@/plugins/lib@profile/groups/z-group-table.vue'),
		'a-remove-task-modal': () => import('@/plugins/lib@profile/groups/modals/a-remove-group-modal.vue'),
		'svg-active-tasks-icon': () => import('@/plugins/appzmudri/_theme/icon/active-tasks.svg?inline'),
		'svg-inactive-tasks-icon': () => import('@/plugins/appzmudri/_theme/icon/inactive-tasks.svg?inline'),
	},

	data() {
		return {
			adminActiveTaskFields: [
				{key: 'courseImg', label: 'Kurz'},
				{key: 'course', label: ''},
				{key: 'deadline', label: 'Do kedy splnil?'},
				{key: 'created_at', label: 'Dátum zadania'},
				{key: 'watch_or_reactivate', label: ''},
			],
			adminInactiveTaskFields: [
				{key: 'courseImg', label: 'Kurz'},
				{key: 'course', label: '', class: 'white'},
				{key: 'deadline', label: 'Do kedy splnil?', class: 'white'},
				{key: 'created_at', label: 'Dátum zadania', class: 'white'},
				{key: 'watch_or_reactivate', label: ''},
				// {key: 'remove', label: '', class: 'white'}
			],
			memberActiveTaskFields: [
				{key: 'courseImg', label: 'Kurz'},
				{key: 'course', label: ''},
				{key: 'deadline', label: 'Do kedy splnil?'},
				{key: 'created_at', label: 'Dátum zadania'},
				{key: 'exams_count', label: 'Počet bodov'},
				{key: 'go_to_course', label: ''},
			],
			memberInactiveTaskFields: [
				{key: 'courseImg', label: 'Kurz'},
				{key: 'course', label: ''},
				{key: 'deadline', label: 'Do kedy splnil?'},
				{key: 'created_at', label: 'Dátum zadania'},
				{key: 'exams_count', label: 'Počet bodov'},
				{key: 'space', label: ''},
			],
			tasks: null,
			modalConfig: {
				adminHeaderText: 'Odstrániť úlohu',
				adminBodyText: 'Ste si istý, že chcete odstraniť túto úlohu',
			}
		}
	},

	async created() {
		await this._loadTasks()
	},

	computed: {
		...mapGetters('wAuth', [
			'user'
		]),
		isAdmin() {
			return this.user.profile.type == 'teacher'
		}
	},

	methods: {
		async reactivateTask(tid) {
			this.$wToast.clear()
			try {
				this.$wToast.success()
				await api.admin.reactivateTask(this.$route.params.id, tid, {
					'deadline': moment().add(7, 'days').format('YYYY-MM-DD HH:MM:SS').toString()
				})
				this.$nextTick(async () => {
					await this._loadTasks()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async removeTask(tid) {
			this.$wToast.clear()
			try {
				this.$wToast.success()
				await api.admin.removeTask(this.$route.params.id, tid.id)
				this.$nextTick(async () => {
					await this._loadTasks()
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async _loadTasks() {
			const r = this.isAdmin ? await api.admin.getTasks(this.$route.params.id) : await api.member.getTasks(this.$route.params.id)
			sortBy(r, e=> e.course.name)
			this.tasks = partition(r, (e) => {
				return e.is_active
			})
		}
	}
}
</script>

<style lang="scss" scoped>
	h1 {
		font-size: 24px;
		display: inline-block;
		color:#2B2E4A;
	}

	/deep/ .white {
		color: white !important;
	}

	.inactive-tasks {
		color: white !important;
		background-color: #0064FF;
	}
</style>
<template>
	<div>
		<b-row>
			<b-col>
				<div class="a-profile-subpage-title pb-0">
					<b-col>
						<div class="d-flex align-items-center">
							<div class="z-icon-wrapper border cp" @click="$router.push(`/triedy/${$route.params.groupId}`)">
								<svg-arrow-left-icon/>
							</div>
							<div v-if="student">
								<h1 class="ml-4">{{student.user.name}}</h1>
								<h1 class="ml-1">{{student.user.surname}}</h1>
							</div>
						</div>
					</b-col>
					<b-col>
						<div class="d-flex flex-md-row-reverse group">
							<p v-if="group">
								Trieda: <b class="group-name fw-bold cp" @click="$router.replace({path: `/triedy/${$route.params.groupId}`})">{{group.name}} {{group.second_name}}</b>
							</p>
						</div>
						<div class="d-flex flex-md-row-reverse">
							<b-button class="p-0" variant="outline" v-if="student"
								@click="$refs['a-remove-student-modal'].showModal()">Vymazať z triedy</b-button>
						</div>
					</b-col>
				</div>
			</b-col>
		</b-row>
		<b-row>
			<b-col>
				<b-tabs>
					<b-col>
						<b-tab title="Úlohy" active>
							<z-group-table v-if="student" :tableItems="student.tasks" :tableFields="adminStudentTasksFields"/>
						</b-tab>
						<b-tab title="Informácie">
							<b-row class="mt-3">
								<b-col class="cell-border mt-3 mr-3 pb-2">
									<p class="mb-2">Meno</p>
									<b v-if="student" >{{student.user.name}}</b>
								</b-col>
								<b-col class="cell-border mt-3 mr-3 pb-2">
									<p class="mb-2">Priezvisko</p>
									<b v-if="student">{{student.user.surname}}</b>
								</b-col>
								<b-col class="cell-border mt-3 mr-3 pb-2">
									<p class="mb-2">E-mail</p>
									<b v-if="student">{{student.user.email}}</b>
								</b-col>

							</b-row>
							<b-row class="mt-3">
								<b-col class="cell-border mt-3 mr-3 pb-2">
									<p class="mb-2">Počet vypracovaných zadaní</p>
									<b v-if="student && student.tasks_count">{{student.tasks_count.tasks_done_count}} / {{student.tasks_count.tasks_count}}</b>
								</b-col>
								<b-col class="cell-border mt-3 mr-3 pb-2">
									<p class="mb-2">Dátum úlohy</p>
									<b v-if="student">{{formatDate(student.user.created_at)}}</b>
								</b-col>
								<b-col>
								</b-col>
							</b-row>
						</b-tab>
					</b-col>
				</b-tabs>
			</b-col>
		</b-row>
		<a-remove-student-modal ref="a-remove-student-modal" v-if="student && group"
			:student="student" :group="group"
			@remove="removeStudentFromGroup"/>
	</div>
</template>

<script>
import moment from 'moment'
import wAxios from '@/plugins/w/axios'

const api = {
	getTasks: (gid, mid) => wAxios.get_auth_data(`v1/group-admin/groups/${gid}/members/${mid}/tasks`),
	getGroup: (gid) => wAxios.get_auth_data(`v1/group-admin/groups/${gid}`),
	removeStudentFromGroup: (gid, mid) => wAxios.delete_auth(`v1/group-admin/groups/${gid}/members/${mid}`)
}

export default {
	layout: 'profile',

	components: {
		'z-group-table': () => import('@/plugins/lib@profile/groups/z-group-table.vue'),
		'a-remove-student-modal': () => import('@/plugins/lib@profile/groups/modals/a-remove-student-modal.vue'),
		'svg-arrow-left-icon': () => import('@/plugins/appzmudri/_theme/icon/arrow-left.svg?inline')
	},

	data() {
		return {
			adminStudentTasksFields: [
				{key: 'courseImg', label: 'Kurz'},
				{key: 'course', label: ''},
				{key: 'deadline', label: 'Do kedy splnil?'},
				{key: 'created_at', label: 'Dátum zadania'},
				{key: 'exams_count', label: 'Počet bodov'},
			],
			student: null,
			group: null
		}
	},

	async created() {
		await this._loadUser()
	},

	methods: {
		formatDate(t) {
			return moment(t).format('L')
		},
		async removeStudentFromGroup(student) {
			this.$wToast.clear()
			try {
				await api.removeStudentFromGroup(this.$route.params.groupId, student.id)
				this.$wToast.success()
				this.$nextTick(() => {
					this.$router.push(`/triedy/${this.$route.params.groupId}`)
				})
			} catch (e) {
				this.$wToast.error(e)
			}
		},
		async _loadUser() {
			this.student = await api.getTasks(this.$route.params.groupId,this.$route.params.studentId)
			this.$nextTick(async () => {
				await this._loadGroup()
			})
		},
		async _loadGroup() {
			this.group = await api.getGroup(this.$route.params.groupId)
		}
	}
}
</script>

<style lang="scss" scoped>
	.cp {
		cursor: pointer;
	}

	h1 {
		font-size: 24px;
		color: #2B2E4A;
		display: inline-block;
	}

	button {
		width: 192px;
		height: 39px;
		left: 1384px;
		top: 337px;
		border-radius: 100px;
		font-weight: 700;
		color: #86B2CD !important;
		border-color: #86B2CD !important;
		font-size: small;
		font-weight: bold;
		&:hover {
			background-color: #2B2E4A;
			color: #FFFFFF !important;
			border-color: #2B2E4A !important;
		}
	}

	p {
		color: #2B2E4A;
		font-weight: lighter;
		font-family: Poppins;
		font-style: normal;
		font-weight: normal;
		font-size: 14px;
		line-height: 21px;
	}

	.group {
		p {
			color: #86B2CD;
		}
		b {
			font-family: Poppins;
			font-style: normal;
			font-weight: bold;
			font-size: 14px;
			line-height: 40px;
		}
	}



	.group-name {
		color: #333333,
	}

	.a-icon-wrapper {
		cursor: pointer;
	}

	.cell-border {
		border-bottom: 1px solid #CADEEA;
	}


</style>